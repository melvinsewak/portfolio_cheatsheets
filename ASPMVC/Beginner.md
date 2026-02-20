# ASP.NET MVC Beginner Cheatsheet

## Project Setup

### Creating a New MVC Project
```bash
# Create project with the MVC template
dotnet new mvc -n MyMvcApp
cd MyMvcApp
dotnet run
```

### Program.cs (Minimal Hosting Model - .NET 6+)
```csharp
var builder = WebApplication.CreateBuilder(args);

// Add MVC services to the container
builder.Services.AddControllersWithViews();

var app = builder.Build();

// Configure the HTTP request pipeline
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

// Default route: /{controller}/{action}/{id?}
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

## Controllers

### Basic Controller
```csharp
using Microsoft.AspNetCore.Mvc;

namespace MyMvcApp.Controllers
{
    public class HomeController : Controller
    {
        // GET /Home/Index
        public IActionResult Index()
        {
            return View();
        }

        // GET /Home/About
        public IActionResult About()
        {
            return View();
        }
    }
}
```

### Action Results
```csharp
public class ProductsController : Controller
{
    // Return a view
    public IActionResult Index()
    {
        return View();
    }

    // Return a view with a model
    public IActionResult Details(int id)
    {
        var product = new Product { Id = id, Name = "Laptop", Price = 999.99m };
        return View(product);
    }

    // Redirect to another action
    public IActionResult RedirectExample()
    {
        return RedirectToAction("Index");
    }

    // Redirect to action in another controller
    public IActionResult GoHome()
    {
        return RedirectToAction("Index", "Home");
    }

    // Return JSON data
    public IActionResult GetJson()
    {
        return Json(new { name = "Laptop", price = 999 });
    }

    // Return HTTP status codes
    public IActionResult NotFoundExample()
    {
        return NotFound();              // 404
    }

    public IActionResult UnauthorizedExample()
    {
        return Unauthorized();          // 401
    }

    public IActionResult BadRequestExample()
    {
        return BadRequest("Invalid input");  // 400
    }

    // Return a file download
    public IActionResult Download()
    {
        byte[] fileBytes = System.IO.File.ReadAllBytes("report.pdf");
        return File(fileBytes, "application/pdf", "report.pdf");
    }

    // Return plain content
    public IActionResult PlainText()
    {
        return Content("Hello, World!");
    }
}
```

### Passing Data to Views

```csharp
public class HomeController : Controller
{
    public IActionResult Index()
    {
        // ViewData - dictionary, requires casting
        ViewData["Title"] = "Home Page";
        ViewData["Count"] = 42;

        // ViewBag - dynamic wrapper over ViewData
        ViewBag.Message = "Welcome to my app!";
        ViewBag.Year = DateTime.Now.Year;

        // TempData - survives one redirect
        TempData["SuccessMessage"] = "Record saved!";

        return View();
    }
}
```

### HTTP Method Attributes
```csharp
public class AccountController : Controller
{
    // Responds to GET requests
    [HttpGet]
    public IActionResult Login()
    {
        return View();
    }

    // Responds to POST requests
    [HttpPost]
    public IActionResult Login(string username, string password)
    {
        // Handle login logic
        return RedirectToAction("Index", "Home");
    }

    // Responds to PUT requests
    [HttpPut]
    public IActionResult Update(int id)
    {
        return Ok();
    }

    // Responds to DELETE requests
    [HttpDelete]
    public IActionResult Delete(int id)
    {
        return Ok();
    }
}
```

## Routing

### Convention-Based Routing
```csharp
// In Program.cs
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

// Examples that match the default route:
// /                     -> HomeController.Index()
// /Products             -> ProductsController.Index()
// /Products/Details     -> ProductsController.Details()
// /Products/Details/5   -> ProductsController.Details(id: 5)
```

### Attribute Routing
```csharp
[Route("products")]
public class ProductsController : Controller
{
    // GET /products
    [Route("")]
    [HttpGet]
    public IActionResult Index()
    {
        return View();
    }

    // GET /products/5
    [Route("{id:int}")]
    [HttpGet]
    public IActionResult Details(int id)
    {
        return View();
    }

    // GET /products/search?query=laptop
    [Route("search")]
    [HttpGet]
    public IActionResult Search(string query)
    {
        return View();
    }
}
```

### Route Constraints
```csharp
// id must be an integer
[Route("products/{id:int}")]

// id must be a GUID
[Route("items/{id:guid}")]

// name must be at least 3 characters
[Route("users/{name:minlength(3)}")]

// id must be between 1 and 100
[Route("pages/{id:range(1,100)}")]
```

## Models

### Basic Model Class
```csharp
namespace MyMvcApp.Models
{
    public class Product
    {
        public int Id { get; set; }
        public string Name { get; set; } = string.Empty;
        public decimal Price { get; set; }
        public string? Description { get; set; }
        public bool IsAvailable { get; set; }
        public DateTime CreatedAt { get; set; }
    }
}
```

### Model with Data Annotations
```csharp
using System.ComponentModel.DataAnnotations;

namespace MyMvcApp.Models
{
    public class Product
    {
        public int Id { get; set; }

        [Required(ErrorMessage = "Name is required")]
        [StringLength(100, MinimumLength = 2, ErrorMessage = "Name must be 2-100 characters")]
        [Display(Name = "Product Name")]
        public string Name { get; set; } = string.Empty;

        [Required]
        [Range(0.01, 10000, ErrorMessage = "Price must be between $0.01 and $10,000")]
        [DataType(DataType.Currency)]
        public decimal Price { get; set; }

        [StringLength(500)]
        public string? Description { get; set; }

        [Display(Name = "In Stock")]
        public bool IsAvailable { get; set; }

        [DataType(DataType.Date)]
        [Display(Name = "Created")]
        public DateTime CreatedAt { get; set; } = DateTime.Now;
    }
}
```

## Views and Razor Syntax

### Basic Razor View (Index.cshtml)
```html
@* This is a Razor comment *@

@{
    ViewData["Title"] = "Home Page";
    var greeting = "Hello";
}

<h1>@ViewData["Title"]</h1>
<p>@greeting, World!</p>

@* Display current date *@
<p>Today is @DateTime.Now.ToLongDateString()</p>
```

### View with Model
```html
@model MyMvcApp.Models.Product

@{
    ViewData["Title"] = Model.Name;
}

<h1>@Model.Name</h1>
<p>Price: @Model.Price.ToString("C")</p>
<p>Description: @Model.Description</p>

@if (Model.IsAvailable)
{
    <span class="badge bg-success">In Stock</span>
}
else
{
    <span class="badge bg-danger">Out of Stock</span>
}
```

### Razor Loops
```html
@model IEnumerable<MyMvcApp.Models.Product>

<table class="table">
    <thead>
        <tr>
            <th>Name</th>
            <th>Price</th>
            <th>Actions</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var product in Model)
        {
            <tr>
                <td>@product.Name</td>
                <td>@product.Price.ToString("C")</td>
                <td>
                    <a asp-action="Details" asp-route-id="@product.Id">Details</a>
                </td>
            </tr>
        }
    </tbody>
</table>

@if (!Model.Any())
{
    <p>No products found.</p>
}
```

### Razor Conditionals
```html
@{
    var score = 85;
}

@if (score >= 90)
{
    <p>Excellent!</p>
}
else if (score >= 70)
{
    <p>Good</p>
}
else
{
    <p>Needs improvement</p>
}

@* Inline conditional *@
<p class="@(score >= 70 ? "text-success" : "text-danger")">Score: @score</p>
```

## Tag Helpers

Tag Helpers replace HTML helper methods and look like standard HTML attributes.

### Anchor Tag Helper
```html
@* Generates: <a href="/Products/Details/5">View Details</a> *@
<a asp-controller="Products" asp-action="Details" asp-route-id="5">View Details</a>

@* Generates: <a href="/Home/Index">Home</a> *@
<a asp-action="Index" asp-controller="Home">Home</a>
```

### Form Tag Helper
```html
@model MyMvcApp.Models.Product

<form asp-action="Create" asp-controller="Products" method="post">
    <div class="mb-3">
        <label asp-for="Name" class="form-label"></label>
        <input asp-for="Name" class="form-control" />
        <span asp-validation-for="Name" class="text-danger"></span>
    </div>

    <div class="mb-3">
        <label asp-for="Price" class="form-label"></label>
        <input asp-for="Price" class="form-control" />
        <span asp-validation-for="Price" class="text-danger"></span>
    </div>

    <div class="mb-3 form-check">
        <input asp-for="IsAvailable" class="form-check-input" />
        <label asp-for="IsAvailable" class="form-check-label"></label>
    </div>

    <button type="submit" class="btn btn-primary">Save</button>
</form>
```

### Select Tag Helper
```html
@{
    var categories = new List<SelectListItem>
    {
        new SelectListItem { Value = "1", Text = "Electronics" },
        new SelectListItem { Value = "2", Text = "Clothing" },
        new SelectListItem { Value = "3", Text = "Books" }
    };
}

<select asp-for="CategoryId" asp-items="categories" class="form-select">
    <option value="">-- Select Category --</option>
</select>
```

### Image Tag Helper
Appends a version query string to image URLs so browsers load the updated file after a change.
```html
@* Generates: <img src="/images/logo.png?v=xyz123" alt="Logo"> *@
<img asp-append-version="true" src="~/images/logo.png" alt="Logo" />
```

### Link and Script Tag Helpers
Serve a local fallback if a CDN resource fails to load.
```html
<head>
    @* asp-append-version adds a cache-busting hash *@
    <link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />

    @* CDN with local fallback *@
    <link rel="stylesheet"
          href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css"
          asp-fallback-href="~/lib/bootstrap/dist/css/bootstrap.min.css"
          asp-fallback-test-class="sr-only"
          asp-fallback-test-property="position"
          asp-fallback-test-value="absolute" />
</head>

<body>
    @* asp-append-version on scripts *@
    <script src="~/js/site.js" asp-append-version="true"></script>

    @* CDN with local fallback *@
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"
            asp-fallback-src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"
            asp-fallback-test="window.bootstrap">
    </script>
</body>
```

### Environment Tag Helper
Renders content based on the current hosting environment (`Development`, `Staging`, `Production`).
```html
@* Render only in Development *@
<environment include="Development">
    <link rel="stylesheet" href="~/css/site.css" />
</environment>

@* Render in Staging and Production *@
<environment exclude="Development">
    <link rel="stylesheet"
          href="https://cdn.example.com/css/site.min.css"
          asp-fallback-href="~/css/site.min.css"
          asp-fallback-test-class="sr-only"
          asp-fallback-test-property="position"
          asp-fallback-test-value="absolute" />
</environment>
```

### Custom Tag Helper
Create your own Tag Helpers to encapsulate reusable HTML components.
```csharp
// TagHelpers/AlertTagHelper.cs
using Microsoft.AspNetCore.Razor.TagHelpers;

// Matches <alert type="success">...</alert>
[HtmlTargetElement("alert")]
public class AlertTagHelper : TagHelper
{
    // Maps to the "type" attribute: success, danger, warning, info
    public string Type { get; set; } = "info";

    public override async Task ProcessAsync(TagHelperContext context, TagHelperOutput output)
    {
        output.TagName = "div";
        output.Attributes.SetAttribute("class", $"alert alert-{Type} alert-dismissible");
        output.Attributes.SetAttribute("role", "alert");

        var content = await output.GetChildContentAsync();
        output.Content.SetHtmlContent(content.GetContent());
    }
}
```

```html
@* Register in _ViewImports.cshtml *@
@addTagHelper *, MyMvcApp

@* Use in a view *@
<alert type="success">Product saved successfully!</alert>
<alert type="danger">An error occurred.</alert>
```

## Layouts and Partial Views

### Layout File (_Layout.cshtml)
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <title>@ViewData["Title"] - MyMvcApp</title>
    <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="~/css/site.css" />
</head>
<body>
    <nav class="navbar navbar-expand-sm navbar-dark bg-dark">
        <a class="navbar-brand" asp-controller="Home" asp-action="Index">MyMvcApp</a>
        <div class="navbar-nav">
            <a class="nav-link" asp-controller="Products" asp-action="Index">Products</a>
        </div>
    </nav>

    <main class="container mt-4">
        @* RenderBody renders the content of the current view *@
        @RenderBody()
    </main>

    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>

    @* Optional sections defined by child views *@
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

### _ViewStart.cshtml
```html
@* Automatically sets the layout for all views *@
@{
    Layout = "_Layout";
}
```

### Partial Views
```html
@* In a view, render a partial view *@
@await Html.PartialAsync("_ProductCard", product)

@* Using Partial Tag Helper *@
<partial name="_ProductCard" model="product" />
```

```html
@* _ProductCard.cshtml *@
@model MyMvcApp.Models.Product

<div class="card">
    <div class="card-body">
        <h5 class="card-title">@Model.Name</h5>
        <p class="card-text">@Model.Price.ToString("C")</p>
    </div>
</div>
```

## Model Binding

### Binding from Route, Query String, and Form
```csharp
public class ProductsController : Controller
{
    // Binds from route: /products/5
    public IActionResult Details(int id)
    {
        return View();
    }

    // Binds from query string: /products/search?query=laptop&page=2
    public IActionResult Search(string query, int page = 1)
    {
        return View();
    }

    // Binds from posted form data
    [HttpPost]
    public IActionResult Create(Product product)
    {
        if (ModelState.IsValid)
        {
            // Save product
            return RedirectToAction("Index");
        }
        return View(product);
    }

    // Bind specific properties only (security: prevent over-posting)
    [HttpPost]
    public IActionResult Edit([Bind("Id,Name,Price")] Product product)
    {
        if (ModelState.IsValid)
        {
            // Update product
            return RedirectToAction("Index");
        }
        return View(product);
    }
}
```

## Do's and Don'ts

### ✅ Do's

✅ **Keep controllers thin – put business logic in services**
```csharp
// Good: controller delegates to a service
public class ProductsController : Controller
{
    private readonly IProductService _productService;

    public ProductsController(IProductService productService)
    {
        _productService = productService;
    }

    public async Task<IActionResult> Index()
    {
        var products = await _productService.GetAllAsync();
        return View(products);
    }
}
```

✅ **Always validate ModelState before processing form submissions**
```csharp
[HttpPost]
public IActionResult Create(Product product)
{
    if (!ModelState.IsValid)
    {
        return View(product);  // Return form with validation errors
    }
    // Safe to process
    return RedirectToAction("Index");
}
```

✅ **Use Tag Helpers over HTML helpers for cleaner markup**
```html
<!-- Good: Tag Helper -->
<a asp-action="Details" asp-route-id="@product.Id">Details</a>

<!-- Less clean: HTML Helper -->
@Html.ActionLink("Details", "Details", new { id = product.Id })
```

✅ **Use `[Bind]` or ViewModels to prevent over-posting attacks**
```csharp
// Only bind the fields you expect
[HttpPost]
public IActionResult Edit([Bind("Id,Name,Price")] Product product) { }
```

### ❌ Don'ts

❌ **Don't put database queries directly in views or controllers**
```csharp
// Bad: querying in controller action directly
public IActionResult Index()
{
    var products = _dbContext.Products.ToList();  // Move this to a service/repository
    return View(products);
}
```

❌ **Don't skip CSRF protection on POST forms**
```html
<!-- Bad: no anti-forgery token -->
<form method="post">...</form>

<!-- Good: include anti-forgery token (Tag Helpers add it automatically) -->
<form asp-action="Create" method="post">...</form>
```

```csharp
// Good: validate anti-forgery token on POST actions
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult Create(Product product) { }
```

❌ **Don't ignore return types – always return IActionResult or Task<IActionResult>**
```csharp
// Bad
public void Index() { }

// Good
public IActionResult Index()
{
    return View();
}
```

❌ **Don't hardcode URLs – use Tag Helpers or Url.Action()**
```html
<!-- Bad -->
<a href="/Products/Details/5">Details</a>

<!-- Good -->
<a asp-action="Details" asp-route-id="5">Details</a>
```

---

Ready for more? Continue to [Intermediate.md](Intermediate.md)! 🚀
