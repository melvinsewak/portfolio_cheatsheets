# LangChain Intermediate Cheatsheet

## Retrieval-Augmented Generation (RAG)

### Complete RAG Pipeline
```python
from langchain_openai import ChatOpenAI, OpenAIEmbeddings
from langchain_community.document_loaders import WebBaseLoader
from langchain_text_splitters import RecursiveCharacterTextSplitter
from langchain_chroma import Chroma
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough

# 1. Load documents
loader = WebBaseLoader("https://python.langchain.com/docs/introduction/")
docs = loader.load()

# 2. Split into chunks
splitter = RecursiveCharacterTextSplitter(chunk_size=1000, chunk_overlap=200)
chunks = splitter.split_documents(docs)

# 3. Embed and store in vector store
embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
vectorstore = Chroma.from_documents(chunks, embeddings)

# 4. Create retriever
retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 4},   # retrieve top-4 most similar chunks
)

# 5. Build RAG chain
def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)

rag_prompt = ChatPromptTemplate.from_messages([
    ("system",
     "You are an assistant for question-answering tasks. "
     "Use the following retrieved context to answer the question. "
     "If you don't know, say you don't know.\n\nContext:\n{context}"),
    ("user", "{question}"),
])

llm = ChatOpenAI(model="gpt-4o-mini")

rag_chain = (
    {"context": retriever | format_docs, "question": RunnablePassthrough()}
    | rag_prompt
    | llm
    | StrOutputParser()
)

# 6. Query
answer = rag_chain.invoke("What is LangChain used for?")
print(answer)
```

### Vector Stores
```python
# Chroma (in-memory or persisted)
from langchain_chroma import Chroma

# Create from documents
vectorstore = Chroma.from_documents(chunks, embeddings)

# Persist to disk
vectorstore = Chroma.from_documents(
    chunks, embeddings,
    persist_directory="./chroma_db"
)

# Load existing persisted store
vectorstore = Chroma(
    persist_directory="./chroma_db",
    embedding_function=embeddings,
)

# FAISS (in-memory, fast)
from langchain_community.vectorstores import FAISS

vectorstore = FAISS.from_documents(chunks, embeddings)
vectorstore.save_local("faiss_index")
vectorstore = FAISS.load_local("faiss_index", embeddings,
                                allow_dangerous_deserialization=True)

# Similarity search
results = vectorstore.similarity_search("What is RAG?", k=3)
results_with_score = vectorstore.similarity_search_with_score("RAG", k=3)
for doc, score in results_with_score:
    print(f"Score: {score:.3f} | {doc.page_content[:100]}")
```

### Advanced Retriever Types
```python
from langchain.retrievers import (
    MultiQueryRetriever,
    ContextualCompressionRetriever,
)
from langchain.retrievers.document_compressors import LLMChainExtractor

base_retriever = vectorstore.as_retriever(search_kwargs={"k": 6})

# MultiQuery: generates multiple query variations to improve recall
multi_retriever = MultiQueryRetriever.from_llm(
    retriever=base_retriever,
    llm=llm,
)

# Contextual compression: extract only relevant parts of retrieved docs
compressor = LLMChainExtractor.from_llm(llm)
compression_retriever = ContextualCompressionRetriever(
    base_compressor=compressor,
    base_retriever=base_retriever,
)

# Ensemble retriever (combines keyword + semantic search)
from langchain.retrievers import EnsembleRetriever
from langchain_community.retrievers import BM25Retriever

bm25_retriever = BM25Retriever.from_documents(chunks)
bm25_retriever.k = 4

ensemble = EnsembleRetriever(
    retrievers=[bm25_retriever, base_retriever],
    weights=[0.4, 0.6],
)
```

## Agents and Tools

### Built-in Tools
```python
from langchain_community.tools import DuckDuckGoSearchRun, WikipediaQueryRun
from langchain_community.utilities import WikipediaAPIWrapper
from langchain.tools import tool

# DuckDuckGo web search
search_tool = DuckDuckGoSearchRun()
print(search_tool.invoke("latest Python 3.13 features"))

# Wikipedia
wiki_api = WikipediaAPIWrapper(top_k_results=2, doc_content_chars_max=2000)
wiki_tool = WikipediaQueryRun(api_wrapper=wiki_api)

# Custom tool using @tool decorator
@tool
def get_word_count(text: str) -> int:
    """Count the number of words in the given text."""
    return len(text.split())

@tool
def celsius_to_fahrenheit(celsius: float) -> float:
    """Convert a temperature from Celsius to Fahrenheit."""
    return celsius * 9 / 5 + 32
```

### Creating an Agent with Tools
```python
from langchain.agents import create_tool_calling_agent, AgentExecutor
from langchain_core.prompts import ChatPromptTemplate

tools = [search_tool, wiki_tool, get_word_count, celsius_to_fahrenheit]

agent_prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant. Use tools when needed."),
    ("placeholder", "{chat_history}"),
    ("user", "{input}"),
    ("placeholder", "{agent_scratchpad}"),
])

agent = create_tool_calling_agent(llm, tools, agent_prompt)

agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,       # print reasoning steps
    max_iterations=5,   # safety limit
    return_intermediate_steps=True,
)

result = agent_executor.invoke({
    "input": "What is 100°C in Fahrenheit and who invented the Celsius scale?",
    "chat_history": [],
})

print(result["output"])
print(result["intermediate_steps"])
```

### Structured Tool Input
```python
from langchain.tools import StructuredTool
from pydantic import BaseModel, Field

class SearchInput(BaseModel):
    query: str = Field(description="The search query")
    max_results: int = Field(default=3, description="Max number of results")

def advanced_search(query: str, max_results: int = 3) -> list[str]:
    """Search the web and return a list of results."""
    tool = DuckDuckGoSearchRun()
    result = tool.invoke(query)
    return result.split("\n")[:max_results]

structured_tool = StructuredTool.from_function(
    func=advanced_search,
    name="advanced_search",
    description="Search the web for information",
    args_schema=SearchInput,
)
```

## Memory and Conversation History

### In-Memory Chat History
```python
from langchain_core.chat_history import InMemoryChatMessageHistory
from langchain_core.runnables.history import RunnableWithMessageHistory

# Simple in-memory storage (for demos/testing)
store: dict[str, InMemoryChatMessageHistory] = {}

def get_session_history(session_id: str) -> InMemoryChatMessageHistory:
    if session_id not in store:
        store[session_id] = InMemoryChatMessageHistory()
    return store[session_id]

prompt_with_history = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    ("placeholder", "{chat_history}"),
    ("user", "{input}"),
])

chain = prompt_with_history | llm | StrOutputParser()

chain_with_history = RunnableWithMessageHistory(
    chain,
    get_session_history,
    input_messages_key="input",
    history_messages_key="chat_history",
)

# Invoke with a session ID (history is persisted per session)
config = {"configurable": {"session_id": "user-123"}}

resp1 = chain_with_history.invoke({"input": "My name is Alice."}, config=config)
resp2 = chain_with_history.invoke({"input": "What is my name?"}, config=config)
print(resp2)   # "Your name is Alice."
```

### Trimming Message History
```python
from langchain_core.messages import trim_messages

trimmer = trim_messages(
    max_tokens=2000,
    strategy="last",        # keep most recent messages
    token_counter=llm,      # use model to count tokens
    include_system=True,    # always keep system message
    allow_partial=False,
    start_on="human",       # ensure history starts with human message
)

chain = prompt_with_history | trimmer | llm | StrOutputParser()
```

## Streaming

### Synchronous Streaming
```python
# Stream tokens one by one
for chunk in chain.stream({"question": "Explain neural networks step by step."}):
    print(chunk, end="", flush=True)
print()

# Stream with intermediate steps (agent)
for event in agent_executor.stream({"input": "Search for Python 3.13 news"}):
    if "actions" in event:
        for action in event["actions"]:
            print(f"Tool: {action.tool}({action.tool_input})")
    elif "output" in event:
        print(f"Answer: {event['output']}")
```

### Async Streaming
```python
import asyncio

async def stream_response(question: str) -> None:
    async for chunk in chain.astream({"question": question}):
        print(chunk, end="", flush=True)
    print()

async def stream_events(question: str) -> None:
    async for event in chain.astream_events({"question": question}, version="v2"):
        if event["event"] == "on_chat_model_stream":
            content = event["data"]["chunk"].content
            if content:
                print(content, end="", flush=True)

asyncio.run(stream_response("Tell me about quantum computing"))
```

## Document Loaders

```python
from langchain_community.document_loaders import (
    PyPDFLoader,
    CSVLoader,
    JSONLoader,
    UnstructuredMarkdownLoader,
    GitLoader,
)

# PDF
pdf_loader = PyPDFLoader("document.pdf")
pdf_docs = pdf_loader.load_and_split()   # auto-split by page

# CSV
csv_loader = CSVLoader("data.csv", csv_args={"delimiter": ","})
csv_docs = csv_loader.load()

# JSON with jq selector
json_loader = JSONLoader(
    file_path="data.json",
    jq_schema=".[] | {content: .body, source: .url}",
    content_key="content",
)

# Markdown
md_loader = UnstructuredMarkdownLoader("README.md")

# Git repository
git_loader = GitLoader(
    clone_url="https://github.com/langchain-ai/langchain",
    repo_path="./langchain_repo",
    branch="master",
    file_filter=lambda x: x.endswith(".py"),
)
```

## Output Parsers (Advanced)

```python
from langchain_core.output_parsers import PydanticOutputParser
from pydantic import BaseModel, Field
from typing import Optional

class MovieReview(BaseModel):
    title: str = Field(description="Movie title")
    rating: float = Field(description="Rating out of 10", ge=0, le=10)
    pros: list[str] = Field(description="List of positive aspects")
    cons: list[str] = Field(description="List of negative aspects")
    summary: str = Field(description="Brief summary")
    recommended: bool = Field(description="Whether to recommend")

parser = PydanticOutputParser(pydantic_object=MovieReview)

prompt = ChatPromptTemplate.from_messages([
    ("system",
     "You are a movie critic. Respond in the following format:\n"
     "{format_instructions}"),
    ("user", "Review the movie: {movie_title}"),
]).partial(format_instructions=parser.get_format_instructions())

chain = prompt | llm | parser
review = chain.invoke({"movie_title": "Inception"})
print(f"{review.title}: {review.rating}/10")
print(f"Recommended: {review.recommended}")

# With retry on parse failure
from langchain.output_parsers import OutputFixingParser

robust_parser = OutputFixingParser.from_llm(parser=parser, llm=llm)
chain = prompt | llm | robust_parser
```

## Caching LLM Calls
```python
from langchain_core.globals import set_llm_cache
from langchain_community.cache import SQLiteCache, InMemoryCache

# In-memory cache (resets on restart)
set_llm_cache(InMemoryCache())

# SQLite cache (persists between runs)
set_llm_cache(SQLiteCache(database_path=".langchain.db"))

# Identical calls are served from cache instantly — saves tokens & money
result1 = llm.invoke("What is 2 + 2?")
result2 = llm.invoke("What is 2 + 2?")   # from cache
```

## Best Practices (Do's)

✅ **Use `with_structured_output` for reliable structured responses**
```python
from pydantic import BaseModel

class Analysis(BaseModel):
    sentiment: str
    confidence: float
    keywords: list[str]

structured_llm = llm.with_structured_output(Analysis)
result = structured_llm.invoke("This product is amazing!")
print(result.sentiment, result.confidence)
```

✅ **Persist vector stores to avoid re-embedding**
```python
# First run — embed and save
vectorstore = Chroma.from_documents(chunks, embeddings,
                                     persist_directory="./db")

# Subsequent runs — load from disk
vectorstore = Chroma(persist_directory="./db",
                     embedding_function=embeddings)
```

✅ **Use `RunnableWithMessageHistory` for stateful conversations**
```python
chain_with_history = RunnableWithMessageHistory(
    chain,
    get_session_history,
    input_messages_key="input",
    history_messages_key="chat_history",
)
```

✅ **Add metadata to documents for better filtering**
```python
from langchain_core.documents import Document

doc = Document(
    page_content="Python is a programming language.",
    metadata={"source": "docs/intro.md", "topic": "python", "version": 3.13},
)
# Filter by metadata during retrieval
results = vectorstore.similarity_search(
    "Python features",
    k=5,
    filter={"topic": "python"},
)
```

## Common Mistakes (Don'ts)

❌ **Don't skip document chunking**
```python
# Bad — entire document as one chunk exceeds context window
vectorstore = Chroma.from_documents(docs, embeddings)

# Good — split first
chunks = splitter.split_documents(docs)
vectorstore = Chroma.from_documents(chunks, embeddings)
```

❌ **Don't use agents for simple, predictable tasks**
```python
# Overkill — agents have overhead and unpredictability
agent_executor.invoke({"input": "Translate 'hello' to French"})

# Better for deterministic tasks
chain = prompt | llm | StrOutputParser()
chain.invoke({"text": "hello", "lang": "French"})
```

❌ **Don't ignore retriever `k` — too few or too many hurts quality**
```python
# Too few — may miss relevant context
retriever = vectorstore.as_retriever(search_kwargs={"k": 1})

# Too many — dilutes relevant context and wastes tokens
retriever = vectorstore.as_retriever(search_kwargs={"k": 50})

# Good — typically 3–6 for most use cases
retriever = vectorstore.as_retriever(search_kwargs={"k": 4})
```

❌ **Don't forget to handle agent failures gracefully**
```python
# Bad
result = agent_executor.invoke({"input": user_query})

# Good
try:
    result = agent_executor.invoke({"input": user_query})
except Exception as e:
    result = {"output": f"Sorry, I encountered an error: {e}"}
```
