# Python Beginner Cheatsheet

## Data Types

### Numeric Types
```python
# Integers (unlimited precision)
count = 42
big_num = 1_000_000    # underscores for readability

# Floats
price = 19.99
sci = 1.5e-3           # 0.0015

# Complex
z = 3 + 4j

# Booleans (subclass of int)
is_valid = True        # == 1
is_done  = False       # == 0

# Type conversion
int("42")              # 42
float("3.14")          # 3.14
str(100)               # "100"
bool(0)                # False
bool("hello")          # True
```

### Strings
```python
name = "Alice"
greeting = 'Hello'
multi = """This is
a multi-line string."""

# f-strings (Python 3.6+) — preferred
message = f"Hello, {name}!"
expr    = f"2 + 2 = {2 + 2}"

# Common string methods
text = "  Hello, World!  "
text.strip()           # "Hello, World!"
text.lower()           # "  hello, world!  "
text.upper()           # "  HELLO, WORLD!  "
text.replace("World", "Python")  # "  Hello, Python!  "
text.split(",")        # ['  Hello', ' World!  ']
",".join(["a", "b"])   # "a,b"
text.startswith("  H") # True
"World" in text        # True
len(text)              # 18

# Indexing and slicing
s = "Python"
s[0]    # "P"
s[-1]   # "n"
s[0:3]  # "Pyt"
s[::-1] # "nohtyP"  (reversed)
```

### None
```python
result = None          # represents absence of value
if result is None:     # always compare with 'is', not '=='
    print("No result yet")
```

## Variables and Assignment

```python
# Simple assignment
age = 25
name = "Bob"

# Multiple assignment
x, y, z = 1, 2, 3

# Swap values (no temp variable needed)
x, y = y, x

# Augmented assignment
count = 0
count += 1   # count = count + 1
count -= 1
count *= 2
count //= 2  # integer division

# Walrus operator := (Python 3.8+)
if (n := len(name)) > 3:
    print(f"Name is {n} characters long")
```

## Operators

```python
# Arithmetic
10 + 3   # 13
10 - 3   # 7
10 * 3   # 30
10 / 3   # 3.3333...  (true division, returns float)
10 // 3  # 3          (floor division)
10 % 3   # 1          (modulo)
2 ** 8   # 256        (exponentiation)

# Comparison
5 == 5   # True
5 != 3   # True
5 > 3    # True
5 < 10   # True
5 >= 5   # True
5 <= 4   # False

# Logical
True and False  # False
True or False   # True
not True        # False

# Identity
x is None       # True if x is the None object
x is not None   # True if x is not None

# Membership
3 in [1, 2, 3]       # True
"py" in "python"     # True
5 not in [1, 2, 3]   # True
```

## Control Flow

### If / Elif / Else
```python
score = 85

if score >= 90:
    grade = "A"
elif score >= 80:
    grade = "B"
elif score >= 70:
    grade = "C"
else:
    grade = "F"

print(f"Grade: {grade}")  # Grade: B

# One-liner (ternary expression)
status = "Pass" if score >= 60 else "Fail"
```

### Match Statement (Python 3.10+)
```python
command = "quit"

match command:
    case "quit":
        print("Quitting...")
    case "help":
        print("Help text here")
    case "go" | "run":
        print("Going!")
    case _:
        print(f"Unknown command: {command}")

# Match with capture
point = (1, 0)
match point:
    case (0, 0):
        print("Origin")
    case (x, 0):
        print(f"X-axis at {x}")
    case (0, y):
        print(f"Y-axis at {y}")
    case (x, y):
        print(f"Point at ({x}, {y})")
```

## Loops

### For Loop
```python
# Iterate over a list
fruits = ["apple", "banana", "cherry"]
for fruit in fruits:
    print(fruit)

# Range
for i in range(5):          # 0, 1, 2, 3, 4
    print(i)

for i in range(1, 6):       # 1, 2, 3, 4, 5
    print(i)

for i in range(0, 10, 2):   # 0, 2, 4, 6, 8
    print(i)

# Enumerate (index + value)
for index, fruit in enumerate(fruits, start=1):
    print(f"{index}. {fruit}")

# Zip (iterate multiple lists together)
names  = ["Alice", "Bob"]
scores = [90, 85]
for name, score in zip(names, scores):
    print(f"{name}: {score}")
```

### While Loop
```python
count = 0
while count < 5:
    print(count)
    count += 1

# Break and continue
for i in range(10):
    if i == 3:
        continue    # skip 3
    if i == 7:
        break       # stop at 7
    print(i)        # prints 0, 1, 2, 4, 5, 6

# For/while + else
for i in range(5):
    print(i)
else:
    print("Loop finished normally")  # runs if no break
```

## Functions

```python
# Basic function
def greet(name):
    print(f"Hello, {name}!")

greet("Alice")  # Hello, Alice!

# Return value
def add(a, b):
    return a + b

result = add(3, 4)  # 7

# Default parameters
def power(base, exponent=2):
    return base ** exponent

power(3)      # 9
power(3, 3)   # 27

# Keyword arguments
def describe(name, age, city):
    print(f"{name}, {age}, from {city}")

describe(age=30, city="NYC", name="Alice")

# Variable-length arguments
def total(*args):          # args is a tuple
    return sum(args)

total(1, 2, 3, 4)  # 10

def show_info(**kwargs):   # kwargs is a dict
    for key, value in kwargs.items():
        print(f"{key}: {value}")

show_info(name="Alice", age=30)

# Type hints (recommended)
def multiply(x: int, y: int) -> int:
    return x * y

# Docstrings
def divide(a: float, b: float) -> float:
    """
    Divide a by b and return the result.

    Args:
        a: The numerator.
        b: The denominator (must not be zero).

    Returns:
        The result of a / b.
    """
    if b == 0:
        raise ValueError("Cannot divide by zero")
    return a / b
```

## Lists

```python
# Create
numbers = [1, 2, 3, 4, 5]
mixed   = [1, "hello", 3.14, True]
empty   = []

# Access
numbers[0]    # 1  (first)
numbers[-1]   # 5  (last)
numbers[1:3]  # [2, 3]  (slice)

# Modify
numbers.append(6)          # [1, 2, 3, 4, 5, 6]
numbers.insert(0, 0)       # [0, 1, 2, 3, 4, 5, 6]
numbers.remove(0)          # removes first occurrence of 0
numbers.pop()              # removes and returns last item
numbers.pop(0)             # removes and returns item at index 0

# Other operations
len(numbers)               # length
numbers.sort()             # sort in place
sorted(numbers)            # return new sorted list
numbers.reverse()          # reverse in place
numbers.count(3)           # count occurrences
numbers.index(3)           # find index of first occurrence
numbers.extend([7, 8])     # add multiple items
numbers.clear()            # remove all items
3 in numbers               # membership check

# List unpacking
first, *rest = [1, 2, 3, 4]  # first=1, rest=[2, 3, 4]
```

## Tuples

```python
# Create (immutable sequences)
point  = (3, 4)
single = (42,)      # single-element tuple needs trailing comma
coords = 1, 2, 3    # parentheses optional

# Access (same as lists)
point[0]   # 3

# Unpack
x, y = point
print(x, y)  # 3 4

# Named tuple
from collections import namedtuple
Point = namedtuple("Point", ["x", "y"])
p = Point(x=3, y=4)
print(p.x, p.y)  # 3 4
```

## Dictionaries

```python
# Create
person = {"name": "Alice", "age": 30, "city": "NYC"}
empty  = {}
also   = dict(name="Bob", age=25)

# Access
person["name"]             # "Alice"
person.get("email")        # None (safe access)
person.get("email", "N/A") # "N/A" (default value)

# Modify
person["email"] = "alice@example.com"  # add/update key
del person["city"]                     # delete key
person.pop("age")                      # remove and return value

# Iterate
for key in person:
    print(key)

for value in person.values():
    print(value)

for key, value in person.items():
    print(f"{key}: {value}")

# Other operations
len(person)                # number of keys
"name" in person           # True (key check)
person.keys()              # dict_keys([...])
person.values()            # dict_values([...])
person.update({"age": 31, "job": "Dev"})  # merge/update

# Dictionary unpacking (Python 3.9+)
defaults = {"color": "blue", "size": 10}
custom   = {"size": 20, "weight": 5}
merged   = defaults | custom  # {"color": "blue", "size": 20, "weight": 5}
```

## Sets

```python
# Create (unordered, unique elements)
fruits = {"apple", "banana", "cherry"}
also   = set(["apple", "apple", "banana"])  # {"apple", "banana"}
empty  = set()   # NOT {} which creates an empty dict

# Add / remove
fruits.add("orange")
fruits.discard("banana")  # no error if not found
fruits.remove("cherry")   # raises KeyError if not found

# Set operations
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

a | b   # Union:        {1, 2, 3, 4, 5, 6}
a & b   # Intersection: {3, 4}
a - b   # Difference:   {1, 2}
a ^ b   # Symmetric diff: {1, 2, 5, 6}

# Membership (O(1) average)
"apple" in fruits  # True
```

## Input / Output

```python
# Print
print("Hello, World!")
print("Name:", "Alice", "Age:", 30)
print("Pi =", 3.14159, end="\n")  # end default is "\n"
print("A", "B", "C", sep=", ")   # A, B, C

# Formatted output
name, score = "Alice", 95.5
print(f"Student: {name}, Score: {score:.1f}")   # Score: 95.5
print(f"{'Name':<10} {'Score':>6}")             # alignment

# User input (always returns str)
name = input("Enter your name: ")
age  = int(input("Enter your age: "))    # convert to int
price = float(input("Enter price: "))    # convert to float

# Safe input parsing
raw = input("Enter a number: ")
if raw.isdigit():
    number = int(raw)
    print(f"You entered: {number}")
else:
    print("Not a valid integer")
```

## Best Practices (Do's)

✅ **Use f-strings for string formatting**
```python
# Good
message = f"Hello, {name}! You are {age} years old."

# Avoid
message = "Hello, " + name + "! You are " + str(age) + " years old."
```

✅ **Use meaningful variable names**
```python
# Good
user_count = 42
total_price = 99.95

# Bad
n = 42
tp = 99.95
```

✅ **Use `enumerate` instead of manual counters**
```python
# Good
for i, item in enumerate(items):
    print(f"{i}: {item}")

# Avoid
i = 0
for item in items:
    print(f"{i}: {item}")
    i += 1
```

✅ **Use `is None` not `== None`**
```python
# Good
if result is None:
    ...

# Bad
if result == None:
    ...
```

✅ **Prefer `get()` for safe dict access**
```python
# Good
value = data.get("key", "default")

# Risky
value = data["key"]  # raises KeyError if missing
```

## Common Mistakes (Don'ts)

❌ **Don't use mutable default arguments**
```python
# Bad — the list is shared across all calls!
def add_item(item, lst=[]):
    lst.append(item)
    return lst

# Good
def add_item(item, lst=None):
    if lst is None:
        lst = []
    lst.append(item)
    return lst
```

❌ **Don't use bare `except:`**
```python
# Bad
try:
    risky()
except:
    pass

# Good
try:
    risky()
except ValueError as e:
    print(f"Value error: {e}")
```

❌ **Don't modify a list while iterating over it**
```python
# Bad
for item in my_list:
    if condition(item):
        my_list.remove(item)

# Good
my_list = [item for item in my_list if not condition(item)]
```

❌ **Don't compare booleans with ==**
```python
# Bad
if is_valid == True:
    ...

# Good
if is_valid:
    ...
```

❌ **Don't use `import *`**
```python
# Bad — pollutes namespace and hides dependencies
from math import *

# Good
from math import sqrt, pi
import math
```
