# ASP.NET Core MVC with C# — Zero to Expert in 7 Classes

> **Audience:** Beginners with basic programming knowledge → working developers preparing for senior roles
> **Stack:** C# 12+, .NET 8 / .NET 9 / .NET 10 (LTS), ASP.NET Core MVC, Entity Framework Core, SQL Server / SQLite
> **Format:** Each class has several topics. Every topic ends with **🎯 Interview Questions** (with model answers) and **🧩 Scenario-Based Questions** (with a suggested approach).

---

## 📚 Course Map

| Class | Title | Level | Key Outcomes |
|---|---|---|---|
| 1 | Foundations: .NET, MVC Pattern & Your First App | Beginner | Understand the platform, create and run an MVC app, read `Program.cs` |
| 2 | Routing, Controllers, Actions & Model Binding | Beginner → Intermediate | Map URLs to code, return the right results, bind input safely |
| 3 | Views, Razor, Layouts, Tag Helpers & View Components | Intermediate | Build reusable, strongly typed UI |
| 4 | Models, Validation & Entity Framework Core | Intermediate | Persist data, validate input, write efficient queries |
| 5 | Dependency Injection, Middleware, Configuration, Logging & Errors | Intermediate → Advanced | Understand the framework internals that every app relies on |
| 6 | Security: Identity, Authentication, Authorization & Threats | Advanced | Secure an app end to end |
| 7 | Expert Topics: Filters, Caching, Performance, APIs, Testing, Architecture & Deployment | Expert | Build, test, scale and ship production-grade apps |

**Prerequisites:** Install the latest LTS .NET SDK, Visual Studio 2022+/VS Code with C# Dev Kit/JetBrains Rider, and SQL Server LocalDB or SQLite.

**Running project across the course:** We build **"BookStore"** — a catalog with books, authors, categories, orders and admin features. Each class adds to it.

---
---

# 🟢 CLASS 1 — Foundations: .NET, MVC Pattern & Your First App

**Goal:** Understand what ASP.NET Core is, how MVC works, and how a request travels through an app.

---

## 1.1 The .NET Ecosystem & ASP.NET Core

### Concepts
- **.NET** is a free, open-source, cross-platform developer platform (Windows, Linux, macOS).
- **.NET Framework** (Windows-only, up to 4.8.x) is the legacy platform. **.NET (Core)** is the modern one (versions 5, 6, 7, 8, 9, 10...).
- **LTS vs STS:** Even-numbered releases (.NET 8, .NET 10) are Long-Term Support (3 years). Odd-numbered (.NET 9) are Standard-Term Support.
- **CLR (Common Language Runtime):** Executes code, manages memory (Garbage Collector), handles JIT compilation.
- **IL (Intermediate Language):** C# compiles to IL; the JIT compiles IL to native code at runtime. Native AOT compiles ahead of time.
- **ASP.NET Core** is the web framework on top of .NET. It supports several programming models:
  - **MVC** — Controllers + Views (server-rendered HTML)
  - **Razor Pages** — Page-focused model
  - **Web API** — Controllers returning JSON
  - **Minimal APIs** — Lambda-based endpoints
  - **Blazor** — Component-based UI with C#
- **Kestrel** is the built-in cross-platform web server. In production it can run alone or behind a reverse proxy (IIS, Nginx, Apache, YARP).

### C# features you will use constantly
```csharp
// Records — immutable data types, great for DTOs
public record BookDto(int Id, string Title, decimal Price);

// Nullable reference types
string? subtitle = null;

// Pattern matching
string Describe(object o) => o switch
{
    int n when n > 100 => "Large number",
    int => "Number",
    string s => $"Text of length {s.Length}",
    null => "Nothing",
    _ => "Unknown"
};

// async/await
public async Task<string> GetPageAsync(HttpClient client) =>
    await client.GetStringAsync("https://example.com");

// LINQ
var cheapBooks = books.Where(b => b.Price < 20).OrderBy(b => b.Title).ToList();

// Primary constructors (C# 12)
public class BookService(IBookRepository repo)
{
    public Task<Book?> GetAsync(int id) => repo.GetByIdAsync(id);
}
```

### 🎯 Interview Questions
1. **What is the difference between .NET Framework and .NET (Core)?**
   .NET Framework is Windows-only, mature, and in maintenance mode. .NET is cross-platform, open source, faster, modular, supports containers and side-by-side installs, and is where all new development happens.
2. **What is Kestrel?**
   The default, cross-platform, high-performance web server built into ASP.NET Core. It can face the internet directly or sit behind a reverse proxy.
3. **What is the role of the CLR?**
   It loads assemblies, JIT-compiles IL to native code, manages memory through the GC, enforces type safety, and handles exceptions and threading.
4. **What is the difference between LTS and STS releases?**
   LTS releases get 3 years of support and suit enterprises. STS releases get a shorter support window (now 2 years) and deliver features sooner.
5. **What is the difference between a `record` and a `class`?**
   Records have value-based equality, built-in `ToString`, `with`-expressions for non-destructive mutation, and concise positional syntax. Classes have reference equality by default.

### 🧩 Scenario-Based Questions
1. **Your company has a 10-year-old ASP.NET MVC 5 app on .NET Framework 4.8 and wants to host on Linux containers. What do you recommend?**
   Plan a migration to ASP.NET Core. Use the .NET Upgrade Assistant for analysis, migrate class libraries to .NET Standard/.NET first, then move the web project incrementally (YARP-based "strangler fig" pattern lets old and new apps share routes during migration). Replace `System.Web` dependencies, `Global.asax`, and `web.config` settings with middleware and `appsettings.json`.
2. **A manager asks whether to start a new project on .NET 9 or .NET 10.**
   Prefer the current LTS (.NET 10) for long-lived business apps. Choose STS only if a specific new feature is needed and the team can upgrade regularly.

---

## 1.2 The MVC Pattern

### Concepts
**MVC = Model–View–Controller**, a pattern that separates concerns:

| Part | Responsibility | In ASP.NET Core |
|---|---|---|
| **Model** | Data and business rules | C# classes, ViewModels, EF Core entities, services |
| **View** | Presentation (HTML) | `.cshtml` Razor files |
| **Controller** | Handles the request, calls the model, picks a view | Classes deriving from `Controller` |

**Request flow:**
```
Browser → Kestrel → Middleware pipeline → Routing → Controller Action
       → (Model/Services/DB) → View (Razor renders HTML) → Response → Browser
```

**Benefits:** separation of concerns, testability, parallel development (UI vs backend), maintainability.

**MVC vs Razor Pages vs Web API:**
- **MVC:** Best for larger apps with many related actions and shared logic, or apps that serve both HTML and API.
- **Razor Pages:** Each page has its own handler (`OnGet`, `OnPost`). Simpler for page-centric CRUD.
- **Web API:** No views; returns data (JSON/XML).

### 🎯 Interview Questions
1. **Explain MVC in one minute.**
   The controller receives the request, works with the model (data + business logic), and passes a model to the view, which renders the HTML. Each part has a single responsibility, which improves testability and maintainability.
2. **Should business logic live in controllers?**
   No. Controllers should be thin — they coordinate. Business logic belongs in services/domain classes so it can be reused and unit-tested.
3. **What is the difference between MVC and Razor Pages?**
   MVC groups actions by controller; Razor Pages groups by page with a PageModel. Both use Razor, routing, model binding, and filters.
4. **What is a ViewModel?**
   A class shaped specifically for a view, combining or trimming data from one or more domain models. It avoids exposing entities directly to the UI.

### 🧩 Scenario-Based Questions
1. **A `HomeController` has grown to 3,000 lines with SQL queries inside the actions. How do you refactor?**
   Extract data access into repositories or use EF Core directly in services; move business rules into services; inject them via DI; split the controller by feature (e.g., `ProductsController`, `OrdersController`); introduce ViewModels; add unit tests around services before and after refactoring.
2. **A team wants to build a public JSON API and an admin website. One project or two?**
   One project can host both (MVC controllers + `[ApiController]` controllers). Separate them if they have different scaling, security, or release needs. In both cases share business logic through a class library.

---

## 1.3 Creating Your First MVC Project

### Steps
```bash
dotnet --list-sdks                 # check installed SDKs
dotnet new mvc -n BookStore        # create project
cd BookStore
dotnet run                         # run (or dotnet watch for hot reload)
```

### Project Structure
```
BookStore/
├── Controllers/        → HomeController.cs
├── Models/             → ErrorViewModel.cs
├── Views/
│   ├── Home/           → Index.cshtml, Privacy.cshtml
│   ├── Shared/         → _Layout.cshtml, Error.cshtml, _ValidationScriptsPartial.cshtml
│   ├── _ViewImports.cshtml
│   └── _ViewStart.cshtml
├── wwwroot/            → css, js, lib (static files)
├── Properties/launchSettings.json → local run profiles (ports, environment)
├── appsettings.json
├── appsettings.Development.json
├── Program.cs          → app startup
└── BookStore.csproj    → project file (target framework, packages)
```

### Useful CLI commands
```bash
dotnet build
dotnet watch run
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet new sln -n BookStore && dotnet sln add BookStore/BookStore.csproj
dotnet test
dotnet publish -c Release -o ./publish
```

### 🎯 Interview Questions
1. **What is `wwwroot`?**
   The web root folder for static files (CSS, JS, images). Only files here are served by the static files middleware by default.
2. **What is `launchSettings.json` used for?**
   Local development only: run profiles, URLs/ports, and environment variables like `ASPNETCORE_ENVIRONMENT`. It is not used in production.
3. **What is the `.csproj` file?**
   An MSBuild file that defines the SDK (`Microsoft.NET.Sdk.Web`), target framework, package references, and build settings.
4. **What does `dotnet watch` do?**
   Rebuilds and restarts (or hot-reloads) the app when files change.

### 🧩 Scenario-Based Questions
1. **The app works locally but CSS is missing in production. What do you check?**
   Confirm static file middleware (`UseStaticFiles`/`MapStaticAssets`) is registered; confirm files are in `wwwroot` and included in publish output; check paths (`~/css/site.css`) and case sensitivity on Linux; check the reverse proxy configuration and browser dev tools for 404s.
2. **Two developers get different behavior because one runs in Production mode locally.**
   Check `ASPNETCORE_ENVIRONMENT` in `launchSettings.json` and system environment variables; standardize the launch profile in source control.

---

## 1.4 `Program.cs`, Hosting Model & the Request Pipeline

### The minimal hosting model
```csharp
var builder = WebApplication.CreateBuilder(args);

// 1) Register services (DI container)
builder.Services.AddControllersWithViews();

var app = builder.Build();

// 2) Configure the HTTP request pipeline (middleware, ORDER MATTERS)
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();          // .NET 9+: app.MapStaticAssets(); is the optimized option
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();

// 3) Map endpoints
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

### What `WebApplication.CreateBuilder` sets up
- Configuration (appsettings.json, appsettings.{Environment}.json, user secrets in Development, environment variables, command-line args)
- Logging (Console, Debug, EventSource)
- DI container
- Kestrel web server
- Environment (`IWebHostEnvironment`)

### Middleware pipeline (mental model)
```
Request  →  ExceptionHandler → HSTS → HttpsRedirection → StaticFiles → Routing
         →  Authentication → Authorization → Endpoint (Controller action)
Response ←  ...each middleware can act on the way back...
```
Each middleware can: handle the request and short-circuit, or call `next()` and act before and after.

### 🎯 Interview Questions
1. **What happened to `Startup.cs`?**
   Since .NET 6, the minimal hosting model merges `Startup.ConfigureServices` and `Startup.Configure` into `Program.cs`. `Startup` still works if you prefer it.
2. **Why does middleware order matter?**
   Middleware runs in registration order on the request and reverse order on the response. For example, `UseAuthentication` must come before `UseAuthorization`, and both after `UseRouting` so the endpoint metadata (e.g., `[Authorize]`) is known.
3. **What is the difference between `AddControllersWithViews`, `AddControllers`, `AddRazorPages`, and `AddMvc`?**
   `AddControllers` = API controllers only (no views). `AddControllersWithViews` = controllers + views. `AddRazorPages` = Razor Pages. `AddMvc` = all of them.
4. **What is the Generic Host?**
   The host abstraction that manages app lifetime, DI, configuration, and logging for web and non-web apps (e.g., worker services).

### 🧩 Scenario-Based Questions
1. **`[Authorize]` attributes are ignored — anonymous users can reach protected pages. What's wrong?**
   Likely `UseAuthorization()` is missing or placed after the endpoints are mapped, or `UseAuthentication()` is missing or in the wrong order. Verify order: `UseRouting → UseAuthentication → UseAuthorization → Map...`.
2. **Static files are being sent through authentication and are slow. How do you fix it?**
   Place `UseStaticFiles` before `UseAuthentication`/`UseAuthorization` so static files short-circuit early. Use `MapStaticAssets` (.NET 9+) for compression and caching headers.

---

## 1.5 Your First Controller and View

### Controller
```csharp
// Controllers/BooksController.cs
using Microsoft.AspNetCore.Mvc;

public class BooksController : Controller
{
    public IActionResult Index()
    {
        var books = new List<string> { "Clean Code", "C# in Depth", "Refactoring" };
        return View(books);  // looks for Views/Books/Index.cshtml
    }
}
```

### View
```cshtml
@* Views/Books/Index.cshtml *@
@model List<string>
@{
    ViewData["Title"] = "Books";
}
<h1>Books</h1>
<ul>
    @foreach (var book in Model)
    {
        <li>@book</li>
    }
</ul>
```

Navigate to `/Books` (default route → `BooksController.Index`).

### View discovery order
1. `Views/{Controller}/{Action}.cshtml`
2. `Views/Shared/{Action}.cshtml`
3. (Pages/Shared in apps with Razor Pages)

### 🎯 Interview Questions
1. **How does MVC find a view when you call `View()`?**
   It uses the action name and controller name to look in `Views/{Controller}/` then `Views/Shared/`. You can pass an explicit name or path: `View("Details")` or `View("~/Views/Custom/Page.cshtml")`.
2. **What does the `Controller` suffix mean?**
   By convention, MVC discovers public classes ending in `Controller` (or deriving from `Controller`/`ControllerBase`, or marked `[Controller]`). The route uses the name without the suffix.
3. **What is the difference between `Controller` and `ControllerBase`?**
   `ControllerBase` provides API features (no view support). `Controller` adds view-related features: `View()`, `PartialView()`, `ViewBag`, `ViewData`, `TempData`.

### 🧩 Scenario-Based Questions
1. **You get "InvalidOperationException: The view 'Index' was not found". How do you troubleshoot?**
   The exception message lists searched locations. Check the folder name matches the controller name (without `Controller`), the file name matches the action, the file is a `.cshtml`, and its build action is correct (Content).
2. **A public helper method on a controller is reachable via URL by accident. How do you prevent this?**
   Make it `private`/`protected`, move it to a service, or mark it `[NonAction]`.

---

## ✅ Class 1 Homework
1. Create the BookStore MVC project and add a `BooksController` with `Index` and `About` actions.
2. Add a nav link in `_Layout.cshtml` to Books.
3. Write a custom inline middleware in `Program.cs` that logs request path and elapsed time.
4. Explain, in writing, the full lifecycle of a request to `/Books/Index`.

---
---

# 🟢 CLASS 2 — Routing, Controllers, Actions & Model Binding

**Goal:** Master how URLs map to code, how actions return results, and how input flows into C# objects.

---

## 2.1 Routing

### Conventional routing
```csharp
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

// More specific routes go first
app.MapControllerRoute(
    name: "bookBySlug",
    pattern: "book/{slug}",
    defaults: new { controller = "Books", action = "BySlug" });
```

### Attribute routing
```csharp
[Route("catalog")]
public class CatalogController : Controller
{
    [HttpGet("")]                         // GET /catalog
    public IActionResult Index() => View();

    [HttpGet("{id:int}")]                 // GET /catalog/5
    public IActionResult Details(int id) => View();

    [HttpGet("{category}/{page:int:min(1)=1}")]  // GET /catalog/fiction/2
    public IActionResult ByCategory(string category, int page) => View();
}
```

### Route constraints
| Constraint | Example | Matches |
|---|---|---|
| `int` | `{id:int}` | 123 |
| `guid` | `{id:guid}` | GUID values |
| `min/max/range` | `{age:range(18,99)}` | 18–99 |
| `alpha` | `{name:alpha}` | letters only |
| `regex` | `{code:regex(^[A-Z]{{3}}$)}` | ABC |
| `bool`, `datetime`, `decimal`, `length(n)`, `minlength`, `maxlength` | | |

Custom constraints: implement `IRouteConstraint` and register via `builder.Services.Configure<RouteOptions>(o => o.ConstraintMap.Add("slug", typeof(SlugConstraint)));`.

### Areas (large-app organization)
```csharp
[Area("Admin")]
public class DashboardController : Controller { public IActionResult Index() => View(); }

app.MapControllerRoute(
    name: "areas",
    pattern: "{area:exists}/{controller=Dashboard}/{action=Index}/{id?}");
```
Folder: `Areas/Admin/Controllers`, `Areas/Admin/Views`.

### Generating URLs
```csharp
Url.Action("Details", "Books", new { id = 5 });   // /Books/Details/5
RedirectToAction("Index", "Home");
```
```cshtml
<a asp-controller="Books" asp-action="Details" asp-route-id="5">View</a>
```

### Endpoint routing
Since ASP.NET Core 3.0, routing is split into `UseRouting` (match the endpoint) and endpoint execution, so middleware between them (auth, CORS) can read endpoint metadata.

### 🎯 Interview Questions
1. **Conventional vs attribute routing?**
   Conventional routing defines URL patterns centrally; good for standard MVC sites. Attribute routing puts routes on controllers/actions; preferred for APIs and precise URLs. Attribute routes take precedence for the actions that have them.
2. **What is endpoint routing?**
   A routing system where endpoints are matched early (`UseRouting`) and executed later, letting middleware like authorization and CORS inspect the selected endpoint's metadata.
3. **What are route constraints, and should they be used for validation?**
   They restrict which values match a route segment. They are for disambiguating routes, not input validation — a failed constraint returns 404, not a helpful 400.
4. **What are Areas?**
   A way to partition a large app into functional groups (Admin, Customer), each with its own controllers and views.
5. **What is the token `[controller]` in attribute routes?**
   Token replacement: `[Route("api/[controller]")]` becomes `api/books` for `BooksController`. `[action]` and `[area]` also exist.

### 🧩 Scenario-Based Questions
1. **SEO requires URLs like `/books/clean-code` instead of `/Books/Details/42`. How do you implement it?**
   Add a `Slug` column (unique index), use attribute route `[HttpGet("books/{slug}")]`, look up by slug, and optionally redirect old ID URLs with a 301 (`RedirectToActionPermanent`). Enable lowercase URLs: `builder.Services.AddRouting(o => o.LowercaseUrls = true);`.
2. **You get `AmbiguousMatchException` on `/Books/Details/5`.**
   Two actions match the same route and verb. Differentiate by HTTP verb attributes, constraints, or distinct templates; or remove the duplicate.
3. **An old site URL `/products.aspx?id=7` must keep working after migration.**
   Use URL Rewrite middleware (`AddRedirect`/a custom rule) or a dedicated route that issues 301 redirects to the new URL.

---

## 2.2 Controllers & Action Results

### Common action results
| Helper | Type | Use |
|---|---|---|
| `View()` | `ViewResult` | Render a Razor view |
| `PartialView()` | `PartialViewResult` | Render a partial (AJAX) |
| `Json(obj)` | `JsonResult` | JSON response |
| `Content("text")` | `ContentResult` | Raw text |
| `RedirectToAction()` | `RedirectToActionResult` | 302 to an action |
| `Redirect(url)` / `LocalRedirect(url)` | | Redirect (Local is safe from open redirects) |
| `File(bytes, "application/pdf", "a.pdf")` | `FileResult` | Downloads |
| `NotFound()`, `BadRequest()`, `Ok()`, `Unauthorized()`, `Forbid()`, `NoContent()` | Status results | |
| `StatusCode(503)` | | Any status code |
| `Challenge()` | | Trigger authentication |

```csharp
public class BooksController(IBookService service) : Controller
{
    public async Task<IActionResult> Details(int id)
    {
        var book = await service.GetAsync(id);
        if (book is null) return NotFound();
        return View(book);
    }

    public IActionResult Download(int id)
    {
        var bytes = System.IO.File.ReadAllBytes($"files/{id}.pdf");
        return File(bytes, "application/pdf", $"book-{id}.pdf");
    }
}
```

### Return types
- `IActionResult` — any result type.
- `ActionResult<T>` — APIs: return `T` or an `ActionResult`; improves OpenAPI documentation.
- Specific types (`ViewResult`) — less flexible.
- Always prefer `async Task<IActionResult>` when doing I/O.

### 🎯 Interview Questions
1. **`IActionResult` vs `ActionResult<T>`?**
   `IActionResult` is flexible and untyped. `ActionResult<T>` allows returning either `T` or a status result, enabling implicit conversion and better Swagger metadata — ideal for APIs.
2. **`Redirect` vs `RedirectToAction` vs `LocalRedirect`?**
   `Redirect` goes to any URL; `RedirectToAction` builds a URL from route values; `LocalRedirect` throws if the URL is not local — use it for `returnUrl` to prevent open redirect attacks.
3. **What is the difference between 401 and 403?**
   401 = not authenticated (who are you?). 403 = authenticated but not allowed (`Forbid()`).
4. **Why should actions be async?**
   Async I/O frees threads while waiting for databases/network, improving scalability under load. It doesn't make one request faster.

### 🧩 Scenario-Based Questions
1. **A report action generates a 200 MB CSV and the server runs out of memory.**
   Stream the response instead of building it in memory: write rows to `Response.Body` asynchronously, or return a `FileStreamResult`. Consider generating large reports in a background job and emailing/linking the result.
2. **After login you redirect to `returnUrl` and a pentest flags an open redirect.**
   Validate with `Url.IsLocalUrl(returnUrl)` or use `LocalRedirect(returnUrl)`; fall back to the home page.

---

## 2.3 Model Binding

### Sources (in default order for complex types in MVC)
Form fields → Route values → Query string (body for `[FromBody]`, JSON).

### Explicit binding attributes
```csharp
public IActionResult Search(
    [FromQuery] string term,             // ?term=
    [FromRoute] int page,                // {page}
    [FromHeader(Name = "X-Client")] string? client,
    [FromServices] ILogger<BooksController> logger)
{ ... }

[HttpPost]
public IActionResult Create([FromForm] BookCreateVm vm) { ... }

[HttpPost("api/books")]
public IActionResult CreateApi([FromBody] BookDto dto) { ... }  // JSON
```

### Complex types, collections, files
```csharp
public class BookCreateVm
{
    public string Title { get; set; } = "";
    public decimal Price { get; set; }
    public List<int> CategoryIds { get; set; } = new();
    public IFormFile? Cover { get; set; }
}
```
```cshtml
<form asp-action="Create" method="post" enctype="multipart/form-data">
    <input asp-for="Title" />
    <input asp-for="Price" />
    <input type="file" asp-for="Cover" />
    <button type="submit">Save</button>
</form>
```

### Overposting (mass assignment) protection
```csharp
// BAD: binds directly to entity — attacker can post IsAdmin=true or Price=0
public IActionResult Edit(Book book) { ... }

// GOOD: bind to a ViewModel exposing only editable fields
public IActionResult Edit(int id, BookEditVm vm) { ... }

// Alternative: [Bind("Title,Price")] — works but brittle
```

### Custom model binder (advanced)
Implement `IModelBinder` (e.g., to bind a comma-separated list `?ids=1,2,3`) and register via `[ModelBinder(typeof(CsvIntBinder))]` or an `IModelBinderProvider`.

### 🎯 Interview Questions
1. **What is model binding?**
   The process of mapping HTTP request data (route, query, form, headers, body) to action parameters and model properties, including type conversion.
2. **Why can only one parameter use `[FromBody]`?**
   The request body is a non-rewindable stream read once by an input formatter, so only one parameter can be deserialized from it.
3. **What is overposting and how do you prevent it?**
   An attacker posts extra fields that bind to properties you didn't intend (e.g., `IsAdmin`). Prevent with ViewModels/DTOs containing only allowed fields, `[Bind]`, or `TryUpdateModelAsync` with an include list.
4. **What does `[ApiController]` change about binding?**
   It infers binding sources (complex types → body), requires attribute routing, and automatically returns 400 with ProblemDetails when `ModelState` is invalid.
5. **What is `ModelState`?**
   A dictionary holding binding and validation results. Check `ModelState.IsValid` before processing input.

### 🧩 Scenario-Based Questions
1. **A form posting a decimal price like "12,50" fails for European users.**
   Model binding uses the current culture. Configure request localization (`UseRequestLocalization`) with supported cultures, and align client-side validation (jQuery Validate globalize) — or standardize input formats.
2. **An API returns 415 Unsupported Media Type.**
   The client didn't send `Content-Type: application/json`, or the action expects form data. Fix the client header or use the correct `[FromForm]`/`[FromBody]` attribute.
3. **Users upload profile images and one user uploads a 2 GB file.**
   Enforce limits: `[RequestSizeLimit]`, `FormOptions.MultipartBodyLengthLimit`, Kestrel `MaxRequestBodySize`; validate extension, MIME type, and file signature; store outside `wwwroot` with generated names; scan for malware; stream large files.

---

## 2.4 HTTP Verbs, Action Selectors & the PRG Pattern

### Verbs & selectors
```csharp
[HttpGet]  public IActionResult Create() => View();
[HttpPost] [ValidateAntiForgeryToken]
public async Task<IActionResult> Create(BookCreateVm vm)
{
    if (!ModelState.IsValid) return View(vm);      // redisplay with errors
    await _service.CreateAsync(vm);
    TempData["Success"] = "Book created!";
    return RedirectToAction(nameof(Index));        // PRG
}

[ActionName("Delete")] [HttpPost]
public IActionResult DeleteConfirmed(int id) { ... }

[NonAction] public void Helper() { }
```

### Post/Redirect/Get (PRG)
After a successful POST, redirect to a GET. This prevents duplicate submissions on browser refresh and gives a bookmarkable URL.

### Idempotency
| Verb | Safe | Idempotent |
|---|---|---|
| GET | ✅ | ✅ |
| PUT | ❌ | ✅ |
| DELETE | ❌ | ✅ |
| POST | ❌ | ❌ |
| PATCH | ❌ | ❌ (usually) |

### 🎯 Interview Questions
1. **What is the PRG pattern and why use it?**
   Post → Redirect → Get. After processing a form, redirect so refreshing doesn't resubmit the POST.
2. **Why should deletes never be GET requests?**
   GET must be safe; crawlers, prefetching, or image tags could trigger deletes, and GET can't be protected by anti-forgery tokens.
3. **What does `[ActionName]` do?**
   Sets the name used for routing, allowing two C# methods with the same signature (e.g., GET `Delete(int id)` and POST `DeleteConfirmed(int id)`) to share the action name "Delete".

### 🧩 Scenario-Based Questions
1. **Customers report being charged twice when they double-click "Pay".**
   Client-side: disable the button after submit. Server-side: use an idempotency key (unique token per order form stored in DB with a unique constraint), check order status before charging, and use PRG.
2. **A validation error on POST loses all dropdown options in the view.**
   Dropdown data isn't posted back. Repopulate select lists before `return View(vm)` — ideally via a private method used by both GET and POST.

---

## ✅ Class 2 Homework
1. Add `Create`, `Edit`, `Details`, `Delete` actions for books using an in-memory list service.
2. Implement slug URLs and an `Admin` area.
3. Add a file upload for book covers with size/type validation.
4. Demonstrate an overposting attack against an entity-bound action, then fix it with a ViewModel.

---
---

# 🟡 CLASS 3 — Views, Razor, Layouts, Tag Helpers & View Components

**Goal:** Build clean, reusable, strongly typed user interfaces.

---

## 3.1 Razor Syntax

### Essentials
```cshtml
@model BookStore.Models.BookDetailsVm
@using BookStore.Helpers
@inject IConfiguration Config

<h1>@Model.Title</h1>                         @* implicit expression *@
<p>Price: @(Model.Price * 1.05m)</p>            @* explicit expression *@
<p>Email: support@@bookstore.com</p>            @* escape @ *@

@if (Model.InStock)
{
    <span class="badge bg-success">In stock</span>
}
else
{
    <span class="badge bg-danger">Out of stock</span>
}

@foreach (var review in Model.Reviews)
{
    <partial name="_Review" model="review" />
}

@{
    var discount = Model.Price > 50 ? 0.1m : 0m;   // code block
}

@switch (Model.Format)
{
    case "Ebook": <text>Instant download</text> break;
    default: <text>Ships in 2 days</text> break;
}

@Html.Raw(Model.SafeHtmlDescription)   @* DANGER: only for sanitized HTML *@
```

### Key facts
- Razor **HTML-encodes** output by default (`@value`) → protects against XSS.
- `@Html.Raw` disables encoding — only use with trusted/sanitized content.
- Views compile into C# classes (build-time compilation; runtime compilation optional for development with `Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation`).

### 🎯 Interview Questions
1. **Does Razor protect against XSS?**
   Yes, `@` output is HTML-encoded by default. Risk appears with `Html.Raw`, unencoded JavaScript contexts, or building HTML strings manually.
2. **Implicit vs explicit Razor expressions?**
   Implicit: `@Model.Title`. Explicit: `@(expression)` — needed for arithmetic, generics, or when text follows directly.
3. **What is `@inject` in a view?**
   Injects a service from DI directly into a view (e.g., localization, configuration). Use sparingly to keep views simple.
4. **Are Razor views compiled?**
   Yes, at build time by default, into classes in the app assembly, which improves startup and catches errors early.

### 🧩 Scenario-Based Questions
1. **Product descriptions are written by admins in a rich text editor and must render as HTML.**
   Sanitize on save (and/or render) with a library like HtmlSanitizer that whitelists safe tags/attributes, then use `Html.Raw` on the sanitized output only. Add a Content Security Policy header as defense in depth.
2. **A designer wants to tweak views without rebuilding during development.**
   Enable Razor runtime compilation in Development only, or use `dotnet watch` hot reload.

---

## 3.2 Layouts, `_ViewStart`, `_ViewImports` & Sections

### `_Layout.cshtml`
```cshtml
<!DOCTYPE html>
<html>
<head>
    <title>@ViewData["Title"] - BookStore</title>
    <link rel="stylesheet" href="~/css/site.css" asp-append-version="true" />
    @await RenderSectionAsync("Styles", required: false)
</head>
<body>
    <partial name="_Nav" />
    <main class="container">
        @RenderBody()
    </main>
    <script src="~/js/site.js" asp-append-version="true"></script>
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

### Using a section in a view
```cshtml
@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}
```

### `_ViewStart.cshtml` — runs before every view in its folder and below
```cshtml
@{ Layout = "_Layout"; }
```

### `_ViewImports.cshtml` — shared directives
```cshtml
@using BookStore
@using BookStore.Models
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@addTagHelper *, BookStore
```

### 🎯 Interview Questions
1. **`_ViewStart` vs `_ViewImports`?**
   `_ViewStart` runs code before each view (usually sets `Layout`). `_ViewImports` shares directives (`@using`, `@addTagHelper`, `@inject`, `@model` base) — it doesn't run code.
2. **What is `RenderBody` vs `RenderSection`?**
   `RenderBody` renders the main content of the view (exactly once). `RenderSection` renders named optional/required blocks such as scripts.
3. **What does `asp-append-version="true"` do?**
   Appends a hash of the file content to the URL (`site.css?v=abc123`) for cache busting.
4. **How do you use different layouts for admin and public pages?**
   Put a `_ViewStart.cshtml` in the Admin area/folder setting `Layout = "_AdminLayout"`, or set `Layout` per view.

### 🧩 Scenario-Based Questions
1. **After deploying new CSS, users still see the old style.**
   Use `asp-append-version` or `MapStaticAssets` fingerprinting so URLs change with content; configure cache headers appropriately; purge CDN cache if used.
2. **A page throws "The following sections have been defined but have not been rendered".**
   The view defines a section that the layout doesn't render. Add `RenderSectionAsync` in the layout or remove/ignore the section (`IgnoreSection`).

---

## 3.3 Passing Data to Views: ViewData, ViewBag, TempData & Strongly Typed Models

| Mechanism | Type | Lifetime | Typical use |
|---|---|---|---|
| **Strongly typed model** | `@model T` | Current request | Primary data — **preferred** |
| **ViewData** | `ViewDataDictionary` (string keys) | Current request | Small extras (Title) |
| **ViewBag** | `dynamic` wrapper over ViewData | Current request | Same as ViewData, no compile-time checks |
| **TempData** | Cookie/session-backed | Until read (next request) | Flash messages after redirect |
| **Session** | `ISession` | Across requests | User-specific state (use sparingly) |

```csharp
public IActionResult Index()
{
    ViewData["Title"] = "All Books";
    ViewBag.Count = 42;
    TempData["Success"] = "Saved";        // survives one redirect
    var vm = new BookListVm { Books = _service.GetAll() };
    return View(vm);
}
```
`TempData.Keep("key")` keeps a value for another request; `TempData.Peek("key")` reads without marking for deletion.

### ViewModels
```csharp
public class BookListVm
{
    public IReadOnlyList<BookRowVm> Books { get; init; } = [];
    public int Page { get; init; }
    public int TotalPages { get; init; }
    public string? SearchTerm { get; init; }
}
```

### 🎯 Interview Questions
1. **ViewData vs ViewBag vs TempData?**
   ViewData: dictionary for the current request. ViewBag: dynamic wrapper over the same data. TempData: persists until read, typically across one redirect, stored in a cookie by default (or session).
2. **Why prefer strongly typed views?**
   Compile-time checking, IntelliSense, refactoring safety, and clearer contracts between controller and view.
3. **Where is TempData stored by default?**
   In an encrypted cookie via the cookie-based TempData provider. It can be switched to session-based.
4. **What are risks of using Session?**
   Server affinity/scaling issues (needs a distributed store like Redis for multiple servers), memory use, and stale state. Keep it small.

### 🧩 Scenario-Based Questions
1. **A success message after redirect sometimes doesn't show.**
   Check that TempData is set before the redirect, that the cookie isn't blocked or too large, that cookie consent policy (`CheckConsentNeeded`) isn't blocking non-essential cookies, and that it isn't read (consumed) by another request first such as an AJAX call.
2. **A shopping cart stored in Session disappears when the app scales to 3 servers.**
   In-memory session is per server. Use a distributed cache (Redis/SQL Server) for session or persist the cart in the database tied to the user/anonymous ID.

---

## 3.4 Partial Views & View Components

### Partial views — reusable markup, no logic
```cshtml
<partial name="_BookCard" model="book" />
@await Html.PartialAsync("_BookCard", book)
```

### View Components — reusable UI **with its own logic/data**
```csharp
// ViewComponents/CartSummaryViewComponent.cs
public class CartSummaryViewComponent(ICartService cart) : ViewComponent
{
    public async Task<IViewComponentResult> InvokeAsync()
    {
        var summary = await cart.GetSummaryAsync(UserClaimsPrincipal);
        return View(summary);  // Views/Shared/Components/CartSummary/Default.cshtml
    }
}
```
```cshtml
@await Component.InvokeAsync("CartSummary")
@* or as a tag helper: *@
<vc:cart-summary></vc:cart-summary>
```

### Partial for AJAX
```csharp
public async Task<IActionResult> SearchPartial(string term)
    => PartialView("_BookResults", await _service.SearchAsync(term));
```

### 🎯 Interview Questions
1. **Partial view vs View Component?**
   A partial view renders markup from data passed in by the parent. A view component has its own class, DI, and async logic to fetch data independently — like a mini-controller. Use view components for widgets (cart, menu, recent posts).
2. **Can a view component be called directly via URL?**
   No, not by default. They aren't endpoints; you can return one from a controller via `ViewComponent("Name")`.
3. **Why not use child actions?**
   `Html.Action`/child actions from MVC 5 don't exist in ASP.NET Core; view components replace them.

### 🧩 Scenario-Based Questions
1. **The layout's navigation menu needs categories from the database on every page.**
   Use a view component (`CategoryMenuViewComponent`) that loads categories via a service, ideally cached with `IMemoryCache`, and invoke it in `_Layout`.
2. **A page with 20 partials loading data individually is slow.**
   Avoid N+1 queries: load data once in the controller into a ViewModel and pass slices to partials; cache expensive widgets; use output caching or `<cache>` tag helper.

---

## 3.5 Tag Helpers & HTML Helpers

### Built-in tag helpers
```cshtml
<form asp-controller="Books" asp-action="Edit" asp-route-id="@Model.Id" method="post">
    <div asp-validation-summary="ModelOnly" class="text-danger"></div>

    <label asp-for="Title"></label>
    <input asp-for="Title" class="form-control" />
    <span asp-validation-for="Title" class="text-danger"></span>

    <select asp-for="CategoryId" asp-items="Model.Categories"></select>

    <button type="submit">Save</button>
</form>

<environment include="Development"><script src="~/js/debug.js"></script></environment>
<cache expires-after="@TimeSpan.FromMinutes(10)">@await Component.InvokeAsync("TopSellers")</cache>
<img src="~/images/logo.png" asp-append-version="true" />
```
`<form>` tag helper automatically adds an **anti-forgery token** for POST forms.

### HTML helper equivalents (older style)
```cshtml
@Html.LabelFor(m => m.Title)
@Html.TextBoxFor(m => m.Title, new { @class = "form-control" })
@Html.ValidationMessageFor(m => m.Title)
```

### Custom tag helper
```csharp
// <email address="support@bookstore.com"></email>
[HtmlTargetElement("email")]
public class EmailTagHelper : TagHelper
{
    public string Address { get; set; } = "";

    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        output.TagName = "a";
        output.Attributes.SetAttribute("href", $"mailto:{Address}");
        output.Content.SetContent(Address);
    }
}
```
Register with `@addTagHelper *, BookStore` in `_ViewImports`.

### 🎯 Interview Questions
1. **Tag helpers vs HTML helpers?**
   Tag helpers look like HTML attributes/elements, are designer-friendly, and get IntelliSense. HTML helpers are C# method calls. Both generate similar markup; tag helpers are the modern default.
2. **What does `asp-for` do?**
   Binds an element to a model expression: generates `id`, `name`, `value`, type and validation `data-val-*` attributes based on the property and its annotations.
3. **How does the form tag helper help security?**
   It injects a hidden `__RequestVerificationToken` for anti-forgery validation.
4. **How do you build a custom tag helper?**
   Inherit `TagHelper`, override `Process`/`ProcessAsync`, target elements with `[HtmlTargetElement]`, and register via `@addTagHelper`.

### 🧩 Scenario-Based Questions
1. **Every form needs a consistent Bootstrap structure (label, input, validation).**
   Create a custom tag helper (e.g., `<form-field for="Title" />`) using `IHtmlGenerator`, or a reusable partial/editor template (`Views/Shared/EditorTemplates`).
2. **Client-side validation is not working on a form.**
   Ensure `_ValidationScriptsPartial` (jQuery, jquery.validate, unobtrusive) is included in the `Scripts` section, inputs use `asp-for`, and `data-val` attributes appear in the rendered HTML.

---

## ✅ Class 3 Homework
1. Build a layout with nav, footer, and a `Scripts` section.
2. Create `_BookCard` partial and a `CartSummary` view component.
3. Build Create/Edit forms with tag helpers and validation messages.
4. Write a custom `<price>` tag helper that formats decimals by culture and shows a "Sale" badge.

---
---

# 🟡 CLASS 4 — Models, Validation & Entity Framework Core

**Goal:** Validate input properly and persist data efficiently with EF Core.

---

## 4.1 Models: Entities, ViewModels & DTOs

| Type | Purpose | Example |
|---|---|---|
| **Entity** | Maps to a DB table | `Book` with `Id`, navigation properties |
| **ViewModel** | Shaped for a view | `BookEditVm` with `SelectList Categories` |
| **DTO** | Data transfer across boundaries (API) | `BookDto` record |
| **Domain model** | Business rules and behavior | `Order.AddItem()` enforcing invariants |

### Mapping
- Manual mapping (explicit, fast, clear) — extension methods like `book.ToDto()`.
- Libraries: Mapster, or AutoMapper (note: AutoMapper moved to a commercial license in 2025 — check licensing before choosing).
- EF Core projections: `.Select(b => new BookRowVm { ... })` — maps in SQL, loads only needed columns.

### 🎯 Interview Questions
1. **Why not pass EF entities directly to views or APIs?**
   Overposting risk, leaking sensitive fields, lazy-loading surprises, circular references in JSON, and coupling UI to DB schema.
2. **ViewModel vs DTO?**
   ViewModel serves a specific view (may include UI data like select lists). DTO is a plain transfer object between layers/services, often for APIs.
3. **What is an anemic domain model?**
   Entities with only data and no behavior; all logic lives in services. A rich domain model puts invariants and behavior inside entities.

### 🧩 Scenario-Based Questions
1. **An API returning `User` entities accidentally exposes `PasswordHash`.**
   Return DTOs with only safe fields; add tests asserting the response contract; review all endpoints returning entities.
2. **Mapping code is duplicated across 30 controllers.**
   Centralize mapping in extension methods or a mapping library configuration; prefer EF projections for read models.

---

## 4.2 Validation

### Data annotations
```csharp
public class BookCreateVm
{
    [Required, StringLength(200, MinimumLength = 2)]
    public string Title { get; set; } = "";

    [Range(0.01, 10000)]
    [DataType(DataType.Currency)]
    public decimal Price { get; set; }

    [Required, RegularExpression(@"^\d{13}$", ErrorMessage = "ISBN must be 13 digits")]
    public string Isbn { get; set; } = "";

    [EmailAddress] public string? ContactEmail { get; set; }

    [Display(Name = "Published on")]
    [DataType(DataType.Date)]
    public DateOnly PublishedOn { get; set; }

    [Compare(nameof(Isbn))] public string? ConfirmIsbn { get; set; }

    [Remote(action: "IsIsbnUnique", controller: "Books")]
    public string? UniqueIsbn { get; set; }
}
```

### Server-side check
```csharp
[HttpPost, ValidateAntiForgeryToken]
public async Task<IActionResult> Create(BookCreateVm vm)
{
    if (await _service.IsbnExistsAsync(vm.Isbn))
        ModelState.AddModelError(nameof(vm.Isbn), "ISBN already exists.");

    if (!ModelState.IsValid) return View(vm);
    ...
}
```

### Custom validation attribute
```csharp
public class NotInFutureAttribute : ValidationAttribute
{
    protected override ValidationResult? IsValid(object? value, ValidationContext ctx)
        => value is DateOnly d && d > DateOnly.FromDateTime(DateTime.Today)
            ? new ValidationResult(ErrorMessage ?? "Date cannot be in the future.")
            : ValidationResult.Success;
}
```

### Cross-property validation with `IValidatableObject`
```csharp
public class PromotionVm : IValidatableObject
{
    public DateTime Start { get; set; }
    public DateTime End { get; set; }

    public IEnumerable<ValidationResult> Validate(ValidationContext ctx)
    {
        if (End <= Start)
            yield return new ValidationResult("End must be after Start", [nameof(End)]);
    }
}
```

### FluentValidation (popular alternative)
```csharp
public class BookValidator : AbstractValidator<BookCreateVm>
{
    public BookValidator()
    {
        RuleFor(x => x.Title).NotEmpty().MaximumLength(200);
        RuleFor(x => x.Price).GreaterThan(0);
    }
}
```
Call validators explicitly in actions/filters (automatic MVC pipeline integration is no longer recommended by the FluentValidation project).

### Client-side validation
Uses jQuery Unobtrusive Validation reading `data-val-*` attributes generated by tag helpers. **Never trust client-side validation alone.**

### 🎯 Interview Questions
1. **Is client-side validation enough?**
   No. It improves UX but can be bypassed. Server-side validation is mandatory.
2. **How do you add a validation error manually?**
   `ModelState.AddModelError("PropertyName", "Message")`; use an empty key for model-level errors.
3. **`[Required]` on a non-nullable `int` — does it work?**
   An `int` always has a value (0), so `[Required]` won't catch missing input. Use `int?` with `[Required]` to detect missing values. With nullable reference types enabled, non-nullable `string` properties are treated as implicitly required.
4. **What is `[Remote]` validation?**
   Client-side validation that calls a server action via AJAX (e.g., to check username availability). You still must validate on the server.
5. **Data annotations vs FluentValidation?**
   Annotations are simple and built in; FluentValidation is better for complex, conditional, testable rules and keeps models clean.

### 🧩 Scenario-Based Questions
1. **Two users register the same email at the same moment; both pass "email exists" validation.**
   Race condition. Enforce a unique index in the DB and catch `DbUpdateException` to show a friendly error. Validation checks are UX; constraints are the source of truth.
2. **A wizard form has 3 steps and validation of step 3 fails for fields from step 1.**
   Use separate ViewModels per step, or validate only relevant fields (`ModelState.Remove` for others, or FluentValidation rule sets). Persist intermediate state server-side.

---

## 4.3 Entity Framework Core Fundamentals

### Setup
```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet tool install --global dotnet-ef
```

### Entities & DbContext
```csharp
public class Book
{
    public int Id { get; set; }
    public string Title { get; set; } = "";
    public decimal Price { get; set; }
    public int AuthorId { get; set; }
    public Author Author { get; set; } = null!;
    public ICollection<Category> Categories { get; set; } = new List<Category>();
    public byte[] RowVersion { get; set; } = [];   // concurrency token
}

public class Author
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public ICollection<Book> Books { get; set; } = new List<Book>();
}

public class Category
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public ICollection<Book> Books { get; set; } = new List<Book>();
}

public class AppDbContext(DbContextOptions<AppDbContext> options) : DbContext(options)
{
    public DbSet<Book> Books => Set<Book>();
    public DbSet<Author> Authors => Set<Author>();
    public DbSet<Category> Categories => Set<Category>();

    protected override void OnModelCreating(ModelBuilder b)
    {
        b.Entity<Book>(e =>
        {
            e.Property(x => x.Title).HasMaxLength(200).IsRequired();
            e.Property(x => x.Price).HasPrecision(10, 2);
            e.Property(x => x.RowVersion).IsRowVersion();
            e.HasIndex(x => x.Title);
        });
        b.ApplyConfigurationsFromAssembly(typeof(AppDbContext).Assembly);
    }
}
```

### Register & migrate
```csharp
builder.Services.AddDbContext<AppDbContext>(o =>
    o.UseSqlServer(builder.Configuration.GetConnectionString("Default")));
```
```bash
dotnet ef migrations add InitialCreate
dotnet ef database update
dotnet ef migrations script --idempotent -o migrate.sql   # for production deployments
```

### CRUD
```csharp
// Create
_db.Books.Add(book); await _db.SaveChangesAsync();

// Read
var book = await _db.Books.FindAsync(id);
var list = await _db.Books.AsNoTracking().Where(b => b.Price < 20).ToListAsync();

// Update (tracked)
var b = await _db.Books.FindAsync(id); b!.Price = 15; await _db.SaveChangesAsync();

// Bulk update/delete without loading (EF Core 7+)
await _db.Books.Where(b => b.Price == 0).ExecuteDeleteAsync();
await _db.Books.Where(b => b.AuthorId == 3)
    .ExecuteUpdateAsync(s => s.SetProperty(b => b.Price, b => b.Price * 1.1m));
```

### 🎯 Interview Questions
1. **What is a DbContext and what's its lifetime?**
   A unit of work + repository over the database: tracks changes and saves them. Registered as **scoped** (one per request). It is not thread-safe.
2. **Code-first vs database-first?**
   Code-first: C# classes → migrations → DB. Database-first: scaffold classes from an existing DB (`dotnet ef dbcontext scaffold`).
3. **What does `AsNoTracking` do?**
   Disables change tracking for read-only queries — faster and less memory.
4. **How should migrations be applied in production?**
   Prefer generated idempotent SQL scripts or migration bundles (`dotnet ef migrations bundle`) run in the deployment pipeline, rather than `Database.Migrate()` at app startup (race conditions with multiple instances, permission issues).
5. **What is `ExecuteUpdate`/`ExecuteDelete`?**
   Set-based operations that translate to a single SQL UPDATE/DELETE without loading entities into memory (they bypass the change tracker).
6. **Fluent API vs data annotations for EF configuration?**
   Fluent API is more powerful (composite keys, relationships, precision, indexes) and keeps entities clean; annotations are quicker for simple cases.

### 🧩 Scenario-Based Questions
1. **Two admins edit the same book at once; one silently overwrites the other.**
   Use optimistic concurrency with a `rowversion` concurrency token. Include it in the edit form (hidden field), and catch `DbUpdateConcurrencyException` to show the user the conflicting values.
2. **A migration added a non-nullable column to a table with 5 million rows and timed out in production.**
   Split into steps: add nullable column → backfill in batches → make non-nullable; review generated SQL before deploy; run during low traffic; increase command timeout for the migration.
3. **You inject `AppDbContext` into a singleton service and get strange errors.**
   Captive dependency — a scoped DbContext inside a singleton lives forever and is shared across threads. Inject `IDbContextFactory<AppDbContext>` or `IServiceScopeFactory` and create scopes as needed.

---

## 4.4 Relationships, Loading Strategies & Efficient Queries

### Relationship types
- **One-to-many:** Author → Books
- **One-to-one:** User → Profile
- **Many-to-many:** Books ↔ Categories (EF Core 5+ creates the join table automatically; use an explicit join entity if it needs extra columns like `AddedOn`)

### Loading strategies
```csharp
// Eager loading
var books = await _db.Books.Include(b => b.Author).Include(b => b.Categories).ToListAsync();

// Explicit loading
await _db.Entry(book).Collection(b => b.Categories).LoadAsync();

// Lazy loading (opt-in: UseLazyLoadingProxies + virtual navigations) — use with care

// Projection (often best)
var rows = await _db.Books
    .Select(b => new BookRowVm { Id = b.Id, Title = b.Title, Author = b.Author.Name })
    .ToListAsync();

// Split queries to avoid cartesian explosion
var authors = await _db.Authors.Include(a => a.Books).AsSplitQuery().ToListAsync();
```

### The N+1 problem
```csharp
// BAD: 1 query for books + N queries for authors (with lazy loading)
foreach (var b in _db.Books.ToList()) Console.WriteLine(b.Author.Name);

// GOOD: 1 query
var data = _db.Books.Select(b => new { b.Title, AuthorName = b.Author.Name }).ToList();
```

### IQueryable vs IEnumerable
```csharp
IQueryable<Book> q = _db.Books;           // builds SQL, executes on DB
q = q.Where(b => b.Price > 10);           // still SQL
var list = await q.ToListAsync();         // executes

IEnumerable<Book> e = _db.Books.AsEnumerable();
e.Where(b => b.Price > 10);               // filters IN MEMORY after loading ALL rows!
```

### Pagination
```csharp
var page = await _db.Books.AsNoTracking()
    .OrderBy(b => b.Id)
    .Skip((pageNumber - 1) * pageSize).Take(pageSize)
    .ToListAsync();
// For deep pages use keyset pagination: .Where(b => b.Id > lastSeenId).Take(pageSize)
```

### Raw SQL safely
```csharp
var books = await _db.Books
    .FromSql($"SELECT * FROM Books WHERE Title LIKE {pattern}")   // parameterized
    .ToListAsync();
// NEVER: FromSqlRaw("... WHERE Title = '" + input + "'")
```

### Global query filters (soft delete / multi-tenant)
```csharp
b.Entity<Book>().HasQueryFilter(x => !x.IsDeleted);
// bypass with .IgnoreQueryFilters()
```

### 🎯 Interview Questions
1. **What is the N+1 query problem?**
   Loading a list (1 query) then a related entity for each item (N queries). Fix with `Include`, projection, or batching.
2. **IQueryable vs IEnumerable?**
   `IQueryable` builds an expression tree translated to SQL and executed on the database. `IEnumerable` executes in memory. Calling `AsEnumerable`/`ToList` too early pulls all rows into memory.
3. **Eager vs lazy vs explicit loading?**
   Eager: `Include` in the same query. Lazy: loads on property access (risk of N+1). Explicit: you call `Load` manually when needed.
4. **What is a cartesian explosion and how do you fix it?**
   Including multiple collections in one query multiplies rows. Use `AsSplitQuery()` or projections.
5. **Is `FromSql` safe from SQL injection?**
   `FromSql`/`FromSqlInterpolated` parameterize interpolated values. `FromSqlRaw` with string concatenation is unsafe.
6. **Offset vs keyset pagination?**
   Offset (`Skip/Take`) gets slower on deep pages and can skip/duplicate rows when data changes. Keyset uses `WHERE Id > last` — consistent and fast with an index.

### 🧩 Scenario-Based Questions
1. **A book listing page takes 8 seconds. How do you investigate?**
   Enable EF Core logging of SQL (`LogTo` / `EnableSensitiveDataLogging` in dev only), look for N+1 and `SELECT *`, check execution plans and missing indexes, use projections and `AsNoTracking`, add pagination, and measure with MiniProfiler/Application Insights.
2. **A "Delete" button must not physically delete data due to auditing rules.**
   Implement soft delete: `IsDeleted` + `DeletedAt` fields, global query filter, and override `SaveChanges` or use an interceptor to convert deletes into updates.
3. **A SaaS app serves multiple companies from one database.**
   Add `TenantId` to entities, resolve the tenant per request (subdomain/claim), apply a global query filter with the current tenant, and add indexes on `TenantId`. Test for data leakage.

---

## 4.5 Repository, Unit of Work & Async Data Access

### Should you add a repository over EF Core?
- **EF Core already implements** Repository (`DbSet`) and Unit of Work (`DbContext`).
- Add your own repository when you want to: hide EF from domain/application layers, centralize complex queries, or make unit testing simpler.
- Avoid generic repositories that just wrap `DbSet` and leak `IQueryable` — they add layers without value.

```csharp
public interface IBookRepository
{
    Task<Book?> GetByIdAsync(int id, CancellationToken ct = default);
    Task<PagedResult<BookRowVm>> SearchAsync(string? term, int page, int size, CancellationToken ct = default);
    void Add(Book book);
}

public class BookRepository(AppDbContext db) : IBookRepository
{
    public Task<Book?> GetByIdAsync(int id, CancellationToken ct = default)
        => db.Books.Include(b => b.Author).FirstOrDefaultAsync(b => b.Id == id, ct);

    public async Task<PagedResult<BookRowVm>> SearchAsync(string? term, int page, int size, CancellationToken ct = default)
    {
        var q = db.Books.AsNoTracking();
        if (!string.IsNullOrWhiteSpace(term)) q = q.Where(b => b.Title.Contains(term));
        var total = await q.CountAsync(ct);
        var items = await q.OrderBy(b => b.Title)
            .Skip((page - 1) * size).Take(size)
            .Select(b => new BookRowVm { Id = b.Id, Title = b.Title, Price = b.Price })
            .ToListAsync(ct);
        return new PagedResult<BookRowVm>(items, total, page, size);
    }

    public void Add(Book book) => db.Books.Add(book);
}
```

### Transactions
```csharp
await using var tx = await _db.Database.BeginTransactionAsync();
try
{
    _db.Orders.Add(order);
    await _db.SaveChangesAsync();
    await _inventory.ReserveAsync(order);   // same DbContext
    await _db.SaveChangesAsync();
    await tx.CommitAsync();
}
catch { await tx.RollbackAsync(); throw; }
```
A single `SaveChanges` call is already transactional.

### Async best practices
- Use `async` all the way; never `.Result` or `.Wait()` (thread pool starvation/deadlocks).
- Pass `CancellationToken` (MVC binds `HttpContext.RequestAborted` automatically to a `CancellationToken` action parameter).
- Don't run parallel queries on the same `DbContext`.

### 🎯 Interview Questions
1. **Is the Repository pattern needed with EF Core?**
   Not necessarily. `DbContext` is a Unit of Work and `DbSet` a repository. Custom repositories are useful for abstraction and testability but generic wrappers often add little value. Be ready to argue both sides.
2. **Is `SaveChanges` transactional?**
   Yes — all changes in one `SaveChanges` call are committed in a single transaction.
3. **Why avoid `.Result` in ASP.NET Core?**
   It blocks a thread pool thread, reducing scalability, and can cause thread pool starvation under load.
4. **How do you cancel a long query when the user navigates away?**
   Accept a `CancellationToken` in the action and pass it to EF Core async methods.

### 🧩 Scenario-Based Questions
1. **Under load, the app's response time climbs and the thread pool count grows rapidly.**
   Classic sync-over-async/thread pool starvation. Find `.Result`/`.Wait()`/sync I/O calls (dotnet-counters, dotnet-dump), make the code async end to end.
2. **Placing an order must update Orders, Inventory, and send an email; the email fails.**
   Commit the DB transaction for Orders + Inventory; send the email outside the transaction via a background queue/outbox pattern so email failure doesn't roll back the order and can be retried.

---

## ✅ Class 4 Homework
1. Replace the in-memory list with EF Core + SQL Server/SQLite; create migrations.
2. Add Authors and Categories (many-to-many) with eager loading and projections.
3. Implement search + pagination + sorting.
4. Add optimistic concurrency to book editing and handle conflicts in the UI.
5. Implement soft delete with a global query filter.

---
---

# 🟠 CLASS 5 — Dependency Injection, Middleware, Configuration, Logging & Error Handling

**Goal:** Understand the framework internals every professional ASP.NET Core developer must know.

---

## 5.1 Dependency Injection (DI)

### Concepts
- **Inversion of Control:** classes receive dependencies instead of creating them.
- ASP.NET Core has a built-in DI container (`IServiceCollection` / `IServiceProvider`).
- **Constructor injection** is the default. Also: `[FromServices]` in actions, `@inject` in views, `HttpContext.RequestServices` (service locator — avoid).

### Service lifetimes
| Lifetime | Created | Use for |
|---|---|---|
| **Transient** | Every time requested | Lightweight, stateless services |
| **Scoped** | Once per request (scope) | DbContext, unit of work, per-request state |
| **Singleton** | Once for app lifetime | Caches, configuration, thread-safe stateless services, `HttpClient` handlers |

```csharp
builder.Services.AddTransient<IPriceCalculator, PriceCalculator>();
builder.Services.AddScoped<IBookRepository, BookRepository>();
builder.Services.AddSingleton<IClock, SystemClock>();

// Multiple implementations
builder.Services.AddScoped<INotifier, EmailNotifier>();
builder.Services.AddScoped<INotifier, SmsNotifier>();
// inject IEnumerable<INotifier> to get both

// Keyed services (.NET 8+)
builder.Services.AddKeyedScoped<IPaymentGateway, StripeGateway>("stripe");
builder.Services.AddKeyedScoped<IPaymentGateway, PayPalGateway>("paypal");
public class CheckoutService([FromKeyedServices("stripe")] IPaymentGateway gateway) { }

// HttpClient the right way
builder.Services.AddHttpClient<IIsbnLookupClient, IsbnLookupClient>(c =>
    c.BaseAddress = new Uri("https://api.example.com/"));
```

### Captive dependency rule
A service must not depend on a service with a **shorter** lifetime (singleton → scoped is wrong). In Development, `ValidateScopes` catches this at runtime.

### 🎯 Interview Questions
1. **Explain the three lifetimes with examples.**
   Transient: new each injection (e.g., a formatter). Scoped: one per HTTP request (DbContext). Singleton: one for the app (memory cache, config).
2. **What is a captive dependency?**
   A longer-lived service holding a shorter-lived one (singleton holding scoped DbContext), causing stale state and thread-safety bugs.
3. **How do you use a scoped service from a singleton or background service?**
   Inject `IServiceScopeFactory`, create a scope, and resolve the service inside it.
4. **Why use `IHttpClientFactory` instead of `new HttpClient()`?**
   Creating/disposing `HttpClient` per call can exhaust sockets; a single static client ignores DNS changes. The factory pools handlers and supports resilience policies (`Microsoft.Extensions.Http.Resilience`).
5. **What are keyed services?**
   A .NET 8 feature to register multiple implementations of one interface under keys and resolve a specific one.
6. **Service locator vs DI?**
   Service locator (pulling from `IServiceProvider` manually) hides dependencies and hurts testability; DI makes them explicit in constructors.

### 🧩 Scenario-Based Questions
1. **Payment provider depends on the customer's country: Stripe for EU, Razorpay for India.**
   Register implementations as keyed services or behind a factory (`IPaymentGatewayFactory.Get(country)`) — Strategy pattern. Keep selection logic in one place and test it.
2. **A singleton cache service occasionally returns another user's data.**
   It probably stores per-user/per-request state in fields. Singletons must be stateless or thread-safe and must not hold user context; move per-user state to scoped services or include the user ID in cache keys.
3. **Unit tests are hard because a service calls `DateTime.Now` directly.**
   Inject `TimeProvider` (.NET 8+) and use `FakeTimeProvider` in tests.

---

## 5.2 Middleware

### Building blocks
```csharp
app.Use(async (context, next) =>           // can pass along
{
    var sw = Stopwatch.StartNew();
    await next(context);
    app.Logger.LogInformation("{Path} took {Ms} ms", context.Request.Path, sw.ElapsedMilliseconds);
});

app.Map("/health-simple", branch => branch.Run(async ctx => await ctx.Response.WriteAsync("OK")));

app.UseWhen(ctx => ctx.Request.Path.StartsWithSegments("/api"),
            branch => branch.UseMiddleware<ApiKeyMiddleware>());

app.Run(async ctx => await ctx.Response.WriteAsync("Terminal"));   // ends pipeline
```

### Class-based middleware
```csharp
public class CorrelationIdMiddleware(RequestDelegate next)
{
    private const string Header = "X-Correlation-ID";

    public async Task InvokeAsync(HttpContext context, ILogger<CorrelationIdMiddleware> logger)
    {
        var id = context.Request.Headers[Header].FirstOrDefault() ?? Guid.NewGuid().ToString();
        context.Response.Headers[Header] = id;

        using (logger.BeginScope(new Dictionary<string, object> { ["CorrelationId"] = id }))
        {
            await next(context);
        }
    }
}

// Program.cs
app.UseMiddleware<CorrelationIdMiddleware>();
```
Middleware classes are effectively **singletons** — inject scoped services into `InvokeAsync`, not the constructor (or implement `IMiddleware` for per-request activation).

### Recommended order
```
ExceptionHandler → HSTS → HttpsRedirection → StaticFiles → Routing → CORS
→ RateLimiter → Authentication → Authorization → Session → custom → Endpoints
```

### Middleware vs Filters
- **Middleware:** applies to all requests, knows nothing about MVC actions (but can read endpoint metadata).
- **Filters:** MVC-specific, run around actions with access to `ActionContext`, model binding results, action arguments.

### 🎯 Interview Questions
1. **`Use` vs `Run` vs `Map`?**
   `Use` can call the next middleware; `Run` is terminal; `Map` branches the pipeline by path; `UseWhen` conditionally branches and rejoins.
2. **What is short-circuiting?**
   A middleware writes the response and does not call `next`, ending the pipeline (e.g., static files, auth failure).
3. **Why inject scoped services into `InvokeAsync` rather than the middleware constructor?**
   Convention-based middleware is created once (singleton-like), so constructor-injected scoped services would be captive.
4. **When would you choose a filter over middleware?**
   When you need MVC context: action arguments, ModelState, the action result, or behavior applied to specific controllers/actions.

### 🧩 Scenario-Based Questions
1. **You need a header `X-Response-Time` on every response, but headers are already sent when you try to add it after `next()`.**
   Register `context.Response.OnStarting(() => { ...set header...; return Task.CompletedTask; })` before calling `next`.
2. **A maintenance mode must show a "We'll be back" page except for admins' IPs.**
   Custom middleware early in the pipeline that checks a config flag (via `IOptionsMonitor` for live changes) and allowed IPs, returning 503 with a `Retry-After` header for everyone else.
3. **Requests through Nginx show the proxy's IP instead of the client IP and HTTPS redirects loop.**
   Configure `ForwardedHeadersMiddleware` (`X-Forwarded-For`, `X-Forwarded-Proto`) early in the pipeline with known proxies/networks.

---

## 5.3 Configuration, Options Pattern & Environments

### Configuration sources (later overrides earlier)
1. `appsettings.json`
2. `appsettings.{Environment}.json`
3. User secrets (Development only)
4. Environment variables (`BookStore__Smtp__Host` → `BookStore:Smtp:Host`)
5. Command-line arguments
(+ Azure Key Vault / Azure App Configuration / AWS Secrets Manager if added)

### `appsettings.json`
```json
{
  "ConnectionStrings": { "Default": "Server=(localdb)\\mssqllocaldb;Database=BookStore;Trusted_Connection=True;" },
  "Smtp": { "Host": "smtp.example.com", "Port": 587, "From": "noreply@bookstore.com" },
  "Features": { "EnableReviews": true }
}
```

### Options pattern
```csharp
public class SmtpOptions
{
    public const string Section = "Smtp";
    [Required] public string Host { get; set; } = "";
    [Range(1, 65535)] public int Port { get; set; }
    [EmailAddress] public string From { get; set; } = "";
}

builder.Services.AddOptions<SmtpOptions>()
    .BindConfiguration(SmtpOptions.Section)
    .ValidateDataAnnotations()
    .ValidateOnStart();          // fail fast at startup

public class EmailSender(IOptions<SmtpOptions> options) { private readonly SmtpOptions _o = options.Value; }
```

| Interface | Lifetime | Reloads? | Use |
|---|---|---|---|
| `IOptions<T>` | Singleton | No | Static settings |
| `IOptionsSnapshot<T>` | Scoped | Per request | Settings that may change, in scoped services |
| `IOptionsMonitor<T>` | Singleton | Yes + change notifications | Singletons needing live updates; named options |

### Secrets
```bash
dotnet user-secrets init
dotnet user-secrets set "Smtp:Password" "dev-password"
```
Production: environment variables, Azure Key Vault, or managed identity — **never commit secrets** to source control.

### Environments
```csharp
if (app.Environment.IsDevelopment()) { ... }
if (app.Environment.IsEnvironment("Staging")) { ... }
```
```cshtml
<environment include="Development">...</environment>
```

### Feature flags
`Microsoft.FeatureManagement.AspNetCore` → `[FeatureGate("Reviews")]`, `<feature name="Reviews">` tag helper.

### 🎯 Interview Questions
1. **What is the configuration precedence?**
   Later providers override earlier ones: appsettings.json < appsettings.{Env}.json < user secrets (dev) < environment variables < command line.
2. **`IOptions` vs `IOptionsSnapshot` vs `IOptionsMonitor`?**
   See table above. Key point: `IOptionsSnapshot` is scoped, so it can't be injected into singletons; use `IOptionsMonitor` there.
3. **How do you represent nested keys in environment variables?**
   Double underscore: `Smtp__Host`.
4. **How do you fail fast on invalid configuration?**
   `ValidateDataAnnotations()` / `Validate(...)` with `ValidateOnStart()`.
5. **Where should production secrets live?**
   A secret store (Key Vault, AWS Secrets Manager, HashiCorp Vault) or environment variables injected by the platform — preferably accessed via managed identity.

### 🧩 Scenario-Based Questions
1. **A developer committed the production DB password to Git.**
   Immediately rotate the credential; remove it from history (git filter-repo/BFG) though assume it's compromised; move secrets to Key Vault; add secret scanning (GitHub secret scanning, pre-commit hooks).
2. **Ops wants to change a rate-limit value without redeploying or restarting.**
   Use a reloadable source (Azure App Configuration with refresh, or `reloadOnChange` JSON) and consume via `IOptionsMonitor`.
3. **The app starts in production with a missing SMTP host and only fails when sending the first email hours later.**
   Add options validation with `ValidateOnStart()` so it fails at deployment time, plus a health check.

---

## 5.4 Logging & Observability

### Built-in logging
```csharp
public class OrdersController(ILogger<OrdersController> logger) : Controller
{
    public IActionResult Place(int bookId)
    {
        logger.LogInformation("Placing order for book {BookId} by {User}", bookId, User.Identity?.Name);
        ...
    }
}
```
- Use **message templates** (`{BookId}`), not string interpolation → enables structured logging and avoids allocations.
- Levels: `Trace < Debug < Information < Warning < Error < Critical`.
- Configure per category in `appsettings.json`:
```json
"Logging": { "LogLevel": { "Default": "Information", "Microsoft.AspNetCore": "Warning", "Microsoft.EntityFrameworkCore.Database.Command": "Warning" } }
```

### High-performance logging (source generator)
```csharp
public static partial class Log
{
    [LoggerMessage(Level = LogLevel.Warning, Message = "Book {BookId} out of stock")]
    public static partial void OutOfStock(ILogger logger, int bookId);
}
```

### Serilog (popular)
```csharp
builder.Host.UseSerilog((ctx, cfg) => cfg
    .ReadFrom.Configuration(ctx.Configuration)
    .Enrich.FromLogContext()
    .WriteTo.Console()
    .WriteTo.Seq("http://localhost:5341"));
app.UseSerilogRequestLogging();
```

### Observability
- **OpenTelemetry**: logs, traces, metrics → Application Insights, Grafana, Jaeger, Datadog.
- **.NET Aspire** dashboard for local distributed tracing.
- **dotnet-counters / dotnet-trace / dotnet-dump** for diagnostics.

### 🎯 Interview Questions
1. **Why use message templates instead of string interpolation in logs?**
   Templates create structured properties (searchable fields like `BookId`) and avoid formatting cost when the level is disabled.
2. **What is structured logging?**
   Logging events as key/value data rather than plain text, enabling queries like "all errors for BookId=42".
3. **What should never be logged?**
   Passwords, tokens, full card numbers, secrets, and unnecessary personal data (GDPR/PCI concerns).
4. **What is a log scope?**
   Contextual properties (e.g., CorrelationId, OrderId) attached to all log entries within a block.
5. **What are the three pillars of observability?**
   Logs, metrics, and traces.

### 🧩 Scenario-Based Questions
1. **Production log storage costs exploded after a release.**
   Someone left `Debug`/EF command logging enabled or logs inside a hot loop. Tune levels per category, use sampling, and review logs added in the release.
2. **A user reports "error at 3:14 PM" and you can't find it across 5 servers.**
   Centralize logs (Seq/ELK/App Insights), add correlation IDs shown on the error page so users can report them, and use distributed tracing.

---

## 5.5 Error Handling

### Development vs production
```csharp
if (app.Environment.IsDevelopment())
    app.UseDeveloperExceptionPage();          // default in Development anyway
else
    app.UseExceptionHandler("/Home/Error");   // friendly page, no stack trace

app.UseStatusCodePagesWithReExecute("/Home/Status/{0}");   // custom 404/403 pages
```

### `IExceptionHandler` (.NET 8+)
```csharp
public class GlobalExceptionHandler(ILogger<GlobalExceptionHandler> logger) : IExceptionHandler
{
    public async ValueTask<bool> TryHandleAsync(HttpContext ctx, Exception ex, CancellationToken ct)
    {
        logger.LogError(ex, "Unhandled exception");
        var (status, title) = ex switch
        {
            NotFoundException => (StatusCodes.Status404NotFound, "Not found"),
            ValidationException => (StatusCodes.Status400BadRequest, "Validation failed"),
            _ => (StatusCodes.Status500InternalServerError, "Server error")
        };
        ctx.Response.StatusCode = status;
        await ctx.Response.WriteAsJsonAsync(new ProblemDetails { Status = status, Title = title }, ct);
        return true;
    }
}

builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
builder.Services.AddProblemDetails();
app.UseExceptionHandler();
```

### Error page action
```csharp
[ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
public IActionResult Error()
{
    var feature = HttpContext.Features.Get<IExceptionHandlerPathFeature>();
    return View(new ErrorViewModel { RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier });
}
```

### Guidelines
- Don't use exceptions for normal control flow (e.g., "not found" can be a null/Result).
- Catch specific exceptions where you can handle them; let others bubble to the global handler.
- Return **ProblemDetails** (RFC 9457) for APIs.

### 🎯 Interview Questions
1. **How do you implement global exception handling in ASP.NET Core?**
   `UseExceptionHandler` with an error path or `IExceptionHandler` implementations, plus `AddProblemDetails` for APIs; exception filters for MVC-specific handling.
2. **`UseStatusCodePages` vs `UseExceptionHandler`?**
   Exception handler deals with thrown exceptions (500). Status code pages handle responses with error status codes but no body (404, 403).
3. **What is ProblemDetails?**
   A standard JSON format for HTTP API errors with `type`, `title`, `status`, `detail`, `instance`, and extensions.
4. **Exception filter vs exception middleware?**
   Filters only catch exceptions from MVC actions/filters; middleware catches everything in the pipeline after it.

### 🧩 Scenario-Based Questions
1. **A stack trace with connection strings appeared to users in production.**
   The environment was set to Development or the developer exception page is always on. Set `ASPNETCORE_ENVIRONMENT=Production`, use `UseExceptionHandler`, and audit config.
2. **The API returns HTML error pages to a mobile app.**
   Use ProblemDetails/JSON for `/api` routes (`[ApiController]`, `AddProblemDetails`), or branch error handling by path or `Accept` header.

---

## ✅ Class 5 Homework
1. Register services with correct lifetimes; demonstrate and fix a captive dependency with `ValidateScopes`.
2. Write a correlation ID middleware and request timing middleware.
3. Bind `SmtpOptions` with validation on start; store the password in user secrets.
4. Add Serilog with structured logs and a global `IExceptionHandler` + custom 404 page.

---
---

# 🔴 CLASS 6 — Security: Identity, Authentication, Authorization & Threats

**Goal:** Secure an MVC application end to end.

---

## 6.1 Authentication vs Authorization & ASP.NET Core Identity

### Definitions
- **Authentication (AuthN):** Who are you? → produces a `ClaimsPrincipal` (`User`).
- **Authorization (AuthZ):** What can you do? → checks claims, roles, policies.
- **Claims:** key/value facts about the user (`email`, `role`, `department`).

### ASP.NET Core Identity
A membership system: users, password hashing (PBKDF2), roles, claims, lockout, 2FA, email confirmation, external logins.

```bash
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
dotnet add package Microsoft.AspNetCore.Identity.UI
```
```csharp
public class AppUser : IdentityUser { public string? FullName { get; set; } }
public class AppDbContext(DbContextOptions<AppDbContext> o) : IdentityDbContext<AppUser>(o) { ... }

builder.Services.AddDefaultIdentity<AppUser>(o =>
{
    o.SignIn.RequireConfirmedAccount = true;
    o.Password.RequiredLength = 10;
    o.Lockout.MaxFailedAccessAttempts = 5;
    o.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(15);
})
.AddRoles<IdentityRole>()
.AddEntityFrameworkStores<AppDbContext>();

builder.Services.AddRazorPages();   // Identity UI uses Razor Pages
...
app.MapRazorPages();
```
Scaffold and customize pages: `dotnet aspnet-codegenerator identity` (or Visual Studio "Add → New Scaffolded Item → Identity").

### Key services
- `UserManager<TUser>` — create users, passwords, roles, claims.
- `SignInManager<TUser>` — sign in/out, 2FA, lockout.
- `RoleManager<TRole>` — manage roles.

### Custom login action
```csharp
[HttpPost, AllowAnonymous, ValidateAntiForgeryToken]
public async Task<IActionResult> Login(LoginVm vm, string? returnUrl = null)
{
    if (!ModelState.IsValid) return View(vm);
    var result = await _signIn.PasswordSignInAsync(vm.Email, vm.Password, vm.RememberMe, lockoutOnFailure: true);
    if (result.Succeeded) return LocalRedirect(returnUrl ?? "/");
    if (result.RequiresTwoFactor) return RedirectToAction("LoginWith2fa", new { returnUrl, vm.RememberMe });
    if (result.IsLockedOut) return View("Lockout");
    ModelState.AddModelError("", "Invalid login attempt.");   // don't reveal which part was wrong
    return View(vm);
}
```

### 🎯 Interview Questions
1. **Authentication vs authorization?**
   AuthN establishes identity; AuthZ decides access. AuthN happens first.
2. **How does Identity store passwords?**
   As salted hashes (PBKDF2 with many iterations by default) — never plain text or reversible encryption.
3. **What is a ClaimsPrincipal?**
   The user object containing one or more `ClaimsIdentity` instances, each with claims. Available as `User` in controllers and views.
4. **Why lock out accounts and why generic login errors?**
   Lockout slows brute force attacks; generic messages prevent user enumeration.
5. **What is the security stamp?**
   A value that changes when credentials change; cookies are revalidated against it so changing a password can sign out other sessions.

### 🧩 Scenario-Based Questions
1. **Business wants "Login with Google and Microsoft" in addition to local accounts.**
   Add external providers (`AddGoogle`, `AddMicrosoftAccount`) to Identity; register apps in each provider; link external logins to local accounts; handle email collisions carefully.
2. **Enterprise customers want single sign-on with their Azure AD (Microsoft Entra ID).**
   Use OpenID Connect (`Microsoft.Identity.Web`) — the app delegates authentication to Entra ID; map tenant/groups to roles/claims.
3. **An admin's password was leaked. How do you kill all their active sessions?**
   Change the password and call `UserManager.UpdateSecurityStampAsync`; configure `SecurityStampValidatorOptions.ValidationInterval` to a short interval.

---

## 6.2 Cookie Authentication & JWT Bearer

### Cookie authentication (classic MVC)
```csharp
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(o =>
    {
        o.LoginPath = "/Account/Login";
        o.AccessDeniedPath = "/Account/Denied";
        o.ExpireTimeSpan = TimeSpan.FromHours(8);
        o.SlidingExpiration = true;
        o.Cookie.HttpOnly = true;
        o.Cookie.SecurePolicy = CookieSecurePolicy.Always;
        o.Cookie.SameSite = SameSiteMode.Lax;
    });

// Manual sign-in without Identity
var claims = new List<Claim> { new(ClaimTypes.Name, user.Email), new(ClaimTypes.Role, "Admin") };
var identity = new ClaimsIdentity(claims, CookieAuthenticationDefaults.AuthenticationScheme);
await HttpContext.SignInAsync(new ClaimsPrincipal(identity));
```

### JWT bearer (APIs, SPAs, mobile)
```csharp
builder.Services.AddAuthentication()
    .AddJwtBearer(o =>
    {
        o.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true, ValidIssuer = cfg["Jwt:Issuer"],
            ValidateAudience = true, ValidAudience = cfg["Jwt:Audience"],
            ValidateLifetime = true, ClockSkew = TimeSpan.FromMinutes(1),
            ValidateIssuerSigningKey = true,
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(cfg["Jwt:Key"]!))
        };
    });

[Authorize(AuthenticationSchemes = JwtBearerDefaults.AuthenticationScheme)]
[ApiController, Route("api/[controller]")]
public class OrdersApiController : ControllerBase { ... }
```

### Cookie vs JWT
| | Cookie | JWT |
|---|---|---|
| Storage | Browser-managed, HttpOnly | Client stores (memory/secure storage) |
| Best for | Server-rendered MVC, same-site | APIs, mobile, third-party clients |
| CSRF risk | Yes (mitigate with anti-forgery, SameSite) | Not if sent via Authorization header |
| Revocation | Easy (server state/security stamp) | Hard — use short expiry + refresh tokens |

### 🎯 Interview Questions
1. **What is a JWT and its parts?**
   A signed token: Header.Payload.Signature (Base64URL). The payload carries claims. It's signed, not encrypted — don't put secrets in it.
2. **How do you revoke a JWT?**
   Keep access tokens short-lived, use refresh tokens stored/revocable server-side, or keep a denylist of token IDs (`jti`).
3. **Why HttpOnly and Secure cookies?**
   HttpOnly blocks JavaScript access (XSS token theft); Secure ensures cookies only travel over HTTPS.
4. **Can one app use both cookies and JWT?**
   Yes — register both schemes and choose per endpoint with `[Authorize(AuthenticationSchemes = ...)]` or a policy scheme selector.
5. **Symmetric vs asymmetric JWT signing?**
   Symmetric (HMAC) shares one secret; asymmetric (RSA/ECDSA) signs with a private key and validates with a public key — better when multiple services validate tokens.

### 🧩 Scenario-Based Questions
1. **A SPA stores JWTs in `localStorage` and a security review flags it.**
   `localStorage` is readable by any XSS. Prefer the BFF (Backend-for-Frontend) pattern: the server holds tokens and the SPA uses an HttpOnly, SameSite cookie to the BFF.
2. **Users complain they get logged out every 20 minutes while actively working.**
   Enable `SlidingExpiration`, adjust `ExpireTimeSpan`, check the data protection keys aren't regenerated on restart or differ across servers (persist keys to shared storage).

---

## 6.3 Authorization: Roles, Claims, Policies & Resource-Based

```csharp
[Authorize]                                   // any authenticated user
[Authorize(Roles = "Admin,Manager")]          // role-based (either)
[Authorize(Policy = "CanManageInventory")]    // policy-based
[AllowAnonymous]                              // override for an action
```

### Policies
```csharp
builder.Services.AddAuthorizationBuilder()
    .AddPolicy("CanManageInventory", p => p.RequireClaim("permission", "inventory.manage"))
    .AddPolicy("Adult", p => p.Requirements.Add(new MinimumAgeRequirement(18)))
    .SetFallbackPolicy(new AuthorizationPolicyBuilder().RequireAuthenticatedUser().Build());
```

### Custom requirement + handler
```csharp
public record MinimumAgeRequirement(int Age) : IAuthorizationRequirement;

public class MinimumAgeHandler : AuthorizationHandler<MinimumAgeRequirement>
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext ctx, MinimumAgeRequirement req)
    {
        var dob = ctx.User.FindFirst(ClaimTypes.DateOfBirth)?.Value;
        if (dob is not null && DateTime.Today.Year - DateTime.Parse(dob).Year >= req.Age)
            ctx.Succeed(req);
        return Task.CompletedTask;
    }
}
builder.Services.AddSingleton<IAuthorizationHandler, MinimumAgeHandler>();
```

### Resource-based authorization
```csharp
public class OrderOwnerHandler : AuthorizationHandler<OperationAuthorizationRequirement, Order>
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext ctx,
        OperationAuthorizationRequirement req, Order order)
    {
        if (order.UserId == ctx.User.FindFirstValue(ClaimTypes.NameIdentifier) || ctx.User.IsInRole("Admin"))
            ctx.Succeed(req);
        return Task.CompletedTask;
    }
}

// In the controller
var order = await _orders.GetAsync(id);
var result = await _authz.AuthorizeAsync(User, order, Operations.Read);
if (!result.Succeeded) return Forbid();
```

### In views
```cshtml
@inject IAuthorizationService Authz
@if ((await Authz.AuthorizeAsync(User, "CanManageInventory")).Succeeded)
{
    <a asp-area="Admin" asp-controller="Inventory" asp-action="Index">Inventory</a>
}
```

### 🎯 Interview Questions
1. **Role-based vs claims-based vs policy-based authorization?**
   Roles are coarse groups. Claims are attributes. Policies combine requirements (roles, claims, custom logic) into named rules — the most flexible and recommended.
2. **What is resource-based authorization?**
   Decisions depend on the specific resource (e.g., only the owner can edit this order). Done imperatively with `IAuthorizationService.AuthorizeAsync(User, resource, requirement)`.
3. **What is a fallback policy?**
   A policy applied to endpoints without any `[Authorize]`/`[AllowAnonymous]` — e.g., require authentication everywhere by default (secure by default).
4. **Does hiding a button in the view secure the action?**
   No. Always enforce authorization on the server action as well.
5. **What is IDOR?**
   Insecure Direct Object Reference — changing an ID in the URL (`/orders/1002`) to access someone else's data. Fixed with resource-based checks or by scoping queries to the current user.

### 🧩 Scenario-Based Questions
1. **A customer changed `/Orders/Details/105` to `/106` and saw another customer's order.**
   IDOR. Filter queries by current user (`Where(o => o.UserId == userId)`) or apply resource-based authorization; add tests; consider non-sequential IDs as defense in depth (not a fix by itself).
2. **Permissions must be configurable by admins at runtime (no redeploy), like "Editor can publish".**
   Store role→permission mappings in the DB, add permission claims via `IClaimsTransformation` (cached), and create a dynamic policy provider (`IAuthorizationPolicyProvider`) for `[Authorize(Policy = "perm:books.publish")]`.
3. **New controllers keep getting deployed without `[Authorize]`.**
   Set a fallback policy requiring authenticated users, and mark public endpoints explicitly with `[AllowAnonymous]`. Add an architecture test that checks every controller.

---

## 6.4 Common Web Threats & Defenses (OWASP)

| Threat | What happens | Defense in ASP.NET Core |
|---|---|---|
| **XSS** | Attacker's script runs in victims' browsers | Razor auto-encoding; avoid `Html.Raw`; sanitize HTML; CSP header; HttpOnly cookies |
| **CSRF** | A malicious site makes the user's browser submit a form | Anti-forgery tokens (`[ValidateAntiForgeryToken]`, `AutoValidateAntiforgeryTokenAttribute`), SameSite cookies |
| **SQL Injection** | Malicious SQL in input | EF Core/LINQ, parameterized queries; never concatenate |
| **Overposting** | Extra form fields modify protected properties | ViewModels/DTOs |
| **Open Redirect** | `returnUrl` redirects to phishing site | `LocalRedirect`, `Url.IsLocalUrl` |
| **IDOR / Broken access control** | Access other users' data | Resource-based authorization |
| **Sensitive data exposure** | Secrets/PII leaked | HTTPS, HSTS, encryption, Key Vault, minimal logging |
| **Brute force** | Password guessing | Lockout, rate limiting, 2FA, CAPTCHA |
| **Clickjacking** | Site framed by attacker | `X-Frame-Options: DENY` / CSP `frame-ancestors` |
| **Vulnerable dependencies** | Known CVEs in packages | `dotnet list package --vulnerable`, Dependabot |

### Global anti-forgery for MVC
```csharp
builder.Services.AddControllersWithViews(o =>
    o.Filters.Add(new AutoValidateAntiforgeryTokenAttribute()));   // validates all unsafe verbs
```
For AJAX: send the token in a header (`RequestVerificationToken`) and configure `AddAntiforgery(o => o.HeaderName = "RequestVerificationToken")`.

### Security headers
```csharp
app.Use(async (ctx, next) =>
{
    ctx.Response.Headers["X-Content-Type-Options"] = "nosniff";
    ctx.Response.Headers["X-Frame-Options"] = "DENY";
    ctx.Response.Headers["Referrer-Policy"] = "strict-origin-when-cross-origin";
    ctx.Response.Headers["Content-Security-Policy"] = "default-src 'self'; frame-ancestors 'none'";
    await next();
});
```

### CORS
```csharp
builder.Services.AddCors(o => o.AddPolicy("Spa", p =>
    p.WithOrigins("https://app.bookstore.com").AllowAnyHeader().AllowAnyMethod().AllowCredentials()));
app.UseCors("Spa");   // after UseRouting, before UseAuthorization
```
`AllowAnyOrigin()` cannot be combined with `AllowCredentials()`.

### Rate limiting (.NET 7+)
```csharp
builder.Services.AddRateLimiter(o =>
{
    o.RejectionStatusCode = StatusCodes.Status429TooManyRequests;
    o.AddFixedWindowLimiter("login", l => { l.PermitLimit = 5; l.Window = TimeSpan.FromMinutes(1); });
});
app.UseRateLimiter();
// [EnableRateLimiting("login")] on the Login action
```

### HTTPS & HSTS
`UseHttpsRedirection()` + `UseHsts()` (production only; HSTS tells browsers to use HTTPS for a max-age period).

### 🎯 Interview Questions
1. **How does anti-forgery protection work?**
   The server issues a token in a cookie and a matching token in the form (or header). On POST, both must be present and match; a cross-site attacker can't read the token to forge it.
2. **Is CSRF a concern for JWT APIs using the Authorization header?**
   Generally no, because browsers don't attach that header automatically. It is a concern when tokens/cookies are sent automatically.
3. **What does CORS do — is it a security feature for the server?**
   CORS relaxes the browser's same-origin policy for specified origins. It protects users in browsers; it does not stop non-browser clients from calling your API.
4. **What is HSTS?**
   A response header instructing browsers to only use HTTPS for the domain for a period, preventing downgrade/SSL-stripping attacks.
5. **What is a Content Security Policy?**
   A header that restricts sources of scripts, styles, frames, etc., reducing XSS impact.

### 🧩 Scenario-Based Questions
1. **A pentest shows a stored XSS through book reviews.**
   Find where reviews are rendered with `Html.Raw` or inserted into JavaScript/attributes unencoded; encode output (use `JsonSerializer`/`@Json.Serialize` for JS contexts); sanitize rich text; add a CSP; review all `Html.Raw` usage.
2. **The login endpoint receives 10,000 attempts per minute from many IPs.**
   Credential stuffing. Enable rate limiting (per IP and per username), account lockout, 2FA, bot detection/CAPTCHA, WAF rules, breached-password checks, and alerting.
3. **An AJAX POST fails with 400 after enabling `AutoValidateAntiforgeryToken`.**
   The AJAX call doesn't send the anti-forgery token. Read it from `@Html.AntiForgeryToken()`/`IAntiforgery.GetAndStoreTokens` and send it in the configured header.

---

## 6.5 Data Protection & Secrets

### Data Protection API
Used internally for cookies, anti-forgery tokens, and TempData. You can also use it directly:
```csharp
public class TokenService(IDataProtectionProvider provider)
{
    private readonly IDataProtector _p = provider.CreateProtector("BookStore.Invites.v1");
    public string Protect(string data) => _p.Protect(data);
    public string Unprotect(string data) => _p.Unprotect(data);
}
// time-limited: provider.CreateProtector("x").ToTimeLimitedDataProtector()
```

### Multi-server key sharing
```csharp
builder.Services.AddDataProtection()
    .SetApplicationName("BookStore")
    .PersistKeysToAzureBlobStorage(blobUri)        // or PersistKeysToDbContext / FileSystem / Redis
    .ProtectKeysWithAzureKeyVault(keyId, credential);
```

### 🎯 Interview Questions
1. **What is the Data Protection API used for?**
   Encrypting and signing data such as auth cookies, anti-forgery tokens, and TempData; it manages key rotation automatically.
2. **Why do users get logged out after every deployment or across load-balanced servers?**
   Data protection keys aren't persisted/shared, so new instances can't decrypt cookies. Persist keys to shared storage.
3. **Hashing vs encryption?**
   Hashing is one-way (passwords). Encryption is reversible with a key (data you need to read back).

### 🧩 Scenario-Based Questions
1. **After moving to Kubernetes with 3 pods, users get "antiforgery token could not be decrypted".**
   Each pod has its own keys. Persist keys to shared storage (Redis/Blob/DB), set the same application name, and protect the keys at rest.
2. **You need password reset links that expire in 1 hour.**
   Use Identity's `GeneratePasswordResetTokenAsync` (token lifespan configurable via `DataProtectionTokenProviderOptions.TokenLifespan`), or a time-limited data protector for custom tokens.

---

## ✅ Class 6 Homework
1. Add ASP.NET Core Identity with roles (Admin, Customer); seed an admin user.
2. Protect the Admin area with a policy; implement resource-based authorization for orders.
3. Add global anti-forgery validation, security headers, and a rate-limited login.
4. Add a JWT-protected `/api/orders` endpoint alongside cookie-auth MVC pages.
5. Persist data protection keys and test with two app instances.

---
---

# 🟣 CLASS 7 — Expert Topics: Filters, Caching, Performance, APIs, Testing, Architecture & Deployment

**Goal:** Build, test, scale and ship production-grade applications — and talk about them like a senior engineer.

---

## 7.1 Filters Deep Dive

### Filter pipeline (execution order)
```
Authorization filters → Resource filters → [Model binding] → Action filters
→ Action executes → Action filters (after) → Exception filters (on error)
→ Result filters → Result executes → Result filters (after) → Resource filters (after)
```

| Filter | Interface | Typical use |
|---|---|---|
| Authorization | `IAuthorizationFilter` / `IAsyncAuthorizationFilter` | Custom auth checks (prefer policies) |
| Resource | `IResourceFilter` | Caching, short-circuit before model binding |
| Action | `IActionFilter` / `IAsyncActionFilter` | Validation, logging, argument inspection |
| Exception | `IExceptionFilter` | Translate MVC exceptions to results |
| Result | `IResultFilter` | Modify results, add headers |
| Endpoint filters | `IEndpointFilter` | Minimal APIs |

### Async action filter
```csharp
public class ValidateModelFilter : IAsyncActionFilter
{
    public async Task OnActionExecutionAsync(ActionExecutingContext ctx, ActionExecutionDelegate next)
    {
        if (!ctx.ModelState.IsValid)
        {
            ctx.Result = ctx.Controller is Controller c
                ? c.View(ctx.ActionArguments.Values.FirstOrDefault())
                : new BadRequestObjectResult(ctx.ModelState);
            return;                                   // short-circuit
        }
        var executed = await next();                  // run the action
        // executed.Exception / executed.Result available here
    }
}
```

### Registration & scope
```csharp
builder.Services.AddControllersWithViews(o => o.Filters.Add<ValidateModelFilter>()); // global
[ServiceFilter(typeof(AuditFilter))]   // resolved from DI (must be registered)
[TypeFilter(typeof(AuditFilter), Arguments = ["Books"])]   // DI + extra args, not registered
```
Order: Global → Controller → Action (on the way in), reversed on the way out; override with `Order` property.

### 🎯 Interview Questions
1. **List filter types in execution order.**
   Authorization → Resource → Action → (Exception) → Result.
2. **Resource filter vs action filter?**
   Resource filters run before model binding (can short-circuit early, e.g., cache); action filters run after model binding and can access action arguments.
3. **`ServiceFilter` vs `TypeFilter`?**
   `ServiceFilter` resolves a registered filter from DI. `TypeFilter` creates the filter via `ActivatorUtilities`, injecting dependencies and allowing extra constructor arguments without registration.
4. **How do you short-circuit a filter?**
   Set `context.Result` and don't call `next()`.
5. **Filter vs middleware — which runs first?**
   Middleware runs first; filters run within the MVC endpoint execution.

### 🧩 Scenario-Based Questions
1. **Audit logging is required for every create/update/delete on admin controllers, recording user, action, and arguments.**
   An async action filter registered on the Admin area base controller (or as a convention), with sensitive arguments redacted, writing to an audit table via a scoped service; or capture changes at the EF level with a `SaveChanges` interceptor for data-level auditing.
2. **An API endpoint must reject requests missing a valid `X-Api-Key`.**
   An authorization filter or, better, a custom authentication handler/policy so it integrates with `[Authorize]`; keep keys hashed in storage and rotateable.

---

## 7.2 Caching

| Type | API | Where | Use |
|---|---|---|---|
| In-memory | `IMemoryCache` | Single server memory | Lookups, small data |
| Distributed | `IDistributedCache` (Redis, SQL Server) | Shared | Multi-server, sessions |
| Hybrid (.NET 9+) | `HybridCache` | L1 memory + L2 distributed | Best of both, stampede protection |
| Response caching | `[ResponseCache]` + middleware | Client/proxy (HTTP headers) | Public, cacheable responses |
| Output caching (.NET 7+) | `AddOutputCache`, `[OutputCache]` | Server | Cache rendered responses with tags/eviction |
| Tag helper | `<cache>`, `<distributed-cache>` | Server | Fragments of a view |

```csharp
// In-memory with cache-aside
public async Task<IReadOnlyList<Category>> GetCategoriesAsync()
    => await _cache.GetOrCreateAsync("categories", async e =>
    {
        e.AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(30);
        e.SlidingExpiration = TimeSpan.FromMinutes(5);
        return await _db.Categories.AsNoTracking().ToListAsync();
    }) ?? [];

// HybridCache
builder.Services.AddHybridCache();
public async Task<BookDto?> GetBookAsync(int id, CancellationToken ct) =>
    await _hybrid.GetOrCreateAsync($"book:{id}",
        async token => await _repo.GetDtoAsync(id, token),
        tags: ["books"], cancellationToken: ct);
// invalidate: await _hybrid.RemoveByTagAsync("books");

// Output caching
builder.Services.AddOutputCache(o => o.AddPolicy("Catalog", p => p.Expire(TimeSpan.FromMinutes(5)).Tag("books")));
app.UseOutputCache();
[OutputCache(PolicyName = "Catalog")] public IActionResult Index() => View();
// evict: await _outputCacheStore.EvictByTagAsync("books", ct);

// Redis
builder.Services.AddStackExchangeRedisCache(o => o.Configuration = cfg.GetConnectionString("Redis"));
```

### 🎯 Interview Questions
1. **In-memory vs distributed cache?**
   In-memory is fastest but per-server and lost on restart; distributed is shared across servers and survives app restarts, but adds network latency and serialization.
2. **Response caching vs output caching?**
   Response caching relies on HTTP cache headers and clients/proxies honoring them. Output caching stores responses on the server, ignores client cache headers, and supports tag-based invalidation.
3. **What is a cache stampede and how do you prevent it?**
   Many requests miss an expired key simultaneously and all hit the DB. Prevent with locking/single-flight (HybridCache does this), staggered expirations, or background refresh.
4. **Absolute vs sliding expiration?**
   Absolute expires at a fixed time; sliding resets on access. Combine them so hot items still refresh eventually.
5. **What are the hard problems with caching?**
   Invalidation and consistency, per-user data leakage, memory pressure, and stale data.

### 🧩 Scenario-Based Questions
1. **Home page hits the DB 50 times per request and traffic spikes on sale days.**
   Cache category menus and bestsellers (HybridCache/Redis), output-cache the anonymous home page with tag eviction on product changes, use a CDN for static assets, and load-test.
2. **After enabling output caching, a logged-in user saw another user's name in the header.**
   Personalized content was cached. Output caching by default skips authenticated requests, but custom policies may override that; vary by user or exclude personalized fragments (render the user widget separately or via AJAX).
3. **Prices updated by admin take 30 minutes to appear.**
   Invalidate on write: remove the specific cache keys or evict by tag in the update handler; publish invalidation events across servers if using local caches.

---

## 7.3 Performance & Scalability

### Checklist
- **Async I/O** everywhere; no sync-over-async.
- **EF Core:** `AsNoTracking`, projections, pagination, indexes, avoid N+1, compiled queries for hot paths, `DbContext` pooling (`AddDbContextPool`).
- **Response compression** (`AddResponseCompression`) — often handled by the reverse proxy/CDN.
- **Static assets:** `MapStaticAssets` (.NET 9+), CDN, bundling/minification.
- **Avoid large object allocations;** use `ArrayPool`, `Span<T>`, streaming for big payloads.
- **`IHttpClientFactory`** with resilience (retry, timeout, circuit breaker).
- **Background work** offloaded to queues (`Channel<T>`, Hangfire, Azure Service Bus).
- **Measure first:** BenchmarkDotNet, `dotnet-counters`, `dotnet-trace`, Application Insights, load testing (k6, JMeter, NBomber).
- **Stateless app servers** for horizontal scaling (state in DB/Redis).

### Background services
```csharp
public class EmailQueueWorker(Channel<EmailMessage> channel, IServiceScopeFactory scopes,
    ILogger<EmailQueueWorker> logger) : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        await foreach (var msg in channel.Reader.ReadAllAsync(stoppingToken))
        {
            try
            {
                using var scope = scopes.CreateScope();
                var sender = scope.ServiceProvider.GetRequiredService<IEmailSender>();
                await sender.SendAsync(msg, stoppingToken);
            }
            catch (Exception ex) { logger.LogError(ex, "Failed to send email to {To}", msg.To); }
        }
    }
}

builder.Services.AddSingleton(Channel.CreateUnbounded<EmailMessage>());
builder.Services.AddHostedService<EmailQueueWorker>();
```
In-memory queues lose messages on restart — use a durable queue/outbox for critical work.

### 🎯 Interview Questions
1. **How do you find a performance bottleneck in an ASP.NET Core app?**
   Measure: APM traces to find slow endpoints/dependencies, SQL logs and execution plans, `dotnet-counters` (thread pool, GC, exceptions), `dotnet-trace`/profilers for CPU, memory dumps for leaks. Fix the biggest measured issue first.
2. **What is DbContext pooling?**
   `AddDbContextPool` reuses DbContext instances to reduce allocation overhead in high-throughput apps; the context must not hold per-request state in custom fields.
3. **`IHostedService` vs `BackgroundService`?**
   `BackgroundService` is a base class implementing `IHostedService` with a single `ExecuteAsync` long-running method.
4. **What is thread pool starvation?**
   Threads are blocked (sync waits), so new work queues up, causing latency spikes. Fix by making code async.
5. **What makes an app horizontally scalable?**
   Stateless servers, shared state in external stores, shared data protection keys, distributed cache, idempotent operations, and no reliance on local disk.

### 🧩 Scenario-Based Questions
1. **Memory grows steadily until the app restarts every few hours.**
   Capture memory dumps (`dotnet-gcdump`/`dotnet-dump`) and compare; common causes: unbounded static/memory caches, event handler leaks, singleton holding scoped objects, undisposed `HttpClient`/streams, large tracked DbContexts. Set cache size limits.
2. **Generating an invoice PDF takes 10 seconds and blocks the user.**
   Queue the job in a background worker (durable queue), return immediately with "processing" status, notify via SignalR/email when ready.
3. **An external shipping API is slow and sometimes down, causing your checkout to fail.**
   Add timeouts, retries with jitter, and a circuit breaker (`AddStandardResilienceHandler`); fall back gracefully (queue shipping calculation, show estimate); cache rates.

---

## 7.4 Building Web APIs Inside an MVC App

```csharp
[ApiController]
[Route("api/v1/[controller]")]
[Produces("application/json")]
public class BooksApiController(IBookService service) : ControllerBase
{
    [HttpGet]
    public async Task<ActionResult<PagedResult<BookDto>>> Get([FromQuery] int page = 1, [FromQuery] int size = 20)
        => Ok(await service.SearchAsync(null, page, Math.Clamp(size, 1, 100)));

    [HttpGet("{id:int}")]
    [ProducesResponseType<BookDto>(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<BookDto>> GetById(int id)
        => await service.GetDtoAsync(id) is { } dto ? dto : NotFound();

    [HttpPost]
    public async Task<ActionResult<BookDto>> Create(CreateBookDto dto)
    {
        var created = await service.CreateAsync(dto);
        return CreatedAtAction(nameof(GetById), new { id = created.Id }, created);
    }
}
```
- **OpenAPI:** .NET 9+ includes `Microsoft.AspNetCore.OpenApi` (`AddOpenApi()`, `MapOpenApi()`); use Swagger UI or Scalar for a UI.
- **Versioning:** `Asp.Versioning.Mvc` package (URL segment, header, or query).
- **Status codes:** 200 OK, 201 Created (+ Location), 204 No Content, 400, 401, 403, 404, 409 Conflict, 422, 429, 500.
- **Minimal APIs** are an alternative for small, high-performance endpoints.

### 🎯 Interview Questions
1. **What does `[ApiController]` provide?**
   Attribute routing requirement, automatic 400 responses for invalid models, binding source inference, ProblemDetails for errors.
2. **What should a successful POST return?**
   201 Created with a `Location` header (`CreatedAtAction`) and usually the created resource.
3. **PUT vs PATCH?**
   PUT replaces the whole resource (idempotent); PATCH partially updates (JSON Patch or merge patch).
4. **How do you version APIs?**
   URL path (`/api/v2`), query string, header, or media type. Use `Asp.Versioning` and keep old versions until clients migrate.
5. **Controllers vs Minimal APIs?**
   Controllers offer structure, filters, conventions — good for large apps. Minimal APIs are lightweight with less ceremony and slightly better performance; they support endpoint filters and route groups.

### 🧩 Scenario-Based Questions
1. **A mobile app breaks when you rename a JSON property.**
   Introduce API versioning, keep v1 contracts stable (DTOs, not entities), add contract tests, and publish deprecation timelines.
2. **An endpoint returns 100,000 books and clients time out.**
   Enforce pagination with max page size, add filtering, return lightweight DTOs, enable compression, and consider streaming (`IAsyncEnumerable<T>`).

---

## 7.5 Testing

### Test pyramid
Many **unit tests** → fewer **integration tests** → few **end-to-end/UI tests** (Playwright).

### Unit testing a controller/service (xUnit + Moq/NSubstitute)
```csharp
public class BooksControllerTests
{
    [Fact]
    public async Task Details_ReturnsNotFound_WhenBookMissing()
    {
        var service = new Mock<IBookService>();
        service.Setup(s => s.GetAsync(42)).ReturnsAsync((BookDetailsVm?)null);
        var controller = new BooksController(service.Object);

        var result = await controller.Details(42);

        Assert.IsType<NotFoundResult>(result);
    }

    [Fact]
    public async Task Create_RedisplaysView_WhenModelInvalid()
    {
        var controller = new BooksController(Mock.Of<IBookService>());
        controller.ModelState.AddModelError("Title", "Required");

        var result = await controller.Create(new BookCreateVm());

        var view = Assert.IsType<ViewResult>(result);
        Assert.IsType<BookCreateVm>(view.Model);
    }
}
```

### Integration testing with `WebApplicationFactory`
```csharp
public class BooksPageTests(WebApplicationFactory<Program> factory)
    : IClassFixture<WebApplicationFactory<Program>>
{
    [Fact]
    public async Task Index_ReturnsSuccessAndHtml()
    {
        var client = factory.WithWebHostBuilder(b => b.ConfigureServices(services =>
        {
            // replace real DB with test DB (SQLite in-memory or Testcontainers)
        })).CreateClient();

        var response = await client.GetAsync("/Books");

        response.EnsureSuccessStatusCode();
        Assert.Equal("text/html; charset=utf-8", response.Content.Headers.ContentType?.ToString());
    }
}
// Add `public partial class Program { }` at the end of Program.cs so the test project can reference it.
```

### Testing tools
- **xUnit / NUnit / MSTest** — frameworks
- **Moq / NSubstitute** — mocks
- **FluentAssertions / Shouldly** — readable assertions (check licensing for FluentAssertions v8+)
- **Testcontainers** — real SQL Server/Redis in Docker for integration tests
- **Bogus** — fake data
- **Playwright** — browser E2E tests
- **NetArchTest / ArchUnitNET** — architecture rules

### 🎯 Interview Questions
1. **Unit vs integration tests?**
   Unit tests isolate a class with mocked dependencies — fast. Integration tests run real components together (pipeline, DB) — slower but catch wiring issues.
2. **What is `WebApplicationFactory`?**
   Boots the app in memory with a test server for integration tests, allowing service overrides and an `HttpClient` to call endpoints.
3. **Why is EF Core InMemory provider discouraged for testing?**
   It doesn't behave like a relational DB (no constraints, transactions, or SQL translation). Use SQLite in-memory or Testcontainers with the real engine.
4. **What should you test in a controller?**
   Thin logic: status codes, chosen views, redirects, ModelState handling. Business logic should be tested in services.
5. **Mock vs stub vs fake?**
   Stub returns canned data; mock also verifies interactions; fake is a working lightweight implementation (in-memory repo).

### 🧩 Scenario-Based Questions
1. **Tests pass locally but the production query fails due to a SQL translation error.**
   Tests used the InMemory provider. Switch integration tests to the real database engine via Testcontainers.
2. **A legacy controller with no tests needs refactoring.**
   Write characterization (integration) tests capturing current behavior first, then refactor in small steps, extracting services with unit tests.

---

## 7.6 Architecture: Clean Architecture, CQRS & Design Principles

### SOLID quick reference
- **S**ingle Responsibility — one reason to change
- **O**pen/Closed — extend without modifying
- **L**iskov Substitution — subtypes are substitutable
- **I**nterface Segregation — small, focused interfaces
- **D**ependency Inversion — depend on abstractions

### Clean Architecture layout
```
src/
├── BookStore.Domain          → Entities, value objects, domain events, rules (no dependencies)
├── BookStore.Application     → Use cases, interfaces (IBookRepository, IEmailSender), DTOs, validators
├── BookStore.Infrastructure  → EF Core, repositories, email, file storage, external APIs
└── BookStore.Web             → MVC controllers, views, DI composition root
tests/
├── BookStore.UnitTests
└── BookStore.IntegrationTests
```
Dependencies point **inward**: Web → Application → Domain; Infrastructure → Application/Domain.

### Alternatives
- **Vertical Slice Architecture:** organize by feature (`Features/Books/Create`) — each slice owns its request, handler, validation, and view.
- **Modular monolith:** bounded modules with clear contracts in one deployable; can split into microservices later.

### CQRS with a mediator
```csharp
public record GetBookQuery(int Id) : IRequest<BookDto?>;

public class GetBookHandler(AppDbContext db) : IRequestHandler<GetBookQuery, BookDto?>
{
    public Task<BookDto?> Handle(GetBookQuery q, CancellationToken ct) =>
        db.Books.AsNoTracking().Where(b => b.Id == q.Id)
          .Select(b => new BookDto(b.Id, b.Title, b.Price)).FirstOrDefaultAsync(ct);
}

// Controller
public async Task<IActionResult> Details(int id) =>
    await _mediator.Send(new GetBookQuery(id)) is { } dto ? View(dto) : NotFound();
```
Note: MediatR moved to a commercial license in 2025 — alternatives include writing simple handler interfaces yourself or libraries like Wolverine/Mediator (source-generated).

### Other patterns to know
- **Result pattern** instead of exceptions for expected failures
- **Domain events** & **Outbox pattern** for reliable messaging
- **Specification pattern** for reusable query criteria
- **Decorator** (e.g., caching repository via Scrutor)

### 🎯 Interview Questions
1. **Explain Clean Architecture.**
   Concentric layers with business rules at the center; outer layers (UI, DB, frameworks) depend on inner abstractions, so core logic is independent and testable.
2. **What is CQRS? Do you always need separate databases?**
   Separating commands (writes) from queries (reads) into different models/handlers. No — separate databases are optional; most apps use CQRS within one DB.
3. **Clean Architecture vs Vertical Slice?**
   Clean organizes by technical layer with strict dependency rules; Vertical Slice organizes by feature, minimizing cross-feature coupling. Many teams combine them.
4. **What is the Outbox pattern?**
   Save domain changes and outgoing messages in the same DB transaction; a background process publishes messages, guaranteeing at-least-once delivery without distributed transactions.
5. **Monolith vs microservices for a new product?**
   Usually start with a well-structured modular monolith; move to microservices when independent scaling, deployment, or team autonomy justifies the operational cost.

### 🧩 Scenario-Based Questions
1. **A startup with 3 developers wants microservices from day one.**
   Recommend a modular monolith with clear module boundaries and contracts; it's cheaper to operate and easier to change. Extract services later when there's a measured need.
2. **Order placement must publish an event to inventory and email services reliably.**
   Outbox pattern: write the order and the `OrderPlaced` message in one transaction; a background dispatcher publishes to a broker (Service Bus/RabbitMQ); consumers are idempotent.
3. **Interviewer: "Your application layer has 200 generic repository methods. What went wrong?"**
   Over-abstraction. Use query objects/handlers per use case, EF projections for reads, and keep repositories (if any) focused on aggregates.

---

## 7.7 Deployment, DevOps & Production Readiness

### Publish
```bash
dotnet publish -c Release -o ./publish
```

### Docker
```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build
WORKDIR /src
COPY . .
RUN dotnet restore && dotnet publish BookStore/BookStore.csproj -c Release -o /app

FROM mcr.microsoft.com/dotnet/aspnet:10.0
WORKDIR /app
COPY --from=build /app .
USER app
ENV ASPNETCORE_URLS=http://+:8080
EXPOSE 8080
ENTRYPOINT ["dotnet", "BookStore.dll"]
```
(Or `dotnet publish /t:PublishContainer` to build images without a Dockerfile.)

### Hosting options
- **IIS** (Windows) with the ASP.NET Core Hosting Bundle (in-process or out-of-process)
- **Linux + Nginx** reverse proxy with systemd
- **Azure App Service**, **Azure Container Apps**, **AKS/Kubernetes**, **AWS ECS/Elastic Beanstalk**

### Health checks
```csharp
builder.Services.AddHealthChecks()
    .AddDbContextCheck<AppDbContext>("db");
app.MapHealthChecks("/health/live", new HealthCheckOptions { Predicate = _ => false });
app.MapHealthChecks("/health/ready");
```

### CI/CD pipeline (GitHub Actions / Azure DevOps)
1. Restore → build → test (unit + integration with Testcontainers)
2. Security scan (`dotnet list package --vulnerable`, SAST, secret scanning)
3. Build container image → push to registry
4. Run DB migrations (idempotent script or migration bundle)
5. Deploy to staging → smoke tests → swap/promote to production (blue-green or canary)

### Production readiness checklist
- [ ] `ASPNETCORE_ENVIRONMENT=Production`, no developer exception page
- [ ] HTTPS, HSTS, security headers, secure cookies
- [ ] Secrets in a vault; data protection keys persisted
- [ ] Centralized logging, tracing, metrics, alerts
- [ ] Health checks wired to the load balancer/orchestrator
- [ ] Rate limiting, request size limits
- [ ] Database backups and tested restore
- [ ] Forwarded headers configured behind a proxy
- [ ] Graceful shutdown handled (`CancellationToken` in background services)

### 🎯 Interview Questions
1. **In-process vs out-of-process hosting on IIS?**
   In-process runs the app inside the IIS worker process (`w3wp.exe`) — faster. Out-of-process runs Kestrel separately with IIS as reverse proxy.
2. **Liveness vs readiness probes?**
   Liveness: is the process alive (restart if not)? Readiness: can it serve traffic now (DB reachable)? Don't include dependencies in liveness checks.
3. **How do you deploy without downtime?**
   Blue-green or rolling deployments with health checks, backward-compatible DB migrations (expand/contract), and deployment slots.
4. **Why run containers as a non-root user?**
   It limits damage if the app is compromised. .NET 8+ images include an `app` user.
5. **What is a framework-dependent vs self-contained deployment?**
   Framework-dependent requires the .NET runtime installed; self-contained bundles the runtime (bigger, no dependency on host).

### 🧩 Scenario-Based Questions
1. **A deployment renamed a column and the old version (still running during rollout) crashed.**
   Use the expand/contract pattern: add the new column, deploy code writing both, backfill, switch reads, then drop the old column in a later release.
2. **The app returns 502 Bad Gateway on IIS after deployment.**
   Check the Hosting Bundle is installed for the right .NET version, enable stdout logs in `web.config`, check the Event Viewer, confirm app pool settings (No Managed Code), and run `dotnet BookStore.dll` manually to see startup errors.
3. **Kubernetes keeps restarting the pods during a DB outage, making things worse.**
   The liveness probe checks the database. Move dependency checks to readiness; keep liveness lightweight.

---

## ✅ Class 7 Homework
1. Add an audit action filter and an output-cached catalog with tag eviction on edits.
2. Add a background email queue worker.
3. Expose a versioned `/api/v1/books` with OpenAPI.
4. Write unit tests for controllers/services and integration tests with `WebApplicationFactory` + Testcontainers.
5. Restructure the solution into Clean Architecture projects.
6. Containerize the app, add health checks, and create a CI pipeline.

---
---

# 🏁 Capstone Project — BookStore Pro

Build a production-quality online bookstore that demonstrates everything from the course:

| Feature | Concepts exercised |
|---|---|
| Catalog with search, filters, pagination, slugs | Routing, EF Core projections, keyset pagination |
| Book details with reviews (rich text, sanitized) | Razor, partials, XSS defense |
| Cart (anonymous + logged in) and checkout | Session/distributed cache, PRG, idempotency, transactions |
| Customer accounts with 2FA and Google login | Identity, external auth |
| Order history with owner-only access | Resource-based authorization |
| Admin area: CRUD, inventory, reports, audit log | Areas, policies, filters, concurrency, background jobs |
| Public REST API with JWT | `[ApiController]`, versioning, OpenAPI |
| Email notifications via queue | BackgroundService, outbox |
| Caching and performance tuning | HybridCache, output cache, profiling |
| Tests, CI/CD, Docker, health checks | xUnit, Testcontainers, GitHub Actions |

**Evaluation criteria:** correctness, security, code organization, test coverage of business logic, performance under a basic load test, and a short architecture write-up explaining your decisions.

---

# 🎤 Final Mock Interview — Rapid-Fire Round

**Fundamentals**
1. Walk through the lifecycle of an MVC request from Kestrel to rendered HTML.
2. What's the difference between `AddControllersWithViews` and `AddMvc`?
3. Why does `UseAuthorization` need to be after `UseRouting`?

**Data**
4. How do you detect and fix N+1 queries?
5. How do you handle concurrent edits in EF Core?
6. When would you choose Dapper over EF Core? *(Performance-critical read queries, complex hand-tuned SQL; many apps use both.)*

**Framework internals**
7. Explain DI lifetimes and a bug you'd get from mixing them.
8. Middleware vs filters — give an example of each.
9. `IOptionsMonitor` vs `IOptionsSnapshot`?

**Security**
10. How do you prevent CSRF, XSS, SQL injection, overposting, and open redirects in ASP.NET Core?
11. Cookies or JWT for an MVC site plus a mobile app?
12. How would you implement permission-based authorization editable at runtime?

**Expert**
13. Output cache vs response cache vs HybridCache.
14. How do you make the app scale horizontally to 10 instances? List everything that must change.
15. Design an order placement flow that never loses an order and never double-charges.
16. How do you diagnose a production memory leak?
17. How would you migrate a large .NET Framework MVC 5 app to .NET 10 without a big-bang rewrite?

### Senior-level scenario (whiteboard)
> *"Black Friday is in 3 weeks. Last year the site crashed at 5× normal traffic. Walk me through your plan."*

**Strong answer outline:**
1. **Measure:** load test to reproduce, establish baseline, find bottlenecks with APM.
2. **Database:** fix slow queries, add indexes, read replicas for catalog reads, connection pool sizing.
3. **Caching:** CDN for static assets, output cache for anonymous pages, HybridCache/Redis for catalog data with tag invalidation.
4. **App tier:** ensure statelessness, autoscaling rules, shared data protection keys, distributed session.
5. **Resilience:** timeouts/circuit breakers on payment/shipping APIs, queue non-critical work (emails, invoices), rate limiting to protect from abuse.
6. **Checkout safety:** idempotency keys, outbox pattern, inventory reservation with concurrency control.
7. **Operations:** dashboards, alerts, feature flags to switch off heavy features, runbooks, code freeze, and a rehearsed rollback.

---

# 🗺️ Learning Roadmap After This Course

| Stage | Topics |
|---|---|
| **Next** | Minimal APIs, Blazor, SignalR (real-time), gRPC |
| **Data** | Advanced EF Core (interceptors, compiled models, temporal tables), Dapper, Redis, Cosmos DB |
| **Cloud** | Azure App Service, Container Apps, Key Vault, Service Bus, .NET Aspire |
| **Architecture** | DDD, event-driven design, microservices, message brokers (RabbitMQ, Kafka) |
| **Quality** | TDD, mutation testing, architecture tests, performance benchmarking |
| **Security** | OWASP Top 10 in depth, OAuth 2.0/OpenID Connect, threat modeling |

### Recommended references
- Microsoft Learn — ASP.NET Core documentation (learn.microsoft.com/aspnet/core)
- Microsoft Learn — Entity Framework Core documentation
- OWASP Cheat Sheet Series (.NET Security, CSRF, XSS)
- .NET Blog (devblogs.microsoft.com/dotnet) for release updates

---

> **Teaching tip:** Each class works well as a 3–4 hour session: ~60% concept + live coding, ~25% hands-on lab, ~15% interview/scenario discussion. Have students answer the scenario questions out loud — explaining trade-offs is exactly what interviewers look for.
