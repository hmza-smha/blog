[Back](./README.md)

---

## The Options Pattern in a nutshell

ASP.NET Core's options pattern is a way to turn configuration (appsettings.json, environment variables, code) into strongly-typed C# objects that get injected into your services. The interfaces you listed are the *pipeline stages* of building one of those objects.

Think of it as an assembly line for a settings object:

```
new TOptions()  →  Configure  →  PostConfigure  →  Validate  →  handed to you via IOptions<T>
```

---

### `IOptions<TOptions>`

The **consumer** side. This is what you inject to *read* the settings.

```csharp
public class EmailService
{
    private readonly SmtpSettings _settings;
    public EmailService(IOptions<SmtpSettings> options) => _settings = options.Value;
}
```

Key traits:
- Registered as a **singleton**, so `.Value` is computed once and cached forever. Changes to appsettings.json after startup are **not** picked up.
- Cannot be used with named options.
- Can be injected into singleton services safely.

Its two siblings matter here:

| Interface | Lifetime | Reloads config? | Named options? |
|---|---|---|---|
| `IOptions<T>` | Singleton | No | No |
| `IOptionsSnapshot<T>` | Scoped | Yes, once per request | Yes |
| `IOptionsMonitor<T>` | Singleton | Yes, live + `OnChange` callback | Yes |

Use `IOptionsMonitor<T>` when a singleton needs fresh values; use `IOptionsSnapshot<T>` in controllers/scoped services.

---

### `IConfigureOptions<TOptions>`

The **first stage** of the assembly line. It mutates a freshly constructed `TOptions` instance.

When you call `services.Configure<SmtpSettings>(Configuration.GetSection("Smtp"))`, the framework is really registering an `IConfigureOptions<SmtpSettings>` behind the scenes. You implement it yourself when the configuration needs **dependencies from DI** — something the lambda overload can't easily do.

```csharp
public class ConfigureSmtpSettings : IConfigureOptions<SmtpSettings>
{
    private readonly ISecretStore _secrets;   // DI works here
    public ConfigureSmtpSettings(ISecretStore secrets) => _secrets = secrets;

    public void Configure(SmtpSettings options)
    {
        options.Host = "smtp.contoso.com";
        options.Password = _secrets.Get("smtp-pw");
    }
}

// registration
services.AddSingleton<IConfigureOptions<SmtpSettings>, ConfigureSmtpSettings>();
```

You can register **many** of these for the same `TOptions`. They all run, **in registration order**, each seeing the mutations of the previous one. For named options there's `IConfigureNamedOptions<TOptions>`.

---

### `IPostConfigureOptions<TOptions>`

The **second stage**. Runs after *every* `IConfigureOptions` has finished — including ones registered by third-party libraries you don't control.

This is the "last word" hook. Use it for:
- Applying defaults only if the user didn't set something
- Overriding a value a library forced on you
- Deriving one property from others

```csharp
services.PostConfigure<SmtpSettings>(o =>
{
    if (string.IsNullOrEmpty(o.From))
        o.From = $"noreply@{o.Host}";
});
```

ASP.NET Core itself leans on this heavily — e.g. JWT bearer auth uses `IPostConfigureOptions<JwtBearerOptions>` to build the default token validation parameters after your `AddJwtBearer(...)` lambda ran.

---

### `IValidateOptions<TOptions>`

The **third stage**. Returns `ValidateOptionsResult.Success` or `ValidateOptionsResult.Fail(...)`. Runs lazily the first time `.Value` is accessed, unless you opt into eager validation.

```csharp
public class SmtpSettingsValidator : IValidateOptions<SmtpSettings>
{
    public ValidateOptionsResult Validate(string? name, SmtpSettings options)
    {
        var failures = new List<string>();
        if (string.IsNullOrWhiteSpace(options.Host))
            failures.Add("Smtp:Host is required.");
        if (options.Port is < 1 or > 65535)
            failures.Add($"Smtp:Port {options.Port} is out of range.");

        return failures.Count > 0
            ? ValidateOptionsResult.Fail(failures)
            : ValidateOptionsResult.Success;
    }
}

services.AddSingleton<IValidateOptions<SmtpSettings>, SmtpSettingsValidator>();
```

Lighter-weight alternatives:

```csharp
services.AddOptions<SmtpSettings>()
        .Bind(Configuration.GetSection("Smtp"))
        .ValidateDataAnnotations()          // [Required], [Range] on the POCO
        .Validate(o => o.Port > 0, "Port must be positive")
        .ValidateOnStart();                 // fail at startup, not at first use
```

`ValidateOnStart()` is almost always what you want — a misconfigured app should refuse to boot rather than throw `OptionsValidationException` at 3am on the first request that touches it.

> Note: you wrote `ValidateOptions<TOptions>` — the interface is `IValidateOptions<TOptions>`. There *is* a concrete helper class `ValidateOptions<TOptions>` (and generic variants `ValidateOptions<TOptions, TDep>`) used internally by the `.Validate(...)` fluent method, but you rarely reference it directly.

---

## How this differs from middleware

These are **orthogonal concepts** that people conflate because both are "things you register in `Program.cs`."

**Middleware** is about the *HTTP request pipeline*. It's a chain of components, each wrapping the next, that get a chance to inspect/modify the request on the way in and the response on the way out:

```csharp
app.UseAuthentication();
app.UseAuthorization();
app.UseMiddleware<RequestLoggingMiddleware>();
app.MapControllers();
```

```csharp
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    public RequestLoggingMiddleware(RequestDelegate next) => _next = next;

    public async Task InvokeAsync(HttpContext context)
    {
        // before
        await _next(context);
        // after
    }
}
```

The differences that actually matter:

| Aspect | Options interfaces | Middleware |
|---|---|---|
| **Concern** | Building & validating configuration objects | Handling HTTP requests |
| **When it runs** | At app startup / first `.Value` access | Once per HTTP request |
| **Knows about HTTP?** | No — zero `HttpContext` awareness | Yes — `HttpContext` is its whole job |
| **Ordering model** | Sequential stages: Configure → PostConfigure → Validate | Russian-doll nesting: each wraps the next, code runs before *and* after `_next()` |
| **Registration** | `services.*` (`IServiceCollection`, DI container) | `app.Use*` / `app.UseMiddleware<>` (`IApplicationBuilder`) |
| **Can short-circuit?** | No (validation can throw, but there's nothing to short-circuit) | Yes — skip calling `_next()` and the rest of the pipeline never runs |
| **Typical lifetime** | Singleton | Singleton instance, per-request `Invoke` |

The relationship between them is one-directional: **middleware consumes options, not the other way around.** A middleware typically takes `IOptions<T>` (or `IOptionsMonitor<T>` if it must react to config changes) in its constructor:

```csharp
public class ApiKeyMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IOptionsMonitor<ApiKeySettings> _options;

    public ApiKeyMiddleware(RequestDelegate next, IOptionsMonitor<ApiKeySettings> options)
        => (_next, _options) = (next, options);

    public async Task InvokeAsync(HttpContext ctx)
    {
        var expected = _options.CurrentValue.Key;   // fresh on every request
        if (ctx.Request.Headers["X-Api-Key"] != expected)
        {
            ctx.Response.StatusCode = 401;
            return;                                  // short-circuit
        }
        await _next(ctx);
    }
}
```

One gotcha: middleware constructors are resolved **once** at pipeline build time, so injecting `IOptionsSnapshot<T>` (scoped) into a middleware constructor throws. Either use `IOptionsMonitor<T>` in the constructor, or resolve the scoped service from `HttpContext.RequestServices` inside `InvokeAsync`.

---

## Full comparison table

| | `IOptions<T>` | `IConfigureOptions<T>` | `IPostConfigureOptions<T>` | `IValidateOptions<T>` | Middleware |
|---|---|---|---|---|---|
| **Role** | Consume the finished object | Build/populate the object | Final adjustments after all Configure | Approve or reject the object | Handle an HTTP request |
| **Direction** | Read | Write | Write | Read-only check | Read/write `HttpContext` |
| **Pipeline order** | — (endpoint of the chain) | 1st | 2nd | 3rd | N/A — different pipeline |
| **Runs when** | First `.Value` access (cached) | During options construction | After all `IConfigureOptions` | After PostConfigure | Every request |
| **Multiple allowed?** | N/A | Yes, run in registration order | Yes, run in registration order | Yes, **all** run, failures aggregate | Yes, nested |
| **DI supported** | Injected | Yes (constructor injection) | Yes | Yes | Yes (constructor + `InvokeAsync` params) |
| **Named options** | No (use `IOptionsSnapshot`/`Monitor`) | Via `IConfigureNamedOptions<T>` | Yes (`name` param, `null` = all) | Yes (`name` param) | N/A |
| **Registered on** | Auto (`AddOptions`) | `IServiceCollection` | `IServiceCollection` | `IServiceCollection` | `IApplicationBuilder` |
| **Typical API** | ctor injection | `services.Configure<T>(...)` | `services.PostConfigure<T>(...)` | `.Validate()` / `.ValidateDataAnnotations()` / `.ValidateOnStart()` | `app.UseMiddleware<T>()` |
| **On failure** | Throws `OptionsValidationException` | Exception bubbles | Exception bubbles | Returns `Fail(...)` → exception at `.Value` | Whatever you code (500, 401, etc.) |
| **HTTP-aware** | ❌ | ❌ | ❌ | ❌ | ✅ |

---

**Rule of thumb:** the three `I*Options` interfaces are a *construction pipeline* for a config object; `IOptions<T>` is the *delivery mechanism* for the result; middleware is a completely separate *request pipeline* that happens to be one of the many consumers of that result.

---
**`IOptions<T>`** — Use when a service just needs to read settings that won't change after startup. Reach for `IOptionsSnapshot<T>` instead if you need per-request reloads, or `IOptionsMonitor<T>` if a singleton needs live values.

**`IConfigureOptions<T>`** — Use when populating your options requires DI services (a secret store, `IHttpClientFactory`, a DB lookup) that a `services.Configure<T>(o => ...)` lambda can't cleanly reach. For plain config binding, stick with `Configure<T>(section)`.

**`IPostConfigureOptions<T>`** — Use when you need the last word: overriding a value a library set, or deriving one property from others after all binding is done. Classic case is fixing up `JwtBearerOptions` after `AddJwtBearer(...)` has run.

**`IValidateOptions<T>`** — Use when validation is cross-field or non-trivial (e.g. "if `UseTls` is true then `CertPath` is required"). For simple rules, `.ValidateDataAnnotations()` or `.Validate(o => ...)` is enough — and pair either with `.ValidateOnStart()`.

**Middleware** — Use when logic must run on every HTTP request and can inspect or short-circuit it: auth checks, logging, correlation IDs, exception handling, response compression.
---
## Scenario Questions

**Q: You need to convert all incoming `DateTime` values to UTC across every request. What do you use?**
A: A custom `JsonConverter<DateTime>` registered in `AddJsonOptions` (or a model binder / `IModelBinderProvider` for form & query values). **Not middleware** — by the time middleware runs, the body is still a raw stream; and after MVC binds it, middleware can't reach the bound model. The converter hooks the exact moment the string becomes a `DateTime`.

```csharp
builder.Services.AddControllers().AddJsonOptions(o =>
    o.JsonSerializerOptions.Converters.Add(new UtcDateTimeConverter()));
```

---

**Q: You need to reject requests missing an `X-Api-Key` header before they reach any controller. What do you use?**
A: Middleware — it runs per-request, sees `HttpContext`, and can short-circuit by not calling `_next()`. Place it early in the pipeline.

---

**Q: Your `SmtpSettings.Password` must come from Azure Key Vault, not appsettings. What do you use?**
A: `IConfigureOptions<SmtpSettings>` — it supports constructor injection, so you can inject the secret client. A `Configure<T>(o => ...)` lambda can't cleanly resolve DI services.

---

**Q: A third-party library sets `JwtBearerOptions.TokenValidationParameters` and you need to override one property afterward. What do you use?**
A: `IPostConfigureOptions<JwtBearerOptions>` / `services.PostConfigure<T>(...)` — it runs after every `IConfigureOptions`, so you get the last word.

---

**Q: The app should refuse to start if `ConnectionStrings:Default` is empty. What do you use?**
A: `.Validate(...)` or `IValidateOptions<T>` combined with **`.ValidateOnStart()`**. Without `ValidateOnStart`, validation is lazy and only throws on first `.Value` access.

---

**Q: A singleton background service must pick up appsettings.json changes without a restart. What do you inject?**
A: `IOptionsMonitor<T>` and read `.CurrentValue` each time (or subscribe via `OnChange`). `IOptions<T>` caches forever; `IOptionsSnapshot<T>` is scoped and can't be injected into a singleton.

---

**Q: You inject `IOptionsSnapshot<T>` into a middleware constructor and get an exception. Why?**
A: Middleware is instantiated once at pipeline-build time (effectively singleton), so a scoped service can't be captured. Use `IOptionsMonitor<T>` in the constructor, or resolve from `HttpContext.RequestServices` inside `InvokeAsync`.

---

**Q: You need to add a `X-Correlation-Id` header to every response. Middleware or filter?**
A: Middleware — it wraps the whole pipeline including static files, and can write headers before the response starts. Filters only run for MVC/controller endpoints.

---

**Q: You need to log the request body for POSTs to `/api/orders` only. Middleware or action filter?**
A: Action filter (or endpoint filter in Minimal APIs) — you get the already-bound model for free. Doing it in middleware means manually buffering the stream with `EnableBuffering()`.

---

**Q: Multiple `IConfigureOptions<T>` are registered for the same type. Which wins?**
A: They all run, in registration order, each mutating the same instance — so the **last one to set a property wins**. Then all `IPostConfigureOptions<T>` run, then validation.

---

**Q: You need different `HttpClient` timeout settings for "PaymentApi" and "ShippingApi" using the same options class. What do you use?**
A: **Named options** — `services.Configure<ApiSettings>("PaymentApi", section)` and retrieve with `IOptionsSnapshot<T>.Get("PaymentApi")`. `IOptions<T>` can't read named options.

---

**Q: What's the full order of operations when `IOptions<T>.Value` is first accessed?**
A: `new TOptions()` → all `IConfigureOptions<T>` → all `IPostConfigureOptions<T>` → all `IValidateOptions<T>` → cached and returned.

---

**Q: Global exception handling — middleware or filter?**
A: Middleware (`UseExceptionHandler` or custom), placed first, because it catches exceptions from the entire pipeline. An `IExceptionFilter` only catches exceptions thrown inside MVC actions.

---

**Q: Why does `UseAuthentication()` have to come before `UseAuthorization()`?**
A: Middleware is a nested chain — authentication populates `HttpContext.User`, and authorization reads it. Reversed, authorization sees an anonymous user.

---

**Q: When would you use `IValidateOptions<T>` over `.ValidateDataAnnotations()`?**
A: When the rule is cross-field or conditional — e.g. "if `UseTls == true` then `CertPath` is required." Data annotations only validate properties in isolation.
## Added Question

**Q: You need a `SystemContext` (or `ICurrentUser`) service exposing the current `UserId`, `TenantId`, `Language`, `TimeZone`, etc. — available anywhere in the app. What do you use?**

A: **A scoped service, not the options pattern.** Options are startup-time configuration; this is per-request ambient state that differs on every call. Two common approaches:

**Option 1 — Lazy read from `IHttpContextAccessor` (simplest):**

```csharp
public interface ISystemContext
{
    Guid? UserId { get; }
    Guid? TenantId { get; }
    string Language { get; }
}

public class SystemContext : ISystemContext
{
    private readonly IHttpContextAccessor _accessor;
    public SystemContext(IHttpContextAccessor accessor) => _accessor = accessor;

    private ClaimsPrincipal? User => _accessor.HttpContext?.User;

    public Guid? UserId => Guid.TryParse(User?.FindFirst("sub")?.Value, out var id) ? id : null;
    public Guid? TenantId => Guid.TryParse(User?.FindFirst("tenant_id")?.Value, out var id) ? id : null;
    public string Language => _accessor.HttpContext?.Request.Headers.AcceptLanguage.FirstOrDefault() ?? "en";
}

services.AddHttpContextAccessor();
services.AddScoped<ISystemContext, SystemContext>();
```

**Option 2 — Middleware populates a scoped, settable context (better when values need DB lookups):**

```csharp
public class SystemContextMiddleware
{
    private readonly RequestDelegate _next;
    public SystemContextMiddleware(RequestDelegate next) => _next = next;

    // scoped services come in via InvokeAsync params, NOT the constructor
    public async Task InvokeAsync(HttpContext ctx, SystemContext context, ITenantRepository repo)
    {
        context.UserId  = ...;
        context.TenantId = await repo.ResolveAsync(ctx.Request.Host);
        context.Language = ...;
        await _next(ctx);
    }
}
```

Register the concrete class once and expose the read-only interface from the same instance:

```csharp
services.AddScoped<SystemContext>();
services.AddScoped<ISystemContext>(sp => sp.GetRequiredService<SystemContext>());
```

**Key interview points:**
- **Scoped lifetime** — one instance per request, so every service in that request sees the same values.
- **Never inject it into a singleton** — that's a captive dependency and you'll leak one user's context across all requests. If a singleton needs it, resolve per-operation from an `IServiceScopeFactory`.
- **Inject scoped services into middleware via `InvokeAsync` parameters**, not the constructor (middleware is effectively a singleton).
- **Why not `IOptions<T>`?** Options are resolved from configuration and cached at startup — they have no concept of "the current request."
- Works outside HTTP too (background jobs, message consumers): create a scope manually and populate `SystemContext` from the job's metadata, and the rest of your code doesn't change.

[Back](./README.md)
