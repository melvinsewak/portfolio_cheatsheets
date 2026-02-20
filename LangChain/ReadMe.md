# LangChain Cheatsheet

Welcome to the LangChain cheatsheet! This guide covers LangChain 0.3.x, the leading Python framework for building applications powered by large language models (LLMs).

## Overview

LangChain provides composable abstractions for LLMs, prompt management, retrieval-augmented generation (RAG), agents, and multi-step chains. It integrates with OpenAI, Anthropic, Google, Azure, Hugging Face, local models via Ollama, and dozens of vector stores, document loaders, and tools.

## Contents

- [Beginner.md](Beginner.md) - Installation, LLMs, prompts, output parsers, and simple chains
- [Intermediate.md](Intermediate.md) - RAG, vector stores, agents, tools, memory, and streaming
- [Advanced.md](Advanced.md) - Custom chains, multi-agent systems, LangGraph, callbacks, and production patterns

## Quick Reference

### Install
```bash
pip install langchain langchain-openai langchain-community
```

### Simple LLM Call
```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4o-mini")
response = llm.invoke("What is the capital of France?")
print(response.content)   # Paris
```

### Prompt + Chain
```python
from langchain_core.prompts import ChatPromptTemplate

prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    ("user", "{question}"),
])

chain = prompt | llm
result = chain.invoke({"question": "Explain async/await in Python."})
print(result.content)
```

### Key Features

- **Model Agnostic**: Works with OpenAI, Anthropic, Gemini, Mistral, local models, and more
- **LCEL (LangChain Expression Language)**: Pipe `|` operator for composing chains declaratively
- **RAG Support**: Built-in loaders, splitters, embeddings, and vector store integrations
- **Agents & Tools**: LLM-driven decision loops with tool use (web search, code execution, APIs)
- **LangGraph**: Graph-based multi-agent orchestration
- **Streaming**: First-class async streaming of tokens and events
- **Observability**: LangSmith tracing for debugging and monitoring

## Getting Started

### Installation
```bash
# Core library
pip install langchain langchain-core langchain-community

# LLM providers (install what you need)
pip install langchain-openai         # OpenAI / Azure OpenAI
pip install langchain-anthropic      # Anthropic Claude
pip install langchain-google-genai   # Google Gemini
pip install langchain-ollama         # Local models via Ollama

# Vector stores
pip install langchain-chroma         # ChromaDB
pip install langchain-pinecone       # Pinecone

# Additional tools
pip install langchain-text-splitters
pip install faiss-cpu                # FAISS vector store
```

### Environment Setup
```bash
# Set API keys (use .env file with python-dotenv in practice)
export OPENAI_API_KEY="sk-..."
export ANTHROPIC_API_KEY="sk-ant-..."
export LANGCHAIN_TRACING_V2=true     # Enable LangSmith tracing
export LANGCHAIN_API_KEY="ls__..."
```

#### Azure OpenAI Service
```bash
# Azure OpenAI requires different environment variables
export AZURE_OPENAI_API_KEY="..."
export AZURE_OPENAI_ENDPOINT="https://<your-resource-name>.openai.azure.com/"
export OPENAI_API_VERSION="2024-12-01-preview"   # use the API version that matches your deployment
```

```python
from dotenv import load_dotenv
load_dotenv()   # loads .env file automatically
```

### Supported LLM Providers
```python
from langchain_openai import ChatOpenAI, AzureChatOpenAI
from langchain_anthropic import ChatAnthropic
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain_ollama import ChatOllama   # local models

# OpenAI Chat
llm = ChatOpenAI(model="gpt-4o", temperature=0.7)

# Azure OpenAI Service (reads AZURE_OPENAI_API_KEY, AZURE_OPENAI_ENDPOINT,
# and OPENAI_API_VERSION from the environment)
llm = AzureChatOpenAI(
    azure_deployment="gpt-4o",   # your Azure deployment name
)

# Anthropic Claude
llm = ChatAnthropic(model="claude-3-5-sonnet-20241022")

# Google Gemini
llm = ChatGoogleGenerativeAI(model="gemini-1.5-pro")

# Local Ollama (no API key required)
llm = ChatOllama(model="llama3.2")
```

## Core Concepts

### Messages
```python
from langchain_core.messages import HumanMessage, AIMessage, SystemMessage

messages = [
    SystemMessage("You are a Python tutor."),
    HumanMessage("What is a list comprehension?"),
]

response = llm.invoke(messages)
print(response.content)    # AIMessage
print(type(response))      # <class 'langchain_core.messages.ai.AIMessage'>
```

### LCEL (LangChain Expression Language)
```python
# Chain components using | (pipe operator)
chain = prompt | llm | output_parser

# All components share the same interface:
# - invoke(input)        — synchronous single call
# - ainvoke(input)       — async single call
# - stream(input)        — sync streaming
# - astream(input)       — async streaming
# - batch([inputs])      — parallel batch
```

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────┐
│                    Your Application                     │
└──────────────────────────┬──────────────────────────────┘
                           │
          ┌────────────────▼─────────────────┐
          │        LangChain Chains           │
          │  PromptTemplate → LLM → Parser   │
          └────────────────┬─────────────────┘
                    ┌──────┴──────┐
                    │             │
          ┌─────────▼───┐  ┌──────▼──────┐
          │  LLM Models │  │ Vector Store│
          │ OpenAI/Ollama│  │ Chroma/FAISS│
          └─────────────┘  └─────────────┘
```

## Quick Tips

✅ **DO:**
- Use LCEL (`|`) to compose chains declaratively
- Use environment variables for API keys (never hard-code them)
- Enable LangSmith tracing during development for debugging
- Use async methods (`ainvoke`, `astream`) in web applications
- Pin LangChain versions in production (`langchain==0.3.x`)

❌ **DON'T:**
- Hard-code API keys in source code
- Use deprecated `LLMChain` (replaced by LCEL pipes)
- Ignore token limits — chunk documents appropriately
- Skip rate limit handling for production workloads
- Forget to test prompts systematically before deploying

## Additional Resources

- [LangChain Documentation](https://python.langchain.com/)
- [LangChain API Reference](https://api.python.langchain.com/)
- [LangSmith (Observability)](https://smith.langchain.com/)
- [LangGraph](https://langchain-ai.github.io/langgraph/)
- [LangChain Hub (Prompts)](https://smith.langchain.com/hub)
- [LangChain GitHub](https://github.com/langchain-ai/langchain)
- [What's New in LangChain 0.3](https://python.langchain.com/docs/versions/v0_3/)
