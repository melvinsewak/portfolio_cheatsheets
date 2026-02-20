# Microsoft AutoGen Beginner Cheatsheet

## Installation

```bash
pip install autogen-agentchat "autogen-ext[openai]" python-dotenv
```

```python
import os
from dotenv import load_dotenv
load_dotenv()   # OPENAI_API_KEY=sk-... in .env file
```

## Model Clients

### OpenAI Model Client
```python
from autogen_ext.models.openai import OpenAIChatCompletionClient

# Basic OpenAI client
model_client = OpenAIChatCompletionClient(
    model="gpt-4o-mini",
    temperature=0.7,
    max_tokens=2048,
)
```

### Azure OpenAI Model Client
```python
from autogen_ext.models.openai import AzureOpenAIChatCompletionClient

azure_client = AzureOpenAIChatCompletionClient(
    model="gpt-4o",
    azure_deployment="my-gpt4o",
    api_version="2024-08-01-preview",
    azure_endpoint="https://my-resource.openai.azure.com/",
    # api_key="..."  # or use AZURE_OPENAI_API_KEY env var
)
```

## AssistantAgent — The Core Agent

```python
import asyncio
from autogen_agentchat.agents import AssistantAgent
from autogen_agentchat.ui import Console
from autogen_ext.models.openai import OpenAIChatCompletionClient

model_client = OpenAIChatCompletionClient(model="gpt-4o-mini")

# Create an agent
agent = AssistantAgent(
    name="assistant",
    model_client=model_client,
    system_message=(
        "You are a helpful assistant. "
        "Be concise and clear in your responses."
    ),
)

# Run a task — use Console for pretty-printed output during development
async def main():
    await Console(agent.run_stream(task="Explain Python generators in 3 bullet points."))

asyncio.run(main())
```

### Collecting the Final Response
```python
from autogen_agentchat.base import TaskResult

async def get_response(task: str) -> str:
    result: TaskResult = await agent.run(task=task)
    # result.messages[-1] is the final agent message
    return result.messages[-1].content

answer = asyncio.run(get_response("What is the Pythagorean theorem?"))
print(answer)
```

### Processing the Event Stream
```python
from autogen_agentchat.messages import TextMessage, ToolCallMessage

async def stream_agent(task: str) -> None:
    async for message in agent.run_stream(task=task):
        if isinstance(message, TextMessage):
            print(f"[{message.source}]: {message.content}")

asyncio.run(stream_agent("Tell me a fun fact about space."))
```

## Agent Conversations (Multi-Turn)

### Resuming a Conversation
```python
async def multi_turn() -> None:
    agent = AssistantAgent("assistant", model_client=model_client)

    # First turn
    result1 = await agent.run(task="My name is Alice.")

    # Second turn — agent remembers previous turns
    result2 = await agent.run(task="What is my name?")
    print(result2.messages[-1].content)   # "Your name is Alice."

asyncio.run(multi_turn())
```

### Resetting Agent State
```python
async def main():
    agent = AssistantAgent("assistant", model_client=model_client)
    await agent.run(task="Remember the number 42.")

    await agent.reset()   # clears message history

    result = await agent.run(task="What number did I ask you to remember?")
    print(result.messages[-1].content)   # won't know the number

asyncio.run(main())
```

## Tools

### Defining Tools
```python
from autogen_core.tools import FunctionTool

# Simple Python function as tool
def get_weather(city: str, unit: str = "celsius") -> str:
    """Get the current weather for a city.

    Args:
        city: Name of the city.
        unit: Temperature unit ('celsius' or 'fahrenheit').
    """
    # In real code, call a weather API here
    temp = 22 if unit == "celsius" else 72
    return f"The weather in {city} is {temp}°{'C' if unit == 'celsius' else 'F'} and sunny."

weather_tool = FunctionTool(get_weather, description="Get current weather for a city.")

def calculate(expression: str) -> str:
    """Evaluate a basic mathematical expression."""
    try:
        allowed = set("0123456789+-*/.() ")
        if any(c not in allowed for c in expression):
            return "Error: Only basic math expressions allowed."
        return str(round(eval(expression), 4))
    except Exception as e:
        return f"Error: {e}"

calc_tool = FunctionTool(calculate, description="Evaluate a mathematical expression.")
```

### Agent with Tools
```python
async def main():
    agent = AssistantAgent(
        name="assistant_with_tools",
        model_client=model_client,
        tools=[weather_tool, calc_tool],
        system_message=(
            "You are a helpful assistant with access to tools. "
            "Use tools when you need real-time data or calculations."
        ),
    )

    await Console(agent.run_stream(
        task="What's the weather in Tokyo? Also, what is 127 * 43 + 999?"
    ))

asyncio.run(main())
```

### Async Tools
```python
import aiohttp

async def fetch_joke() -> str:
    """Fetch a random programming joke from the internet."""
    async with aiohttp.ClientSession() as session:
        async with session.get("https://official-joke-api.appspot.com/jokes/programming/random") as resp:
            data = await resp.json()
            joke = data[0]
            return f"{joke['setup']} ... {joke['punchline']}"

joke_tool = FunctionTool(fetch_joke, description="Fetch a random programming joke.")
```

## Termination Conditions

```python
from autogen_agentchat.conditions import (
    MaxMessageTermination,
    TextMentionTermination,
    TokenUsageTermination,
    TimeoutTermination,
)

# Stop after N messages total
max_msg_termination = MaxMessageTermination(max_messages=10)

# Stop when agent says a specific phrase
text_termination = TextMentionTermination("TASK_COMPLETE")

# Stop if token budget exceeded
token_termination = TokenUsageTermination(max_total_token=50_000)

# Stop after N seconds
timeout_termination = TimeoutTermination(timeout_seconds=30)

# Combine conditions with OR (stop if ANY condition met)
from autogen_agentchat.conditions import ExternalTermination
combined = max_msg_termination | text_termination
```

## Two-Agent Conversation

### RoundRobinGroupChat
```python
from autogen_agentchat.teams import RoundRobinGroupChat
from autogen_agentchat.conditions import MaxMessageTermination, TextMentionTermination

async def two_agent_chat():
    teacher = AssistantAgent(
        name="teacher",
        model_client=model_client,
        system_message=(
            "You are a Python teacher. Explain concepts clearly with examples. "
            "When the student understands, end with 'DONE'."
        ),
    )

    student = AssistantAgent(
        name="student",
        model_client=model_client,
        system_message=(
            "You are a beginner Python student. Ask clarifying questions. "
            "When you understand, say 'I understand!'"
        ),
    )

    termination = TextMentionTermination("DONE") | MaxMessageTermination(10)

    team = RoundRobinGroupChat(
        participants=[teacher, student],
        termination_condition=termination,
    )

    await Console(team.run_stream(task="Teach me about Python list comprehensions."))

asyncio.run(two_agent_chat())
```

## UserProxyAgent — Human-in-the-Loop

```python
from autogen_agentchat.agents import UserProxyAgent

async def human_in_loop():
    assistant = AssistantAgent(
        name="assistant",
        model_client=model_client,
        system_message="You are a helpful assistant.",
    )

    # UserProxyAgent prompts a real human for input
    human = UserProxyAgent(name="human_proxy")

    team = RoundRobinGroupChat(
        participants=[assistant, human],
        termination_condition=MaxMessageTermination(6),
    )

    # Console displays messages; UserProxyAgent will prompt for input
    await Console(team.run_stream(task="Let's discuss AI ethics together."))

asyncio.run(human_in_loop())
```

## Simple Code Execution

```python
import tempfile
from autogen_ext.code_executors.local import LocalCommandLineCodeExecutor
from autogen_agentchat.agents import CodeExecutorAgent, AssistantAgent
from autogen_agentchat.teams import RoundRobinGroupChat
from autogen_agentchat.conditions import TextMentionTermination

async def coding_team():
    with tempfile.TemporaryDirectory() as work_dir:
        # Code executor (runs code locally — use Docker in production)
        executor = LocalCommandLineCodeExecutor(work_dir=work_dir)

        # Agent that writes code
        coder = AssistantAgent(
            name="coder",
            model_client=model_client,
            system_message=(
                "You write Python code to solve tasks. "
                "Always output code in a Python code block. "
                "When the task is verified working, say 'VERIFIED'."
            ),
        )

        # Agent that executes code
        executor_agent = CodeExecutorAgent(
            name="executor",
            code_executor=executor,
        )

        termination = TextMentionTermination("VERIFIED") | MaxMessageTermination(10)
        team = RoundRobinGroupChat([coder, executor_agent], termination_condition=termination)

        await Console(team.run_stream(
            task="Write and run a Python script that prints the first 10 Fibonacci numbers."
        ))

asyncio.run(coding_team())
```

## Saving and Loading State

```python
import json

async def save_and_restore():
    agent = AssistantAgent("assistant", model_client=model_client)
    await agent.run(task="Remember: the secret code is 7749.")

    # Save agent state
    state = await agent.save_state()
    state_json = json.dumps(state)

    # Create a new agent and restore state
    new_agent = AssistantAgent("assistant", model_client=model_client)
    await new_agent.load_state(json.loads(state_json))

    # New agent remembers previous conversation
    result = await new_agent.run(task="What is the secret code?")
    print(result.messages[-1].content)   # Should mention 7749

asyncio.run(save_and_restore())
```

## Best Practices (Do's)

✅ **Always use asyncio for all AutoGen operations**
```python
import asyncio

async def main():
    result = await agent.run(task="Hello!")
    print(result.messages[-1].content)

asyncio.run(main())
```

✅ **Set termination conditions to prevent infinite loops**
```python
termination = MaxMessageTermination(20) | TextMentionTermination("DONE")
team = RoundRobinGroupChat(agents, termination_condition=termination)
```

✅ **Use descriptive system messages for agent specialization**
```python
agent = AssistantAgent(
    name="data_analyst",
    model_client=model_client,
    system_message=(
        "You are a data analyst specializing in Python pandas and visualization. "
        "Always explain your reasoning before writing code."
    ),
)
```

✅ **Use `Console` for development, event streams for production**
```python
# Development
await Console(agent.run_stream(task=task))

# Production — process messages programmatically
async for message in agent.run_stream(task=task):
    if isinstance(message, TextMessage):
        await send_to_frontend(message.content)
```

✅ **Add type hints to tool function signatures**
```python
def get_stock_price(symbol: str, currency: str = "USD") -> dict:
    """Get the current stock price."""
    ...
```

## Common Mistakes (Don'ts)

❌ **Don't run synchronous code in an async agent**
```python
# Bad — blocks the event loop
def bad_tool(query: str) -> str:
    return requests.get(url).text   # blocking!

# Good — use async
async def good_tool(query: str) -> str:
    async with aiohttp.ClientSession() as session:
        async with session.get(url) as resp:
            return await resp.text()
```

❌ **Don't use `LocalCommandLineCodeExecutor` with untrusted input in production**
```python
# Dangerous — executes arbitrary code on your system
executor = LocalCommandLineCodeExecutor(work_dir="/tmp")

# Safe for production — use Docker sandbox
from autogen_ext.code_executors.docker import DockerCommandLineCodeExecutor
executor = DockerCommandLineCodeExecutor(image="python:3.12-slim", work_dir="/tmp")
```

❌ **Don't create agents without meaningful names**
```python
# Bad — hard to debug
agent = AssistantAgent("agent1", ...)

# Good — meaningful names appear in logs and messages
agent = AssistantAgent("python_code_reviewer", ...)
```

❌ **Don't forget to reset agent state between independent tasks**
```python
# Bad — previous conversation pollutes next task
await agent.run(task="Task A")
await agent.run(task="Unrelated Task B")  # carries context from Task A

# Good
await agent.run(task="Task A")
await agent.reset()
await agent.run(task="Unrelated Task B")
```
