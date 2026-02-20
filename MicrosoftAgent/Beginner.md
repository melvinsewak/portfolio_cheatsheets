# Microsoft Agent Framework (.NET) Beginner Cheatsheet

## Installation

```bash
# Create a new console project
dotnet new console -n MyAgentApp
cd MyAgentApp

# Install core packages
dotnet add package Microsoft.Agents.AI --prerelease
dotnet add package Microsoft.Agents.AI.OpenAI --prerelease
dotnet add package Azure.Identity
```

## Your First Agent

### Using Azure OpenAI
```csharp
using Microsoft.Agents.AI;
using Azure.AI.OpenAI;
using Azure.Identity;

// Create an agent backed by Azure OpenAI
AIAgent agent = new AzureOpenAIClient(
        new Uri(Environment.GetEnvironmentVariable("AZURE_OPENAI_ENDPOINT")!),
        new AzureCliCredential())
    .GetChatClient(Environment.GetEnvironmentVariable("AZURE_OPENAI_DEPLOYMENT_NAME")!)
    .AsAIAgent(instructions: "You are a helpful assistant. Be concise and clear.");

// Single turn — returns complete response as string
string response = await agent.RunAsync("What is the difference between value and reference types in C#?");
Console.WriteLine(response);
```

### Using OpenAI Directly
```csharp
using Microsoft.Agents.AI;
using OpenAI;

AIAgent agent = new OpenAIClient(
        Environment.GetEnvironmentVariable("OPENAI_API_KEY")!)
    .GetChatClient("gpt-4o-mini")
    .AsAIAgent(
        name: "CSharpTutor",
        instructions: "You are an expert C# tutor. Explain concepts with short code examples."
    );

string response = await agent.RunAsync("Explain async/await in C# with an example.");
Console.WriteLine(response);
```

### Using GitHub Models (Free Development Tier)
```csharp
using Microsoft.Agents.AI;
using OpenAI;

var options = new OpenAIClientOptions
{
    Endpoint = new Uri("https://models.inference.ai.azure.com"),
};

AIAgent agent = new OpenAIClient(
        new ApiKeyCredential(Environment.GetEnvironmentVariable("GITHUB_TOKEN")!),
        options)
    .GetChatClient("gpt-4o-mini")
    .AsAIAgent(instructions: "You are a helpful assistant.");

string response = await agent.RunAsync("List 3 best practices for C# exception handling.");
Console.WriteLine(response);
```

## Streaming Responses

```csharp
// Stream tokens as they arrive — better UX for long responses
await foreach (string chunk in agent.RunStreamingAsync("Write a detailed explanation of LINQ in C#."))
{
    Console.Write(chunk);
}
Console.WriteLine();

// Collect and stream simultaneously
var fullResponse = new System.Text.StringBuilder();
await foreach (string chunk in agent.RunStreamingAsync("Explain dependency injection."))
{
    Console.Write(chunk);
    fullResponse.Append(chunk);
}
Console.WriteLine();
string complete = fullResponse.ToString();
```

## Multi-Turn Conversations

```csharp
using Microsoft.Agents.AI;

AIAgent agent = new OpenAIClient(apiKey)
    .GetChatClient("gpt-4o-mini")
    .AsAIAgent(instructions: "You are a helpful assistant with memory of our conversation.");

// First turn
string reply1 = await agent.RunAsync("My name is Alice and I love C#.");
Console.WriteLine($"Agent: {reply1}");

// Second turn — agent remembers previous messages
string reply2 = await agent.RunAsync("What is my name and what language do I love?");
Console.WriteLine($"Agent: {reply2}");   // "Your name is Alice and you love C#."

// Third turn
string reply3 = await agent.RunAsync("Recommend a C# book for me.");
Console.WriteLine($"Agent: {reply3}");
```

## Adding Tools to Agents

### Simple Tool Method
```csharp
using Microsoft.Agents.AI;
using System.ComponentModel;

// Tools are regular C# static methods decorated with [Description]
public static class AgentTools
{
    [Description("Get the current UTC date and time in ISO 8601 format.")]
    public static string GetCurrentDateTime()
    {
        return DateTime.UtcNow.ToString("yyyy-MM-ddTHH:mm:ssZ");
    }

    [Description("Calculate the result of a basic math expression. " +
                 "Supports +, -, *, / operations on two numbers.")]
    public static double Calculate(
        [Description("The first number")] double a,
        [Description("The operation: add, subtract, multiply, divide")] string operation,
        [Description("The second number")] double b)
    {
        return operation.ToLower() switch
        {
            "add" or "+" => a + b,
            "subtract" or "-" => a - b,
            "multiply" or "*" => a * b,
            "divide" or "/" => b == 0 ? throw new DivideByZeroException("Cannot divide by zero") : a / b,
            _ => throw new ArgumentException($"Unknown operation: {operation}"),
        };
    }

    [Description("Get current weather for a city. Returns temperature and conditions.")]
    public static string GetWeather(
        [Description("The city name to get weather for")] string city)
    {
        // In production, call a real weather API here
        var random = new Random();
        double temp = Math.Round(random.NextDouble() * 25 + 5, 1);
        string[] conditions = ["Sunny", "Cloudy", "Rainy", "Partly cloudy"];
        return $"Weather in {city}: {temp}°C, {conditions[random.Next(conditions.Length)]}";
    }
}

// Register tools when creating the agent
AIAgent agentWithTools = new AzureOpenAIClient(endpoint, credential)
    .GetChatClient(deployment)
    .AsAIAgent(
        instructions: "You are a helpful assistant. Use tools when needed.",
        tools: [
            AgentTools.GetCurrentDateTime,
            AgentTools.Calculate,
            AgentTools.GetWeather,
        ]
    );

string result = await agentWithTools.RunAsync(
    "What time is it? Also, what is 127 * 43? And what's the weather in Tokyo?"
);
Console.WriteLine(result);
```

### Async Tool Methods
```csharp
using System.Net.Http;
using System.Text.Json;

public static class AsyncTools
{
    private static readonly HttpClient _http = new();

    [Description("Fetch a random programming joke from the internet.")]
    public static async Task<string> GetProgrammingJoke()
    {
        try
        {
            string json = await _http.GetStringAsync(
                "https://official-joke-api.appspot.com/jokes/programming/random"
            );
            using var doc = JsonDocument.Parse(json);
            var joke = doc.RootElement[0];
            return $"{joke.GetProperty("setup").GetString()} ... {joke.GetProperty("punchline").GetString()}";
        }
        catch (Exception ex)
        {
            return $"Could not fetch a joke: {ex.Message}";
        }
    }

    [Description("Search for information about a topic on Wikipedia. Returns a brief summary.")]
    public static async Task<string> SearchWikipedia(
        [Description("The topic to search for on Wikipedia")] string topic)
    {
        try
        {
            string encoded = Uri.EscapeDataString(topic.Replace(" ", "_"));
            string url = $"https://en.wikipedia.org/api/rest_v1/page/summary/{encoded}";
            string json = await _http.GetStringAsync(url);
            using var doc = JsonDocument.Parse(json);
            return doc.RootElement.TryGetProperty("extract", out var extract)
                ? extract.GetString() ?? "No description available."
                : "Topic not found on Wikipedia.";
        }
        catch (HttpRequestException)
        {
            return $"Could not find '{topic}' on Wikipedia.";
        }
    }
}
```

## System Messages and Personas

```csharp
// Specialized agent with a detailed system prompt
AIAgent reviewerAgent = new OpenAIClient(apiKey)
    .GetChatClient("gpt-4o")
    .AsAIAgent(
        name: "CodeReviewer",
        instructions: """
            You are a senior C# code reviewer with 10+ years of experience.
            When reviewing code:
            1. Check for correctness and potential bugs
            2. Identify performance issues
            3. Suggest adherence to SOLID principles
            4. Point out missing error handling
            5. Format feedback as: Issues Found / Suggestions / Summary
            Be constructive and specific.
            """,
        temperature: 0.3f   // lower temperature = more consistent, focused output
    );

string code = """
    public int Divide(int a, int b)
    {
        return a / b;
    }
    """;

string review = await reviewerAgent.RunAsync($"Please review this C# method:\n```csharp\n{code}\n```");
Console.WriteLine(review);
```

## Working with Agent Configuration

```csharp
using Microsoft.Agents.AI;

// Full configuration options when creating an agent
var agentOptions = new AIAgentOptions
{
    Name = "DataAnalyst",
    Instructions = "You are a data analysis assistant. Help with C# data processing tasks.",
    Temperature = 0.5f,
    MaxOutputTokens = 2048,
    // Tools will be inferred from registered methods
};

// Configuration from appsettings.json (recommended for production)
// appsettings.json:
// {
//   "Agent": {
//     "Endpoint": "https://my.openai.azure.com/",
//     "Deployment": "gpt-4o",
//     "Instructions": "You are a helpful assistant."
//   }
// }

// In Program.cs with Generic Host:
// builder.Services.Configure<AgentConfig>(builder.Configuration.GetSection("Agent"));
```

## Error Handling

```csharp
using Microsoft.Agents.AI;

try
{
    string response = await agent.RunAsync("Explain generics in C#.");
    Console.WriteLine(response);
}
catch (AgentException ex) when (ex.ErrorCode == "content_filter")
{
    Console.WriteLine("Response was filtered by content policy.");
}
catch (AgentException ex) when (ex.ErrorCode == "rate_limit_exceeded")
{
    Console.WriteLine("Rate limit hit. Retrying after delay...");
    await Task.Delay(TimeSpan.FromSeconds(10));
    string retryResponse = await agent.RunAsync("Explain generics in C#.");
    Console.WriteLine(retryResponse);
}
catch (TaskCanceledException)
{
    Console.WriteLine("Request timed out.");
}
catch (Exception ex)
{
    Console.WriteLine($"Unexpected error: {ex.Message}");
}
```

## Simple Console Chat Application

```csharp
using Microsoft.Agents.AI;
using OpenAI;

class Program
{
    static async Task Main(string[] args)
    {
        AIAgent agent = new OpenAIClient(
                Environment.GetEnvironmentVariable("OPENAI_API_KEY")!)
            .GetChatClient("gpt-4o-mini")
            .AsAIAgent(
                name: "Assistant",
                instructions: "You are a helpful assistant. Be friendly and concise."
            );

        Console.WriteLine("Chat with your AI assistant (type 'quit' to exit)\n");

        while (true)
        {
            Console.Write("You: ");
            string? input = Console.ReadLine()?.Trim();

            if (string.IsNullOrEmpty(input)) continue;
            if (input.Equals("quit", StringComparison.OrdinalIgnoreCase)) break;

            Console.Write("Assistant: ");
            await foreach (string chunk in agent.RunStreamingAsync(input))
            {
                Console.Write(chunk);
            }
            Console.WriteLine("\n");
        }
    }
}
```

## Best Practices (Do's)

✅ **Use Azure managed identity in production**
```csharp
// Good — no secrets in code or config files
var agent = new AzureOpenAIClient(endpoint, new ManagedIdentityCredential())
    .GetChatClient(deployment)
    .AsAIAgent(instructions: "...");

// Bad — secret in code!
var agent = new OpenAIClient("sk-hardcoded...")
    .GetChatClient("gpt-4o")
    .AsAIAgent(instructions: "...");
```

✅ **Use `[Description]` attributes for clear tool documentation**
```csharp
[Description("Search a product catalog by keyword. Returns matching products with prices.")]
public static async Task<string> SearchProducts(
    [Description("The search keyword")] string keyword,
    [Description("Maximum number of results (1-20)")] int maxResults = 5)
{ ... }
```

✅ **Stream responses for real-time UX**
```csharp
await foreach (string chunk in agent.RunStreamingAsync(userInput))
{
    Console.Write(chunk);
}
```

✅ **Read configuration from environment variables or appsettings.json**
```csharp
string endpoint = Environment.GetEnvironmentVariable("AZURE_OPENAI_ENDPOINT")
    ?? throw new InvalidOperationException("AZURE_OPENAI_ENDPOINT not configured");
```

✅ **Be specific in system instructions**
```csharp
.AsAIAgent(instructions: """
    You are a C# code reviewer. Only review C# code.
    Format your response as: Issues / Suggestions / Overall Rating (1-10).
    """)
```

## Common Mistakes (Don'ts)

❌ **Don't hard-code API keys**
```csharp
// Bad
var client = new OpenAIClient("sk-abc123...");

// Good
var client = new OpenAIClient(Environment.GetEnvironmentVariable("OPENAI_API_KEY")!);
```

❌ **Don't ignore tool parameter validation**
```csharp
// Bad — LLM may pass invalid values
public static double Divide(double a, double b) => a / b;

// Good — validate inputs
[Description("Divide a by b. b must not be zero.")]
public static double Divide(double a, double b)
{
    if (b == 0) throw new ArgumentException("Divisor cannot be zero.");
    return a / b;
}
```

❌ **Don't use vague system instructions**
```csharp
// Bad — too vague
.AsAIAgent(instructions: "You are an assistant.")

// Good — clear role and behavior
.AsAIAgent(instructions: "You are a C# code assistant. Only answer C# programming questions. For other topics, politely decline and redirect.")
```

❌ **Don't forget to handle `AgentException`**
```csharp
// Bad
string result = await agent.RunAsync(userInput);   // unhandled network/API errors

// Good
try { result = await agent.RunAsync(userInput); }
catch (AgentException ex) { /* handle gracefully */ }
```
