# Python Advanced Cheatsheet

## Async Programming (asyncio)

### Basics
```python
import asyncio

# Coroutine function
async def fetch_data(url: str) -> str:
    await asyncio.sleep(1)   # non-blocking wait
    return f"Data from {url}"

# Run a coroutine
result = asyncio.run(fetch_data("https://example.com"))

# Await multiple coroutines concurrently
async def main():
    urls = [
        "https://api.example.com/users",
        "https://api.example.com/posts",
        "https://api.example.com/comments",
    ]
    # Run all concurrently and gather results
    results = await asyncio.gather(*[fetch_data(url) for url in urls])
    for r in results:
        print(r)

asyncio.run(main())
```

### Tasks and Timeouts
```python
async def slow_operation():
    await asyncio.sleep(5)
    return "done"

async def main():
    # Create a task (starts immediately in the background)
    task = asyncio.create_task(slow_operation())

    # Do other work while task runs
    print("Doing other work...")
    await asyncio.sleep(0)   # yield control

    # Cancel if taking too long
    try:
        result = await asyncio.wait_for(task, timeout=2.0)
    except asyncio.TimeoutError:
        print("Task timed out!")
        task.cancel()

asyncio.run(main())
```

### Async Context Managers and Iterators
```python
import aiofiles   # pip install aiofiles

# Async context manager
async def read_file(path: str) -> str:
    async with aiofiles.open(path, encoding="utf-8") as f:
        return await f.read()

# Async generator
async def paginated_results(api_url: str, pages: int):
    for page in range(1, pages + 1):
        await asyncio.sleep(0.1)        # simulate API call
        yield {"page": page, "data": f"results_{page}"}

async def consume():
    async for page in paginated_results("https://api.example.com", 3):
        print(page)

asyncio.run(consume())
```

### asyncio Queue (Producer/Consumer)
```python
async def producer(queue: asyncio.Queue, items: list) -> None:
    for item in items:
        await queue.put(item)
        print(f"Produced: {item}")
    await queue.put(None)   # sentinel to signal completion

async def consumer(queue: asyncio.Queue) -> None:
    while True:
        item = await queue.get()
        if item is None:
            break
        print(f"Consumed: {item}")
        queue.task_done()

async def main():
    queue: asyncio.Queue = asyncio.Queue(maxsize=5)
    await asyncio.gather(
        producer(queue, [1, 2, 3, 4, 5]),
        consumer(queue),
    )

asyncio.run(main())
```

## Concurrency and Parallelism

### Threading (I/O-bound tasks)
```python
import threading
from concurrent.futures import ThreadPoolExecutor
import urllib.request

def download(url: str) -> str:
    with urllib.request.urlopen(url) as resp:
        return resp.read().decode()

urls = [
    "https://httpbin.org/get?page=1",
    "https://httpbin.org/get?page=2",
]

# ThreadPoolExecutor — simpler than raw threads
with ThreadPoolExecutor(max_workers=4) as executor:
    results = list(executor.map(download, urls))

# Thread-safe shared state
lock = threading.Lock()
shared_counter = 0

def increment():
    global shared_counter
    with lock:                  # acquire and release automatically
        shared_counter += 1
```

### Multiprocessing (CPU-bound tasks)
```python
from concurrent.futures import ProcessPoolExecutor
import multiprocessing

def cpu_intensive(n: int) -> int:
    return sum(i * i for i in range(n))

if __name__ == "__main__":   # required on Windows
    data = [10_000_000, 20_000_000, 30_000_000]

    with ProcessPoolExecutor(max_workers=multiprocessing.cpu_count()) as ex:
        results = list(ex.map(cpu_intensive, data))

    print(results)

# Shared memory between processes (Python 3.8+)
from multiprocessing import shared_memory
shm = shared_memory.SharedMemory(create=True, size=1024)
# ... write to shm.buf ...
shm.close()
shm.unlink()
```

### Free-Threaded Python (3.13+ experimental)
```python
# Run Python with --disable-gil for true thread parallelism
# python3.13t script.py   (use the 'free-threaded' build)

# Without GIL, ThreadPoolExecutor can speed up CPU-bound work:
import threading

results = [0] * 4

def compute(idx):
    results[idx] = sum(i * i for i in range(5_000_000))

threads = [threading.Thread(target=compute, args=(i,)) for i in range(4)]
for t in threads:
    t.start()
for t in threads:
    t.join()
```

## Generators and Coroutines (Advanced)

```python
# Send values into a generator
def accumulator():
    total = 0
    while True:
        value = yield total    # yield suspends and returns total; send() passes value
        if value is None:
            break
        total += value

gen = accumulator()
next(gen)           # prime the generator (advance to first yield)
gen.send(10)        # total = 10
gen.send(20)        # total = 30
gen.send(5)         # total = 35

# Generator delegation with yield from
def chain_generators(*iterables):
    for it in iterables:
        yield from it

combined = list(chain_generators([1, 2], [3, 4], [5]))
# [1, 2, 3, 4, 5]

# Infinite pipeline
def naturals():
    n = 0
    while True:
        yield n
        n += 1

def take(n, iterable):
    for i, item in enumerate(iterable):
        if i >= n:
            return
        yield item

print(list(take(5, naturals())))  # [0, 1, 2, 3, 4]
```

## Metaclasses

```python
# Metaclass controls class creation
class SingletonMeta(type):
    _instances: dict = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]


class Database(metaclass=SingletonMeta):
    def __init__(self, url: str) -> None:
        self.url = url
        print(f"Connecting to {url}")

db1 = Database("postgresql://localhost/mydb")
db2 = Database("postgresql://localhost/other")  # won't create new instance
print(db1 is db2)   # True


# Auto-register subclasses
class PluginMeta(type):
    registry: dict[str, type] = {}

    def __init__(cls, name, bases, namespace):
        super().__init__(name, bases, namespace)
        if bases:   # skip the base class itself
            PluginMeta.registry[name] = cls


class Plugin(metaclass=PluginMeta):
    def run(self): ...

class CSVPlugin(Plugin):
    def run(self): print("Processing CSV")

class JSONPlugin(Plugin):
    def run(self): print("Processing JSON")

for name, plugin_cls in PluginMeta.registry.items():
    print(name, plugin_cls)   # CSVPlugin, JSONPlugin
```

## Descriptors

```python
class Validator:
    """Descriptor that validates a numeric attribute stays within a range."""

    def __set_name__(self, owner, name: str) -> None:
        self.name = name
        self.private_name = f"_{name}"

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.private_name, None)

    def __set__(self, obj, value: float) -> None:
        if not isinstance(value, (int, float)):
            raise TypeError(f"{self.name} must be numeric, got {type(value).__name__}")
        setattr(obj, self.private_name, value)


class Temperature:
    celsius = Validator()

    def __init__(self, celsius: float) -> None:
        self.celsius = celsius

    @property
    def fahrenheit(self) -> float:
        return self.celsius * 9 / 5 + 32


t = Temperature(100)
print(t.fahrenheit)  # 212.0
t.celsius = "hot"    # TypeError: celsius must be numeric, got str
```

## Advanced Type Hints (Python 3.12+)

```python
from typing import TypeVar, ParamSpec, Concatenate, overload, TypedDict, Literal, Never

# TypedDict for structured dicts
class UserPayload(TypedDict):
    name: str
    age: int
    email: str | None

def create_user(payload: UserPayload) -> None:
    print(payload["name"])

# Literal types
Mode = Literal["r", "w", "rb", "wb"]

def open_file(path: str, mode: Mode = "r"):
    return open(path, mode)

# TypeVarTuple for variadic generics (Python 3.11+)
from typing import TypeVarTuple, Unpack

Ts = TypeVarTuple("Ts")

def zip_with(*args: Unpack[Ts]) -> tuple[Unpack[Ts]]:
    return args   # type: ignore

# Generic classes (Python 3.12 type statement)
type Vector[T] = list[T]         # Python 3.12+ type alias syntax

def scale[T: (int, float)](v: list[T], factor: T) -> list[T]:
    return [x * factor for x in v]

# Function overloads
@overload
def process(x: int) -> int: ...
@overload
def process(x: str) -> str: ...
def process(x):
    if isinstance(x, int):
        return x * 2
    return x.upper()
```

## Testing with pytest

```python
# test_calculator.py
import pytest
from calculator import add, divide   # assume these exist

# Basic test
def test_add():
    assert add(2, 3) == 5
    assert add(-1, 1) == 0

# Parametrize to run multiple inputs
@pytest.mark.parametrize("a, b, expected", [
    (2, 3, 5),
    (0, 0, 0),
    (-1, 1, 0),
    (100, -50, 50),
])
def test_add_parametrized(a, b, expected):
    assert add(a, b) == expected

# Test exceptions
def test_divide_by_zero():
    with pytest.raises(ValueError, match="Cannot divide by zero"):
        divide(10, 0)

# Fixtures
@pytest.fixture
def sample_data() -> list[int]:
    return [1, 2, 3, 4, 5]

def test_sum(sample_data):
    assert sum(sample_data) == 15

# Mocking
from unittest.mock import patch, MagicMock

def test_fetch_with_mock():
    with patch("mymodule.requests.get") as mock_get:
        mock_get.return_value = MagicMock(status_code=200, json=lambda: {"ok": True})
        from mymodule import fetch_status
        result = fetch_status("https://api.example.com")
        assert result == {"ok": True}
        mock_get.assert_called_once_with("https://api.example.com")
```

## Performance Optimization

### Profiling
```python
import cProfile
import pstats
import io

# Profile a function
pr = cProfile.Profile()
pr.enable()

# --- code to profile ---
result = sorted(range(100_000), key=lambda x: -x)
# --- end ---

pr.disable()
s = io.StringIO()
ps = pstats.Stats(pr, stream=s).sort_stats("cumulative")
ps.print_stats(10)
print(s.getvalue())

# Line profiler (pip install line_profiler)
# @profile decorator — run: kernprof -l -v script.py
```

### Memory Optimization
```python
import sys
from __future__ import annotations

# __slots__ reduces per-instance memory
class Point:
    __slots__ = ("x", "y")

    def __init__(self, x: float, y: float) -> None:
        self.x = x
        self.y = y

normal_class_instance = object.__new__(object)
print(sys.getsizeof(Point(1, 2)))      # smaller than dict-based instance

# Memory-mapped files for huge datasets
import mmap

with open("huge_file.bin", "r+b") as f:
    mm = mmap.mmap(f.fileno(), 0)
    print(mm[0:10])   # read first 10 bytes without loading all
    mm.close()

# Use array instead of list for homogeneous numerics
import array
arr = array.array("i", range(1_000_000))  # typed C array, ~4× smaller than list
```

### Caching and Memoization
```python
from functools import lru_cache, cache

# LRU cache
@lru_cache(maxsize=256)
def fib(n: int) -> int:
    return n if n < 2 else fib(n - 1) + fib(n - 2)

# Unbounded cache (Python 3.9+)
@cache
def factorial(n: int) -> int:
    return 1 if n == 0 else n * factorial(n - 1)

# Cache with TTL using third-party cachetools
from cachetools import TTLCache, cached   # pip install cachetools

ttl_store = TTLCache(maxsize=100, ttl=60)

@cached(cache=ttl_store)
def get_weather(city: str) -> dict:
    # expensive API call
    ...
```

## Design Patterns in Python

### Strategy Pattern
```python
from abc import ABC, abstractmethod

class SortStrategy(ABC):
    @abstractmethod
    def sort(self, data: list) -> list: ...

class QuickSort(SortStrategy):
    def sort(self, data: list) -> list:
        return sorted(data)       # simplified

class BubbleSort(SortStrategy):
    def sort(self, data: list) -> list:
        d = data[:]
        for i in range(len(d)):
            for j in range(len(d) - i - 1):
                if d[j] > d[j + 1]:
                    d[j], d[j + 1] = d[j + 1], d[j]
        return d

class Sorter:
    def __init__(self, strategy: SortStrategy) -> None:
        self._strategy = strategy

    def sort(self, data: list) -> list:
        return self._strategy.sort(data)

sorter = Sorter(QuickSort())
print(sorter.sort([3, 1, 4, 1, 5]))
```

### Observer Pattern
```python
from collections import defaultdict
from typing import Callable

class EventEmitter:
    def __init__(self) -> None:
        self._listeners: dict[str, list[Callable]] = defaultdict(list)

    def on(self, event: str, listener: Callable) -> None:
        self._listeners[event].append(listener)

    def emit(self, event: str, *args, **kwargs) -> None:
        for listener in self._listeners[event]:
            listener(*args, **kwargs)


emitter = EventEmitter()
emitter.on("data", lambda x: print(f"Listener A got: {x}"))
emitter.on("data", lambda x: print(f"Listener B got: {x}"))
emitter.emit("data", {"key": "value"})
```

## Best Practices (Do's)

✅ **Use `asyncio.gather` for concurrent async calls**
```python
# Good — runs concurrently
results = await asyncio.gather(fetch(url1), fetch(url2), fetch(url3))

# Slow — sequential
r1 = await fetch(url1)
r2 = await fetch(url2)
```

✅ **Prefer `ProcessPoolExecutor` for CPU-bound, `ThreadPoolExecutor` for I/O-bound**
```python
# CPU-bound
with ProcessPoolExecutor() as ex:
    results = list(ex.map(cpu_work, data))

# I/O-bound
with ThreadPoolExecutor() as ex:
    results = list(ex.map(io_work, data))
```

✅ **Use `__slots__` for memory-critical classes**
```python
class Coordinate:
    __slots__ = ("lat", "lon")
    def __init__(self, lat, lon):
        self.lat, self.lon = lat, lon
```

✅ **Type-annotate public APIs completely**
```python
def search(query: str, max_results: int = 10) -> list[dict[str, str]]:
    ...
```

✅ **Use `@dataclass(frozen=True)` for immutable value objects**
```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Money:
    amount: float
    currency: str
```

## Common Mistakes (Don'ts)

❌ **Don't mix sync and async code carelessly**
```python
# Bad — blocking call inside async function
async def bad():
    time.sleep(1)       # blocks the event loop!

# Good
async def good():
    await asyncio.sleep(1)  # non-blocking
```

❌ **Don't forget `if __name__ == "__main__":` for multiprocessing**
```python
# Required on Windows to prevent recursive spawning
if __name__ == "__main__":
    with ProcessPoolExecutor() as ex:
        ex.map(work, data)
```

❌ **Don't use `lru_cache` on methods with mutable arguments**
```python
# Bad — lists are unhashable and can't be cached
@lru_cache
def process(data: list):   # TypeError!
    ...

# Good — convert to tuple first
@lru_cache
def process(data: tuple):
    ...
```

❌ **Don't raise or catch `BaseException` unless absolutely necessary**
```python
# Bad — catches SystemExit and KeyboardInterrupt!
try:
    ...
except BaseException:
    pass

# Good
try:
    ...
except Exception:
    ...
```

❌ **Don't leave coroutines unawaited**
```python
async def main():
    fetch_data("url")   # creates coroutine object but never awaits it — silent bug!
    await fetch_data("url")  # correct
```
