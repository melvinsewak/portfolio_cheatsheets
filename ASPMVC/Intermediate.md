# ASP.NET MVC Intermediate Cheatsheet

## Entity Framework Core

### Installing EF Core
```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package Microsoft.EntityFrameworkCore.Design
```

### Defining a DbContext
```csharp
using Microsoft.EntityFrameworkCore;

namespace MyMvcApp.Data
{
    public class AppDbContext : DbContext
    {
        public AppDbContext(DbContextOptions<AppDbContext> options)
            : base(options) { }

        public DbSet<Product> Products { get; set; }
        public DbSet<Category> Categories { get; set; }
        public DbSet<Order> Orders { get; set; }

        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            // Seed data
            modelBuilder.Entity<Category>().HasData(
                new Category { Id = 1, Name = "Electronics" },
                new Category { Id = 2, Name = "Clothing" }
            );

            // Configure relationships
            modelBuilder.Entity<Product>()
                .HasOne(p => p.Category)
                .WithMany(c => c.Products)
                .HasForeignKey(p => p.CategoryId);
        }
    }
}
```

### Registering DbContext in Program.cs
```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
```

### Connection String in appsettings.json
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=MyMvcApp;Trusted_Connection=True;"
  }
}
```

### Migrations
```bash
# Add a new migration
dotnet ef migrations add InitialCreate

# Apply pending migrations
dotnet ef database update

# Remove the last migration (if not yet applied)
dotnet ef migrations remove

# Generate SQL script for a migration
dotnet ef migrations script
```

### CRUD with EF Core (Async)
```csharp
public class ProductRepository
{
    private readonly AppDbContext _context;

    public ProductRepository(AppDbContext context)
    {
        _context = context;
    }

    // Read all
    public async Task<List<Product>> GetAllAsync()
    {
        return await _context.Products
            .Include(p => p.Category)
            .OrderBy(p => p.Name)
            .ToListAsync();
    }

    // Read one
    public async Task<Product?> GetByIdAsync(int id)
    {
        return await _context.Products
            .Include(p => p.Category)
            .FirstOrDefaultAsync(p => p.Id == id);
    }

    // Create
    public async Task AddAsync(Product product)
    {
        _context.Products.Add(product);
        await _context.SaveChangesAsync();
    }

    // Update
    public async Task UpdateAsync(Product product)
    {
        _context.Products.Update(product);
        await _context.SaveChangesAsync();
    }

    // Delete
    public async Task DeleteAsync(int id)
    {
        var product = await _context.Products.FindAsync(id);
        if (product != null)
        {
            _context.Products.Remove(product);
            await _context.SaveChangesAsync();
        }
    }
}
```

### LINQ Queries with EF Core
```csharp
// Filter
var affordable = await _context.Products
    .Where(p => p.Price < 100)
    .ToListAsync();

// Order
var sorted = await _context.Products
    .OrderByDescending(p => p.Price)
    .ToListAsync();

// Select (projection)
var names = await _context.Products
    .Select(p => new { p.Id, p.Name })
    .ToListAsync();

// First or default
var product = await _context.Products
    .FirstOrDefaultAsync(p => p.Name == "Laptop");

// Count
int count = await _context.Products.CountAsync();

// Any
bool exists = await _context.Products.AnyAsync(p => p.Price > 500);

// Pagination
int page = 1;
int pageSize = 10;
var paged = await _context.Products
    .Skip((page - 1) * pageSize)
    .Take(pageSize)
    .ToListAsync();
```

## ViewModels

ViewModels are classes specifically designed to pass data between a controller and a view, as opposed to domain models used for database storage.

### Defining a ViewModel
```csharp
namespace MyMvcApp.ViewModels
{
    public class ProductListViewModel
    {
        public IEnumerable<Product> Products { get; set; } = new List<Product>();
        public string? SearchQuery { get; set; }
        public int CurrentPage { get; set; }
        public int TotalPages { get; set; }
        public List<SelectListItem> Categories { get; set; } = new();
    }

    public class ProductCreateViewModel
    {
        [Required]
        [StringLength(100)]
        public string Name { get; set; } = string.Empty;

        [Required]
        [Range(0.01, 10000)]
        public decimal Price { get; set; }

        [StringLength(500)]
        public string? Description { get; set; }

        [Display(Name = "Category")]
        public int CategoryId { get; set; }

        public List<SelectListItem> Categories { get; set; } = new();
    }
}
```

### Using a ViewModel in a Controller
```csharp
public class ProductsController : Controller
{
    private readonly AppDbContext _context;

    public ProductsController(AppDbContext context)
    {
        _context = context;
    }

    public async Task<IActionResult> Index(string? search)
    {
        var query = _context.Products.AsQueryable();

        if (!string.IsNullOrEmpty(search))
            query = query.Where(p => p.Name.Contains(search));

        var viewModel = new ProductListViewModel
        {
            Products = await query.ToListAsync(),
            SearchQuery = search
        };

        return View(viewModel);
    }

    [HttpGet]
    public async Task<IActionResult> Create()
    {
        var viewModel = new ProductCreateViewModel
        {
            Categories = await _context.Categories
                .Select(c => new SelectListItem { Value = c.Id.ToString(), Text = c.Name })
                .ToListAsync()
        };
        return View(viewModel);
    }
}
```

## Data Validation

### Server-Side Validation with Data Annotations
```csharp
using System.ComponentModel.DataAnnotations;

public class RegisterViewModel
{
    [Required(ErrorMessage = "Username is required")]
    [StringLength(50, MinimumLength = 3)]
    public string Username { get; set; } = string.Empty;

    [Required]
    [EmailAddress(ErrorMessage = "Invalid email address")]
    public string Email { get; set; } = string.Empty;

    [Required]
    [DataType(DataType.Password)]
    [MinLength(8, ErrorMessage = "Password must be at least 8 characters")]
    public string Password { get; set; } = string.Empty;

    [Required]
    [DataType(DataType.Password)]
    [Compare("Password", ErrorMessage = "Passwords do not match")]
    [Display(Name = "Confirm Password")]
    public string ConfirmPassword { get; set; } = string.Empty;

    [Range(18, 120, ErrorMessage = "Age must be between 18 and 120")]
    public int Age { get; set; }

    [Url(ErrorMessage = "Invalid URL format")]
    public string? Website { get; set; }

    [Phone]
    public string? PhoneNumber { get; set; }
}
```

### Custom Validation Attribute
```csharp
public class FutureDateAttribute : ValidationAttribute
{
    protected override ValidationResult? IsValid(object? value, ValidationContext context)
    {
        if (value is DateTime date && date <= DateTime.Now)
        {
            return new ValidationResult("Date must be in the future.");
        }
        return ValidationResult.Success;
    }
}

// Usage
public class EventViewModel
{
    [Required]
    public string Title { get; set; } = string.Empty;

    [FutureDate]
    public DateTime EventDate { get; set; }
}
```

### Handling Validation in the Controller
```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Create(ProductCreateViewModel viewModel)
{
    if (!ModelState.IsValid)
    {
        // Re-populate dropdown before returning the view
        viewModel.Categories = await GetCategorySelectListAsync();
        return View(viewModel);
    }

    var product = new Product
    {
        Name = viewModel.Name,
        Price = viewModel.Price,
        Description = viewModel.Description,
        CategoryId = viewModel.CategoryId
    };

    _context.Products.Add(product);
    await _context.SaveChangesAsync();

    TempData["Success"] = "Product created successfully!";
    return RedirectToAction("Index");
}

// Manually add a validation error
[HttpPost]
public IActionResult Login(LoginViewModel model)
{
    if (ModelState.IsValid)
    {
        bool valid = ValidateCredentials(model.Username, model.Password);
        if (!valid)
        {
            ModelState.AddModelError(string.Empty, "Invalid username or password.");
            return View(model);
        }
        return RedirectToAction("Index", "Home");
    }
    return View(model);
}
```

### Client-Side Validation
```html
@* In _Layout.cshtml or the view *@
@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}
```

```html
@* _ValidationScriptsPartial.cshtml - already included in new MVC projects *@
<script src="~/lib/jquery-validation/dist/jquery.validate.min.js"></script>
<script src="~/lib/jquery-validation-unobtrusive/jquery.validate.unobtrusive.min.js"></script>
```

## Dependency Injection

### Registering Services
```csharp
// In Program.cs

// Singleton: one instance for the entire application lifetime
builder.Services.AddSingleton<IEmailService, SmtpEmailService>();

// Scoped: one instance per HTTP request
builder.Services.AddScoped<IProductRepository, ProductRepository>();
builder.Services.AddScoped<IOrderService, OrderService>();

// Transient: a new instance every time it is requested
builder.Services.AddTransient<INotificationService, NotificationService>();
```

### Injecting Services into Controllers
```csharp
public class ProductsController : Controller
{
    private readonly IProductRepository _repository;
    private readonly ILogger<ProductsController> _logger;

    public ProductsController(IProductRepository repository, ILogger<ProductsController> logger)
    {
        _repository = repository;
        _logger = logger;
    }

    public async Task<IActionResult> Index()
    {
        _logger.LogInformation("Fetching all products");
        var products = await _repository.GetAllAsync();
        return View(products);
    }
}
```

### Service and Interface Pattern
```csharp
// Interface
public interface IProductRepository
{
    Task<IEnumerable<Product>> GetAllAsync();
    Task<Product?> GetByIdAsync(int id);
    Task AddAsync(Product product);
    Task UpdateAsync(Product product);
    Task DeleteAsync(int id);
}

// Implementation
public class ProductRepository : IProductRepository
{
    private readonly AppDbContext _context;

    public ProductRepository(AppDbContext context)
    {
        _context = context;
    }

    public async Task<IEnumerable<Product>> GetAllAsync()
        => await _context.Products.ToListAsync();

    public async Task<Product?> GetByIdAsync(int id)
        => await _context.Products.FindAsync(id);

    public async Task AddAsync(Product product)
    {
        _context.Products.Add(product);
        await _context.SaveChangesAsync();
    }

    public async Task UpdateAsync(Product product)
    {
        _context.Products.Update(product);
        await _context.SaveChangesAsync();
    }

    public async Task DeleteAsync(int id)
    {
        var product = await _context.Products.FindAsync(id);
        if (product != null)
        {
            _context.Products.Remove(product);
            await _context.SaveChangesAsync();
        }
    }
}
```

## Authentication and Authorization

### Setting Up Identity
```bash
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
dotnet add package Microsoft.AspNetCore.Identity.UI
```

```csharp
// In Program.cs
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));

builder.Services.AddDefaultIdentity<IdentityUser>(options =>
{
    options.SignIn.RequireConfirmedAccount = false;
    options.Password.RequireDigit = true;
    options.Password.RequiredLength = 8;
    options.Password.RequireUppercase = true;
})
.AddRoles<IdentityRole>()
.AddEntityFrameworkStores<AppDbContext>();

// ...
app.UseAuthentication();
app.UseAuthorization();
```

### Protecting Actions and Controllers
```csharp
using Microsoft.AspNetCore.Authorization;

// Require authentication for entire controller
[Authorize]
public class DashboardController : Controller
{
    public IActionResult Index() => View();
}

// Require a specific role
[Authorize(Roles = "Admin")]
public class AdminController : Controller
{
    public IActionResult Index() => View();
}

// Allow anonymous access to a specific action in an [Authorize] controller
public class AccountController : Controller
{
    [AllowAnonymous]
    public IActionResult Login() => View();

    [Authorize]
    public IActionResult Profile() => View();
}

// Require multiple roles (user must have at least one)
[Authorize(Roles = "Admin,Manager")]
public IActionResult Reports() => View();
```

### Accessing the Current User
```csharp
using System.Security.Claims;

public class ProfileController : Controller
{
    public IActionResult Index()
    {
        string? userId = User.FindFirstValue(ClaimTypes.NameIdentifier);
        string? email = User.FindFirstValue(ClaimTypes.Email);
        bool isAdmin = User.IsInRole("Admin");

        return View();
    }
}
```

## Configuration and appsettings.json

### appsettings.json
```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=MyMvcApp;Trusted_Connection=True;"
  },
  "AppSettings": {
    "SiteName": "My MVC App",
    "PageSize": 10,
    "AllowRegistration": true
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  }
}
```

### Accessing Configuration
```csharp
// In Program.cs
var siteName = builder.Configuration["AppSettings:SiteName"];

// Using Options pattern (recommended)
builder.Services.Configure<AppSettings>(
    builder.Configuration.GetSection("AppSettings"));
```

```csharp
// AppSettings.cs
public class AppSettings
{
    public string SiteName { get; set; } = string.Empty;
    public int PageSize { get; set; }
    public bool AllowRegistration { get; set; }
}

// Injecting in a controller
public class HomeController : Controller
{
    private readonly AppSettings _settings;

    public HomeController(IOptions<AppSettings> options)
    {
        _settings = options.Value;
    }

    public IActionResult Index()
    {
        ViewBag.SiteName = _settings.SiteName;
        return View();
    }
}
```

## Areas

Areas help organize a large MVC application into smaller functional groups.

### Creating an Area
```
MyMvcApp/
├── Areas/
│   └── Admin/
│       ├── Controllers/
│       │   └── DashboardController.cs
│       └── Views/
│           └── Dashboard/
│               └── Index.cshtml
```

```csharp
// Areas/Admin/Controllers/DashboardController.cs
[Area("Admin")]
[Authorize(Roles = "Admin")]
public class DashboardController : Controller
{
    public IActionResult Index() => View();
}
```

```csharp
// Register area route in Program.cs
app.MapControllerRoute(
    name: "areas",
    pattern: "{area:exists}/{controller=Dashboard}/{action=Index}/{id?}");
```

## Do's and Don'ts

### ✅ Do's

✅ **Use async/await for all database operations**
```csharp
// Good
public async Task<IActionResult> Index()
{
    var products = await _context.Products.ToListAsync();
    return View(products);
}
```

✅ **Use ViewModels instead of passing domain models directly to views**
```csharp
// Good: ViewModel separates database model from view concerns
var viewModel = new ProductDetailsViewModel
{
    Product = product,
    RelatedProducts = relatedProducts,
    UserCanEdit = User.IsInRole("Admin")
};
return View(viewModel);
```

✅ **Use the repository/service pattern to keep controllers thin**
```csharp
// Good: controller only orchestrates
public async Task<IActionResult> Index()
{
    var products = await _productService.GetFeaturedAsync();
    return View(products);
}
```

✅ **Apply [ValidateAntiForgeryToken] to all POST, PUT, DELETE actions**
```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Delete(int id) { }
```

### ❌ Don'ts

❌ **Don't expose domain models with sensitive fields directly in forms**
```csharp
// Bad: exposes IsAdmin, PasswordHash, etc. to binding
public IActionResult Edit(User user) { }

// Good: use a ViewModel with only the fields you need
public IActionResult Edit(UserEditViewModel model) { }
```

❌ **Don't catch generic exceptions and swallow errors silently**
```csharp
// Bad
try { await _context.SaveChangesAsync(); }
catch (Exception) { }  // Never do this

// Good
try { await _context.SaveChangesAsync(); }
catch (DbUpdateException ex)
{
    _logger.LogError(ex, "Error saving product");
    ModelState.AddModelError(string.Empty, "Could not save. Please try again.");
    return View(model);
}
```

❌ **Don't use synchronous EF Core calls in an async context**
```csharp
// Bad
var products = _context.Products.ToList();  // Blocks thread

// Good
var products = await _context.Products.ToListAsync();
```

❌ **Don't store secrets in appsettings.json committed to source control**
```bash
# Good: use User Secrets in development
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "your-connection-string"

# Good: use environment variables or Azure Key Vault in production
```

---

Ready for more? Continue to [Advanced.md](Advanced.md)! 🚀
