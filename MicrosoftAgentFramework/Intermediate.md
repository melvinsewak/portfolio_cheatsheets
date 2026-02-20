# Microsoft AutoGen Intermediate Cheatsheet

## Multi-Agent Teams

### SelectorGroupChat — LLM-Driven Routing
```python
import asyncio
from autogen_agentchat.agents import AssistantAgent
from autogen_agentchat.teams import SelectorGroupChat
from autogen_agentchat.conditions import TextMentionTermination, MaxMessageTermination
from autogen_agentchat.ui import Console
from autogen_ext.models.openai import OpenAIChatCompletionClient

model_client = OpenAIChatCompletionClient(model="gpt-4o")

# Specialized agents
planner = AssistantAgent(
    name="planner",
    model_client=model_client,
    system_message=(
        "You are a project planner. Break complex tasks into sub-tasks "
        "and assign them to the appropriate specialist. Say 'DONE' when complete."
    ),
)

researcher = AssistantAgent(
    name="researcher",
    model_client=model_client,
    system_message=(
        "You are a research specialist. Gather information and summarize findings. "
        "Provide detailed, accurate information with sources when possible."
    ),
)

writer = AssistantAgent(
    name="writer",
    model_client=model_client,
    system_message=(
        "You are a technical writer. Transform research and plans into "
        "clear, well-structured documents and reports."
    ),
)

critic = AssistantAgent(
    name="critic",
    model_client=model_client,
    system_message=(
        "You are a quality reviewer. Critically evaluate outputs for accuracy, "
        "completeness, and clarity. Provide specific, actionable feedback."
    ),
)

termination = TextMentionTermination("DONE") | MaxMessageTermination(20)

# SelectorGroupChat uses an LLM to decide who speaks next
team = SelectorGroupChat(
    participants=[planner, researcher, writer, critic],
    model_client=model_client,
    termination_condition=termination,
    selector_prompt=(
        "You are managing a team of specialists: {participants}. "
        "Based on the conversation so far, select the most appropriate participant "
        "to contribute next. Return only the participant's name."
    ),
    allow_repeated_speaker=False,   # prevent same agent speaking twice in a row
)

async def main():
    await Console(team.run_stream(
        task="Create a comprehensive beginner's guide to Python decorators."
    ))

asyncio.run(main())
```

### Swarm — Handoff-Based Routing
```python
from autogen_agentchat.teams import Swarm
from autogen_agentchat.messages import HandoffMessage

# Swarm agents hand off to each other directly
triage_agent = AssistantAgent(
    name="triage",
    model_client=model_client,
    handoffs=["billing_agent", "tech_support_agent", "general_agent"],
    system_message=(
        "You are a customer service triage agent. "
        "Determine the nature of the customer's issue and hand off to the right team:\n"
        "- billing_agent: billing, payments, invoices\n"
        "- tech_support_agent: technical issues, bugs, setup\n"
        "- general_agent: general questions and information\n"
        "Use HANDOFF_TO_<agent_name> to route."
    ),
)

billing_agent = AssistantAgent(
    name="billing_agent",
    model_client=model_client,
    handoffs=["triage"],
    system_message=(
        "You are a billing specialist. Handle billing and payment queries professionally. "
        "Hand off back to triage if the issue is not billing-related."
    ),
)

tech_support_agent = AssistantAgent(
    name="tech_support_agent",
    model_client=model_client,
    handoffs=["triage"],
    system_message="You are a technical support specialist. Troubleshoot technical issues step by step.",
)

general_agent = AssistantAgent(
    name="general_agent",
    model_client=model_client,
    handoffs=["triage"],
    system_message="You handle general queries and provide helpful information.",
)

swarm_team = Swarm(
    participants=[triage_agent, billing_agent, tech_support_agent, general_agent],
    termination_condition=MaxMessageTermination(15),
)

async def customer_service(query: str):
    await Console(swarm_team.run_stream(task=query))

asyncio.run(customer_service("I was charged twice for my subscription last month!"))
```

## Code Execution

### Docker-Based Code Executor (Recommended for Production)
```python
from autogen_ext.code_executors.docker import DockerCommandLineCodeExecutor
from autogen_agentchat.agents import CodeExecutorAgent, AssistantAgent
from autogen_agentchat.teams import RoundRobinGroupChat
from autogen_agentchat.conditions import TextMentionTermination, MaxMessageTermination

async def data_analysis_team():
    # Docker provides an isolated sandbox — safe for untrusted code
    async with DockerCommandLineCodeExecutor(
        image="python:3.12-slim",
        work_dir="/tmp/autogen_work",
        timeout=30,             # kill container after 30 seconds
        auto_remove=True,       # remove container when done
    ) as executor:

        coder = AssistantAgent(
            name="data_scientist",
            model_client=model_client,
            system_message=(
                "You write Python code for data analysis tasks. "
                "Include all imports in your code blocks. "
                "When satisfied with results, say 'ANALYSIS COMPLETE'."
            ),
        )

        executor_agent = CodeExecutorAgent(
            name="code_runner",
            code_executor=executor,
        )

        termination = (
            TextMentionTermination("ANALYSIS COMPLETE") | MaxMessageTermination(12)
        )
        team = RoundRobinGroupChat([coder, executor_agent], termination_condition=termination)

        await Console(team.run_stream(
            task=(
                "Generate a list of 100 random numbers, calculate their mean, "
                "median, and standard deviation, then identify the top 5 outliers."
            )
        ))

asyncio.run(data_analysis_team())
```

### Jupyter Kernel Executor (Interactive Notebooks)
```python
from autogen_ext.code_executors.jupyter import JupyterCodeExecutor
from pathlib import Path

async def notebook_analysis():
    async with JupyterCodeExecutor(
        output_dir=Path("/tmp/autogen_notebooks"),
        kernel_name="python3",
        timeout=60,
    ) as executor:

        coder = AssistantAgent(
            name="notebook_coder",
            model_client=model_client,
            system_message=(
                "You write Python code for data analysis. "
                "Variables persist between code blocks. Say 'DONE' when complete."
            ),
        )

        executor_agent = CodeExecutorAgent("runner", code_executor=executor)
        termination = TextMentionTermination("DONE") | MaxMessageTermination(10)
        team = RoundRobinGroupChat([coder, executor_agent], termination_condition=termination)

        await Console(team.run_stream(
            task="Create a matplotlib plot of a sine wave and save it to sine_wave.png"
        ))

asyncio.run(notebook_analysis())
```

## Advanced Tool Use

### Tools with Complex Return Types
```python
from autogen_core.tools import FunctionTool
from pydantic import BaseModel

class WeatherData(BaseModel):
    city: str
    temperature_celsius: float
    humidity_percent: int
    condition: str
    wind_speed_kmh: float

async def get_detailed_weather(city: str) -> WeatherData:
    """Fetch detailed weather data for a city."""
    # Simulate API call
    import random
    return WeatherData(
        city=city,
        temperature_celsius=round(random.uniform(10, 35), 1),
        humidity_percent=random.randint(30, 90),
        condition=random.choice(["Sunny", "Cloudy", "Rainy", "Partly cloudy"]),
        wind_speed_kmh=round(random.uniform(5, 50), 1),
    )

weather_tool = FunctionTool(
    get_detailed_weather,
    description="Get detailed weather data for a city, including temperature, humidity, and wind speed.",
)

async def main():
    agent = AssistantAgent(
        name="weather_assistant",
        model_client=model_client,
        tools=[weather_tool],
        system_message="You are a weather assistant. Provide weather forecasts and comparisons.",
    )

    await Console(agent.run_stream(
        task="Compare the current weather in Tokyo, London, and New York."
    ))

asyncio.run(main())
```

### Tool with Error Handling
```python
import aiohttp

async def search_wikipedia(query: str, sentences: int = 3) -> str:
    """Search Wikipedia and return a summary.

    Args:
        query: The topic to search for.
        sentences: Number of summary sentences to return (1-10).
    """
    sentences = max(1, min(10, sentences))
    url = "https://en.wikipedia.org/api/rest_v1/page/summary/" + query.replace(" ", "_")

    try:
        async with aiohttp.ClientSession() as session:
            async with session.get(url, timeout=aiohttp.ClientTimeout(total=10)) as resp:
                if resp.status == 404:
                    return f"No Wikipedia article found for '{query}'"
                if resp.status != 200:
                    return f"Error fetching data: HTTP {resp.status}"
                data = await resp.json()
                extract = data.get("extract", "No description available.")
                # Return first N sentences
                parts = extract.split(". ")
                return ". ".join(parts[:sentences]) + ("." if len(parts) > sentences else "")
    except aiohttp.ClientTimeout:
        return "Request timed out — Wikipedia may be unreachable."
    except Exception as e:
        return f"Unexpected error: {e}"

wiki_tool = FunctionTool(
    search_wikipedia,
    description="Search Wikipedia for information about a topic and return a concise summary.",
)
```

## Memory and Context Management

### Agent Memory with External Storage
```python
import json
from pathlib import Path
from autogen_agentchat.agents import AssistantAgent

class PersistentAgent:
    """Wrapper that persists agent state to disk across sessions."""

    def __init__(self, name: str, model_client, state_file: str = None):
        self.agent = AssistantAgent(name=name, model_client=model_client)
        self.state_file = Path(state_file or f"{name}_state.json")

    async def __aenter__(self):
        if self.state_file.exists():
            state = json.loads(self.state_file.read_text())
            await self.agent.load_state(state)
            print(f"Loaded state from {self.state_file}")
        return self.agent

    async def __aexit__(self, *args):
        state = await self.agent.save_state()
        self.state_file.write_text(json.dumps(state))
        print(f"Saved state to {self.state_file}")


async def persistent_session():
    async with PersistentAgent("assistant", model_client) as agent:
        result = await agent.run(task="Remember: project deadline is March 15, 2026.")
        print(result.messages[-1].content)

async def continue_session():
    async with PersistentAgent("assistant", model_client) as agent:
        result = await agent.run(task="What is the project deadline?")
        print(result.messages[-1].content)

asyncio.run(persistent_session())
asyncio.run(continue_session())   # agent remembers across Python processes
```

### Message History Management
```python
from autogen_core.memory import ListMemory
from autogen_agentchat.agents import AssistantAgent

# AutoGen 0.4.x — pass initial messages for context injection
from autogen_agentchat.messages import TextMessage
from autogen_core import CancellationToken

async def agent_with_context():
    agent = AssistantAgent(
        name="assistant",
        model_client=model_client,
        system_message="You are a helpful assistant.",
    )

    # Inject prior conversation context
    context_messages = [
        TextMessage(content="Hello, I am working on a Django web app.", source="user"),
        TextMessage(content="Great! I can help with Django development.", source="assistant"),
    ]

    # Re-run the agent with an initial conversation history
    result = await agent.run(
        task="What framework am I using?",
    )
    print(result.messages[-1].content)

asyncio.run(agent_with_context())
```

## Human-in-the-Loop Patterns

### Approval-Based Execution
```python
from autogen_agentchat.agents import UserProxyAgent, AssistantAgent
from autogen_agentchat.teams import RoundRobinGroupChat
from autogen_agentchat.conditions import TextMentionTermination, MaxMessageTermination

async def approval_workflow():
    """An AI drafts content; a human approves or requests changes."""

    content_writer = AssistantAgent(
        name="content_writer",
        model_client=model_client,
        system_message=(
            "You write marketing content. When asked to revise, do so immediately. "
            "When the human says 'APPROVED', say 'Thank you! TASK_COMPLETE'."
        ),
    )

    # UserProxyAgent reads from stdin — human types responses
    human_reviewer = UserProxyAgent(
        name="human_reviewer",
        description="A human who reviews and approves content before publishing.",
    )

    termination = TextMentionTermination("TASK_COMPLETE") | MaxMessageTermination(10)

    team = RoundRobinGroupChat(
        participants=[content_writer, human_reviewer],
        termination_condition=termination,
    )

    await Console(team.run_stream(
        task="Write a 2-sentence tweet about the launch of our new Python course."
    ))

asyncio.run(approval_workflow())
```

### Automated Human Proxy (Testing/CI)
```python
from autogen_agentchat.agents import UserProxyAgent

class AutomatedUserProxy(UserProxyAgent):
    """Simulates human input for testing — no stdin required."""

    def __init__(self, responses: list[str], **kwargs):
        super().__init__(**kwargs)
        self._responses = iter(responses)

    async def on_messages_stream(self, messages, cancellation_token):
        response = next(self._responses, "DONE")
        from autogen_agentchat.messages import TextMessage
        yield TextMessage(content=response, source=self.name)


# Use in tests
automated_human = AutomatedUserProxy(
    responses=["Looks good, but make it shorter.", "APPROVED"],
    name="test_reviewer",
)
```

## Event-Driven Architecture

### Processing Agent Events
```python
from autogen_agentchat.messages import (
    TextMessage,
    ToolCallRequestEvent,
    ToolCallExecutionEvent,
    ThoughtEvent,
)
from autogen_agentchat.base import TaskResult

async def handle_events(task: str) -> None:
    agent = AssistantAgent(
        name="assistant",
        model_client=model_client,
        tools=[weather_tool, wiki_tool],
        reflect_on_tool_use=True,   # agent reasons after tool calls
    )

    async for event in agent.run_stream(task=task):
        if isinstance(event, ThoughtEvent):
            print(f"💭 Thinking: {event.content[:100]}...")
        elif isinstance(event, ToolCallRequestEvent):
            for call in event.content:
                print(f"🔧 Tool call: {call.name}({call.arguments})")
        elif isinstance(event, ToolCallExecutionEvent):
            for result in event.content:
                print(f"✅ Tool result: {result.content[:100]}")
        elif isinstance(event, TextMessage):
            print(f"💬 {event.source}: {event.content}")
        elif isinstance(event, TaskResult):
            print(f"\n📋 Task complete with {len(event.messages)} messages")

asyncio.run(handle_events("Search Wikipedia for quantum computing and check the weather in Seattle."))
```

## Parallel Agent Execution

```python
import asyncio
from autogen_agentchat.agents import AssistantAgent

async def run_agents_in_parallel():
    """Run multiple independent agents concurrently."""

    agents = [
        AssistantAgent(
            name=f"analyzer_{topic}",
            model_client=model_client,
            system_message=f"You are an expert in {topic}.",
        )
        for topic in ["machine learning", "web development", "DevOps"]
    ]

    tasks = [
        agent.run(task="What are the top 3 skills beginners should focus on in 2025?")
        for agent in agents
    ]

    # Run all agents simultaneously
    results = await asyncio.gather(*tasks)

    for agent, result in zip(agents, results):
        print(f"\n=== {agent.name} ===")
        print(result.messages[-1].content)

asyncio.run(run_agents_in_parallel())
```

## Best Practices (Do's)

✅ **Use `SelectorGroupChat` for dynamic, intelligent task routing**
```python
team = SelectorGroupChat(
    participants=[researcher, coder, writer, critic],
    model_client=model_client,
    termination_condition=termination,
)
```

✅ **Use Docker for safe code execution in production**
```python
async with DockerCommandLineCodeExecutor(
    image="python:3.12-slim",
    timeout=30,
    auto_remove=True,
) as executor:
    ...
```

✅ **Handle tool errors gracefully**
```python
async def robust_tool(query: str) -> str:
    """Search the web. Returns error message on failure."""
    try:
        return await actual_search(query)
    except Exception as e:
        return f"Search failed: {e}. Try a different query."
```

✅ **Use `reflect_on_tool_use=True` for complex reasoning tasks**
```python
agent = AssistantAgent(
    name="reasoner",
    model_client=model_client,
    tools=[...],
    reflect_on_tool_use=True,   # agent summarizes tool results before responding
)
```

✅ **Log all agent events for observability**
```python
async for event in team.run_stream(task=task):
    logger.info("event", extra={"type": type(event).__name__, "source": getattr(event, "source", None)})
```

## Common Mistakes (Don'ts)

❌ **Don't use `RoundRobinGroupChat` when you need smart routing**
```python
# Bad for complex tasks — always goes in fixed order
team = RoundRobinGroupChat([researcher, writer, critic])

# Good — LLM decides who speaks based on context
team = SelectorGroupChat([researcher, writer, critic], model_client=model_client)
```

❌ **Don't create a new model client for every agent (wastes connections)**
```python
# Bad
agent1 = AssistantAgent("agent1", model_client=OpenAIChatCompletionClient(model="gpt-4o"))
agent2 = AssistantAgent("agent2", model_client=OpenAIChatCompletionClient(model="gpt-4o"))

# Good — share the same client instance
client = OpenAIChatCompletionClient(model="gpt-4o")
agent1 = AssistantAgent("agent1", model_client=client)
agent2 = AssistantAgent("agent2", model_client=client)
```

❌ **Don't forget that tool docstrings are the agent's instructions**
```python
# Bad — vague description
def process(data: str) -> str:
    """Process data."""
    ...

# Good — clear, specific description
def summarize_text(text: str, max_words: int = 100) -> str:
    """Summarize the given text to at most max_words words, preserving key information."""
    ...
```

❌ **Don't let agents run without a termination condition**
```python
# Dangerous — may run indefinitely and incur large API costs
team = RoundRobinGroupChat([agent1, agent2])
await team.run(task="Discuss AI")   # no termination!

# Good
termination = MaxMessageTermination(20) | TextMentionTermination("DONE")
team = RoundRobinGroupChat([agent1, agent2], termination_condition=termination)
```
