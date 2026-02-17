# C# Intermediate Cheatsheet

## Object-Oriented Programming

### Classes and Objects
```csharp
// Class definition
public class Person
{
    // Fields (private by default)
    private string name;
    
    // Properties (recommended over public fields)
    public string Name { get; set; }
    public int Age { get; set; }
    
    // Auto-implemented property with default value
    public string Country { get; set; } = "USA";
    
    // Read-only property
    public string FullInfo => $"{Name}, {Age} years old";
    
    // Constructor
    public Person(string name, int age)
    {
        Name = name;
        Age = age;
    }
    
    // Default constructor
    public Person() { }
    
    // Method
    public void Introduce()
    {
        Console.WriteLine($"Hi, I'm {Name} and I'm {Age} years old.");
    }
}

// Usage
Person person = new Person("Alice", 25);
person.Introduce();
```

### Inheritance
```csharp
// Base class
public class Animal
{
    public string Name { get; set; }
    
    public virtual void MakeSound()
    {
        Console.WriteLine("Some sound");
    }
}

// Derived class
public class Dog : Animal
{
    public override void MakeSound()
    {
        Console.WriteLine("Woof!");
    }
    
    public void Fetch()
    {
        Console.WriteLine($"{Name} is fetching the ball");
    }
}

// Usage
Dog dog = new Dog { Name = "Buddy" };
dog.MakeSound();  // Woof!
dog.Fetch();
```

### Interfaces
```csharp
// Interface definition
public interface IDrawable
{
    void Draw();
    void Resize(int width, int height);
}

// Implementation
public class Circle : IDrawable
{
    public int Radius { get; set; }
    
    public void Draw()
    {
        Console.WriteLine($"Drawing circle with radius {Radius}");
    }
    
    public void Resize(int width, int height)
    {
        Radius = width / 2;
    }
}
```

### Abstract Classes
```csharp
public abstract class Shape
{
    public abstract double CalculateArea();
    
    // Concrete method in abstract class
    public void Display()
    {
        Console.WriteLine($"Area: {CalculateArea()}");
    }
}

public class Rectangle : Shape
{
    public double Width { get; set; }
    public double Height { get; set; }
    
    public override double CalculateArea()
    {
        return Width * Height;
    }
}
```

## Collections

### List<T>
```csharp
using System.Collections.Generic;

// Create list
List<string> names = new List<string>();

// Add items
names.Add("Alice");
names.Add("Bob");
names.AddRange(new[] { "Charlie", "David" });

// Access items
string first = names[0];

// Remove items
names.Remove("Bob");
names.RemoveAt(0);

// Check existence
bool exists = names.Contains("Alice");

// Count
int count = names.Count;

// Iterate
foreach (string name in names)
{
    Console.WriteLine(name);
}

// Sort
names.Sort();
```

### Dictionary<TKey, TValue>
```csharp
// Create dictionary
Dictionary<string, int> ages = new Dictionary<string, int>();

// Add items
ages["Alice"] = 25;
ages["Bob"] = 30;
ages.Add("Charlie", 35);

// Access items
int aliceAge = ages["Alice"];

// Check if key exists
if (ages.ContainsKey("Alice"))
{
    Console.WriteLine($"Alice is {ages["Alice"]} years old");
}

// TryGetValue (safer)
if (ages.TryGetValue("David", out int age))
{
    Console.WriteLine($"Age: {age}");
}

// Remove items
ages.Remove("Bob");

// Iterate
foreach (KeyValuePair<string, int> kvp in ages)
{
    Console.WriteLine($"{kvp.Key}: {kvp.Value}");
}
```

### HashSet<T>
```csharp
// Create set (unique items)
HashSet<int> numbers = new HashSet<int> { 1, 2, 3, 4, 5 };

// Add item
numbers.Add(6);
numbers.Add(3);  // Won't add duplicate

// Set operations
HashSet<int> set1 = new HashSet<int> { 1, 2, 3 };
HashSet<int> set2 = new HashSet<int> { 3, 4, 5 };

set1.UnionWith(set2);        // { 1, 2, 3, 4, 5 }
set1.IntersectWith(set2);    // { 3 }
set1.ExceptWith(set2);       // { 1, 2 }
```

## LINQ (Language Integrated Query)

```csharp
using System.Linq;

List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

// Where - Filter
var evenNumbers = numbers.Where(n => n % 2 == 0);

// Select - Transform
var squared = numbers.Select(n => n * n);

// OrderBy / OrderByDescending
var sorted = numbers.OrderByDescending(n => n);

// First / FirstOrDefault
int first = numbers.First();
int firstOrDefault = numbers.FirstOrDefault();

// Any / All
bool hasEven = numbers.Any(n => n % 2 == 0);
bool allPositive = numbers.All(n => n > 0);

// Sum / Average / Min / Max
int sum = numbers.Sum();
double average = numbers.Average();
int min = numbers.Min();
int max = numbers.Max();

// Take / Skip
var firstThree = numbers.Take(3);
var skipTwo = numbers.Skip(2);

// Complex query
var result = numbers
    .Where(n => n > 3)
    .OrderBy(n => n)
    .Select(n => n * 2)
    .ToList();
```

### LINQ with Objects
```csharp
public class Student
{
    public string Name { get; set; }
    public int Age { get; set; }
    public double Grade { get; set; }
}

List<Student> students = new List<Student>
{
    new Student { Name = "Alice", Age = 20, Grade = 85.5 },
    new Student { Name = "Bob", Age = 22, Grade = 92.0 },
    new Student { Name = "Charlie", Age = 21, Grade = 78.5 }
};

// Filter students with grade > 80
var topStudents = students.Where(s => s.Grade > 80);

// Get names of students
var names = students.Select(s => s.Name);

// Order by grade
var ordered = students.OrderByDescending(s => s.Grade);

// Group by age
var grouped = students.GroupBy(s => s.Age);
```

## Exception Handling

### Try-Catch-Finally
```csharp
try
{
    int result = 10 / 0;  // Will throw exception
}
catch (DivideByZeroException ex)
{
    Console.WriteLine($"Error: {ex.Message}");
}
catch (Exception ex)
{
    Console.WriteLine($"General error: {ex.Message}");
}
finally
{
    // Always executes
    Console.WriteLine("Cleanup code");
}
```

### Throwing Exceptions
```csharp
public void ValidateAge(int age)
{
    if (age < 0)
    {
        throw new ArgumentException("Age cannot be negative");
    }
    
    if (age < 18)
    {
        throw new InvalidOperationException("Must be 18 or older");
    }
}

// Usage
try
{
    ValidateAge(-5);
}
catch (ArgumentException ex)
{
    Console.WriteLine(ex.Message);
}
```

### Custom Exceptions
```csharp
public class InvalidUserException : Exception
{
    public InvalidUserException() { }
    
    public InvalidUserException(string message) : base(message) { }
    
    public InvalidUserException(string message, Exception inner)
        : base(message, inner) { }
}

// Usage
throw new InvalidUserException("User not found");
```

## File I/O

### Reading Files
```csharp
using System.IO;

// Read all text
string content = File.ReadAllText("file.txt");

// Read all lines
string[] lines = File.ReadAllLines("file.txt");

// Read line by line
foreach (string line in File.ReadLines("file.txt"))
{
    Console.WriteLine(line);
}

// Using StreamReader
using (StreamReader reader = new StreamReader("file.txt"))
{
    string line;
    while ((line = reader.ReadLine()) != null)
    {
        Console.WriteLine(line);
    }
}
```

### Writing Files
```csharp
// Write all text
File.WriteAllText("file.txt", "Hello, World!");

// Write all lines
string[] lines = { "Line 1", "Line 2", "Line 3" };
File.WriteAllLines("file.txt", lines);

// Append text
File.AppendAllText("file.txt", "New line\n");

// Using StreamWriter
using (StreamWriter writer = new StreamWriter("file.txt"))
{
    writer.WriteLine("Hello");
    writer.WriteLine("World");
}
```

## Generics

### Generic Methods
```csharp
public T GetFirst<T>(List<T> list)
{
    if (list.Count > 0)
        return list[0];
    return default(T);
}

// Usage
List<int> numbers = new List<int> { 1, 2, 3 };
int first = GetFirst(numbers);
```

### Generic Classes
```csharp
public class Box<T>
{
    private T content;
    
    public void Put(T item)
    {
        content = item;
    }
    
    public T Get()
    {
        return content;
    }
}

// Usage
Box<int> intBox = new Box<int>();
intBox.Put(42);
int value = intBox.Get();

Box<string> stringBox = new Box<string>();
stringBox.Put("Hello");
```

## Best Practices (Do's)

✅ **Use properties instead of public fields**
```csharp
// Good
public string Name { get; set; }

// Avoid
public string name;
```

✅ **Use LINQ for readable collection operations**
```csharp
// Good
var adults = people.Where(p => p.Age >= 18).ToList();

// Less readable
List<Person> adults = new List<Person>();
foreach (var person in people)
{
    if (person.Age >= 18)
        adults.Add(person);
}
```

✅ **Use using statements for IDisposable objects**
```csharp
// Good
using (StreamReader reader = new StreamReader("file.txt"))
{
    // Use reader
}  // Automatically disposed

// Or with C# 8+
using StreamReader reader = new StreamReader("file.txt");
// Use reader
```

✅ **Prefer interfaces over concrete classes**
```csharp
// Good
IList<string> names = new List<string>();

// Less flexible
List<string> names = new List<string>();
```

✅ **Use meaningful exception types**
```csharp
// Good
throw new ArgumentNullException(nameof(parameter));
throw new InvalidOperationException("Invalid state");

// Less informative
throw new Exception("Error");
```

## Common Mistakes (Don'ts)

❌ **Don't catch exceptions without handling them**
```csharp
// Bad
try
{
    // Code
}
catch { }  // Swallowing exceptions

// Better
try
{
    // Code
}
catch (Exception ex)
{
    Log(ex);
    throw;
}
```

❌ **Don't modify collections while iterating**
```csharp
// Wrong
foreach (var item in list)
{
    list.Remove(item);  // Throws InvalidOperationException
}

// Correct
for (int i = list.Count - 1; i >= 0; i--)
{
    list.RemoveAt(i);
}
```

❌ **Don't use exceptions for flow control**
```csharp
// Bad
try
{
    int value = int.Parse(input);
}
catch
{
    value = 0;
}

// Good
if (int.TryParse(input, out int value))
{
    // Use value
}
else
{
    value = 0;
}
```

❌ **Don't forget to dispose of IDisposable objects**
```csharp
// Bad
FileStream fs = new FileStream("file.txt", FileMode.Open);
// Might not get disposed if exception occurs

// Good
using (FileStream fs = new FileStream("file.txt", FileMode.Open))
{
    // Use fs
}
```
