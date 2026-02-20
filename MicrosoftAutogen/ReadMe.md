# Microsoft AutoGen Cheatsheet

Welcome to the Microsoft AutoGen cheatsheet! This guide covers AutoGen 0.4.x, Microsoft's open-source framework for building multi-agent AI systems powered by large language models.

## Overview

AutoGen (from Microsoft Research) enables developers to build systems where multiple AI agents collaborate, delegate tasks, write and execute code, and interact with external tools to solve complex problems. AutoGen 0.4 introduces a complete redesign with an async-first, event-driven architecture and clear separation between the core framework and application components.

## Contents

- [Beginner.md](Beginner.md) - Installation, basic agents, single-agent tasks, and conversations
- [Intermediate.md](Intermediate.md) - Multi-agent teams, tools, code execution, and human-in-the-loop
- [Advanced.md](Advanced.md) - Custom agents, custom teams, orchestration, observability, and production patterns

## Quick Reference

### Install
```bash
pip install autogen-agentchat autogen-ext[openai]
```

### Simple Assistant Agent
```python
import asyncio
from autogen_agentchat.agents import AssistantAgent
from autogen_agentchat.ui import Console
from autogen_ext.models.openai import OpenAIChatCompletionClient

model_client = OpenAIChatCompletionClient(model="gpt-4o-mini")

agent = AssistantAgent(
    name="assistant",
    model_client=model_client,
    system_message="You are a helpful assistant.",
)

asyncio.run(Console(agent.run_stream(task="What is the capital of France?")))
```

### Two-Agent Conversation
```python
from autogen_agentchat.agents import AssistantAgent, UserProxyAgent
from autogen_agentchat.teams import RoundRobinGroupChat

researcher = AssistantAgent("researcher", model_client=model_client)
critic = AssistantAgent("critic", model_client=model_client,
                        system_message="Review the researcher's response critically.")

team = RoundRobinGroupChat([researcher, critic], max_turns=4)
asyncio.run(Console(team.run_stream(task="Explain quantum computing")))
```

### Key Features

- **Multi-Agent Collaboration**: Multiple specialized agents work together to solve tasks
- **Async-First Architecture**: Built on Python asyncio for high-performance applications
- **Code Execution**: Agents can write, execute, and iterate on code in sandboxed environments
- **Flexible Teams**: Round-robin, selector-based, swarm, and custom team topologies
- **Tool Use**: Agents can call Python functions, APIs, and external services
- **Human-in-the-Loop**: Easy integration of human feedback and approval workflows
- **Observable**: Full event stream for monitoring and debugging
- **Model Agnostic**: Works with OpenAI, Azure OpenAI, Anthropic, Gemini, Ollama, and more

## Getting Started

### Installation
```bash
# Core framework
pip install autogen-agentchat

# Model extensions (install what you need)
pip install "autogen-ext[openai]"        # OpenAI and Azure OpenAI
pip install "autogen-ext[anthropic]"     # Anthropic Claude
pip install "autogen-ext[ollama]"        # Local Ollama models
pip install "autogen-ext[gemini]"        # Google Gemini

# Code execution
pip install "autogen-ext[docker]"        # Docker-based code execution (recommended)
pip install "autogen-ext[jupyter]"       # Jupyter kernel executor

# Vector store and RAG
pip install "autogen-ext[chromadb]"
```

### Environment Setup
```bash
export OPENAI_API_KEY="sk-..."
export AZURE_OPENAI_API_KEY="..."
export AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
```

### Supported Model Clients
```python
from autogen_ext.models.openai import OpenAIChatCompletionClient, AzureOpenAIChatCompletionClient

# OpenAI
client = OpenAIChatCompletionClient(model="gpt-4o")

# Azure OpenAI
azure_client = AzureOpenAIChatCompletionClient(
    model="gpt-4o",
    azure_deployment="my-gpt4o-deployment",
    api_version="2024-08-01-preview",
    azure_endpoint="https://my-resource.openai.azure.com/",
)

# Anthropic
from autogen_ext.models.anthropic import AnthropicChatCompletionClient
claude = AnthropicChatCompletionClient(model="claude-3-5-sonnet-20241022")

# Ollama (local)
from autogen_ext.models.ollama import OllamaChatCompletionClient
local = OllamaChatCompletionClient(model="llama3.2")
```

## Core Concepts

### Architecture Overview
```
┌─────────────────────────────────────────────────────┐
│                    Your Application                 │
└────────────────────────┬────────────────────────────┘
                         │
          ┌──────────────▼──────────────┐
          │         Team / Chat         │
          │  (RoundRobin / Selector /   │
          │   Swarm / MagenticOne)      │
          └──────┬──────────────┬───────┘
                 │              │
      ┌──────────▼──┐  ┌────────▼──────────┐
      │  AssistantAgent│  │  CodeExecutorAgent│
      │  (LLM-based) │  │  (runs code)      │
      └──────┬───────┘  └───────────────────┘
             │
      ┌──────▼──────────┐
      │  Model Client   │
      │ OpenAI / Azure  │
      │ Claude / Ollama │
      └─────────────────┘
```

### Message Types
- **TextMessage**: Plain text between agents
- **ToolCallMessage**: Agent requests tool execution
- **ToolCallResultMessage**: Tool execution result
- **HandoffMessage**: Pass control to another agent (Swarm)
- **StopMessage**: Signal to end the conversation

## Quick Tips

✅ **DO:**
- Use `asyncio.run()` or `await` for all AutoGen operations (fully async)
- Use Docker for code execution sandboxing in production
- Set `max_turns` on teams to prevent infinite loops
- Use `Console` for development; process event streams for production
- Leverage `SelectorGroupChat` for intelligent task delegation

❌ **DON'T:**
- Run agents without a `max_turns` or termination condition in production
- Execute untrusted code with `LocalCommandLineCodeExecutor` in production
- Hard-code API keys in source code
- Skip testing agents individually before combining them into teams
- Ignore agent message history when debugging unexpected behavior

## Additional Resources

- [AutoGen Documentation](https://microsoft.github.io/autogen/)
- [AutoGen GitHub](https://github.com/microsoft/autogen)
- [AutoGen 0.4 Migration Guide](https://microsoft.github.io/autogen/stable/user-guide/autogenstudio-user-guide/index.html)
- [AutoGen Studio (No-Code UI)](https://microsoft.github.io/autogen/stable/user-guide/autogenstudio-user-guide/index.html)
- [Model Capabilities Guide](https://microsoft.github.io/autogen/stable/user-guide/agentchat-user-guide/model-capabilities.html)
- [AutoGen Discord Community](https://discord.gg/autogen)
