# Python Cheatsheet

Welcome to the Python cheatsheet! This guide will help you quickly master Python 3.13, one of the most versatile and popular programming languages in the world.

## Overview

Python is a high-level, interpreted, general-purpose programming language emphasizing readability and simplicity. It powers everything from web development and data science to AI/ML, scripting, and automation. Python 3.13 introduces improved error messages, a free-threaded build option (no GIL), and an interactive interpreter (REPL) with better multi-line editing.

## Contents

- [Beginner.md](Beginner.md) - Data types, variables, control flow, functions, lists, dicts, and I/O
- [Intermediate.md](Intermediate.md) - OOP, comprehensions, file I/O, error handling, modules, and decorators
- [Advanced.md](Advanced.md) - Async/await, generators, type hints, metaclasses, concurrency, and testing

## Quick Reference

### Hello World
```python
print("Hello, World!")
```

### Variables & Types
```python
name: str = "Alice"
age: int = 30
price: float = 9.99
is_active: bool = True
```

### Collections
```python
my_list  = [1, 2, 3]
my_tuple = (1, 2, 3)
my_dict  = {"key": "value"}
my_set   = {1, 2, 3}
```

### Function
```python
def greet(name: str = "World") -> str:
    return f"Hello, {name}!"

print(greet("Alice"))  # Hello, Alice!
```

### Key Features

- **Readable Syntax**: Clean, English-like code that's easy to read and write
- **Dynamic Typing**: No need to declare variable types explicitly
- **Rich Standard Library**: Batteries included for networking, files, math, and more
- **Huge Ecosystem**: PyPI hosts 500k+ packages (pip install anything)
- **Cross-Platform**: Runs on Windows, macOS, Linux, and more
- **Multipurpose**: Web, data science, AI/ML, automation, scripting

## Getting Started

### Install Python 3.13

**Windows:**
Download the installer from [python.org](https://www.python.org/downloads/) and run it.
Check "Add Python to PATH" during installation.

**macOS:**
```bash
brew install python@3.13
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt update && sudo apt install python3.13 python3.13-venv python3-pip
```

### Verify Installation
```bash
python --version   # Python 3.13.x
python -m pip --version
```

### Virtual Environments (Recommended)
```bash
# Create virtual environment
python -m venv .venv

# Activate (macOS/Linux)
source .venv/bin/activate

# Activate (Windows)
.venv\Scripts\activate

# Install packages
pip install requests

# Deactivate
deactivate
```

### Run Python Code
```bash
# Interactive REPL
python

# Run a script
python script.py

# Run a module
python -m http.server 8000
```

## Python 3.13 Highlights

- 🚀 **Free-threaded mode** (`--disable-gil`) for true multi-core parallelism
- 🐛 **Improved error messages** with precise location indicators
- 📝 **Enhanced REPL** with multi-line editing and syntax highlighting
- ⚡ **Incremental GC** for reduced pause times
- 🔧 **`copy.replace()`** for immutable objects
- 🧩 **`typing.TypeVar` defaults** for more expressive generics

## Quick Tips

✅ **DO:**
- Use virtual environments for every project
- Follow PEP 8 style guidelines
- Add type hints to function signatures
- Use `f-strings` for string formatting
- Write docstrings for public functions

❌ **DON'T:**
- Use mutable defaults in function parameters (`def f(lst=[])`)
- Catch bare `except:` clauses
- Use `==` to compare with `None` (use `is`)
- Import `*` from modules
- Ignore warnings from the interpreter

## Additional Resources

- [Official Python Documentation](https://docs.python.org/3.13/)
- [PEP 8 Style Guide](https://pep8.org/)
- [Python Package Index (PyPI)](https://pypi.org/)
- [Real Python Tutorials](https://realpython.com/)
- [Python Cheatsheet Community](https://www.pythoncheatsheet.org/)
- [What's New in Python 3.13](https://docs.python.org/3.13/whatsnew/3.13.html)
