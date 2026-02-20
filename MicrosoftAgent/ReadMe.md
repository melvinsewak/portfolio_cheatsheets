# Microsoft Agent Framework (.NET) Cheatsheet

Welcome to the Microsoft Agent Framework cheatsheet! This guide covers the Microsoft Agent Framework (Preview) for .NET — a unified SDK for building, orchestrating, and deploying AI agents and multi-agent workflows in C#.

## Overview

The Microsoft Agent Framework (`Microsoft.Agents.AI`) is a first-class .NET framework for building intelligent agents and multi-agent workflows. It merges concepts from Semantic Kernel and AutoGen into a streamlined, production-ready developer experience with native support for tools, memory, workflows, human-in-the-loop, and observability. The framework integrates with OpenAI, Azure OpenAI, GitHub Models, Anthropic, and other providers via `Microsoft.Extensions.AI`.

## Contents

- [Beginner.md](Beginner.md) - Installation, first agent, messages, streaming, and tools
- [Intermediate.md](Intermediate.md) - Multi-agent workflows, memory, human-in-the-loop, and dependency injection
- [Advanced.md](Advanced.md) - Custom agents, orchestration, observability, MCP integration, and production patterns

## Quick Reference

### Install Packages
```bash
dotnet add package Microsoft.Agents.AI --prerelease
dotnet add package Microsoft.Agents.AI.OpenAI --prerelease
dotnet add package Microsoft.Agents.AI.Workflows --prerelease
```

### Hello World Agent
```csharp
using Microsoft.Agents.AI;

AIAgent agent = new AzureOpenAIClient(
        new Uri(Environment.GetEnvironmentVariable("AZURE_OPENAI_ENDPOINT")!),
        new AzureCliCredential())
    .GetChatClient("gpt-4o-mini")
    .AsAIAgent(instructions: "You are a helpful assistant.");

Console.WriteLine(await agent.RunAsync("What is the capital of France?"));
```

### Two-Agent Workflow
```csharp
using Microsoft.Agents.AI.Workflows;

var workflow = new WorkflowBuilder()
    .AddAgent("researcher", researcherAgent)
    .AddAgent("writer", writerAgent)
    .Connect("researcher", "writer")
    .Build();

string result = await workflow.RunAsync("Write a blog post about async/await in C#.");
Console.WriteLine(result);
```

### Key Features

- **Model Agnostic**: Works with Azure OpenAI, OpenAI, GitHub Models, Anthropic, Ollama, and custom IChatClient providers
- **Native .NET**: First-class dependency injection, middleware, and `IHostedService` integration
- **Workflows**: Chain agents in typed, sequential or graph-based pipelines
- **Tool Use**: Decorate C# methods with `[Description]` to expose them as LLM-callable tools
- **Memory**: Built-in conversation history, session persistence, and context windows
- **Human-in-the-Loop**: Approval steps and manual intervention points in workflows
- **Observability**: OpenTelemetry traces, metrics, and Microsoft Foundry dashboard integration
- **MCP Support**: Model Context Protocol (MCP) for standardized tool interoperability

## Getting Started

### Prerequisites
- .NET 8 or later
- Azure OpenAI resource or OpenAI API key (or GitHub Models for free dev quota)

### Installation
```bash
# Core agent framework
dotnet add package Microsoft.Agents.AI --prerelease
dotnet add package Microsoft.Agents.AI.OpenAI --prerelease

# Workflow orchestration
dotnet add package Microsoft.Agents.AI.Workflows --prerelease

# Azure identity (for managed identity / CLI auth)
dotnet add package Azure.Identity
```

### Environment Configuration
```bash
# Azure OpenAI
export AZURE_OPENAI_ENDPOINT="https://my-resource.openai.azure.com/"
export AZURE_OPENAI_DEPLOYMENT_NAME="gpt-4o"

# OpenAI directly
export OPENAI_API_KEY="sk-..."

# GitHub Models (free tier for development)
export GITHUB_TOKEN="ghp_..."
```

### Supported Model Providers
```csharp
// Azure OpenAI (with Azure CLI / Managed Identity)
using Azure.Identity;

AIAgent azureAgent = new AzureOpenAIClient(
        new Uri(Environment.GetEnvironmentVariable("AZURE_OPENAI_ENDPOINT")!),
        new AzureCliCredential())
    .GetChatClient(Environment.GetEnvironmentVariable("AZURE_OPENAI_DEPLOYMENT_NAME")!)
    .AsAIAgent(instructions: "You are a helpful assistant.");

// OpenAI directly
using OpenAI;

AIAgent openaiAgent = new OpenAIClient(
        Environment.GetEnvironmentVariable("OPENAI_API_KEY")!)
    .GetChatClient("gpt-4o-mini")
    .AsAIAgent(instructions: "You are a helpful assistant.");

// GitHub Models (free tier — great for development)
AIAgent githubAgent = new OpenAIClient(
        new ApiKeyCredential(Environment.GetEnvironmentVariable("GITHUB_TOKEN")!),
        new OpenAIClientOptions { Endpoint = new Uri("https://models.inference.ai.azure.com") })
    .GetChatClient("gpt-4o-mini")
    .AsAIAgent(instructions: "You are a helpful assistant.");
```

## Core Concepts

### Architecture Overview
```
┌─────────────────────────────────────────────────────────┐
│                     Your .NET App                       │
└────────────────────────┬────────────────────────────────┘
                         │
          ┌──────────────▼──────────────┐
          │    Workflow / Orchestration  │
          │  (WorkflowBuilder / Graph)   │
          └──────┬──────────────┬───────┘
                 │              │
      ┌──────────▼──┐  ┌────────▼──────────┐
      │   AIAgent   │  │   AIAgent (Tool)   │
      │  (LLM core) │  │  (w/ C# functions) │
      └──────┬──────┘  └────────────────────┘
             │
      ┌──────▼──────────────────┐
      │   IChatClient Provider   │
      │ Azure OpenAI / OpenAI /  │
      │ GitHub Models / Ollama   │
      └─────────────────────────┘
```

## Quick Tips

✅ **DO:**
- Use `AzureCliCredential` or `ManagedIdentityCredential` — avoid API keys in production
- Decorate tool methods with `[Description("...")]` for best LLM tool selection
- Use `IHostedService` / Generic Host for long-running agent services
- Enable OpenTelemetry for production observability
- Pin preview package versions in CI (`--version x.y.z-preview.N`)

❌ **DON'T:**
- Hard-code API keys or connection strings in source code
- Run tools without input validation — the LLM controls tool arguments
- Skip error handling in tool methods — unhandled exceptions surface to the LLM
- Ignore token limits — keep conversation history trimmed for long-running sessions
- Use preview packages in production without thorough testing

## Additional Resources

- [Microsoft Agent Framework Docs (Learn)](https://learn.microsoft.com/en-us/agent-framework/overview/)
- [Get Started: Your First Agent](https://learn.microsoft.com/en-us/agent-framework/get-started/your-first-agent)
- [GitHub: microsoft/agent-framework](https://github.com/microsoft/agent-framework)
- [Agent Framework Samples](https://github.com/microsoft/Agent-Framework-Samples)
- [NuGet: Microsoft.Agents.AI](https://www.nuget.org/profiles/MicrosoftAgentFramework/)
- [AI Agents for Beginners (Microsoft)](https://microsoft.github.io/ai-agents-for-beginners/14-microsoft-agent-framework/)
- [Microsoft.Extensions.AI](https://learn.microsoft.com/en-us/dotnet/ai/microsoft-extensions-ai)
