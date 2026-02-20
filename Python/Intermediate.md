# Python Intermediate Cheatsheet

## Object-Oriented Programming

### Classes and Objects
```python
class Person:
    # Class variable (shared by all instances)
    species = "Homo sapiens"

    def __init__(self, name: str, age: int) -> None:
        # Instance variables
        self.name = name
        self.age = age
        self._email = None        # convention: "protected"
        self.__id = id(self)      # name-mangled "private"

    # Instance method
    def greet(self) -> str:
        return f"Hi, I'm {self.name} and I'm {self.age} years old."

    # Property (controlled access)
    @property
    def email(self) -> str:
        return self._email

    @email.setter
    def email(self, value: str) -> None:
        if "@" not in value:
            raise ValueError("Invalid email address")
        self._email = value

    # Class method
    @classmethod
    def from_dict(cls, data: dict) -> "Person":
        return cls(data["name"], data["age"])

    # Static method
    @staticmethod
    def is_adult(age: int) -> bool:
        return age >= 18

    # String representations
    def __repr__(self) -> str:
        return f"Person(name={self.name!r}, age={self.age!r})"

    def __str__(self) -> str:
        return self.name


# Usage
alice = Person("Alice", 30)
alice.email = "alice@example.com"
print(alice.greet())
print(Person.is_adult(17))  # False

bob = Person.from_dict({"name": "Bob", "age": 25})
print(repr(bob))            # Person(name='Bob', age=25)
```

### Inheritance and Polymorphism
```python
class Animal:
    def __init__(self, name: str) -> None:
        self.name = name

    def speak(self) -> str:
        raise NotImplementedError("Subclass must implement speak()")

    def __str__(self) -> str:
        return f"{type(self).__name__}({self.name})"


class Dog(Animal):
    def speak(self) -> str:
        return f"{self.name} says Woof!"

    def fetch(self, item: str) -> str:
        return f"{self.name} fetches the {item}"


class Cat(Animal):
    def speak(self) -> str:
        return f"{self.name} says Meow!"


# Polymorphism
animals: list[Animal] = [Dog("Rex"), Cat("Whiskers")]
for animal in animals:
    print(animal.speak())  # Each subclass responds differently

# isinstance / issubclass checks
print(isinstance(animals[0], Animal))  # True
print(issubclass(Dog, Animal))         # True
```

### Abstract Base Classes
```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self) -> float: ...

    @abstractmethod
    def perimeter(self) -> float: ...

    def describe(self) -> str:
        return f"Area={self.area():.2f}, Perimeter={self.perimeter():.2f}"


class Rectangle(Shape):
    def __init__(self, width: float, height: float) -> None:
        self.width = width
        self.height = height

    def area(self) -> float:
        return self.width * self.height

    def perimeter(self) -> float:
        return 2 * (self.width + self.height)


r = Rectangle(4, 5)
print(r.describe())  # Area=20.00, Perimeter=18.00
```

### Dataclasses (Python 3.7+)
```python
from dataclasses import dataclass, field

@dataclass
class Product:
    name: str
    price: float
    tags: list[str] = field(default_factory=list)
    in_stock: bool = True

    def discount(self, pct: float) -> float:
        return self.price * (1 - pct / 100)


p = Product(name="Laptop", price=999.99, tags=["tech", "sale"])
print(p)             # Product(name='Laptop', price=999.99, ...)
print(p.discount(10))  # 899.991
```

## Comprehensions

```python
numbers = range(1, 11)

# List comprehension
squares  = [x ** 2 for x in numbers]
evens    = [x for x in numbers if x % 2 == 0]
pairs    = [(x, y) for x in range(3) for y in range(3) if x != y]

# Dict comprehension
sq_dict  = {x: x ** 2 for x in numbers}

# Set comprehension
sq_set   = {x ** 2 for x in numbers}

# Generator expression (lazy — does NOT build list in memory)
sq_gen   = (x ** 2 for x in numbers)
total    = sum(x ** 2 for x in numbers)  # no extra brackets needed

# Nested comprehension
matrix   = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
flat     = [n for row in matrix for n in row]
```

## Error Handling

### Try / Except / Else / Finally
```python
def divide(a: float, b: float) -> float:
    try:
        result = a / b
    except ZeroDivisionError:
        print("Cannot divide by zero")
        return 0.0
    except TypeError as e:
        print(f"Type error: {e}")
        raise
    else:
        # Runs only if no exception was raised
        print(f"Result: {result}")
        return result
    finally:
        # Always runs (cleanup)
        print("divide() finished")


# Multiple exception types
try:
    value = int(input("Enter number: "))
except (ValueError, OverflowError) as e:
    print(f"Invalid input: {e}")
```

### Custom Exceptions
```python
class AppError(Exception):
    """Base class for application errors."""

class NotFoundError(AppError):
    def __init__(self, resource: str, resource_id: int) -> None:
        self.resource = resource
        self.resource_id = resource_id
        super().__init__(f"{resource} with id={resource_id} not found")

class ValidationError(AppError):
    pass


# Usage
def get_user(user_id: int) -> dict:
    users = {1: {"name": "Alice"}}
    if user_id not in users:
        raise NotFoundError("User", user_id)
    return users[user_id]


try:
    user = get_user(99)
except NotFoundError as e:
    print(e)  # User with id=99 not found
```

### Exception Groups (Python 3.11+)
```python
# Raise multiple exceptions at once
try:
    raise ExceptionGroup("multiple errors", [
        ValueError("bad value"),
        TypeError("wrong type"),
    ])
except* ValueError as eg:
    print("ValueError(s):", eg.exceptions)
except* TypeError as eg:
    print("TypeError(s):", eg.exceptions)
```

## File I/O

```python
from pathlib import Path

path = Path("data.txt")

# Write a file
path.write_text("Hello\nWorld\n", encoding="utf-8")

# Read entire file
content = path.read_text(encoding="utf-8")
lines   = path.read_text().splitlines()

# Stream line-by-line (memory efficient for large files)
with open(path, encoding="utf-8") as f:
    for line in f:
        print(line.rstrip())

# Append
with open(path, "a", encoding="utf-8") as f:
    f.write("New line\n")

# Binary mode
with open("image.png", "rb") as f:
    data = f.read()

# Pathlib helpers
cwd        = Path.cwd()
home       = Path.home()
config     = home / ".config" / "app.json"    # path joining
exists     = config.exists()
config.parent.mkdir(parents=True, exist_ok=True)

# List directory
for p in Path(".").iterdir():
    if p.is_file():
        print(p.name, p.suffix)

# Glob patterns
for py_file in Path("src").glob("**/*.py"):
    print(py_file)
```

### Working with JSON
```python
import json

# Serialize to JSON string
data = {"name": "Alice", "scores": [90, 85, 92]}
json_str = json.dumps(data, indent=2)

# Parse JSON string
obj = json.loads(json_str)

# Read JSON file
with open("config.json", encoding="utf-8") as f:
    config = json.load(f)

# Write JSON file
with open("output.json", "w", encoding="utf-8") as f:
    json.dump(data, f, indent=2, ensure_ascii=False)
```

## Modules and Packages

```python
# Standard library imports
import os
import sys
import math
import random
import datetime

# From imports
from pathlib import Path
from collections import defaultdict, Counter, deque
from itertools import chain, islice, groupby
from functools import reduce, lru_cache, partial

# Aliased imports
import numpy as np          # third-party (pip install numpy)
import pandas as pd         # third-party

# Conditional import
try:
    import ujson as json    # fast JSON library if available
except ImportError:
    import json

# Useful standard library modules
from collections import Counter
words = ["the", "cat", "sat", "the", "the", "cat"]
freq  = Counter(words)
print(freq.most_common(2))  # [('the', 3), ('cat', 2)]

from itertools import islice
first_five = list(islice(range(1000), 5))  # [0, 1, 2, 3, 4]
```

## Decorators

```python
import functools
import time

# Basic decorator
def logger(func):
    @functools.wraps(func)   # preserve metadata
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__}")
        result = func(*args, **kwargs)
        print(f"{func.__name__} returned {result}")
        return result
    return wrapper


@logger
def add(a, b):
    return a + b

add(3, 4)  # prints log messages, returns 7


# Decorator with arguments
def repeat(times: int):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator


@repeat(3)
def say_hello():
    print("Hello!")

say_hello()  # prints "Hello!" 3 times


# Timing decorator
def timer(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        elapsed = time.perf_counter() - start
        print(f"{func.__name__} took {elapsed:.4f}s")
        return result
    return wrapper


# Cache decorator (memoize)
@lru_cache(maxsize=None)
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

print(fibonacci(50))  # very fast due to caching
```

## Iterators and Generators

```python
# Generator function (lazy sequence)
def count_up(start: int = 0, step: int = 1):
    n = start
    while True:
        yield n
        n += step


counter = count_up(1, 2)
print(next(counter))  # 1
print(next(counter))  # 3
print(next(counter))  # 5

# Finite generator
def fibonacci_gen(limit: int):
    a, b = 0, 1
    while a < limit:
        yield a
        a, b = b, a + b

print(list(fibonacci_gen(100)))  # [0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89]

# Generator expression (memory efficient)
total = sum(x ** 2 for x in range(1_000_000))

# Custom iterator class
class CountDown:
    def __init__(self, start: int) -> None:
        self.current = start

    def __iter__(self):
        return self

    def __next__(self) -> int:
        if self.current <= 0:
            raise StopIteration
        self.current -= 1
        return self.current + 1

for n in CountDown(3):
    print(n)   # 3, 2, 1
```

## Context Managers

```python
# Using built-in context managers
with open("file.txt") as f:
    data = f.read()

# Multiple context managers
with open("in.txt") as src, open("out.txt", "w") as dst:
    dst.write(src.read())

# Custom context manager — class-based
class Timer:
    def __enter__(self):
        self.start = time.perf_counter()
        return self

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.elapsed = time.perf_counter() - self.start
        print(f"Elapsed: {self.elapsed:.4f}s")
        return False   # don't suppress exceptions

with Timer() as t:
    time.sleep(0.1)
# prints: Elapsed: 0.1001s

# Custom context manager — decorator-based
from contextlib import contextmanager

@contextmanager
def managed_resource(name: str):
    print(f"Acquiring {name}")
    try:
        yield name
    finally:
        print(f"Releasing {name}")

with managed_resource("DB connection") as res:
    print(f"Using {res}")
```

## Lambda Functions and Functional Tools

```python
from functools import reduce, partial

# Lambda (anonymous function)
square = lambda x: x ** 2
add    = lambda x, y: x + y

# Map / filter / reduce
numbers = [1, 2, 3, 4, 5]
squares  = list(map(lambda x: x ** 2, numbers))
evens    = list(filter(lambda x: x % 2 == 0, numbers))
total    = reduce(lambda acc, x: acc + x, numbers, 0)

# Prefer comprehensions over map/filter for readability
squares = [x ** 2 for x in numbers]
evens   = [x for x in numbers if x % 2 == 0]

# Sorting with key function
people = [{"name": "Charlie", "age": 30}, {"name": "Alice", "age": 25}]
sorted_people = sorted(people, key=lambda p: p["age"])

# Partial function application
def multiply(a, b):
    return a * b

double = partial(multiply, b=2)
triple = partial(multiply, b=3)
print(double(5))  # 10
print(triple(5))  # 15
```

## Type Hints (Python 3.9+)

```python
from typing import Optional, Union, Any, TypeVar, Generic

# Basic annotations
def greet(name: str, times: int = 1) -> None:
    for _ in range(times):
        print(f"Hello, {name}!")

# Built-in generics (Python 3.9+)
def first(lst: list[int]) -> int | None:
    return lst[0] if lst else None

def merge(a: dict[str, int], b: dict[str, int]) -> dict[str, int]:
    return a | b

# Optional and Union
def parse(value: str) -> int | None:   # Python 3.10+ union syntax
    try:
        return int(value)
    except ValueError:
        return None

# TypeVar for generic functions
T = TypeVar("T")

def first_item(items: list[T]) -> T | None:
    return items[0] if items else None

# TypeAlias (Python 3.10+)
from typing import TypeAlias
Vector: TypeAlias = list[float]

# Protocols (structural subtyping)
from typing import Protocol

class Drawable(Protocol):
    def draw(self) -> None: ...

def render(obj: Drawable) -> None:
    obj.draw()
```

## Best Practices (Do's)

✅ **Use dataclasses for data containers**
```python
from dataclasses import dataclass

@dataclass
class Point:
    x: float
    y: float

p = Point(1.0, 2.0)
```

✅ **Use pathlib instead of os.path**
```python
from pathlib import Path

config = Path.home() / ".config" / "myapp" / "settings.json"
config.parent.mkdir(parents=True, exist_ok=True)
```

✅ **Use context managers for resources**
```python
# Good
with open("file.txt") as f:
    data = f.read()

# Bad — file may not be closed on error
f = open("file.txt")
data = f.read()
f.close()
```

✅ **Prefer generators for large data streams**
```python
# Memory efficient
def read_large_file(path):
    with open(path) as f:
        for line in f:
            yield line.strip()
```

✅ **Cache expensive function results**
```python
from functools import lru_cache

@lru_cache(maxsize=128)
def expensive_computation(n: int) -> int:
    return sum(range(n))
```

## Common Mistakes (Don'ts)

❌ **Don't re-invent the standard library**
```python
# Bad
result = []
for x in range(5):
    for y in range(5):
        result.append((x, y))

# Good
from itertools import product
result = list(product(range(5), range(5)))
```

❌ **Don't use bare `except Exception`**
```python
# Bad
try:
    process()
except Exception:
    pass     # silently swallows ALL exceptions

# Good
try:
    process()
except ValueError as e:
    logger.warning("Bad value: %s", e)
```

❌ **Don't use class attributes as mutable defaults**
```python
# Bad — all instances share the same list
class Config:
    items = []

# Good
class Config:
    def __init__(self):
        self.items = []
```

❌ **Don't mix I/O and business logic**
```python
# Bad
def process_user():
    name = input("Name: ")
    print(f"Hello, {name}")

# Good — separate concerns
def greet_user(name: str) -> str:
    return f"Hello, {name}"

name = input("Name: ")
print(greet_user(name))
```
