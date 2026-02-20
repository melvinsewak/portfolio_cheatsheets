# LangChain Advanced Cheatsheet

## LangGraph — Stateful Multi-Agent Workflows

### Basic Graph
```python
# pip install langgraph
from langgraph.graph import StateGraph, END
from typing import TypedDict, Annotated
import operator

# Define graph state
class AgentState(TypedDict):
    messages: Annotated[list, operator.add]   # messages accumulate
    next_action: str

# Define nodes (functions)
def call_llm(state: AgentState) -> AgentState:
    from langchain_openai import ChatOpenAI
    llm = ChatOpenAI(model="gpt-4o-mini")
    response = llm.invoke(state["messages"])
    return {"messages": [response]}

def route(state: AgentState) -> str:
    last_msg = state["messages"][-1]
    if hasattr(last_msg, "tool_calls") and last_msg.tool_calls:
        return "use_tools"
    return END

# Build graph
builder = StateGraph(AgentState)
builder.add_node("agent", call_llm)
builder.set_entry_point("agent")
builder.add_conditional_edges("agent", route, {"use_tools": "tools", END: END})

graph = builder.compile()

# Invoke
result = graph.invoke({"messages": [("user", "What is 2 + 2?")]})
print(result["messages"][-1].content)
```

### ReAct Agent with LangGraph
```python
from langgraph.prebuilt import create_react_agent
from langchain_openai import ChatOpenAI
from langchain.tools import tool

@tool
def calculator(expression: str) -> str:
    """Evaluate a mathematical expression safely."""
    try:
        allowed = set("0123456789+-*/()., ")
        if any(c not in allowed for c in expression):
            return "Invalid expression"
        return str(eval(expression))
    except Exception as e:
        return f"Error: {e}"

@tool
def get_current_time() -> str:
    """Return the current UTC time as an ISO-8601 string."""
    from datetime import datetime, timezone
    return datetime.now(timezone.utc).isoformat()

llm = ChatOpenAI(model="gpt-4o")
agent = create_react_agent(llm, [calculator, get_current_time])

result = agent.invoke({
    "messages": [("user", "What is (127 * 43) + 999? Also what time is it?")]
})
print(result["messages"][-1].content)
```

### Graph with Human-in-the-Loop
```python
from langgraph.checkpoint.memory import MemorySaver
from langgraph.graph import StateGraph, END, START
from langgraph.prebuilt import create_react_agent
from langchain_core.messages import HumanMessage

# Persist state with checkpointing
memory = MemorySaver()
agent = create_react_agent(llm, tools, checkpointer=memory)

config = {"configurable": {"thread_id": "session-001"}}

# First turn
result = agent.invoke(
    {"messages": [HumanMessage("Search for Python 3.13 news")]},
    config=config,
)

# Second turn — memory is preserved across invocations
result = agent.invoke(
    {"messages": [HumanMessage("Summarize what you just found")]},
    config=config,
)
print(result["messages"][-1].content)
```

### Multi-Agent Supervisor
```python
from langgraph.graph import StateGraph, END
from typing import TypedDict, Literal
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from pydantic import BaseModel

llm = ChatOpenAI(model="gpt-4o")

WORKERS = ["researcher", "coder", "writer"]

class SupervisorState(TypedDict):
    messages: list
    next: str

def make_supervisor_chain():
    options = WORKERS + ["FINISH"]
    system_prompt = (
        "You are a supervisor managing: {workers}. "
        "Given the conversation, decide who should act next. "
        "Respond with one of: {options}"
    )
    prompt = ChatPromptTemplate.from_messages([
        ("system", system_prompt),
        ("placeholder", "{messages}"),
    ]).partial(workers=", ".join(WORKERS), options=", ".join(options))

    class NextAction(BaseModel):
        next: Literal[*options]

    return prompt | llm.with_structured_output(NextAction)

supervisor = make_supervisor_chain()

def researcher_node(state: SupervisorState):
    # researcher agent logic
    ...

def coder_node(state: SupervisorState):
    # coder agent logic
    ...

builder = StateGraph(SupervisorState)
builder.add_node("supervisor", lambda s: {"next": supervisor.invoke(s).next})
builder.add_node("researcher", researcher_node)
builder.add_node("coder", coder_node)

builder.set_entry_point("supervisor")
builder.add_conditional_edges(
    "supervisor",
    lambda s: s["next"],
    {w: w for w in WORKERS} | {"FINISH": END},
)
for worker in WORKERS:
    builder.add_edge(worker, "supervisor")

graph = builder.compile()
```

## Custom Runnables

### Custom LCEL Runnable
```python
from langchain_core.runnables import Runnable, RunnableConfig, RunnableLambda
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import JsonOutputParser
from typing import Any, Iterator, AsyncIterator

class SentimentScorer(Runnable):
    """Custom runnable that scores sentiment using an LLM."""

    def __init__(self, llm) -> None:
        self.llm = llm
        self._prompt = ChatPromptTemplate.from_messages([
            ("system", "Rate the sentiment of the text. "
                        "Respond ONLY with a JSON object: "
                        '{"score": <float -1.0 to 1.0>, "label": "positive|negative|neutral"}'),
            ("user", "{text}"),
        ])
        self._chain = self._prompt | self.llm | JsonOutputParser()

    def invoke(self, input: str, config: RunnableConfig | None = None) -> dict:
        return self._chain.invoke({"text": input}, config=config)

    async def ainvoke(self, input: str, config: RunnableConfig | None = None) -> dict:
        return await self._chain.ainvoke({"text": input}, config=config)

    def stream(self, input: str, config: RunnableConfig | None = None) -> Iterator:
        yield from self._chain.stream({"text": input}, config=config)


scorer = SentimentScorer(llm)
result = scorer.invoke("This product is absolutely terrible!")
print(result)   # {'score': -0.9, 'label': 'negative'}

# Can be used in LCEL chains
pipeline = RunnableLambda(str.strip) | scorer
```

### Runnable with Fallbacks
```python
from langchain_openai import ChatOpenAI
from langchain_anthropic import ChatAnthropic
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

primary_llm = ChatOpenAI(model="gpt-4o")
fallback_llm = ChatAnthropic(model="claude-3-5-haiku-20241022")

# Automatically falls back if primary fails
robust_llm = primary_llm.with_fallbacks([fallback_llm])

prompt = ChatPromptTemplate.from_template("Answer the question: {question}")
chain = prompt | robust_llm | StrOutputParser()
result = chain.invoke({"question": "What is AI?"})   # uses fallback if OpenAI is down
```

### Runnable with Retry
```python
from langchain_core.runnables import RunnableRetry
from langchain_core.prompts import ChatPromptTemplate
from langchain_openai import ChatOpenAI
from langchain_core.output_parsers import StrOutputParser

# Define a simple runnable chain
prompt = ChatPromptTemplate.from_template("Answer the question: {question}")
llm = ChatOpenAI(model="gpt-4o-mini")
chain = prompt | llm | StrOutputParser()

# Wrap the chain with retry logic
retry_chain = chain.with_retry(
    retry_if_exception_type=(ValueError, TimeoutError),
    wait_exponential_jitter=True,
    stop_after_attempt=3,
)
```

## Callbacks and Observability

### Custom Callback Handler
```python
from langchain_core.callbacks import BaseCallbackHandler
from langchain_core.outputs import LLMResult
from typing import Any
import time

class TimingCallback(BaseCallbackHandler):
    def __init__(self) -> None:
        self.start_time: float = 0
        self.token_count: int = 0

    def on_llm_start(self, serialized: dict, prompts: list[str], **kwargs) -> None:
        self.start_time = time.perf_counter()
        print(f"🚀 LLM call started: {serialized.get('name', 'unknown')}")

    def on_llm_end(self, response: LLMResult, **kwargs) -> None:
        elapsed = time.perf_counter() - self.start_time
        tokens = response.llm_output.get("token_usage", {})
        print(f"✅ LLM call finished in {elapsed:.2f}s | "
              f"Tokens: {tokens.get('total_tokens', 'N/A')}")

    def on_llm_error(self, error: Exception, **kwargs) -> None:
        print(f"❌ LLM error: {error}")

    def on_chain_start(self, serialized: dict, inputs: dict, **kwargs) -> None:
        print(f"⛓  Chain started: {serialized.get('name', 'chain')}")


timing_cb = TimingCallback()
result = chain.invoke({"question": "Tell me a joke"}, config={"callbacks": [timing_cb]})
```

### LangSmith Tracing
```python
import os

# Enable LangSmith tracing
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_PROJECT"] = "my-rag-app"
os.environ["LANGCHAIN_API_KEY"] = "ls__..."

# All LangChain calls are now automatically traced in LangSmith
# Visit https://smith.langchain.com to view traces

# Add run metadata
result = chain.invoke(
    {"question": "What is RAG?"},
    config={
        "run_name": "rag-query",
        "tags": ["production", "v1.2"],
        "metadata": {"user_id": "user-123", "session": "sess-abc"},
    },
)
```

### Evaluate with LangSmith
```python
from langsmith import Client
from langsmith.evaluation import evaluate

client = Client()

# Create a dataset
dataset = client.create_dataset("rag-eval-set")
client.create_examples(
    inputs=[{"question": "What is LangChain?"}, {"question": "What is RAG?"}],
    outputs=[{"answer": "LangChain is an AI framework."}, {"answer": "RAG combines retrieval with generation."}],
    dataset_id=dataset.id,
)

# Define evaluator
def correctness_evaluator(run, example):
    score = 1 if example.outputs["answer"].lower() in run.outputs["output"].lower() else 0
    return {"key": "correctness", "score": score}

# Run evaluation
results = evaluate(
    lambda inputs: {"output": rag_chain.invoke(inputs["question"])},
    data=dataset.name,
    evaluators=[correctness_evaluator],
    experiment_prefix="rag-v1",
)
```

## Advanced RAG Patterns

### Self-RAG (Retrieve, Grade, Generate)
```python
from langchain_core.prompts import ChatPromptTemplate
from pydantic import BaseModel, Field

class GradeDocuments(BaseModel):
    """Grade whether a document is relevant to a question."""
    binary_score: str = Field(description="'yes' or 'no'")

grader_llm = llm.with_structured_output(GradeDocuments)

grade_prompt = ChatPromptTemplate.from_messages([
    ("system",
     "You are a grader assessing relevance of a retrieved document to a user question. "
     "If the document contains keywords related to the question, grade it as relevant. "
     "Give a binary score 'yes' or 'no'."),
    ("user", "Retrieved document:\n{document}\n\nUser question: {question}"),
])

grader_chain = grade_prompt | grader_llm

def self_rag_chain(question: str) -> str:
    docs = retriever.invoke(question)

    # Grade each document for relevance
    relevant_docs = []
    for doc in docs:
        grade = grader_chain.invoke({
            "document": doc.page_content,
            "question": question,
        })
        if grade.binary_score == "yes":
            relevant_docs.append(doc)

    if not relevant_docs:
        return "I couldn't find relevant information to answer your question."

    # Generate answer from relevant docs only
    context = "\n\n".join(d.page_content for d in relevant_docs)
    return rag_chain.invoke({"question": question, "context": context})
```

### Hypothetical Document Embeddings (HyDE)
```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

hyde_prompt = ChatPromptTemplate.from_messages([
    ("system",
     "Generate a hypothetical document that would contain the answer to this question. "
     "Write it as if it were an excerpt from a real document."),
    ("user", "{question}"),
])

hyde_generator = hyde_prompt | llm | StrOutputParser()

hyde_chain = (
    RunnablePassthrough.assign(
        hypothetical_doc=hyde_generator
    )
    | (lambda x: retriever.invoke(x["hypothetical_doc"]))
)
```

## Structured Output and Tool Calling

### with_structured_output
```python
from pydantic import BaseModel, Field

class ResearchReport(BaseModel):
    topic: str = Field(description="The research topic")
    summary: str = Field(description="Executive summary (3-5 sentences)")
    key_findings: list[str] = Field(description="List of 5 key findings")
    sources: list[str] = Field(description="List of source URLs or references")
    confidence: float = Field(description="Confidence score 0-1", ge=0, le=1)

# Automatically infers tool calling or JSON mode based on model support
structured_llm = llm.with_structured_output(ResearchReport, include_raw=False)

report = structured_llm.invoke(
    "Research the impact of large language models on software development"
)
print(f"Topic: {report.topic}")
print(f"Confidence: {report.confidence:.0%}")
for finding in report.key_findings:
    print(f"  - {finding}")
```

### Parallel Tool Calling
```python
from langchain_core.messages import HumanMessage

# Models like GPT-4o can call multiple tools in parallel
llm_with_tools = llm.bind_tools([calculator, get_current_time, search_tool])

response = llm_with_tools.invoke(
    "What is (17 * 83) + 224? And what time is it? Also search for AI news."
)

# Response will have multiple tool_calls
for tool_call in response.tool_calls:
    print(f"Tool: {tool_call['name']}, Args: {tool_call['args']}")
```

## Production Patterns

### Rate Limiting and Retry
```python
from langchain_core.rate_limiters import InMemoryRateLimiter
from langchain_openai import ChatOpenAI

# Rate limit to avoid hitting API limits
rate_limiter = InMemoryRateLimiter(
    requests_per_second=2,    # max 2 requests/second
    check_every_n_seconds=0.1,
    max_bucket_size=10,
)

llm = ChatOpenAI(
    model="gpt-4o-mini",
    rate_limiter=rate_limiter,
)

# Exponential backoff retry
from tenacity import retry, stop_after_attempt, wait_exponential

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=4, max=60),
)
def robust_invoke(chain, inputs):
    return chain.invoke(inputs)
```

### Async Batch Processing
```python
import asyncio
from langchain_openai import ChatOpenAI

async def process_batch(questions: list[str]) -> list[str]:
    llm = ChatOpenAI(model="gpt-4o-mini")
    chain = prompt | llm | StrOutputParser()

    # Process all questions concurrently
    results = await chain.abatch(
        [{"question": q} for q in questions],
        config={"max_concurrency": 5},   # limit concurrent requests
    )
    return results

questions = [f"What is {topic}?" for topic in ["AI", "ML", "DL", "NLP", "CV"]]
answers = asyncio.run(process_batch(questions))
```

### Caching with Redis
```python
from langchain_community.cache import RedisSemanticCache
from langchain_openai import OpenAIEmbeddings
from langchain_core.globals import set_llm_cache

# Semantic cache — serves cached responses for similar (not just identical) queries
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
set_llm_cache(RedisSemanticCache(
    redis_url="redis://localhost:6379",
    embedding=embeddings,
    score_threshold=0.95,   # cosine similarity threshold
))

# Subsequent similar queries hit cache instead of calling the API
result1 = llm.invoke("What is machine learning?")
result2 = llm.invoke("Define machine learning")   # likely served from cache
```

## Best Practices (Do's)

✅ **Use LangGraph for complex multi-step agentic workflows**
```python
# Good — explicit state machine with LangGraph
agent = create_react_agent(llm, tools, checkpointer=MemorySaver())

# Less suitable for complex workflows
agent_executor = AgentExecutor(agent=agent, tools=tools)
```

✅ **Enable LangSmith tracing in development**
```python
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_PROJECT"] = "my-project"
```

✅ **Use structured output for reliable JSON extraction**
```python
# Reliable — uses native tool calling
structured_llm = llm.with_structured_output(MyModel)

# Fragile — relies on prompt engineering + JSON parsing
result = llm.invoke("Respond in JSON: ...")
```

✅ **Add fallbacks for production resilience**
```python
primary = ChatOpenAI(model="gpt-4o")
fallback = ChatAnthropic(model="claude-3-5-haiku-20241022")
resilient_llm = primary.with_fallbacks([fallback])
```

✅ **Use `max_concurrency` in batch calls to avoid rate limits**
```python
results = await chain.abatch(inputs, config={"max_concurrency": 3})
```

## Common Mistakes (Don'ts)

❌ **Don't build complex agents without state management**
```python
# Bad — loses context between agent calls
agent_executor.invoke({"input": "Follow up on the previous task"})

# Good — use LangGraph with checkpointing
agent = create_react_agent(llm, tools, checkpointer=MemorySaver())
agent.invoke({"messages": [...]}, config={"configurable": {"thread_id": "t1"}})
```

❌ **Don't skip evaluation before production deployment**
```python
# Good — systematic evaluation
results = evaluate(
    lambda x: {"output": chain.invoke(x)},
    data="eval-dataset",
    evaluators=[correctness_evaluator, faithfulness_evaluator],
)
```

❌ **Don't call slow tools in a single-threaded agent loop without timeouts**
```python
# Bad — tool can hang indefinitely
@tool
def slow_api_call(query: str) -> str:
    return requests.get(url).text

# Good — add timeout
@tool
def safe_api_call(query: str) -> str:
    try:
        return requests.get(url, timeout=10).text
    except requests.Timeout:
        return "Request timed out"
```

❌ **Don't ignore context window limits in long RAG chains**
```python
# Bad — unlimited retrieved docs can overflow context
retriever = vectorstore.as_retriever(search_kwargs={"k": 100})

# Good — calculate how many docs fit within your model's context
# gpt-4o: 128k tokens; typical doc chunk: ~300 tokens → k=20 max
retriever = vectorstore.as_retriever(search_kwargs={"k": 6})
```
