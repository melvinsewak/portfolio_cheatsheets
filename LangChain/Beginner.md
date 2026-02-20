# LangChain Beginner Cheatsheet

## Installation

```bash
pip install langchain langchain-openai langchain-community python-dotenv
```

```python
# Load API keys from .env file
from dotenv import load_dotenv
load_dotenv()
# OPENAI_API_KEY=sk-... in your .env file
```

## LLM Models

### Chat Models (Recommended)
```python
from langchain_openai import ChatOpenAI

# Basic setup
llm = ChatOpenAI(
    model="gpt-4o-mini",
    temperature=0.7,    # 0 = deterministic, 1+ = creative
    max_tokens=1024,
)

# Invoke (synchronous)
response = llm.invoke("Explain machine learning in one sentence.")
print(response.content)   # "Machine learning is..."
print(response.usage_metadata)   # token counts
```

### Using Different Providers
```python
from langchain_openai import ChatOpenAI
from langchain_anthropic import ChatAnthropic
from langchain_ollama import ChatOllama

# OpenAI
gpt = ChatOpenAI(model="gpt-4o-mini")

# Anthropic (Claude)
claude = ChatAnthropic(model="claude-3-5-haiku-20241022")

# Local Ollama (free, no API key needed)
ollama = ChatOllama(model="llama3.2")

# They all share the same interface
for model in [gpt, claude, ollama]:
    print(model.invoke("Say hello!").content)
```

## Messages

```python
from langchain_core.messages import (
    HumanMessage,
    AIMessage,
    SystemMessage,
    ToolMessage,
)

# System message sets the assistant's behavior
# Human messages are user inputs
# AI messages are assistant responses
messages = [
    SystemMessage(content="You are a helpful Python coding assistant."),
    HumanMessage(content="How do I read a file in Python?"),
]

response = llm.invoke(messages)
print(response.content)

# Multi-turn conversation
history = [
    SystemMessage(content="You are a helpful assistant."),
    HumanMessage(content="My name is Alice."),
    AIMessage(content="Nice to meet you, Alice!"),
    HumanMessage(content="What is my name?"),
]
answer = llm.invoke(history)
print(answer.content)   # "Your name is Alice."
```

## Prompt Templates

### Basic Templates
```python
from langchain_core.prompts import ChatPromptTemplate, PromptTemplate

# Chat prompt template
chat_prompt = ChatPromptTemplate.from_messages([
    ("system", "You are an expert in {domain}."),
    ("user", "Explain {concept} in simple terms."),
])

# Format to inspect
formatted = chat_prompt.format_messages(
    domain="data science",
    concept="gradient descent",
)
print(formatted)

# String prompt template (for older completion models)
string_prompt = PromptTemplate.from_template(
    "Write a {tone} summary of: {text}"
)
```

### FewShotChatMessagePromptTemplate
```python
from langchain_core.prompts import FewShotChatMessagePromptTemplate

examples = [
    {"input": "happy",    "output": "sad"},
    {"input": "tall",     "output": "short"},
    {"input": "energetic","output": "lethargic"},
]

example_prompt = ChatPromptTemplate.from_messages([
    ("human", "{input}"),
    ("ai",    "{output}"),
])

few_shot_prompt = FewShotChatMessagePromptTemplate(
    examples=examples,
    example_prompt=example_prompt,
)

final_prompt = ChatPromptTemplate.from_messages([
    ("system", "You are an antonym generator."),
    few_shot_prompt,
    ("human", "{word}"),
])

chain = final_prompt | llm
print(chain.invoke({"word": "bright"}).content)   # "dark"
```

## Output Parsers

```python
from langchain_core.output_parsers import (
    StrOutputParser,
    JsonOutputParser,
    CommaSeparatedListOutputParser,
)

# String parser — extract .content automatically
str_parser = StrOutputParser()
chain = llm | str_parser
result = chain.invoke("List 3 fruits")
print(type(result))   # <class 'str'>

# Comma-separated list parser
list_parser = CommaSeparatedListOutputParser()
list_chain = (
    ChatPromptTemplate.from_template("List 5 {topic}, comma-separated.")
    | llm
    | list_parser
)
fruits = list_chain.invoke({"topic": "fruits"})
print(fruits)  # ['apple', 'banana', 'cherry', 'date', 'elderberry']

# JSON parser with Pydantic model
from langchain_core.output_parsers import PydanticOutputParser
from pydantic import BaseModel, Field

class Joke(BaseModel):
    setup: str = Field(description="The setup of the joke")
    punchline: str = Field(description="The punchline of the joke")

parser = PydanticOutputParser(pydantic_object=Joke)

prompt = ChatPromptTemplate.from_messages([
    ("system", "Answer the user query.\n{format_instructions}"),
    ("user", "Tell me a joke about {topic}"),
]).partial(format_instructions=parser.get_format_instructions())

joke_chain = prompt | llm | parser
joke = joke_chain.invoke({"topic": "programmers"})
print(joke.setup)
print(joke.punchline)
```

## LCEL (LangChain Expression Language) Basics

```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# Compose using the | operator
prompt = ChatPromptTemplate.from_template("Tell me a fact about {animal}.")
parser = StrOutputParser()

chain = prompt | llm | parser

# Invoke
result = chain.invoke({"animal": "penguins"})
print(result)

# Batch — run multiple inputs in parallel
results = chain.batch([
    {"animal": "elephants"},
    {"animal": "dolphins"},
    {"animal": "octopuses"},
])
for r in results:
    print(r)

# Stream — print tokens as they arrive
for chunk in chain.stream({"animal": "pandas"}):
    print(chunk, end="", flush=True)

# Async invoke
import asyncio

async def main():
    result = await chain.ainvoke({"animal": "wolves"})
    print(result)

asyncio.run(main())
```

## Chaining Multiple Steps
```python
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# Step 1: Generate a topic
topic_prompt = ChatPromptTemplate.from_template(
    "Give me a single interesting technical topic related to {field}. "
    "Reply with ONLY the topic name."
)

# Step 2: Write content about that topic
content_prompt = ChatPromptTemplate.from_template(
    "Write a short beginner-friendly explanation of: {topic}"
)

parser = StrOutputParser()

# Chain: field → topic → explanation
chain = (
    topic_prompt
    | llm
    | parser
    | (lambda topic: {"topic": topic})   # transform output to next input
    | content_prompt
    | llm
    | parser
)

result = chain.invoke({"field": "machine learning"})
print(result)
```

## RunnableLambda and RunnablePassthrough
```python
from langchain_core.runnables import RunnableLambda, RunnablePassthrough

# RunnableLambda — wrap any function
uppercase = RunnableLambda(lambda x: x.upper())
print(uppercase.invoke("hello"))   # HELLO

# RunnablePassthrough — pass input unchanged
from langchain_core.runnables import RunnableParallel

# Run branches in parallel and merge results
parallel_chain = RunnableParallel({
    "original": RunnablePassthrough(),
    "length":   RunnableLambda(len),
    "upper":    RunnableLambda(str.upper),
})

print(parallel_chain.invoke("hello world"))
# {'original': 'hello world', 'length': 11, 'upper': 'HELLO WORLD'}
```

## Simple Document Loading
```python
from langchain_community.document_loaders import TextLoader, WebBaseLoader

# Load a text file
loader = TextLoader("my_document.txt", encoding="utf-8")
docs = loader.load()
print(docs[0].page_content[:200])   # first 200 chars
print(docs[0].metadata)              # {'source': 'my_document.txt'}

# Load a web page
web_loader = WebBaseLoader("https://python.langchain.com/docs/introduction/")
web_docs = web_loader.load()

# Load multiple files
from langchain_community.document_loaders import DirectoryLoader
dir_loader = DirectoryLoader("./docs", glob="**/*.md")
all_docs = dir_loader.load()
print(f"Loaded {len(all_docs)} documents")
```

## Text Splitting
```python
from langchain_text_splitters import RecursiveCharacterTextSplitter

splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,        # max characters per chunk
    chunk_overlap=200,      # overlap between chunks to preserve context
    length_function=len,
    add_start_index=True,
)

chunks = splitter.split_documents(docs)
print(f"Split into {len(chunks)} chunks")
print(chunks[0].page_content)
print(chunks[0].metadata)   # includes chunk_index
```

## Basic Embeddings
```python
from langchain_openai import OpenAIEmbeddings

embeddings = OpenAIEmbeddings(model="text-embedding-3-small")

# Embed a single text
vector = embeddings.embed_query("What is LangChain?")
print(len(vector))    # 1536 dimensions

# Embed multiple documents
texts = ["Hello world", "Machine learning is great", "Python is fun"]
vectors = embeddings.embed_documents(texts)
print(len(vectors), len(vectors[0]))   # 3 documents, 1536 dimensions each
```

## Best Practices (Do's)

✅ **Always use environment variables for API keys**
```python
import os
from dotenv import load_dotenv

load_dotenv()
# Never do: llm = ChatOpenAI(api_key="sk-hardcoded!")
```

✅ **Use `StrOutputParser` to get plain text from chat models**
```python
from langchain_core.output_parsers import StrOutputParser

chain = prompt | llm | StrOutputParser()
text: str = chain.invoke({"query": "Hello"})
```

✅ **Use `ChatPromptTemplate` for chat models**
```python
# Good — structured messages
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a {role}."),
    ("user", "{query}"),
])

# Works but less flexible
prompt = PromptTemplate.from_template("You are a {role}. {query}")
```

✅ **Stream responses for better UX**
```python
for chunk in chain.stream(inputs):
    print(chunk, end="", flush=True)
print()   # newline at end
```

## Common Mistakes (Don'ts)

❌ **Don't hard-code API keys**
```python
# Bad — exposes secrets in code
llm = ChatOpenAI(api_key="sk-abc123...")

# Good
llm = ChatOpenAI()   # reads OPENAI_API_KEY from environment
```

❌ **Don't ignore token limits**
```python
# Bad — may exceed context window
chain.invoke({"text": huge_string_with_millions_of_chars})

# Good — split first
chunks = splitter.split_text(huge_string)
results = [chain.invoke({"text": c}) for c in chunks]
```

❌ **Don't use deprecated LLMChain**
```python
# Deprecated
from langchain.chains import LLMChain
chain = LLMChain(llm=llm, prompt=prompt)

# Good — use LCEL
chain = prompt | llm | StrOutputParser()
```

❌ **Don't call synchronous methods in async contexts**
```python
# Bad in async code
result = chain.invoke(inputs)   # blocks event loop

# Good
result = await chain.ainvoke(inputs)
```
