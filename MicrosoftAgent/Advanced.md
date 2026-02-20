# Microsoft Agent Framework (.NET) Advanced Cheatsheet

## Custom Agent Implementation

### Building a Custom AIAgent
```csharp
using Microsoft.Agents.AI;
using Microsoft.Extensions.AI;

/// <summary>
/// A custom agent that wraps an IChatClient with domain-specific logic,
/// automatic retry, and response post-processing.
/// </summary>
public class DomainExpertAgent : AIAgent
{
    private readonly IChatClient _chatClient;
    private readonly string _domain;
    private readonly List<ChatMessage> _history = [];
    private readonly ChatMessage _systemMessage;

    public DomainExpertAgent(IChatClient chatClient, string domain, string expertise)
    {
        _chatClient = chatClient;
        _domain = domain;
        Name = $"{domain}Expert";
        _systemMessage = new ChatMessage(ChatRole.System,
            $"You are a world-class expert in {domain}. {expertise} "
            + "Always cite specific principles or techniques in your answers.");
    }

    public override string Name { get; }

    public override async Task<string> RunAsync(
        string input,
        CancellationToken cancellationToken = default)
    {
        // Add user message to history
        _history.Add(new ChatMessage(ChatRole.User, input));

        var allMessages = new List<ChatMessage> { _systemMessage };
        allMessages.AddRange(_history);

        // Call LLM with retry
        ChatCompletion response = await RetryAsync(
            () => _chatClient.CompleteAsync(allMessages, cancellationToken: cancellationToken),
            maxAttempts: 3,
            cancellationToken: cancellationToken
        );

        string content = response.Message.Text ?? "";

        // Post-process: add domain tag
        content = $"[{_domain} Expert] {content}";

        _history.Add(new ChatMessage(ChatRole.Assistant, content));
        return content;
    }

    public override async IAsyncEnumerable<string> RunStreamingAsync(
        string input,
        [System.Runtime.CompilerServices.EnumeratorCancellation] CancellationToken cancellationToken = default)
    {
        _history.Add(new ChatMessage(ChatRole.User, input));
        var allMessages = new List<ChatMessage> { _systemMessage };
        allMessages.AddRange(_history);

        var fullResponse = new System.Text.StringBuilder();
        await foreach (var update in _chatClient.CompleteStreamingAsync(allMessages, cancellationToken: cancellationToken))
        {
            string chunk = update.Text ?? "";
            fullResponse.Append(chunk);
            yield return chunk;
        }

        _history.Add(new ChatMessage(ChatRole.Assistant, fullResponse.ToString()));
    }

    private static async Task<T> RetryAsync<T>(
        Func<Task<T>> operation,
        int maxAttempts,
        CancellationToken cancellationToken)
    {
        for (int attempt = 1; attempt <= maxAttempts; attempt++)
        {
            try
            {
                return await operation();
            }
            catch (Exception ex) when (attempt < maxAttempts && !cancellationToken.IsCancellationRequested)
            {
                int delay = (int)Math.Pow(2, attempt) * 1000;  // exponential backoff
                Console.Error.WriteLine($"Attempt {attempt} failed: {ex.Message}. Retrying in {delay}ms...");
                await Task.Delay(delay, cancellationToken);
            }
        }
        throw new InvalidOperationException($"Operation failed after {maxAttempts} attempts.");
    }
}

// Usage
var client = new AzureOpenAIClient(endpoint, new DefaultAzureCredential())
    .GetChatClient("gpt-4o")
    .AsIChatClient();

var csharpExpert = new DomainExpertAgent(
    client,
    domain: "C#",
    expertise: "Focus on .NET 9 features, performance, and best practices."
);

string advice = await csharpExpert.RunAsync("When should I use Span<T> instead of arrays?");
Console.WriteLine(advice);
```

## Advanced Workflow Patterns

### Graph-Based Workflow
```csharp
using Microsoft.Agents.AI.Workflows;

// Define a content review pipeline with branching:
// Draft → Review → [if rejected: Revise → Review] → [if approved: Publish]

var pipeline = new WorkflowBuilder()
    .AddAgent("drafter",  drafterAgent)
    .AddAgent("reviewer", reviewerAgent)
    .AddAgent("reviser",  reviserAgent)
    .AddAgent("publisher", publisherAgent)
    .Connect("drafter", "reviewer")
    .ConnectConditional("reviewer", condition: output => output.Contains("APPROVED"),
        onTrue:  "publisher",
        onFalse: "reviser")
    .Connect("reviser", "reviewer")   // loop back for re-review
    .WithMaxIterations("reviewer", maxIterations: 3)   // prevent infinite loops
    .Build();

string finalContent = await pipeline.RunAsync(
    "Write a technical blog post about C# pattern matching."
);
Console.WriteLine(finalContent);
```

### Durable / Resumable Workflow
```csharp
using Microsoft.Agents.AI.Workflows;

// Durable workflows persist state and can resume after failures or restarts
var durableWorkflow = new WorkflowBuilder()
    .AddAgent("analyzer",   analyzerAgent)
    .AddAgent("processor",  processorAgent)
    .AddAgent("reporter",   reporterAgent)
    .Connect("analyzer", "processor")
    .Connect("processor", "reporter")
    .WithDurability(
        checkpointStore: new SqliteCheckpointStore("workflow-state.db"),
        resumable: true
    )
    .Build();

string workflowId = Guid.NewGuid().ToString();

try
{
    // Start or resume a workflow by ID
    string result = await durableWorkflow.RunAsync(
        "Analyze our Q4 sales data and produce an executive report.",
        workflowId: workflowId
    );
    Console.WriteLine(result);
}
catch (Exception ex)
{
    Console.Error.WriteLine($"Workflow {workflowId} failed: {ex.Message}");
    // On next run, pass the same workflowId to resume from the last checkpoint
}
```

## Model Context Protocol (MCP) Integration

### Connecting to an MCP Server
```csharp
using Microsoft.Agents.AI;
using Microsoft.Agents.AI.Mcp;

// MCP allows agents to discover and use tools from external MCP servers
// (e.g., filesystem, databases, external APIs)

// Connect to a local MCP server (stdio transport)
var mcpServer = await McpServerConnection.ConnectStdioAsync(
    command: "npx",
    arguments: ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/data"]
);

// Discover available tools from the server
var mcpTools = await mcpServer.ListToolsAsync();
Console.WriteLine($"Discovered {mcpTools.Count} MCP tools");

// Create agent with MCP tools
var agent = new AzureOpenAIClient(endpoint, new DefaultAzureCredential())
    .GetChatClient("gpt-4o")
    .AsAIAgent(
        instructions: "You are a file analysis assistant. Use the available tools to read and analyze files.",
        mcpServers: [mcpServer]     // agent can now use all MCP-provided tools
    );

string result = await agent.RunAsync("Summarize all .md files in the data directory.");
Console.WriteLine(result);
await mcpServer.DisposeAsync();

// Connect to an HTTP-based MCP server
var remoteMcpServer = await McpServerConnection.ConnectHttpAsync(
    new Uri("https://my-mcp-server.example.com/mcp")
);
```

### Publishing Your Own MCP Tools
```csharp
using Microsoft.Agents.AI.Mcp;

// Create an MCP server that exposes your tools to any MCP-compatible client
var server = new McpServerBuilder()
    .AddTool<SearchProductsTool>()
    .AddTool<GetOrderStatusTool>()
    .AddTool<UpdateInventoryTool>()
    .WithTransport(McpTransport.Stdio)
    .Build();

await server.RunAsync(CancellationToken.None);

// Tool class for MCP publication
[McpTool(Name = "search_products", Description = "Search the product catalog by keyword.")]
public class SearchProductsTool : IMcpTool
{
    [McpToolParameter(Name = "keyword", Description = "Search keyword", Required = true)]
    public string Keyword { get; set; } = "";

    [McpToolParameter(Name = "max_results", Description = "Maximum number of results")]
    public int MaxResults { get; set; } = 5;

    public async Task<McpToolResult> ExecuteAsync(CancellationToken ct)
    {
        // implement actual search logic
        var results = await ProductCatalog.SearchAsync(Keyword, MaxResults, ct);
        return McpToolResult.Success(System.Text.Json.JsonSerializer.Serialize(results));
    }
}
```

## Observability with OpenTelemetry

```csharp
using Microsoft.Agents.AI;
using OpenTelemetry;
using OpenTelemetry.Trace;
using OpenTelemetry.Metrics;
using Azure.Monitor.OpenTelemetry.Exporter;

// Configure OpenTelemetry in Program.cs
builder.Services.AddOpenTelemetry()
    .WithTracing(tracing => tracing
        .AddSource("Microsoft.Agents.AI")       // trace all agent activity
        .AddAzureMonitorTraceExporter()          // send to Application Insights
        .AddConsoleExporter()                    // also log to console in dev
    )
    .WithMetrics(metrics => metrics
        .AddMeter("Microsoft.Agents.AI")         // agent metrics (tokens, latency, etc.)
        .AddAzureMonitorMetricExporter()
    );

// The Agent Framework automatically emits spans and metrics for:
// - agent.run (input/output lengths, duration, model)
// - tool.call (tool name, arguments, duration, success/failure)
// - workflow.step (step name, duration, retries)
// - llm.tokens (prompt tokens, completion tokens, total)

// Add custom spans to your own code
using var activity = ActivitySource.StartActivity("custom.processing");
activity?.SetTag("user.id", userId);
activity?.SetTag("agent.name", agentName);
string result = await agent.RunAsync(input);
activity?.SetTag("response.length", result.Length);
```

## Testing Agent Applications

### Unit Testing with Mock IChatClient
```csharp
using Microsoft.Agents.AI;
using Microsoft.Extensions.AI;
using Xunit;

public class AgentTests
{
    [Fact]
    public async Task Agent_Should_Return_Non_Empty_Response()
    {
        // Arrange — mock the underlying chat client
        var mockClient = new MockChatClient(
            response: "The capital of France is Paris."
        );

        var agent = mockClient.AsAIAgent(
            instructions: "You are a helpful geography assistant."
        );

        // Act
        string result = await agent.RunAsync("What is the capital of France?");

        // Assert
        Assert.NotEmpty(result);
        Assert.Contains("Paris", result, StringComparison.OrdinalIgnoreCase);
    }

    [Fact]
    public async Task Agent_Should_Call_Tool_When_Needed()
    {
        // Arrange
        bool toolCalled = false;

        string GetWeather(string city)
        {
            toolCalled = true;
            return $"22°C, Sunny in {city}";
        }

        var mockClient = new MockChatClient(
            responses: [
                // First response: call the tool
                ChatMessage.CreateToolCallMessage("get_weather", new { city = "Tokyo" }),
                // Second response: final answer
                new ChatMessage(ChatRole.Assistant, "The weather in Tokyo is 22°C and sunny."),
            ]
        );

        var agent = mockClient.AsAIAgent(
            instructions: "Use tools to answer weather questions.",
            tools: [GetWeather]
        );

        // Act
        string result = await agent.RunAsync("What's the weather in Tokyo?");

        // Assert
        Assert.True(toolCalled, "Expected the weather tool to be called.");
        Assert.Contains("Tokyo", result);
    }

    [Fact]
    public async Task Workflow_Should_Execute_Steps_In_Order()
    {
        // Arrange
        var executionOrder = new List<string>();

        AIAgent MakeTrackingAgent(string name) => new MockChatClient(
            response: $"{name}_result"
        ).AsAIAgent(name: name);

        var workflow = new WorkflowBuilder()
            .AddAgent("step1", MakeTrackingAgent("step1"))
            .AddAgent("step2", MakeTrackingAgent("step2"))
            .Connect("step1", "step2")
            .Build();

        // Act
        string result = await workflow.RunAsync("test input");

        // Assert
        Assert.Contains("step2_result", result);
    }
}

// Simple mock for IChatClient
public class MockChatClient(string? response = null, IEnumerable<ChatMessage>? responses = null)
    : IChatClient
{
    private readonly Queue<ChatMessage> _responses = new(
        responses ?? [new ChatMessage(ChatRole.Assistant, response ?? "Mock response")]
    );

    public Task<ChatCompletion> CompleteAsync(
        IList<ChatMessage> messages,
        ChatOptions? options = null,
        CancellationToken cancellationToken = default)
    {
        var next = _responses.Count > 0
            ? _responses.Dequeue()
            : new ChatMessage(ChatRole.Assistant, "No more mock responses.");
        return Task.FromResult(new ChatCompletion(next));
    }

    public IAsyncEnumerable<StreamingChatCompletionUpdate> CompleteStreamingAsync(
        IList<ChatMessage> messages,
        ChatOptions? options = null,
        CancellationToken cancellationToken = default)
        => throw new NotImplementedException("Streaming not mocked in this test.");

    public ChatClientMetadata Metadata => new(modelId: "mock-model");
    public TService? GetService<TService>(object? key = null) where TService : class => null;
    public void Dispose() { }
}
```

### Integration Testing
```csharp
using Microsoft.Agents.AI;
using Microsoft.Extensions.DependencyInjection;
using Xunit;

[Collection("IntegrationTests")]   // prevents parallel execution — uses real API
public class AgentIntegrationTests : IClassFixture<AgentTestFixture>
{
    private readonly AIAgent _agent;

    public AgentIntegrationTests(AgentTestFixture fixture)
    {
        _agent = fixture.Agent;
    }

    [Fact]
    [Trait("Category", "Integration")]
    public async Task Agent_Should_Correctly_Perform_Arithmetic_Via_Tool()
    {
        // Act
        string result = await _agent.RunAsync("What is (127 * 43) + 999?");

        // Assert — 127 * 43 = 5461, + 999 = 6460
        Assert.Contains("6460", result);
    }
}

public class AgentTestFixture : IDisposable
{
    public AIAgent Agent { get; }

    public AgentTestFixture()
    {
        var apiKey = Environment.GetEnvironmentVariable("OPENAI_API_KEY")
            ?? throw new SkipException("OPENAI_API_KEY not set — skipping integration tests");

        Agent = new OpenAIClient(apiKey)
            .GetChatClient("gpt-4o-mini")
            .AsAIAgent(
                instructions: "You are a math assistant.",
                tools: [AgentTools.Calculate]
            );
    }

    public void Dispose() { /* cleanup */ }
}
```

## Production Patterns

### Resilient Agent with Polly
```csharp
using Microsoft.Agents.AI;
using Polly;
using Polly.Retry;
using Polly.CircuitBreaker;
using Polly.Timeout;

// Configure a resilience pipeline with Polly v8
var resiliencePipeline = new ResiliencePipelineBuilder<string>()
    .AddRetry(new RetryStrategyOptions<string>
    {
        MaxRetryAttempts = 3,
        BackoffType = DelayBackoffType.Exponential,
        UseJitter = true,
        ShouldHandle = new PredicateBuilder<string>().Handle<AgentException>(),
        OnRetry = args =>
        {
            Console.Error.WriteLine($"Retry {args.AttemptNumber}: {args.Outcome.Exception?.Message}");
            return ValueTask.CompletedTask;
        },
    })
    .AddCircuitBreaker(new CircuitBreakerStrategyOptions<string>
    {
        FailureRatio = 0.5,
        MinimumThroughput = 5,
        SamplingDuration = TimeSpan.FromSeconds(30),
        BreakDuration = TimeSpan.FromSeconds(60),
    })
    .AddTimeout(TimeSpan.FromSeconds(30))
    .Build();

async Task<string> ResilientRunAsync(AIAgent agent, string input)
    => await resiliencePipeline.ExecuteAsync(
        async ct => await agent.RunAsync(input, ct)
    );
```

### Cost and Token Tracking
```csharp
using Microsoft.Agents.AI;
using Microsoft.Extensions.AI;

// Wrap the agent to track token usage and estimated cost
public class CostTrackingAgent(AIAgent inner, ILogger<CostTrackingAgent> logger) : AIAgent
{
    private long _totalPromptTokens;
    private long _totalCompletionTokens;

    // Approximate cost per 1M tokens (gpt-4o-mini as of 2025)
    private const decimal PromptCostPer1M = 0.15m;
    private const decimal CompletionCostPer1M = 0.60m;

    public override string Name => inner.Name;

    public override async Task<string> RunAsync(string input, CancellationToken ct = default)
    {
        string result = await inner.RunAsync(input, ct);

        // Usage metadata is available on the agent after each run
        if (inner.LastUsage is { } usage)
        {
            Interlocked.Add(ref _totalPromptTokens, usage.InputTokenCount);
            Interlocked.Add(ref _totalCompletionTokens, usage.OutputTokenCount);

            decimal cost = (usage.InputTokenCount  * PromptCostPer1M / 1_000_000m)
                         + (usage.OutputTokenCount * CompletionCostPer1M / 1_000_000m);

            logger.LogInformation(
                "Run: in={Input} out={Output} cost=${Cost:F6} | Total in={TotalIn} out={TotalOut}",
                usage.InputTokenCount, usage.OutputTokenCount, cost,
                _totalPromptTokens, _totalCompletionTokens);
        }

        return result;
    }

    public override IAsyncEnumerable<string> RunStreamingAsync(string input, CancellationToken ct = default)
        => inner.RunStreamingAsync(input, ct);

    public decimal EstimatedTotalCost =>
        (_totalPromptTokens     * PromptCostPer1M / 1_000_000m) +
        (_totalCompletionTokens * CompletionCostPer1M / 1_000_000m);
}
```

### Configuration-Driven Agent Factory
```csharp
// appsettings.json
// {
//   "Agents": {
//     "Assistant": { "Deployment": "gpt-4o-mini", "Instructions": "You are a helpful assistant.", "Temperature": 0.7 },
//     "CodeReviewer": { "Deployment": "gpt-4o", "Instructions": "You are a senior C# code reviewer.", "Temperature": 0.2 }
//   }
// }

public class AgentFactory(IConfiguration config, AzureOpenAIClient azureClient) : IAIAgentFactory
{
    public AIAgent CreateAgent(string agentName)
    {
        var section = config.GetSection($"Agents:{agentName}");

        if (!section.Exists())
            throw new InvalidOperationException($"No configuration found for agent '{agentName}'.");

        string deployment  = section["Deployment"]   ?? throw new InvalidOperationException("Deployment not configured");
        string instructions = section["Instructions"] ?? "You are a helpful assistant.";
        float temperature  = section.GetValue<float>("Temperature", 0.7f);

        return azureClient
            .GetChatClient(deployment)
            .AsAIAgent(
                name: agentName,
                instructions: instructions,
                temperature: temperature
            );
    }
}

// Registration
builder.Services.AddSingleton<IAIAgentFactory, AgentFactory>();
```

## Best Practices (Do's)

✅ **Implement custom agents by composing `IChatClient` rather than subclassing**
```csharp
// Preferred — compose via IChatClient pipeline
var agent = chatClient
    .AsBuilder()
    .UseLogging(loggerFactory)
    .UseOpenTelemetry(tracerProvider)
    .Build()
    .AsAIAgent(instructions: "...");
```

✅ **Use Polly v8 `ResiliencePipeline` for production reliability**
```csharp
var pipeline = new ResiliencePipelineBuilder<string>()
    .AddRetry(retryOptions)
    .AddCircuitBreaker(cbOptions)
    .AddTimeout(TimeSpan.FromSeconds(30))
    .Build();
```

✅ **Always set `WithMaxIterations` on graph workflows to prevent runaway loops**
```csharp
.ConnectConditional("reviewer", ..., onFalse: "reviser")
.WithMaxIterations("reviewer", maxIterations: 3)
```

✅ **Validate and sanitize all tool inputs**
```csharp
[Description("Search products by keyword.")]
public static async Task<string> SearchProducts(string keyword)
{
    if (string.IsNullOrWhiteSpace(keyword) || keyword.Length > 100)
        return "Error: keyword must be 1-100 non-whitespace characters.";
    return await catalog.SearchAsync(keyword.Trim());
}
```

✅ **Use OpenTelemetry for full distributed tracing**
```csharp
builder.Services.AddOpenTelemetry()
    .WithTracing(t => t.AddSource("Microsoft.Agents.AI").AddAzureMonitorTraceExporter());
```

## Common Mistakes (Don'ts)

❌ **Don't use `Thread.Sleep` or blocking calls in async tools**
```csharp
// Bad — blocks the thread pool
public static string SlowTool(string input)
{
    Thread.Sleep(5000);   // blocks!
    return Process(input);
}

// Good — non-blocking async
public static async Task<string> SlowTool(string input)
{
    await Task.Delay(5000);
    return await ProcessAsync(input);
}
```

❌ **Don't expose internal system errors to the LLM via tool exceptions**
```csharp
// Bad — leaks internal info
public static string RiskyTool(string q)
    => db.Execute(q);   // throws SqlException with internal DB details

// Good — return safe error messages
public static async Task<string> SafeTool(string q)
{
    try { return await db.ExecuteAsync(q); }
    catch (Exception ex)
    {
        logger.LogError(ex, "Tool error");
        return "Database query failed. Please try a different query.";
    }
}
```

❌ **Don't run untested workflows in production**
```csharp
// Good — always write tests before deploying
[Fact]
public async Task Workflow_Completes_Without_Infinite_Loop()
{
    var result = await testWorkflow.RunAsync("test input");
    Assert.NotNull(result);
}
```

❌ **Don't forget to dispose `McpServerConnection`**
```csharp
// Bad
var mcp = await McpServerConnection.ConnectStdioAsync(...);
// ... process ends, MCP server process leaked

// Good
await using var mcp = await McpServerConnection.ConnectStdioAsync(...);
// automatically disposed
```
