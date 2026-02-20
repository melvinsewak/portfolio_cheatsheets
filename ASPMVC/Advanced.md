# ASP.NET MVC Advanced Cheatsheet

## Web API with ASP.NET Core MVC

ASP.NET Core MVC supports building RESTful APIs alongside traditional MVC views in the same project.

### API Controller
```csharp
using Microsoft.AspNetCore.Mvc;

[ApiController]
[Route("api/[controller]")]
public class ProductsApiController : ControllerBase
{
    private readonly IProductRepository _repository;

    public ProductsApiController(IProductRepository repository)
    {
        _repository = repository;
    }

    // GET api/productsapi
    [HttpGet]
    public async Task<ActionResult<IEnumerable<Product>>> GetAll()
    {
        var products = await _repository.GetAllAsync();
        return Ok(products);
    }

    // GET api/productsapi/5
    [HttpGet("{id:int}")]
    public async Task<ActionResult<Product>> GetById(int id)
    {
        var product = await _repository.GetByIdAsync(id);
        if (product == null)
            return NotFound(new { message = $"Product {id} not found" });

        return Ok(product);
    }

    // POST api/productsapi
    [HttpPost]
    public async Task<ActionResult<Product>> Create(Product product)
    {
        await _repository.AddAsync(product);
        return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
    }

    // PUT api/productsapi/5
    [HttpPut("{id:int}")]
    public async Task<IActionResult> Update(int id, Product product)
    {
        if (id != product.Id)
            return BadRequest();

        var existingProduct = await _repository.GetByIdAsync(id);
        if (existingProduct == null)
            return NotFound();

        await _repository.UpdateAsync(product);
        return NoContent();
    }

    // DELETE api/productsapi/5
    [HttpDelete("{id:int}")]
    public async Task<IActionResult> Delete(int id)
    {
        var product = await _repository.GetByIdAsync(id);
        if (product == null)
            return NotFound();

        await _repository.DeleteAsync(id);
        return NoContent();
    }
}
```

### Returning Problem Details (RFC 7807)
```csharp
// Return a standardized error response
return Problem(
    title: "Validation failed",
    detail: "One or more fields are invalid.",
    statusCode: StatusCodes.Status400BadRequest);
```

### Configuring JSON Serialization
```csharp
// In Program.cs
builder.Services.AddControllers()
    .AddJsonOptions(options =>
    {
        options.JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
        options.JsonSerializerOptions.WriteIndented = true;
        options.JsonSerializerOptions.DefaultIgnoreCondition =
            JsonIgnoreCondition.WhenWritingNull;
    });
```

## Middleware

Middleware are components that form the HTTP request pipeline. Each middleware can run code before and after calling the next component.

### Built-in Middleware Order (Program.cs)
```csharp
var app = builder.Build();

// 1. Exception handling (must be first to catch errors in subsequent middleware)
if (app.Environment.IsDevelopment())
    app.UseDeveloperExceptionPage();
else
    app.UseExceptionHandler("/Home/Error");

// 2. HSTS (HTTP Strict Transport Security)
app.UseHsts();

// 3. HTTPS redirection
app.UseHttpsRedirection();

// 4. Static files (serves wwwroot)
app.UseStaticFiles();

// 5. Routing
app.UseRouting();

// 6. Authentication (must come before Authorization)
app.UseAuthentication();

// 7. Authorization
app.UseAuthorization();

// 8. Map endpoints (controllers, Razor Pages, etc.)
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

### Custom Middleware
```csharp
// Inline middleware with app.Use
app.Use(async (context, next) =>
{
    // Before the next middleware
    Console.WriteLine($"Request: {context.Request.Method} {context.Request.Path}");

    await next(context);

    // After the next middleware
    Console.WriteLine($"Response: {context.Response.StatusCode}");
});

// Short-circuit middleware with app.Run (does not call next)
app.Run(async context =>
{
    await context.Response.WriteAsync("Request terminated here.");
});
```

### Class-Based Middleware
```csharp
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;

    public RequestLoggingMiddleware(RequestDelegate next, ILogger<RequestLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var start = DateTime.UtcNow;

        await _next(context);

        var elapsed = DateTime.UtcNow - start;
        _logger.LogInformation(
            "{Method} {Path} responded {StatusCode} in {Elapsed}ms",
            context.Request.Method,
            context.Request.Path,
            context.Response.StatusCode,
            elapsed.TotalMilliseconds);
    }
}

// Extension method for clean registration
public static class RequestLoggingMiddlewareExtensions
{
    public static IApplicationBuilder UseRequestLogging(this IApplicationBuilder app)
        => app.UseMiddleware<RequestLoggingMiddleware>();
}

// Register in Program.cs
app.UseRequestLogging();
```

## Filters

Filters allow you to run code before or after specific stages in the request pipeline, at the action level rather than middleware level.

### Action Filter
```csharp
using Microsoft.AspNetCore.Mvc.Filters;

public class LogActionFilter : IActionFilter
{
    private readonly ILogger<LogActionFilter> _logger;

    public LogActionFilter(ILogger<LogActionFilter> logger)
    {
        _logger = logger;
    }

    // Runs before the action method
    public void OnActionExecuting(ActionExecutingContext context)
    {
        _logger.LogInformation("Executing action: {Action}", context.ActionDescriptor.DisplayName);
    }

    // Runs after the action method
    public void OnActionExecuted(ActionExecutedContext context)
    {
        _logger.LogInformation("Executed action: {Action}", context.ActionDescriptor.DisplayName);
    }
}
```

### Exception Filter
```csharp
public class GlobalExceptionFilter : IExceptionFilter
{
    private readonly ILogger<GlobalExceptionFilter> _logger;

    public GlobalExceptionFilter(ILogger<GlobalExceptionFilter> logger)
    {
        _logger = logger;
    }

    public void OnException(ExceptionContext context)
    {
        _logger.LogError(context.Exception, "Unhandled exception");

        context.Result = new ObjectResult(new { error = "An unexpected error occurred." })
        {
            StatusCode = StatusCodes.Status500InternalServerError
        };
        context.ExceptionHandled = true;
    }
}
```

### Result Filter
```csharp
public class AddHeaderFilter : IResultFilter
{
    public void OnResultExecuting(ResultExecutingContext context)
    {
        context.HttpContext.Response.Headers["X-App-Version"] = "1.0.0";
    }

    public void OnResultExecuted(ResultExecutedContext context) { }
}
```

### Registering Filters Globally
```csharp
// In Program.cs - apply to all controllers
builder.Services.AddControllersWithViews(options =>
{
    options.Filters.Add<LogActionFilter>();
    options.Filters.Add<GlobalExceptionFilter>();
});
```

### Applying Filters to a Controller or Action
```csharp
// As an attribute on a controller
[ServiceFilter(typeof(LogActionFilter))]
public class ProductsController : Controller { }

// As an attribute on an action
[ServiceFilter(typeof(LogActionFilter))]
public IActionResult Index() => View();
```

### Authorization Filter (Custom Policy)
```csharp
// Define policy
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOnly", policy =>
        policy.RequireRole("Admin"));

    options.AddPolicy("MinAge18", policy =>
        policy.RequireClaim("DateOfBirth")
              .AddRequirements(new MinimumAgeRequirement(18)));
});

// Apply to a controller or action
[Authorize(Policy = "AdminOnly")]
public class AdminController : Controller { }
```

## Caching

### Response Caching
```csharp
// Register in Program.cs
builder.Services.AddResponseCaching();
app.UseResponseCaching();

// Apply to action
[ResponseCache(Duration = 60, Location = ResponseCacheLocation.Client)]
public IActionResult Index()
{
    return View();
}

// Disable caching
[ResponseCache(NoStore = true, Location = ResponseCacheLocation.None)]
public IActionResult Dashboard()
{
    return View();
}
```

### In-Memory Caching
```csharp
// Register
builder.Services.AddMemoryCache();

// Inject and use
public class ProductService
{
    private readonly IMemoryCache _cache;
    private readonly AppDbContext _context;

    public ProductService(IMemoryCache cache, AppDbContext context)
    {
        _cache = cache;
        _context = context;
    }

    public async Task<List<Product>> GetFeaturedAsync()
    {
        const string cacheKey = "featured_products";

        if (!_cache.TryGetValue(cacheKey, out List<Product>? products))
        {
            products = await _context.Products
                .Where(p => p.IsFeatured)
                .ToListAsync();

            var options = new MemoryCacheEntryOptions()
                .SetSlidingExpiration(TimeSpan.FromMinutes(5))
                .SetAbsoluteExpiration(TimeSpan.FromHours(1));

            _cache.Set(cacheKey, products, options);
        }

        return products!;
    }
}
```

## Logging

### Built-in Logging
```csharp
public class HomeController : Controller
{
    private readonly ILogger<HomeController> _logger;

    public HomeController(ILogger<HomeController> logger)
    {
        _logger = logger;
    }

    public IActionResult Index()
    {
        _logger.LogDebug("Debug: entering Index action");
        _logger.LogInformation("Information: serving home page");
        _logger.LogWarning("Warning: something to watch");
        _logger.LogError("Error: something went wrong");
        _logger.LogCritical("Critical: system failure");

        // Structured logging with parameters
        _logger.LogInformation("User {UserId} visited {Page} at {Time}",
            User.Identity?.Name, "Home/Index", DateTime.UtcNow);

        return View();
    }
}
```

### Configuring Log Levels in appsettings.json
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning",
      "MyMvcApp.Controllers": "Debug"
    }
  }
}
```

## Testing

### Unit Testing Controllers
```bash
dotnet add package xunit
dotnet add package Moq
dotnet add package Microsoft.AspNetCore.Mvc.Testing
```

```csharp
using Xunit;
using Moq;
using Microsoft.AspNetCore.Mvc;

public class ProductsControllerTests
{
    private readonly Mock<IProductRepository> _mockRepo;
    private readonly ProductsController _controller;

    public ProductsControllerTests()
    {
        _mockRepo = new Mock<IProductRepository>();
        _controller = new ProductsController(_mockRepo.Object);
    }

    [Fact]
    public async Task Index_ReturnsViewWithProducts()
    {
        // Arrange
        var products = new List<Product>
        {
            new Product { Id = 1, Name = "Laptop", Price = 999 },
            new Product { Id = 2, Name = "Mouse", Price = 29 }
        };
        _mockRepo.Setup(r => r.GetAllAsync()).ReturnsAsync(products);

        // Act
        var result = await _controller.Index();

        // Assert
        var viewResult = Assert.IsType<ViewResult>(result);
        var model = Assert.IsAssignableFrom<IEnumerable<Product>>(viewResult.Model);
        Assert.Equal(2, model.Count());
    }

    [Fact]
    public async Task Details_ReturnsNotFound_WhenProductDoesNotExist()
    {
        // Arrange
        _mockRepo.Setup(r => r.GetByIdAsync(It.IsAny<int>())).ReturnsAsync((Product?)null);

        // Act
        var result = await _controller.Details(99);

        // Assert
        Assert.IsType<NotFoundResult>(result);
    }

    [Fact]
    public async Task Create_Post_RedirectsToIndex_WhenModelIsValid()
    {
        // Arrange
        var product = new Product { Name = "Laptop", Price = 999 };

        // Act
        var result = await _controller.Create(product);

        // Assert
        var redirect = Assert.IsType<RedirectToActionResult>(result);
        Assert.Equal("Index", redirect.ActionName);
        _mockRepo.Verify(r => r.AddAsync(product), Times.Once);
    }
}
```

### Integration Testing with WebApplicationFactory
```csharp
// Note: In .NET 6+ with the minimal hosting model, Program.cs must be accessible to the test project.
// Add the following to the bottom of Program.cs in your MVC project:
//   public partial class Program { }
// Or add [assembly: InternalsVisibleTo("MyMvcApp.Tests")] to expose internals.

using Microsoft.AspNetCore.Mvc.Testing;

public class HomeControllerIntegrationTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;

    public HomeControllerIntegrationTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task Get_HomeIndex_ReturnsSuccessAndCorrectContentType()
    {
        // Act
        var response = await _client.GetAsync("/");

        // Assert
        response.EnsureSuccessStatusCode();
        Assert.Equal("text/html; charset=utf-8",
            response.Content.Headers.ContentType?.ToString());
    }

    [Fact]
    public async Task Get_NonExistentRoute_Returns404()
    {
        var response = await _client.GetAsync("/this-does-not-exist");
        Assert.Equal(System.Net.HttpStatusCode.NotFound, response.StatusCode);
    }
}
```

## Performance Optimization

### Async Throughout
```csharp
// Always use async for I/O-bound work
public async Task<IActionResult> Index()
{
    var products = await _context.Products.ToListAsync();
    return View(products);
}
```

### Projection to Avoid Fetching Unnecessary Data
```csharp
// Only select columns you need
var summaries = await _context.Products
    .Where(p => p.IsAvailable)
    .Select(p => new ProductSummaryDto
    {
        Id = p.Id,
        Name = p.Name,
        Price = p.Price
    })
    .ToListAsync();
```

### Compiled Queries
```csharp
private static readonly Func<AppDbContext, int, Task<Product?>> GetByIdCompiled =
    EF.CompileAsyncQuery((AppDbContext ctx, int id) =>
        ctx.Products.FirstOrDefault(p => p.Id == id));

// Usage
var product = await GetByIdCompiled(_context, id);
```

### Output Caching (.NET 7+)
```csharp
// Register
builder.Services.AddOutputCache();
app.UseOutputCache();

// Apply to action
[OutputCache(Duration = 60)]
public async Task<IActionResult> Index()
{
    var products = await _context.Products.ToListAsync();
    return View(products);
}
```

## Security Best Practices

### Anti-Forgery Tokens (CSRF Protection)
```csharp
// Automatically validated when using Tag Helpers in forms
// Manually add [ValidateAntiForgeryToken] to POST actions
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Delete(int id) { }

// Global validation for all POST actions
builder.Services.AddControllersWithViews(options =>
{
    options.Filters.Add(new AutoValidateAntiforgeryTokenAttribute());
});
```

### Content Security Policy (CSP)
```csharp
// Requires: using System.Security.Cryptography;
// Note: for script-src nonces, generate a unique random value per request
// and include the same nonce in the script tags of your views.
app.Use(async (context, next) =>
{
    // Generate a unique nonce per request for inline scripts
    var nonce = Convert.ToBase64String(RandomNumberGenerator.GetBytes(16));
    context.Items["CSPNonce"] = nonce;
    context.Response.Headers["Content-Security-Policy"] =
        $"default-src 'self'; script-src 'self' 'nonce-{nonce}'; style-src 'self';";
    await next(context);
});
```

### Preventing XSS in Razor Views
```html
@* Razor automatically HTML-encodes output - safe by default *@
<p>@Model.UserInput</p>

@* Explicitly render raw HTML only when you trust the source *@
<p>@Html.Raw(Model.TrustedHtmlContent)</p>
```

### Secure Headers
```csharp
app.UseHsts();  // HTTP Strict Transport Security
app.UseHttpsRedirection();

app.Use(async (context, next) =>
{
    context.Response.Headers["X-Content-Type-Options"] = "nosniff";
    context.Response.Headers["X-Frame-Options"] = "DENY";
    context.Response.Headers["Referrer-Policy"] = "no-referrer";
    await next();
});
```

### Secrets Management
```bash
# Development: use User Secrets (not committed to source control)
dotnet user-secrets init
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "your-connection-string"
dotnet user-secrets set "ApiKeys:Stripe" "sk_test_..."

# Production: use environment variables or a secrets vault (e.g., Azure Key Vault)
```

## Do's and Don'ts

### ✅ Do's

✅ **Use `[ApiController]` for API controllers to get automatic model validation and binding**
```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsApiController : ControllerBase { }
```

✅ **Register middleware in the correct order**
```csharp
// Order matters: exception handling first, routing before authorization
app.UseExceptionHandler("/Home/Error");
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllerRoute(...);
```

✅ **Write unit tests using mock repositories, not real databases**
```csharp
var mockRepo = new Mock<IProductRepository>();
mockRepo.Setup(r => r.GetAllAsync()).ReturnsAsync(fakeProducts);
```

✅ **Use structured logging with parameters, not string concatenation**
```csharp
// Good: structured, searchable
_logger.LogInformation("Order {OrderId} placed by {UserId}", orderId, userId);

// Bad: just a string
_logger.LogInformation("Order " + orderId + " placed by " + userId);
```

✅ **Apply output caching or response caching for read-heavy pages**
```csharp
[OutputCache(Duration = 60)]
public async Task<IActionResult> Index() { }
```

### ❌ Don'ts

❌ **Don't use `[FromBody]` on MVC view actions – that's for API controllers**
```csharp
// Bad in an MVC controller
public IActionResult Create([FromBody] Product product) { }

// Good in an MVC controller (form-posted data)
public IActionResult Create(Product product) { }

// [FromBody] is correct in ApiController
[ApiController]
public class ProductsApiController : ControllerBase
{
    [HttpPost]
    public IActionResult Create([FromBody] Product product) { }
}
```

❌ **Don't trust user-supplied data without validation and sanitization**
```csharp
// Bad: executes raw user input as SQL (SQL injection risk)
var sql = $"SELECT * FROM Products WHERE Name = '{userInput}'";

// Good: use parameterized queries via EF Core
var products = await _context.Products
    .Where(p => p.Name == userInput)
    .ToListAsync();
```

❌ **Don't return sensitive information in error responses**
```csharp
// Bad: reveals internal details
catch (Exception ex)
{
    return StatusCode(500, ex.ToString());
}

// Good: log internally, return generic message
catch (Exception ex)
{
    _logger.LogError(ex, "Unexpected error in {Action}", nameof(Create));
    return StatusCode(500, "An unexpected error occurred.");
}
```

❌ **Don't skip authentication/authorization checks on sensitive actions**
```csharp
// Bad: no protection
public IActionResult DeleteUser(int id) { }

// Good: require authorization
[Authorize(Roles = "Admin")]
[HttpPost, ValidateAntiForgeryToken]
public async Task<IActionResult> DeleteUser(int id) { }
```

❌ **Don't block async code with .Result or .Wait()**
```csharp
// Bad: can cause deadlocks
var product = _repository.GetByIdAsync(id).Result;

// Good
var product = await _repository.GetByIdAsync(id);
```

---

## Security Summary

The following security topics are covered in this cheatsheet:

| Topic | Where Covered |
|---|---|
| CSRF (Cross-Site Request Forgery) | Anti-Forgery Tokens section |
| XSS (Cross-Site Scripting) | Razor auto-encoding; avoid `Html.Raw` |
| SQL Injection | Use EF Core parameterized queries |
| Sensitive data exposure | Secrets Management; avoid returning stack traces |
| Insecure headers | Secure Headers section |
| Broken authentication | Authentication & Authorization section |
| Over-posting | Use `[Bind]` or ViewModels |

---

Happy coding with ASP.NET Core MVC! 🚀
