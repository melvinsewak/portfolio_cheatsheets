# C# Beginner Cheatsheet

## Data Types

### Value Types
```csharp
// Integer types
int number = 42;
long bigNumber = 123456789L;
short smallNumber = 100;
byte tinyNumber = 255;

// Floating point
float price = 19.99f;
double precise = 3.14159265359;
decimal money = 99.95m;  // Best for financial calculations

// Boolean
bool isTrue = true;
bool isFalse = false;

// Character
char letter = 'A';
```

### Reference Types
```csharp
// String
string name = "John Doe";
string message = $"Hello, {name}!";  // String interpolation

// Array
int[] numbers = { 1, 2, 3, 4, 5 };
string[] names = new string[3];
```

## Variables and Constants

```csharp
// Variable declaration
int age = 25;
string name = "Alice";

// Constant (cannot be changed)
const double PI = 3.14159;
const int MAX_USERS = 100;

// Type inference with var
var count = 10;  // Compiler infers int
var text = "Hello";  // Compiler infers string
```

## Operators

```csharp
// Arithmetic
int sum = 5 + 3;        // 8
int diff = 10 - 4;      // 6
int product = 4 * 3;    // 12
int quotient = 15 / 3;  // 5
int remainder = 17 % 5; // 2

// Comparison
bool isEqual = (5 == 5);       // true
bool notEqual = (5 != 3);      // true
bool greater = (10 > 5);       // true
bool less = (3 < 7);           // true

// Logical
bool and = true && false;  // false
bool or = true || false;   // true
bool not = !true;          // false

// Increment/Decrement
int x = 5;
x++;  // x is now 6
x--;  // x is now 5
```

## Control Flow

### If-Else Statements
```csharp
int score = 85;

if (score >= 90)
{
    Console.WriteLine("Grade: A");
}
else if (score >= 80)
{
    Console.WriteLine("Grade: B");
}
else if (score >= 70)
{
    Console.WriteLine("Grade: C");
}
else
{
    Console.WriteLine("Grade: F");
}
```

### Switch Statement
```csharp
int day = 3;

switch (day)
{
    case 1:
        Console.WriteLine("Monday");
        break;
    case 2:
        Console.WriteLine("Tuesday");
        break;
    case 3:
        Console.WriteLine("Wednesday");
        break;
    default:
        Console.WriteLine("Other day");
        break;
}
```

### Ternary Operator
```csharp
int age = 20;
string status = (age >= 18) ? "Adult" : "Minor";
```

## Loops

### For Loop
```csharp
for (int i = 0; i < 5; i++)
{
    Console.WriteLine($"Iteration {i}");
}
```

### While Loop
```csharp
int count = 0;
while (count < 5)
{
    Console.WriteLine(count);
    count++;
}
```

### Do-While Loop
```csharp
int num = 0;
do
{
    Console.WriteLine(num);
    num++;
} while (num < 5);
```

### Foreach Loop
```csharp
string[] fruits = { "Apple", "Banana", "Orange" };

foreach (string fruit in fruits)
{
    Console.WriteLine(fruit);
}
```

## Methods/Functions

```csharp
// Method with no return value
void SayHello()
{
    Console.WriteLine("Hello!");
}

// Method with parameters and return value
int Add(int a, int b)
{
    return a + b;
}

// Method with default parameters
void Greet(string name = "Guest")
{
    Console.WriteLine($"Hello, {name}!");
}

// Calling methods
SayHello();
int result = Add(5, 3);
Greet();
Greet("Alice");
```

## String Operations

```csharp
string text = "Hello World";

// Length
int length = text.Length;  // 11

// Convert case
string upper = text.ToUpper();    // "HELLO WORLD"
string lower = text.ToLower();    // "hello world"

// Substring
string sub = text.Substring(0, 5);  // "Hello"

// Contains
bool contains = text.Contains("World");  // true

// Replace
string replaced = text.Replace("World", "C#");  // "Hello C#"

// Split
string[] words = text.Split(' ');  // ["Hello", "World"]

// Concatenation
string greeting = "Hello" + " " + "World";
string combined = string.Concat("Hello", " ", "World");
```

## Arrays

```csharp
// Declaration and initialization
int[] numbers = { 1, 2, 3, 4, 5 };

// Access elements
int first = numbers[0];  // 1
int last = numbers[4];   // 5

// Modify elements
numbers[0] = 10;

// Array length
int length = numbers.Length;  // 5

// Multi-dimensional array
int[,] matrix = new int[2, 3]
{
    { 1, 2, 3 },
    { 4, 5, 6 }
};

// Jagged array
int[][] jagged = new int[2][];
jagged[0] = new int[] { 1, 2, 3 };
jagged[1] = new int[] { 4, 5 };
```

## Input/Output

```csharp
// Output
Console.WriteLine("Hello, World!");  // With newline
Console.Write("Hello");              // Without newline

// Input
Console.Write("Enter your name: ");
string name = Console.ReadLine();

// Parse input
Console.Write("Enter your age: ");
int age = int.Parse(Console.ReadLine());

// TryParse (safer)
Console.Write("Enter a number: ");
if (int.TryParse(Console.ReadLine(), out int number))
{
    Console.WriteLine($"You entered: {number}");
}
else
{
    Console.WriteLine("Invalid number!");
}
```

## Best Practices (Do's)

✅ **Use meaningful variable names**
```csharp
// Good
int studentAge = 20;
string userName = "Alice";

// Bad
int a = 20;
string s = "Alice";
```

✅ **Use proper naming conventions**
- PascalCase for classes, methods: `MyClass`, `GetUserName()`
- camelCase for variables, parameters: `myVariable`, `userName`
- ALL_CAPS for constants: `MAX_VALUE`

✅ **Always initialize variables before use**
```csharp
int count = 0;  // Good
string name = "";  // Good
```

✅ **Use string interpolation for readability**
```csharp
// Good
string message = $"Hello, {name}!";

// Less readable
string message = "Hello, " + name + "!";
```

## Common Mistakes (Don'ts)

❌ **Don't compare strings with == for case-insensitive comparison**
```csharp
// Wrong
if (name == "alice") { }

// Correct
if (name.Equals("alice", StringComparison.OrdinalIgnoreCase)) { }
```

❌ **Don't forget break in switch statements**
```csharp
// Wrong - will cause compile error
switch (value)
{
    case 1:
        Console.WriteLine("One");
        // Missing break!
    case 2:
        Console.WriteLine("Two");
        break;
}
```

❌ **Don't use magic numbers**
```csharp
// Bad
if (status == 1) { }

// Good
const int STATUS_ACTIVE = 1;
if (status == STATUS_ACTIVE) { }
```

❌ **Don't ignore potential null values**
```csharp
// Risky
string text = GetUserInput();
int length = text.Length;  // Could throw NullReferenceException

// Better
string text = GetUserInput();
if (text != null)
{
    int length = text.Length;
}
```
