# Microsoft AutoGen Advanced Cheatsheet

## Custom Agents

### Building a Custom Agent from Scratch
```python
import asyncio
from typing import AsyncGenerator, Sequence
from autogen_agentchat.agents import BaseChatAgent
from autogen_agentchat.base import Response, TaskResult
from autogen_agentchat.messages import (
    AgentEvent,
    BaseChatMessage,
    ChatMessage,
    TextMessage,
)
from autogen_core import CancellationToken

class SentimentAnalysisAgent(BaseChatAgent):
    """Custom agent that analyses sentiment without calling an LLM."""

    def __init__(self, name: str) -> None:
        super().__init__(name=name, description="Analyses sentiment of text messages.")
        self._history: list[BaseChatMessage] = []

    @property
    def produced_message_types(self) -> list[type[ChatMessage]]:
        return [TextMessage]

    async def on_messages(
        self,
        messages: Sequence[ChatMessage],
        cancellation_token: CancellationToken,
    ) -> Response:
        # Collect all responses from stream
        responses: list[AgentEvent | ChatMessage] = []
        async for msg in self.on_messages_stream(messages, cancellation_token):
            responses.append(msg)
        return Response(chat_message=responses[-1], inner_messages=responses[:-1])

    async def on_messages_stream(
        self,
        messages: Sequence[ChatMessage],
        cancellation_token: CancellationToken,
    ) -> AsyncGenerator[AgentEvent | ChatMessage | Response, None]:
        for message in messages:
            self._history.append(message)

        # Get the last human message
        last = messages[-1].content if messages else ""

        # Simple keyword-based sentiment (replace with a real ML model in production)
        positive_words = {"great", "excellent", "love", "amazing", "fantastic", "good"}
        negative_words = {"bad", "terrible", "hate", "awful", "poor", "worst"}
        words = set(last.lower().split())

        pos = len(words & positive_words)
        neg = len(words & negative_words)

        if pos > neg:
            sentiment, score = "POSITIVE", round(pos / (pos + neg + 1), 2)
        elif neg > pos:
            sentiment, score = "NEGATIVE", round(-neg / (pos + neg + 1), 2)
        else:
            sentiment, score = "NEUTRAL", 0.0

        response = TextMessage(
            content=f"Sentiment: {sentiment} (score: {score:+.2f})",
            source=self.name,
        )
        self._history.append(response)
        yield response

    async def on_reset(self, cancellation_token: CancellationToken) -> None:
        self._history.clear()


# Usage
async def main():
    agent = SentimentAnalysisAgent("sentiment_analyzer")
    result = await agent.run(task="This product is absolutely terrible and awful!")
    print(result.messages[-1].content)   # Sentiment: NEGATIVE (score: -0.67)

asyncio.run(main())
```

### LLM-Powered Custom Agent with Memory
```python
from autogen_agentchat.agents import BaseChatAgent
from autogen_agentchat.base import Response
from autogen_agentchat.messages import TextMessage, ChatMessage, AgentEvent
from autogen_core import CancellationToken
from autogen_ext.models.openai import OpenAIChatCompletionClient
from autogen_core.models import (
    SystemMessage,
    UserMessage,
    AssistantMessage,
    CreateResult,
)
from typing import AsyncGenerator, Sequence

class MemoryEnhancedAgent(BaseChatAgent):
    """Agent that maintains a rolling summary of past conversations."""

    def __init__(self, name: str, model_client: OpenAIChatCompletionClient,
                 max_history: int = 10) -> None:
        super().__init__(name=name, description="Agent with rolling conversation memory.")
        self._model_client = model_client
        self._max_history = max_history
        self._messages: list = [
            SystemMessage(content=(
                "You are a helpful assistant with memory. "
                "Reference previous conversations when relevant."
            ))
        ]
        self._turn_count = 0

    @property
    def produced_message_types(self):
        return [TextMessage]

    async def on_messages_stream(
        self,
        messages: Sequence[ChatMessage],
        cancellation_token: CancellationToken,
    ) -> AsyncGenerator[AgentEvent | ChatMessage | Response, None]:
        self._turn_count += 1

        # Add new messages to history
        for msg in messages:
            self._messages.append(UserMessage(content=msg.content, source=msg.source))

        # Trim history if too long (keep system message + last N messages)
        if len(self._messages) > self._max_history + 1:
            self._messages = [self._messages[0]] + self._messages[-(self._max_history):]

        # Call the model
        result: CreateResult = await self._model_client.create(
            messages=self._messages,
            cancellation_token=cancellation_token,
        )

        response_text = result.content
        self._messages.append(AssistantMessage(content=response_text, source=self.name))

        response_msg = TextMessage(content=response_text, source=self.name)
        yield response_msg

    async def on_messages(self, messages, cancellation_token):
        all_messages = []
        async for msg in self.on_messages_stream(messages, cancellation_token):
            all_messages.append(msg)
        return Response(chat_message=all_messages[-1], inner_messages=all_messages[:-1])

    async def on_reset(self, cancellation_token: CancellationToken) -> None:
        self._messages = [self._messages[0]]  # keep system message
        self._turn_count = 0
```

## Advanced Team Patterns

### MagenticOne — General-Purpose Multi-Agent System
```python
# pip install "autogen-ext[magentic-one]"
from autogen_ext.teams.magentic_one import MagenticOne
from autogen_ext.models.openai import OpenAIChatCompletionClient

async def magentic_one_example():
    client = OpenAIChatCompletionClient(model="gpt-4o")

    # MagenticOne includes Orchestrator + Coder + Executor + FileSurfer + WebSurfer
    m1 = MagenticOne(client=client)

    # Handles complex multi-step tasks autonomously
    result = await m1.run(task=(
        "Research the top 3 Python testing frameworks, "
        "write a comparison table, and save it to comparison.md"
    ))
    print(result.messages[-1].content)

asyncio.run(magentic_one_example())
```

### Custom Team with GraphFlow
```python
from autogen_agentchat.teams import DiGraphBuilder, GraphFlow
from autogen_agentchat.agents import AssistantAgent
from autogen_agentchat.conditions import MaxMessageTermination

async def pipeline_team():
    client = OpenAIChatCompletionClient(model="gpt-4o-mini")

    # Define agents
    extractor = AssistantAgent("extractor", model_client=client,
                               system_message="Extract key requirements from user stories.")
    designer = AssistantAgent("designer", model_client=client,
                              system_message="Design a system architecture from requirements.")
    reviewer = AssistantAgent("reviewer", model_client=client,
                              system_message="Review and approve the design. Say 'APPROVED' if good.")

    # Build a directed graph: extractor → designer → reviewer
    builder = DiGraphBuilder()
    builder.add_node(extractor).add_node(designer).add_node(reviewer)
    builder.add_edge(extractor, designer)
    builder.add_edge(designer, reviewer)
    builder.set_entry_point(extractor)

    graph = builder.build()

    team = GraphFlow(
        participants=[extractor, designer, reviewer],
        graph=graph,
        termination_condition=MaxMessageTermination(6),
    )

    await Console(team.run_stream(
        task="As a user, I want to upload photos and share them with friends."
    ))

asyncio.run(pipeline_team())
```

## Custom Termination Conditions

```python
from autogen_agentchat.base import TerminationCondition, TerminatedException
from autogen_agentchat.messages import ChatMessage, TextMessage, StopMessage
from typing import Sequence

class ConsensusTermination(TerminationCondition):
    """Terminate when N agents all explicitly agree."""

    def __init__(self, required_agents: list[str], agreement_phrase: str = "I AGREE") -> None:
        self._required = set(required_agents)
        self._phrase = agreement_phrase.upper()
        self._agreed: set[str] = set()

    @property
    def terminated(self) -> bool:
        return self._agreed >= self._required

    async def __call__(self, messages: Sequence[ChatMessage]) -> StopMessage | None:
        for msg in messages:
            if (
                isinstance(msg, TextMessage)
                and msg.source in self._required
                and self._phrase in msg.content.upper()
            ):
                self._agreed.add(msg.source)

        if self.terminated:
            return StopMessage(
                content=f"Consensus reached! All agents agreed: {self._agreed}",
                source="ConsensusTermination",
            )
        return None

    async def reset(self) -> None:
        self._agreed.clear()


# Usage
consensus = ConsensusTermination(
    required_agents=["agent_a", "agent_b", "agent_c"],
    agreement_phrase="I AGREE",
)
combined = consensus | MaxMessageTermination(20)
```

## Observability and Tracing

### Structured Event Logging
```python
import logging
import json
from autogen_agentchat.messages import (
    TextMessage, ToolCallRequestEvent, ToolCallExecutionEvent,
    ThoughtEvent, ModelClientStreamingChunkEvent,
)
from autogen_agentchat.base import TaskResult

logging.basicConfig(level=logging.INFO, format="%(asctime)s %(levelname)s %(message)s")
logger = logging.getLogger(__name__)

async def traced_run(team, task: str) -> TaskResult:
    total_tool_calls = 0
    total_messages = 0

    async for event in team.run_stream(task=task):
        if isinstance(event, ThoughtEvent):
            logger.debug("agent.thought source=%s chars=%d", event.source, len(event.content))
        elif isinstance(event, ToolCallRequestEvent):
            total_tool_calls += len(event.content)
            for call in event.content:
                logger.info("tool.call name=%s source=%s", call.name, event.source)
        elif isinstance(event, ToolCallExecutionEvent):
            for r in event.content:
                logger.info("tool.result name=%s success=%s", r.call_id, r.is_error is False)
        elif isinstance(event, TextMessage):
            total_messages += 1
            logger.info("message source=%s words=%d", event.source, len(event.content.split()))
        elif isinstance(event, TaskResult):
            logger.info(
                "task.complete messages=%d tool_calls=%d",
                len(event.messages), total_tool_calls,
            )
            return event

    return None
```

### Integration with OpenTelemetry
```python
# pip install opentelemetry-sdk opentelemetry-exporter-otlp
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter

# Setup tracer
provider = TracerProvider()
provider.add_span_processor(BatchSpanProcessor(OTLPSpanExporter()))
trace.set_tracer_provider(provider)
tracer = trace.get_tracer("autogen.app")

async def traced_agent_run(agent, task: str) -> str:
    with tracer.start_as_current_span("agent.run") as span:
        span.set_attribute("agent.name", agent.name)
        span.set_attribute("task.length", len(task))

        result = await agent.run(task=task)
        final = result.messages[-1].content

        span.set_attribute("response.length", len(final))
        span.set_attribute("messages.count", len(result.messages))
        return final
```

## Production Patterns

### Agent Service with FastAPI
```python
from fastapi import FastAPI, BackgroundTasks
from pydantic import BaseModel
import uuid
from autogen_agentchat.agents import AssistantAgent
from autogen_ext.models.openai import OpenAIChatCompletionClient
from autogen_core.tools import FunctionTool

app = FastAPI(title="AutoGen Agent API")
model_client = OpenAIChatCompletionClient(model="gpt-4o-mini")

class TaskRequest(BaseModel):
    task: str
    session_id: str | None = None

class TaskResponse(BaseModel):
    session_id: str
    result: str
    message_count: int

# Session store (use Redis or database in production)
sessions: dict[str, list] = {}

@app.post("/run", response_model=TaskResponse)
async def run_task(request: TaskRequest) -> TaskResponse:
    session_id = request.session_id or str(uuid.uuid4())

    agent = AssistantAgent(
        name="assistant",
        model_client=model_client,
        system_message="You are a helpful assistant.",
    )

    # Restore session if exists
    if session_id in sessions:
        await agent.load_state(sessions[session_id])

    result = await agent.run(task=request.task)

    # Persist session
    sessions[session_id] = await agent.save_state()

    return TaskResponse(
        session_id=session_id,
        result=result.messages[-1].content,
        message_count=len(result.messages),
    )

@app.delete("/session/{session_id}")
async def clear_session(session_id: str) -> dict:
    sessions.pop(session_id, None)
    return {"deleted": session_id}
```

### Rate Limiting and Cost Control
```python
import asyncio
from collections import deque
from datetime import datetime, timedelta

class RateLimitedModelClient:
    """Wraps a model client with rate limiting."""

    def __init__(self, client, requests_per_minute: int = 60) -> None:
        self._client = client
        self._rpm = requests_per_minute
        self._request_times: deque = deque()
        self._lock = asyncio.Lock()

    async def create(self, *args, **kwargs):
        async with self._lock:
            now = datetime.utcnow()
            cutoff = now - timedelta(minutes=1)

            # Remove requests older than 1 minute
            while self._request_times and self._request_times[0] < cutoff:
                self._request_times.popleft()

            # Wait if we've hit the rate limit
            if len(self._request_times) >= self._rpm:
                oldest = self._request_times[0]
                wait_seconds = (oldest + timedelta(minutes=1) - now).total_seconds()
                if wait_seconds > 0:
                    await asyncio.sleep(wait_seconds)

            self._request_times.append(datetime.utcnow())

        return await self._client.create(*args, **kwargs)

    # Proxy all other attributes to the underlying client
    def __getattr__(self, name):
        return getattr(self._client, name)
```

### Circuit Breaker for Agent Reliability
```python
from enum import Enum
from datetime import datetime, timedelta

class CircuitState(Enum):
    CLOSED = "closed"       # normal operation
    OPEN = "open"           # failing, reject calls
    HALF_OPEN = "half_open" # test recovery

class CircuitBreaker:
    def __init__(self, failure_threshold: int = 5, recovery_timeout: int = 60) -> None:
        self._state = CircuitState.CLOSED
        self._failures = 0
        self._threshold = failure_threshold
        self._timeout = timedelta(seconds=recovery_timeout)
        self._opened_at: datetime | None = None

    async def call(self, func, *args, **kwargs):
        if self._state == CircuitState.OPEN:
            if datetime.utcnow() - self._opened_at >= self._timeout:
                self._state = CircuitState.HALF_OPEN
            else:
                raise RuntimeError("Circuit breaker is OPEN — service unavailable")

        try:
            result = await func(*args, **kwargs)
            self._on_success()
            return result
        except Exception as e:
            self._on_failure()
            raise

    def _on_success(self) -> None:
        self._failures = 0
        self._state = CircuitState.CLOSED

    def _on_failure(self) -> None:
        self._failures += 1
        if self._failures >= self._threshold:
            self._state = CircuitState.OPEN
            self._opened_at = datetime.utcnow()
```

## Testing AutoGen Agents

### Unit Testing with Mock Responses
```python
import pytest
import asyncio
from unittest.mock import AsyncMock, patch, MagicMock
from autogen_agentchat.agents import AssistantAgent
from autogen_agentchat.messages import TextMessage
from autogen_core.models import CreateResult, RequestUsage
from autogen_core import CancellationToken

@pytest.mark.asyncio
async def test_agent_responds_to_greeting():
    """Test that the agent produces a non-empty response."""
    mock_client = AsyncMock()
    mock_client.create.return_value = CreateResult(
        content="Hello! I'm here to help.",
        usage=RequestUsage(prompt_tokens=10, completion_tokens=8),
        finish_reason="stop",
        cached=False,
    )
    mock_client.model_info = {"function_calling": True, "vision": False}

    agent = AssistantAgent(name="test_agent", model_client=mock_client)
    result = await agent.run(task="Hello!")

    assert len(result.messages) > 0
    assert "hello" in result.messages[-1].content.lower()
    mock_client.create.assert_called_once()


@pytest.mark.asyncio
async def test_tool_is_called():
    """Test that the agent calls a specific tool when appropriate."""
    tool_mock = AsyncMock(return_value="22°C, Sunny")

    from autogen_core.tools import FunctionTool
    tool = FunctionTool(tool_mock, name="get_weather",
                        description="Get weather for a city.")

    mock_client = AsyncMock()
    # First call — LLM requests tool use
    from autogen_core.models import FunctionCall
    mock_client.create.side_effect = [
        CreateResult(
            content=[FunctionCall(id="1", name="get_weather", arguments='{"city":"Tokyo"}')],
            usage=RequestUsage(prompt_tokens=20, completion_tokens=10),
            finish_reason="tool_calls",
            cached=False,
        ),
        # Second call — LLM produces final response
        CreateResult(
            content="The weather in Tokyo is 22°C and sunny.",
            usage=RequestUsage(prompt_tokens=30, completion_tokens=12),
            finish_reason="stop",
            cached=False,
        ),
    ]
    mock_client.model_info = {"function_calling": True, "vision": False}

    agent = AssistantAgent(
        name="weather_agent",
        model_client=mock_client,
        tools=[tool],
    )
    result = await agent.run(task="What is the weather in Tokyo?")

    tool_mock.assert_called_once_with(city="Tokyo")
    assert "22" in result.messages[-1].content or "sunny" in result.messages[-1].content.lower()
```

### Integration Testing
```python
@pytest.mark.asyncio
@pytest.mark.integration   # skip in CI without API keys
async def test_full_agent_pipeline():
    """Integration test using real API (requires OPENAI_API_KEY)."""
    import os
    if not os.getenv("OPENAI_API_KEY"):
        pytest.skip("OPENAI_API_KEY not set")

    from autogen_ext.models.openai import OpenAIChatCompletionClient
    client = OpenAIChatCompletionClient(model="gpt-4o-mini")
    agent = AssistantAgent("test_agent", model_client=client)

    result = await agent.run(task="What is 2 + 2? Answer with just the number.")

    assert result.messages[-1].content.strip() == "4"
```

## Agent Configuration and Serialization

### YAML/JSON Configuration
```python
import yaml
from autogen_agentchat.agents import AssistantAgent
from autogen_ext.models.openai import OpenAIChatCompletionClient

CONFIG_YAML = """
agents:
  - name: researcher
    system_message: You are a research specialist. Provide detailed, accurate information.
    model: gpt-4o
    temperature: 0.3
    max_tokens: 2048

  - name: writer
    system_message: You are a technical writer. Write clear, concise content.
    model: gpt-4o-mini
    temperature: 0.7
    max_tokens: 1024
"""

def build_agents_from_config(config_yaml: str) -> list[AssistantAgent]:
    config = yaml.safe_load(config_yaml)
    agents = []
    for agent_config in config["agents"]:
        client = OpenAIChatCompletionClient(
            model=agent_config["model"],
            temperature=agent_config.get("temperature", 0.7),
            max_tokens=agent_config.get("max_tokens", 2048),
        )
        agents.append(AssistantAgent(
            name=agent_config["name"],
            model_client=client,
            system_message=agent_config["system_message"],
        ))
    return agents

agents = build_agents_from_config(CONFIG_YAML)
```

## Best Practices (Do's)

✅ **Use custom agents for reusable, specialized capabilities**
```python
class DomainExpertAgent(BaseChatAgent):
    """Encapsulates domain-specific logic, prompts, and tools."""
    ...
```

✅ **Test agents in isolation before combining into teams**
```python
@pytest.mark.asyncio
async def test_researcher_agent():
    result = await researcher.run(task="Explain transformers")
    assert len(result.messages[-1].content) > 50
```

✅ **Use structured config for team composition**
```python
config = load_config("agents.yaml")
team = build_team(config)   # no hardcoded agent definitions in production code
```

✅ **Implement proper async resource cleanup**
```python
async def main():
    async with DockerCommandLineCodeExecutor(image="python:3.12-slim") as executor:
        agent = CodeExecutorAgent("runner", code_executor=executor)
        await agent.run(task="Print hello world")
    # Docker container is cleaned up here
```

✅ **Use `GraphFlow` for deterministic multi-step pipelines**
```python
# Predictable, testable pipeline
team = GraphFlow(participants=[a, b, c], graph=directed_graph)
```

## Common Mistakes (Don'ts)

❌ **Don't block the event loop with synchronous I/O in agents**
```python
# Bad — blocks asyncio event loop
class BadAgent(BaseChatAgent):
    async def on_messages_stream(self, messages, token):
        data = open("big_file.txt").read()   # blocking file I/O
        ...

# Good — use async I/O
class GoodAgent(BaseChatAgent):
    async def on_messages_stream(self, messages, token):
        async with aiofiles.open("big_file.txt") as f:
            data = await f.read()
        ...
```

❌ **Don't ignore `CancellationToken` in custom agents**
```python
# Bad — can't be cancelled externally
async def on_messages_stream(self, messages, token):
    for item in huge_list:
        await process(item)  # can't interrupt!

# Good — check cancellation
async def on_messages_stream(self, messages, token):
    for item in huge_list:
        if token.is_cancelled():
            raise asyncio.CancelledError()
        await process(item)
```

❌ **Don't put business logic in system messages alone**
```python
# Fragile — LLM might not follow instructions exactly
system_msg = "Always respond with valid JSON. Never use markdown."

# Better — enforce structure at the API level
structured_llm = model_client.with_structured_output(MyModel)
```

❌ **Don't deploy without token/cost budgets**
```python
# No cost control
team = RoundRobinGroupChat(agents)

# Good — multiple safety nets
termination = (
    MaxMessageTermination(50)
    | TokenUsageTermination(max_total_token=100_000)
    | TimeoutTermination(timeout_seconds=300)
)
team = RoundRobinGroupChat(agents, termination_condition=termination)
```
