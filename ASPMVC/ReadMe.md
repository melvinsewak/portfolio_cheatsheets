# ASP.NET MVC Cheatsheet

Welcome to the ASP.NET MVC cheatsheet! This guide will help you quickly get up to speed with building web applications using ASP.NET Core MVC.

## Overview

ASP.NET Core MVC is a rich framework for building web applications and APIs using the Model-View-Controller (MVC) design pattern. It is cross-platform, high-performance, and tightly integrated with the .NET ecosystem, making it the go-to choice for enterprise-grade web development with C#.

## Contents

- [Beginner.md](Beginner.md) - Project setup, controllers, views, Razor syntax, models, routing, and Tag Helpers
- [Intermediate.md](Intermediate.md) - Entity Framework Core, validation, ViewModels, authentication, and dependency injection
- [Advanced.md](Advanced.md) - Web API, middleware, filters, SignalR, testing, performance optimization, and security

## Quick Reference

### Basic Controller
```csharp
using Microsoft.AspNetCore.Mvc;

public class HomeController : Controller
{
    public IActionResult Index()
    {
        return View();
    }

    public IActionResult About()
    {
        ViewData["Message"] = "Your application description.";
        return View();
    }
}
```

### Basic Razor View
```html
@model MyApp.Models.Product

<h1>@Model.Name</h1>
<p>Price: @Model.Price.ToString("C")</p>
```

### Key Features

- **MVC Pattern**: Clear separation of concerns with Models, Views, and Controllers
- **Razor Syntax**: Powerful server-side templating with C# embedded in HTML
- **Tag Helpers**: Server-side rendering helpers that look like standard HTML attributes
- **Routing**: Convention-based and attribute-based routing
- **Model Binding**: Automatic mapping of HTTP request data to action parameters
- **Dependency Injection**: Built-in DI container for clean, testable code
- **Entity Framework Core**: ORM for database access with LINQ support

## Getting Started

### Prerequisites
- [.NET SDK](https://dotnet.microsoft.com/download) (6.0 or later recommended)
- An IDE such as [Visual Studio](https://visualstudio.microsoft.com/) or [VS Code](https://code.visualstudio.com/) with the C# extension

### Create a New Project
```bash
# Create a new ASP.NET Core MVC project
dotnet new mvc -n MyMvcApp

# Navigate into the project
cd MyMvcApp

# Run the application
dotnet run
```

### Navigate to the Application
```
https://localhost:5001
```

## Project Structure
```
MyMvcApp/
├── Controllers/
│   └── HomeController.cs
├── Models/
│   └── ErrorViewModel.cs
├── Views/
│   ├── Home/
│   │   ├── Index.cshtml
│   │   └── Privacy.cshtml
│   ├── Shared/
│   │   ├── _Layout.cshtml
│   │   └── _ValidationScriptsPartial.cshtml
│   ├── _ViewImports.cshtml
│   └── _ViewStart.cshtml
├── wwwroot/
│   ├── css/
│   ├── js/
│   └── lib/
├── appsettings.json
└── Program.cs
```

## Essential CLI Commands

```bash
# Create new MVC project
dotnet new mvc -n MyMvcApp

# Add Entity Framework Core packages
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools

# Add a migration
dotnet ef migrations add InitialCreate

# Apply migrations to the database
dotnet ef database update

# Build the project
dotnet build

# Run the project
dotnet run

# Run tests
dotnet test

# Publish for deployment
dotnet publish -c Release -o ./publish
```

## Learning Path

1. **Start with [Beginner.md](Beginner.md)** - Learn the fundamentals: controllers, views, Razor, models, and routing
2. **Progress to [Intermediate.md](Intermediate.md)** - Master database access, validation, authentication, and DI
3. **Advance to [Advanced.md](Advanced.md)** - Explore Web API, middleware, performance, and testing

## Additional Resources

### Official Documentation
- [ASP.NET Core MVC Overview](https://docs.microsoft.com/en-us/aspnet/core/mvc/overview)
- [Razor Syntax Reference](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/razor)
- [Entity Framework Core](https://docs.microsoft.com/en-us/ef/core/)

### Tools & Libraries
- [Entity Framework Core](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore) - ORM for database access
- [AutoMapper](https://automapper.org/) - Object-to-object mapping
- [FluentValidation](https://fluentvalidation.net/) - Validation library
- [Serilog](https://serilog.net/) - Structured logging

### Community
- [ASP.NET Core GitHub](https://github.com/dotnet/aspnetcore)
- [.NET Blog](https://devblogs.microsoft.com/dotnet/)
- [Stack Overflow - asp.net-core-mvc](https://stackoverflow.com/questions/tagged/asp.net-core-mvc)

## Quick Tips

✅ **Do:**
- Follow the MVC pattern and keep concerns separated
- Use ViewModels to pass data between controllers and views
- Validate input on both client and server sides
- Use dependency injection for services and repositories
- Use async/await for database and I/O operations
- Apply the principle of least privilege for authorization

❌ **Don't:**
- Put business logic directly in controllers or views
- Access the database directly from views
- Store sensitive data in ViewData or session without encryption
- Ignore model validation errors
- Use synchronous database calls that block threads

---

Happy coding with ASP.NET Core MVC! 🚀
