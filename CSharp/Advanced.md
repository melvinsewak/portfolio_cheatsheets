# C# Advanced Cheatsheet

## Asynchronous Programming

### Async/Await Basics
```csharp
using System.Threading.Tasks;

// Async method
public async Task<string> FetchDataAsync()
{
    await Task.Delay(1000);  // Simulate delay
    return "Data loaded";
}

// Async method with no return value
public async Task ProcessDataAsync()
{
    await Task.Delay(1000);
    Console.WriteLine("Processing complete");
}

// Usage
public async Task MainAsync()
{
    string data = await FetchDataAsync();
    Console.WriteLine(data);
    
    await ProcessDataAsync();
}
```

### Multiple Async Operations
```csharp
// Run tasks in parallel
public async Task<(string, string)> FetchMultipleAsync()
{
    Task<string> task1 = FetchData1Async();
    Task<string> task2 = FetchData2Async();
    
    // Wait for both
    await Task.WhenAll(task1, task2);
    
    return (task1.Result, task2.Result);
}

// Wait for first to complete
public async Task<string> FetchFirstAsync()
{
    Task<string> task1 = FetchData1Async();
    Task<string> task2 = FetchData2Async();
    
    Task<string> firstCompleted = await Task.WhenAny(task1, task2);
    return await firstCompleted;
}
```

### ConfigureAwait
```csharp
// In library code, use ConfigureAwait(false)
public async Task<string> LibraryMethodAsync()
{
    string result = await FetchDataAsync().ConfigureAwait(false);
    return result.ToUpper();
}

// In UI code, don't use ConfigureAwait or use ConfigureAwait(true)
public async Task ButtonClickAsync()
{
    string data = await FetchDataAsync();  // Returns to UI thread
    textBox.Text = data;
}
```

## Delegates and Events

### Delegates
```csharp
// Delegate declaration
public delegate int MathOperation(int x, int y);

// Methods matching delegate signature
public int Add(int x, int y) => x + y;
public int Multiply(int x, int y) => x * y;

// Usage
MathOperation operation = Add;
int result = operation(5, 3);  // 8

operation = Multiply;
result = operation(5, 3);  // 15

// Multicast delegates
MathOperation multiOp = Add;
multiOp += Multiply;
multiOp(5, 3);  // Calls both Add and Multiply
```

### Lambda Expressions
```csharp
// Simple lambda
Func<int, int, int> add = (x, y) => x + y;

// With statement body
Func<int, int, int> multiply = (x, y) =>
{
    int result = x * y;
    Console.WriteLine($"Result: {result}");
    return result;
};

// Action (no return value)
Action<string> print = message => Console.WriteLine(message);

// Predicate (returns bool)
Predicate<int> isEven = num => num % 2 == 0;
```

### Events
```csharp
// Publisher class
public class Button
{
    // Event declaration
    public event EventHandler Clicked;
    
    // Event with custom args
    public event EventHandler<ButtonClickEventArgs> ClickedWithInfo;
    
    // Method to raise event
    public void Click()
    {
        Clicked?.Invoke(this, EventArgs.Empty);
        
        ClickedWithInfo?.Invoke(this, new ButtonClickEventArgs
        {
            ClickTime = DateTime.Now,
            MouseButton = "Left"
        });
    }
}

// Custom event args
public class ButtonClickEventArgs : EventArgs
{
    public DateTime ClickTime { get; set; }
    public string MouseButton { get; set; }
}

// Subscriber
Button button = new Button();
button.Clicked += (sender, e) =>
{
    Console.WriteLine("Button clicked!");
};

button.ClickedWithInfo += (sender, e) =>
{
    Console.WriteLine($"Clicked at {e.ClickTime} with {e.MouseButton}");
};

button.Click();
```

## LINQ Advanced

### Query Syntax
```csharp
var result = from student in students
             where student.Grade > 80
             orderby student.Name
             select new { student.Name, student.Grade };
```

### Join Operations
```csharp
var students = new[] {
    new { Id = 1, Name = "Alice" },
    new { Id = 2, Name = "Bob" }
};

var grades = new[] {
    new { StudentId = 1, Grade = 85 },
    new { StudentId = 2, Grade = 92 }
};

// Inner join
var joined = from s in students
             join g in grades on s.Id equals g.StudentId
             select new { s.Name, g.Grade };

// Group join
var grouped = from s in students
              join g in grades on s.Id equals g.StudentId into studentGrades
              select new { s.Name, Grades = studentGrades };
```

### Custom LINQ Extension
```csharp
public static class LinqExtensions
{
    public static IEnumerable<T> WhereNot<T>(
        this IEnumerable<T> source,
        Func<T, bool> predicate)
    {
        return source.Where(item => !predicate(item));
    }
}

// Usage
var oddNumbers = numbers.WhereNot(n => n % 2 == 0);
```

## Nullable Types and Null Handling

### Nullable Value Types
```csharp
// Nullable int
int? nullableInt = null;
int? anotherInt = 42;

// Check for value
if (nullableInt.HasValue)
{
    int value = nullableInt.Value;
}

// Get value or default
int result = nullableInt ?? 0;  // 0 if null

// Null-conditional operator
int? length = text?.Length;
```

### Null-Coalescing and Null-Conditional
```csharp
// Null-coalescing operator
string name = userName ?? "Guest";

// Null-coalescing assignment (C# 8+)
name ??= "Default";

// Null-conditional operator
string upper = text?.ToUpper();
int? length = text?.Length;

// Chain null-conditional
string city = user?.Address?.City;

// Null-conditional with indexer
string first = array?[0];

// Null-conditional with method invocation
result = obj?.Method()?.Property;
```

### Nullable Reference Types (C# 8+)
```csharp
#nullable enable

// Non-nullable reference type
string name = "Alice";  // Cannot be null

// Nullable reference type
string? nullableName = null;  // Can be null

// Compiler warning if not checked
public void PrintLength(string? text)
{
    // Warning: possible null reference
    // Console.WriteLine(text.Length);
    
    // Correct: check for null
    if (text != null)
    {
        Console.WriteLine(text.Length);
    }
}
```

## Attributes

### Built-in Attributes
```csharp
// Obsolete
[Obsolete("Use NewMethod instead")]
public void OldMethod() { }

[Obsolete("This is deprecated", true)]  // Error instead of warning
public void DeprecatedMethod() { }

// Conditional compilation
[Conditional("DEBUG")]
public void DebugOnlyMethod() { }

// Serialization
[Serializable]
public class MyClass { }
```

### Custom Attributes
```csharp
// Define custom attribute
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method)]
public class AuthorAttribute : Attribute
{
    public string Name { get; set; }
    public string Date { get; set; }
    
    public AuthorAttribute(string name)
    {
        Name = name;
        Date = DateTime.Now.ToString();
    }
}

// Usage
[Author("Alice")]
public class MyClass
{
    [Author("Bob")]
    public void MyMethod() { }
}

// Reading attributes using reflection
var attribute = typeof(MyClass)
    .GetCustomAttributes(typeof(AuthorAttribute), false)
    .FirstOrDefault() as AuthorAttribute;
```

## Reflection

```csharp
using System.Reflection;

// Get type information
Type type = typeof(Person);
Type type2 = person.GetType();

// Get properties
PropertyInfo[] properties = type.GetProperties();
foreach (var prop in properties)
{
    Console.WriteLine($"{prop.Name}: {prop.PropertyType}");
}

// Get and set property values
PropertyInfo nameProp = type.GetProperty("Name");
object value = nameProp.GetValue(person);
nameProp.SetValue(person, "New Name");

// Get methods
MethodInfo[] methods = type.GetMethods();

// Invoke method
MethodInfo method = type.GetMethod("Introduce");
method.Invoke(person, null);

// Create instance dynamically
object instance = Activator.CreateInstance(type);
```

## Expression Trees

```csharp
using System.Linq.Expressions;

// Simple expression
Expression<Func<int, bool>> expr = x => x > 5;

// Build expression tree manually
ParameterExpression param = Expression.Parameter(typeof(int), "x");
ConstantExpression constant = Expression.Constant(5);
BinaryExpression comparison = Expression.GreaterThan(param, constant);
Expression<Func<int, bool>> lambda = 
    Expression.Lambda<Func<int, bool>>(comparison, param);

// Compile and execute
Func<int, bool> compiled = expr.Compile();
bool result = compiled(10);  // true
```

## Pattern Matching

### Type Patterns
```csharp
object obj = "Hello";

if (obj is string text)
{
    Console.WriteLine($"String: {text}");
}

// Switch with patterns
object value = 42;
string result = value switch
{
    int i => $"Integer: {i}",
    string s => $"String: {s}",
    null => "Null",
    _ => "Unknown"
};
```

### Property Patterns
```csharp
public record Person(string Name, int Age);

Person person = new Person("Alice", 25);

string message = person switch
{
    { Age: < 18 } => "Minor",
    { Age: >= 18, Age: < 65 } => "Adult",
    { Age: >= 65 } => "Senior",
    _ => "Unknown"
};
```

### Tuple Patterns
```csharp
(int x, int y) point = (5, 10);

string quadrant = point switch
{
    (0, 0) => "Origin",
    (> 0, > 0) => "Quadrant I",
    (< 0, > 0) => "Quadrant II",
    (< 0, < 0) => "Quadrant III",
    (> 0, < 0) => "Quadrant IV",
    _ => "On axis"
};
```

## Records (C# 9+)

```csharp
// Record declaration
public record Person(string Name, int Age);

// Usage
Person person1 = new Person("Alice", 25);
Person person2 = new Person("Alice", 25);

// Value equality
bool equal = person1 == person2;  // true

// With expressions (non-destructive mutation)
Person person3 = person1 with { Age = 26 };

// Deconstruction
var (name, age) = person1;
```

## Dependency Injection

```csharp
using Microsoft.Extensions.DependencyInjection;

// Service interface
public interface IEmailService
{
    void SendEmail(string to, string message);
}

// Implementation
public class EmailService : IEmailService
{
    public void SendEmail(string to, string message)
    {
        Console.WriteLine($"Sending to {to}: {message}");
    }
}

// Setup DI container
var services = new ServiceCollection();
services.AddSingleton<IEmailService, EmailService>();

var serviceProvider = services.BuildServiceProvider();

// Resolve service
var emailService = serviceProvider.GetService<IEmailService>();
emailService.SendEmail("user@example.com", "Hello!");

// Constructor injection
public class UserController
{
    private readonly IEmailService _emailService;
    
    public UserController(IEmailService emailService)
    {
        _emailService = emailService;
    }
    
    public void RegisterUser(string email)
    {
        _emailService.SendEmail(email, "Welcome!");
    }
}
```

## Best Practices (Do's)

✅ **Use async/await for I/O operations**
```csharp
// Good - non-blocking
public async Task<string> LoadDataAsync()
{
    return await File.ReadAllTextAsync("data.txt");
}

// Bad - blocking
public string LoadData()
{
    return File.ReadAllText("data.txt");
}
```

✅ **Use CancellationToken for cancellable operations**
```csharp
public async Task<string> FetchDataAsync(CancellationToken ct)
{
    await Task.Delay(5000, ct);
    ct.ThrowIfCancellationRequested();
    return "Data";
}
```

✅ **Prefer ValueTask for frequently completed async operations**
```csharp
public ValueTask<int> GetCachedValueAsync()
{
    if (cache.TryGetValue(key, out int value))
    {
        return new ValueTask<int>(value);  // Synchronous path
    }
    
    return new ValueTask<int>(FetchFromDatabaseAsync());
}
```

✅ **Use records for immutable data**
```csharp
// Good
public record Point(int X, int Y);

// More verbose
public class Point
{
    public int X { get; init; }
    public int Y { get; init; }
    // Plus Equals, GetHashCode, ToString, etc.
}
```

✅ **Use expression-bodied members for simple operations**
```csharp
public class Circle
{
    public double Radius { get; set; }
    
    // Expression-bodied property
    public double Area => Math.PI * Radius * Radius;
    
    // Expression-bodied method
    public double Circumference() => 2 * Math.PI * Radius;
}
```

## Common Mistakes (Don'ts)

❌ **Don't use async void except for event handlers**
```csharp
// Bad
public async void ProcessData()
{
    await Task.Delay(1000);
}

// Good
public async Task ProcessDataAsync()
{
    await Task.Delay(1000);
}

// OK for event handlers
private async void Button_Click(object sender, EventArgs e)
{
    await ProcessDataAsync();
}
```

❌ **Don't block on async code**
```csharp
// Bad - can cause deadlocks
var result = SomeAsyncMethod().Result;
SomeAsyncMethod().Wait();

// Good
var result = await SomeAsyncMethod();
```

❌ **Don't ignore Task results**
```csharp
// Bad - fire and forget
SomeAsyncMethod();

// Good
await SomeAsyncMethod();

// Or if you truly want fire-and-forget, be explicit
_ = Task.Run(async () => await SomeAsyncMethod());
```

❌ **Don't use reflection in performance-critical code**
```csharp
// Bad for hot paths
for (int i = 0; i < 1000000; i++)
{
    var value = property.GetValue(obj);  // Slow
}

// Better - cache delegate
var getter = CreateGetter(property);
for (int i = 0; i < 1000000; i++)
{
    var value = getter(obj);  // Fast
}
```

❌ **Don't capture loop variables in closures incorrectly**
```csharp
// Bad - all delegates capture same variable
for (int i = 0; i < 5; i++)
{
    tasks.Add(Task.Run(() => Console.WriteLine(i)));
}

// Good - capture copy
for (int i = 0; i < 5; i++)
{
    int copy = i;
    tasks.Add(Task.Run(() => Console.WriteLine(copy)));
}
```
