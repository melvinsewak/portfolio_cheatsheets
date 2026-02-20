# Microsoft Agent Framework (.NET) Intermediate Cheatsheet

## Multi-Agent Workflows

### Sequential Workflow (Agent Pipeline)
```csharp
using Microsoft.Agents.AI;
using Microsoft.Agents.AI.Workflows;
using OpenAI;

var chatClient = new OpenAIClient(Environment.GetEnvironmentVariable("OPENAI_API_KEY")!)
    .GetChatClient("gpt-4o");

// Define specialized agents
var researchAgent = chatClient.AsAIAgent(
    name: "Researcher",
    instructions: "You are a research specialist. Analyze the topic and provide key facts and data points. Be thorough."
);

var summaryAgent = chatClient.AsAIAgent(
    name: "Summarizer",
    instructions: "You receive research output and create a concise, well-structured executive summary. Maximum 3 paragraphs."
);

var writerAgent = chatClient.AsAIAgent(
    name: "Writer",
    instructions: "You receive a summary and expand it into a polished blog post with an introduction, body, and conclusion."
);

// Build sequential workflow: Research → Summarize → Write
var workflow = new WorkflowBuilder()
    .AddAgent("research",  researchAgent)
    .AddAgent("summarize", summaryAgent)
    .AddAgent("write",     writerAgent)
    .Connect("research",  "summarize")
    .Connect("summarize", "write")
    .Build();

string result = await workflow.RunAsync("Write a blog post about C# 13 new features.");
Console.WriteLine(result);
```

### Parallel Workflow (Multiple Agents Simultaneously)
```csharp
using Microsoft.Agents.AI.Workflows;

var translationWorkflow = new WorkflowBuilder()
    .AddAgent("spanish", spanishTranslatorAgent)
    .AddAgent("french",  frenchTranslatorAgent)
    .AddAgent("german",  germanTranslatorAgent)
    .Parallel("spanish", "french", "german")   // all three run concurrently
    .Build();

string input = "Hello! Welcome to our platform.";
var results = await translationWorkflow.RunParallelAsync(input);

foreach (var (agentName, translation) in results)
{
    Console.WriteLine($"{agentName}: {translation}");
}
```

### Conditional Workflow (Routing)
```csharp
using Microsoft.Agents.AI;
using Microsoft.Agents.AI.Workflows;
using System.ComponentModel;

// Router agent classifies the incoming request
var routerAgent = chatClient.AsAIAgent(
    name: "Router",
    instructions: """
        Classify the user's request into exactly one category:
        - BILLING: questions about payments, invoices, subscriptions
        - TECHNICAL: bugs, setup issues, API errors, integration problems
        - GENERAL: everything else
        Respond with only the category name in UPPERCASE.
        """
);

var billingAgent = chatClient.AsAIAgent(
    name: "BillingSupport",
    instructions: "You are a billing specialist. Handle payment and subscription queries professionally."
);

var technicalAgent = chatClient.AsAIAgent(
    name: "TechSupport",
    instructions: "You are a technical support engineer. Troubleshoot issues step-by-step."
);

var generalAgent = chatClient.AsAIAgent(
    name: "GeneralSupport",
    instructions: "You are a general customer support agent. Be friendly and helpful."
);

// Route to the correct specialist based on the router's decision
async Task<string> HandleSupportRequest(string userQuery)
{
    string category = (await routerAgent.RunAsync(userQuery)).Trim();

    AIAgent specialist = category switch
    {
        "BILLING"   => billingAgent,
        "TECHNICAL" => technicalAgent,
        _           => generalAgent,
    };

    Console.WriteLine($"Routing to: {specialist.Name}");
    return await specialist.RunAsync(userQuery);
}

string answer = await HandleSupportRequest("I was charged twice this month!");
Console.WriteLine(answer);
```

## Memory and Conversation State

### Session-Based Memory
```csharp
using Microsoft.Agents.AI;

// Each agent instance maintains its own conversation history automatically.
// To persist state across application restarts, serialize and restore the session.

var agent = chatClient.AsAIAgent(instructions: "You are a helpful assistant.");

// First session
string reply1 = await agent.RunAsync("I'm building a REST API in C#.");
string reply2 = await agent.RunAsync("What frameworks would you recommend?");
// Agent remembers context from reply1

// Save session state for persistence
string sessionJson = await agent.ExportSessionAsync();  // serialized JSON
File.WriteAllText("session.json", sessionJson);

// Restore session in a new process/request
var newAgent = chatClient.AsAIAgent(instructions: "You are a helpful assistant.");
await newAgent.ImportSessionAsync(File.ReadAllText("session.json"));
string reply3 = await newAgent.RunAsync("Which framework did we decide on?");
// newAgent remembers the original conversation
```

### Trimming Conversation History
```csharp
using Microsoft.Agents.AI;

var agent = chatClient.AsAIAgent(
    instructions: "You are a helpful assistant.",
    options: new AIAgentOptions
    {
        // Keep only the last N conversation turns to stay within context limits
        MaxConversationTurns = 10,

        // Or limit by token count
        MaxContextTokens = 8000,
    }
);
```

### External Memory with Vector Store
```csharp
using Microsoft.Agents.AI;
using Microsoft.Extensions.VectorData;
using System.ComponentModel;

// Store conversation facts in a vector store for long-term memory retrieval
public class VectorMemoryTool
{
    private readonly IVectorStore _vectorStore;
    private readonly IEmbeddingGenerator<string, Embedding<float>> _embedder;

    public VectorMemoryTool(IVectorStore vectorStore,
                             IEmbeddingGenerator<string, Embedding<float>> embedder)
    {
        _vectorStore = vectorStore;
        _embedder = embedder;
    }

    [Description("Remember a fact for future reference.")]
    public async Task<string> RememberFact(
        [Description("The fact to remember")] string fact)
    {
        var collection = _vectorStore.GetCollection<int, MemoryRecord>("facts");
        await collection.EnsureCollectionExistsAsync();

        var embedding = await _embedder.GenerateEmbeddingVectorAsync(fact);
        await collection.UpsertAsync(new MemoryRecord
        {
            Key = DateTime.UtcNow.GetHashCode(),
            Fact = fact,
            Vector = embedding,
        });
        return $"Remembered: {fact}";
    }

    [Description("Recall facts related to a query from memory.")]
    public async Task<string> RecallFacts(
        [Description("What to search for in memory")] string query)
    {
        var collection = _vectorStore.GetCollection<int, MemoryRecord>("facts");
        var queryVector = await _embedder.GenerateEmbeddingVectorAsync(query);

        var results = await collection.VectorizedSearchAsync(queryVector, new VectorSearchOptions { Top = 3 });
        var facts = new List<string>();
        await foreach (var result in results.Results)
        {
            facts.Add(result.Record.Fact);
        }
        return facts.Count > 0 ? string.Join("\n", facts) : "No relevant facts found in memory.";
    }
}

public class MemoryRecord
{
    [VectorStoreRecordKey]
    public int Key { get; set; }

    [VectorStoreRecordData]
    public string Fact { get; set; } = "";

    [VectorStoreRecordVector(1536)]
    public ReadOnlyMemory<float> Vector { get; set; }
}
```

## Dependency Injection and Hosted Services

### Generic Host Setup
```csharp
// Program.cs
using Microsoft.Agents.AI;
using Microsoft.Agents.AI.OpenAI;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Azure.Identity;

var host = Host.CreateApplicationBuilder(args);

// Register the Azure OpenAI client
host.Services.AddSingleton(_ =>
    new AzureOpenAIClient(
        new Uri(host.Configuration["Azure:OpenAI:Endpoint"]!),
        new DefaultAzureCredential()));

// Register agents as named services
host.Services.AddAIAgent("assistant", (sp, factory) =>
    sp.GetRequiredService<AzureOpenAIClient>()
        .GetChatClient(host.Configuration["Azure:OpenAI:Deployment"]!)
        .AsAIAgent(instructions: "You are a helpful assistant."));

host.Services.AddAIAgent("code_reviewer", (sp, factory) =>
    sp.GetRequiredService<AzureOpenAIClient>()
        .GetChatClient("gpt-4o")
        .AsAIAgent(
            name: "CodeReviewer",
            instructions: "You are a senior C# code reviewer.",
            tools: [AgentTools.ReviewCode]
        ));

// Register application services
host.Services.AddHostedService<AgentService>();

await host.Build().RunAsync();

// AgentService.cs
public class AgentService(IAIAgentFactory agentFactory, ILogger<AgentService> logger)
    : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        var assistant = agentFactory.GetAgent("assistant");

        while (!stoppingToken.IsCancellationRequested)
        {
            try
            {
                // Process tasks from a queue, message bus, etc.
                string response = await assistant.RunAsync("Daily status update?", stoppingToken);
                logger.LogInformation("Agent response: {Response}", response);
                await Task.Delay(TimeSpan.FromMinutes(60), stoppingToken);
            }
            catch (OperationCanceledException)
            {
                break;
            }
        }
    }
}
```

### Using Agents in ASP.NET Core
```csharp
// Program.cs (ASP.NET Core Minimal API)
using Microsoft.Agents.AI;
using Azure.Identity;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddSingleton(_ =>
    new AzureOpenAIClient(
        new Uri(builder.Configuration["Azure:OpenAI:Endpoint"]!),
        new DefaultAzureCredential()));

builder.Services.AddAIAgent("chat", (sp, _) =>
    sp.GetRequiredService<AzureOpenAIClient>()
        .GetChatClient(builder.Configuration["Azure:OpenAI:Deployment"]!)
        .AsAIAgent(instructions: "You are a helpful assistant."));

var app = builder.Build();

// Session-based multi-turn chat endpoint
app.MapPost("/chat", async (
    ChatRequest request,
    IAIAgentFactory agentFactory,
    ISessionStore sessionStore) =>
{
    // Restore or create session
    var agent = agentFactory.GetAgent("chat");
    if (request.SessionId is not null && await sessionStore.TryRestoreAsync(agent, request.SessionId))
    {
        // session restored
    }

    string response = await agent.RunAsync(request.Message);

    // Persist session
    string sessionId = request.SessionId ?? Guid.NewGuid().ToString();
    await sessionStore.SaveAsync(agent, sessionId);

    return Results.Ok(new { SessionId = sessionId, Response = response });
});

app.Run();

record ChatRequest(string Message, string? SessionId);
```

## Human-in-the-Loop

### Approval Step in Workflow
```csharp
using Microsoft.Agents.AI;
using Microsoft.Agents.AI.Workflows;
using System.ComponentModel;

// Tool that requires human approval before executing a sensitive action
public static class SensitiveTools
{
    [Description("Delete all records in the specified database table. Requires confirmation.")]
    public static async Task<string> DeleteTableRecords(
        [Description("Table name to delete records from")] string tableName)
    {
        // Request human approval
        Console.ForegroundColor = ConsoleColor.Yellow;
        Console.Write($"⚠️  APPROVAL REQUIRED: Delete all records from '{tableName}'? [y/N]: ");
        Console.ResetColor();

        string? input = Console.ReadLine()?.Trim().ToLower();
        if (input != "y" && input != "yes")
        {
            return $"Action cancelled by user — '{tableName}' records were NOT deleted.";
        }

        // Proceed with the action
        // await database.DeleteAllAsync(tableName);
        return $"Successfully deleted all records from '{tableName}'.";
    }

    [Description("Send a mass email to all subscribers in the specified list.")]
    public static async Task<string> SendMassEmail(
        [Description("Subscriber list name")] string listName,
        [Description("Email subject")] string subject,
        [Description("Email body")] string body)
    {
        Console.Write($"📧 APPROVAL REQUIRED: Send email '{subject}' to list '{listName}'? [y/N]: ");
        string? input = Console.ReadLine()?.Trim().ToLower();

        if (input != "y" && input != "yes")
            return "Email send cancelled by user.";

        // await emailService.SendAsync(listName, subject, body);
        return $"Email '{subject}' sent to '{listName}' subscriber list.";
    }
}

// Agent with human-gated tools
var adminAgent = chatClient.AsAIAgent(
    name: "DatabaseAdmin",
    instructions: "You are a database administrator assistant. Always explain what you're about to do before doing it.",
    tools: [SensitiveTools.DeleteTableRecords, SensitiveTools.SendMassEmail]
);

string result = await adminAgent.RunAsync("Please clean up the test_users table and notify the team.");
Console.WriteLine(result);
```

## Tools with Rich Return Types

```csharp
using System.ComponentModel;
using System.Text.Json;

public record ProductSearchResult(
    string Name,
    decimal Price,
    int StockCount,
    string Category,
    double Rating);

public static class CatalogTools
{
    private static readonly List<ProductSearchResult> Products =
    [
        new("C# in Depth",          45.99m, 150, "Books",       4.8),
        new("Pro ASP.NET Core",     52.99m,  80, "Books",       4.6),
        new("JetBrains Rider",     749.00m,   0, "Software",    4.9),
        new("Visual Studio 2022",    0.00m, 999, "Software",    4.7),
    ];

    [Description("Search the product catalog. Returns matching products with prices and stock.")]
    public static string SearchProducts(
        [Description("Search keyword")] string keyword,
        [Description("Maximum results to return (1-10)")] int maxResults = 5)
    {
        var matches = Products
            .Where(p => p.Name.Contains(keyword, StringComparison.OrdinalIgnoreCase)
                     || p.Category.Contains(keyword, StringComparison.OrdinalIgnoreCase))
            .OrderByDescending(p => p.Rating)
            .Take(maxResults)
            .ToList();

        if (!matches.Any())
            return $"No products found matching '{keyword}'.";

        return JsonSerializer.Serialize(matches, new JsonSerializerOptions { WriteIndented = true });
    }

    [Description("Check if a product is in stock by its name.")]
    public static string CheckStock(
        [Description("Exact product name")] string productName)
    {
        var product = Products.FirstOrDefault(p =>
            p.Name.Equals(productName, StringComparison.OrdinalIgnoreCase));

        return product is null
            ? $"Product '{productName}' not found in catalog."
            : product.StockCount > 0
                ? $"'{productName}' is in stock ({product.StockCount} units available) at ${product.Price:F2}"
                : $"'{productName}' is currently OUT OF STOCK.";
    }
}
```

## Middleware and Cross-Cutting Concerns

```csharp
using Microsoft.Agents.AI;
using Microsoft.Extensions.Logging;

// Wrap agent with logging, timing, and retry middleware
public class LoggingAgentWrapper(AIAgent inner, ILogger logger) : AIAgent
{
    public override string Name => inner.Name;

    public override async Task<string> RunAsync(
        string input,
        CancellationToken cancellationToken = default)
    {
        logger.LogInformation("Agent '{Name}' starting. Input length: {Length}", Name, input.Length);
        var stopwatch = System.Diagnostics.Stopwatch.StartNew();

        try
        {
            string result = await inner.RunAsync(input, cancellationToken);
            stopwatch.Stop();
            logger.LogInformation("Agent '{Name}' completed in {Ms}ms. Output length: {Length}",
                Name, stopwatch.ElapsedMilliseconds, result.Length);
            return result;
        }
        catch (Exception ex)
        {
            stopwatch.Stop();
            logger.LogError(ex, "Agent '{Name}' failed after {Ms}ms", Name, stopwatch.ElapsedMilliseconds);
            throw;
        }
    }

    public override IAsyncEnumerable<string> RunStreamingAsync(
        string input,
        CancellationToken cancellationToken = default)
        => inner.RunStreamingAsync(input, cancellationToken);
}
```

## Best Practices (Do's)

✅ **Use Generic Host and DI for production agents**
```csharp
builder.Services.AddAIAgent("assistant", (sp, _) =>
    sp.GetRequiredService<AzureOpenAIClient>()
      .GetChatClient(deployment)
      .AsAIAgent(instructions: "..."));
```

✅ **Validate tool inputs before executing actions**
```csharp
[Description("Delete a user by ID. ID must be a positive integer.")]
public static async Task<string> DeleteUser(int userId)
{
    if (userId <= 0) return "Error: userId must be a positive integer.";
    await userService.DeleteAsync(userId);
    return $"User {userId} deleted.";
}
```

✅ **Use `CancellationToken` for all async operations**
```csharp
protected override async Task ExecuteAsync(CancellationToken stoppingToken)
{
    while (!stoppingToken.IsCancellationRequested)
    {
        string result = await agent.RunAsync(task, stoppingToken);
        await Task.Delay(interval, stoppingToken);
    }
}
```

✅ **Export/import agent sessions for stateful services**
```csharp
string state = await agent.ExportSessionAsync();
await cache.SetStringAsync($"agent:session:{userId}", state, expiry);
```

✅ **Route complex workflows to specialized agents**
```csharp
var specialist = category switch
{
    "BILLING" => billingAgent,
    "TECHNICAL" => techAgent,
    _ => generalAgent,
};
string response = await specialist.RunAsync(userQuery);
```

## Common Mistakes (Don'ts)

❌ **Don't share agent instances across concurrent requests without isolation**
```csharp
// Bad — shared conversation history causes cross-talk between users
private static AIAgent _sharedAgent = ...;
app.MapPost("/chat", async (string msg) => await _sharedAgent.RunAsync(msg));

// Good — create per-request or per-session agent instances
app.MapPost("/chat", async (string msg, IAIAgentFactory factory) =>
{
    var agent = factory.CreateAgent("chat");
    // restore session if needed
    return await agent.RunAsync(msg);
});
```

❌ **Don't create workflows without error handling**
```csharp
// Bad
string result = await workflow.RunAsync(input);

// Good
try
{
    string result = await workflow.RunAsync(input);
}
catch (WorkflowException ex)
{
    logger.LogError(ex, "Workflow '{Name}' failed at step '{Step}'", ex.WorkflowName, ex.FailedStep);
    return FallbackResponse(input);
}
```

❌ **Don't give agents overly broad permissions via tools**
```csharp
// Bad — agent can run ANY SQL command
[Description("Run any SQL query against the database.")]
public static string RunSql(string sql) => db.ExecuteRaw(sql);

// Good — limit to safe, parameterized read-only operations
[Description("Search products by name. Returns up to 10 matches.")]
public static async Task<string> SearchProducts(string name)
    => await db.Products.Where(p => p.Name.Contains(name)).Take(10).ToJsonAsync();
```

❌ **Don't forget to trim conversation history in long-running sessions**
```csharp
// Bad — history grows unbounded and eventually exceeds token limit
while (true)
    await agent.RunAsync(GetNextTask());

// Good — configure MaxConversationTurns or reset periodically
var agent = client.AsAIAgent(instructions: "...",
    options: new AIAgentOptions { MaxConversationTurns = 20 });
```
