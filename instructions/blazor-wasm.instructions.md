---
description: "Blazor WebAssembly development using Service-Based Architecture with Direct Injection"
applyTo: "**/*.razor, **/*.razor.cs, **/*.razor.css, **/*.razor.js"
---

## Architecture Overview

Build Blazor WebAssembly applications using a Service-Based Architecture inspired by Microsoft's eShop reference application. This architecture uses direct service injection with `@inject`, scoped state services with EventCallback notifications, and a two-project structure (Client + ClientComponents) to separate application-specific code from reusable UI components. This approach eliminates MVVM abstraction layers, simplifies testing, improves performance, and maintains excellent code organization through feature-based structure.

## Architectural Principles

**Core Philosophy:**

- **Direct Injection**: Components use `@inject` to access services directly without ViewModel intermediaries
- **State Services**: Scoped services manage feature state and notify components via EventCallback pattern
- **Separation of Concerns**: Client project contains app-specific features; ClientComponents contains reusable UI
- **Hybrid Validation**: DataAnnotations for simple UI validation; FluentValidation for complex business rules
- **Feature Organization**: Group related functionality (pages, services, models) by feature for maintainability

## Project Structure

Follow the two-project architecture separating application-specific code from reusable components:

### Client Project (Application-Specific)

```
Client/
├── Components/
│   ├── Authentication/      # Auth-specific components (.razor, .razor.cs, .razor.css, .razor.js)
│   │   ├── LoginForm.razor
│   │   ├── LoginForm.razor.cs
│   │   └── LoginForm.razor.css
│   ├── UserProfile/         # User profile components
│   │   ├── ProfileEditor.razor
│   │   └── ProfileEditor.razor.cs
│   ├── Basket/              # Shopping basket components
│   └── [Feature]/           # Other feature-specific components
├── Pages/
│   ├── Authentication/      # Auth pages (.razor, .razor.cs, .razor.css, .razor.js)
│   │   ├── Login.razor
│   │   └── Register.razor
│   ├── UserProfile/         # User profile pages
│   │   ├── Profile.razor
│   │   └── EditProfile.razor
│   ├── Basket/              # Shopping basket pages
│   └── [Feature]/           # Other feature pages
├── Services/
│   ├── Authentication/      # Auth services (state + business)
│   │   ├── AuthenticationState.cs      # Scoped state service with EventCallback
│   │   ├── AuthenticationService.cs    # Scoped business service with HttpClient
│   │   └── IClaimsService.cs
│   ├── UserProfile/         # User profile services
│   │   ├── UserProfileState.cs         # Scoped state service
│   │   └── UserProfileService.cs       # Scoped business service
│   ├── Basket/              # Shopping basket services
│   │   ├── BasketState.cs
│   │   └── BasketService.cs
│   └── [Feature]/           # Other feature services
├── Extensions/              # Feature DI registration
│   ├── AuthenticationExtensions.cs     # AddAuthenticationFeature()
│   ├── UserProfileExtensions.cs        # AddUserProfileFeature()
│   ├── BasketExtensions.cs             # AddBasketFeature()
│   └── [Feature]Extensions.cs
├── Models/                  # DTOs and request models
│   ├── Authentication/
│   │   ├── LoginRequest.cs
│   │   └── RegisterRequest.cs
│   ├── UserProfile/
│   │   └── UpdateProfileRequest.cs
│   └── [Feature]/
├── Validators/              # FluentValidation validators
│   ├── Authentication/
│   │   ├── LoginRequestValidator.cs
│   │   └── RegisterRequestValidator.cs
│   └── UserProfile/
│       └── UpdateProfileRequestValidator.cs
├── Infrastructure/          # Cross-cutting concerns
│   ├── Authentication/      # Auth configuration and helpers
│   ├── Http/                # HTTP client configurations
│   ├── Storage/             # Local storage services
│   └── Telemetry/           # Telemetry services (singleton)
│       ├── ITelemetryService.cs
│       └── ApplicationInsightsTelemetryService.cs
├── Layouts/                 # Application layouts
│   ├── MainLayout.razor
│   └── MainLayout.razor.css
├── wwwroot/
│   ├── css/                 # Custom stylesheets
│   ├── js/                  # JavaScript interop files
│   └── assets/              # Static assets
├── _Imports.razor           # Global using statements
├── App.razor                # Root component with routing
├── Program.cs               # Application entry point with DI registration
└── Client.csproj
```

### ClientComponents Project (Reusable Component Library)

```
ClientComponents/
├── Components/
│   ├── Forms/               # Reusable form components
│   │   ├── ValidatedInput.razor
│   │   ├── ValidatedInput.razor.cs
│   │   ├── ValidatedTextArea.razor
│   │   ├── ValidatedSelect.razor
│   │   └── FormSection.razor
│   ├── Tables/              # Reusable data table components
│   │   ├── DataTable.razor          # Generic table with sorting/pagination
│   │   ├── DataTable.razor.cs
│   │   ├── PaginationControls.razor
│   │   └── SortableHeader.razor
│   ├── Cards/               # Reusable card layouts
│   │   ├── InfoCard.razor
│   │   ├── StatsCard.razor
│   │   └── ActionCard.razor
│   ├── Dialogs/             # Reusable dialog components
│   │   ├── ConfirmDialog.razor
│   │   ├── AlertDialog.razor
│   │   └── ConfirmDialog.razor.cs
│   └── Layout/              # Reusable layout components
│       ├── LoadingSpinner.razor
│       └── ErrorBoundary.razor
├── Tests/                   # 100% coverage requirement
│   ├── Forms/
│   │   ├── ValidatedInputTests.cs
│   │   └── FormSectionTests.cs
│   ├── Tables/
│   │   └── DataTableTests.cs
│   └── Cards/
│       └── InfoCardTests.cs
├── _Imports.razor
└── ClientComponents.csproj
```

### Tests Project

```
Client.Tests/
├── Components/              # bUnit component tests
│   ├── Authentication/
│   │   └── LoginFormTests.cs
│   └── UserProfile/
│       └── ProfileEditorTests.cs
├── Services/                # xUnit service tests
│   ├── Authentication/
│   │   ├── AuthenticationStateTests.cs
│   │   └── AuthenticationServiceTests.cs
│   └── UserProfile/
│       ├── UserProfileStateTests.cs
│       └── UserProfileServiceTests.cs
└── Validators/              # FluentValidation tests
    └── Authentication/
        └── LoginRequestValidatorTests.cs
```

### Key Architectural Notes

**Client Project Guidelines:**

- Feature-based organization: group Components, Pages, Services by feature
- State services use scoped lifetime (auto-cleanup on navigation)
- Business services use scoped lifetime with typed HttpClient
- Feature DI registration via extension methods (AddAuthenticationFeature, etc.)
- Validators use FluentValidation for complex business rules
- Components can inject state services and subscribe to OnChange events

**ClientComponents Project Guidelines:**

- Components must meet ALL 6 promotion criteria before inclusion
- Zero business logic - only presentation with EventCallback parameters
- Never inject services (no HttpClient, no state services, no localStorage)
- 100% bUnit test coverage required for all components
- Generic and reusable across different applications
- WASM-specific: stateless, no storage access, no HTTP calls

**Component Promotion Flow:**

1. Start in `Client/Components/[Feature]/` for feature-specific UI
2. When component used in 3+ features, evaluate for promotion
3. Verify ALL 6 criteria met (reusability, zero logic, stable API, generic, documented, tested)
4. Move to `ClientComponents/Components/[Category]/` with full test suite
5. Update Client project to reference shared component

## Component File Organization

Each Blazor component should follow this file organization pattern:

```
ComponentName/
├── ComponentName.razor          # Markup and data binding
├── ComponentName.razor.cs       # Code-behind with component logic
├── ComponentName.razor.css      # Scoped CSS styling
└── ComponentName.razor.js       # Scoped JavaScript (when needed)
```

## Service-Based Pattern Implementation

**State Services should:**

- Be registered as **scoped** services by default (better memory management, auto-cleanup, isolated testing)
- Use **singleton** only for truly global state (AppPreferencesState, CatalogCache, TelemetryService)
- Expose `public event Action? OnChange;` for reactive notifications
- Call `NotifyStateChanged()` method to trigger OnChange event after state updates
- Use `SemaphoreSlim` for thread-safe async operations
- Never directly reference Blazor components or UI elements
- Encapsulate feature-specific state and state transitions

**Business Services should:**

- Handle API calls, data operations, and business logic
- Be registered as scoped services
- Return data to state services for state updates
- Not maintain state themselves (delegate to state services)
- Use typed HttpClient with Polly resilience policies

**Components should:**

- Use `@inject` directive for direct service injection
- Subscribe to state service `OnChange` events in `OnInitialized`
- Call `StateHasChanged()` when state service notifies changes
- Implement `IDisposable` to unsubscribe from events
- Use scoped CSS (.razor.css) for component-specific styling
- Use scoped JavaScript (.razor.js) when JavaScript interop is required
- Delegate business logic to services, focus on UI rendering
- Use EventCallback parameters in shared components (never inject services in ClientComponents)

## State Service Pattern

### Complete State Service Example

```csharp
using System.Threading;

namespace Client.Services.UserProfile;

public class UserProfileState
{
    private readonly SemaphoreSlim _semaphore = new(1, 1);
    private UserProfileModel? _currentProfile;

    // Event for notifying components of state changes
    public event Action? OnChange;

    public UserProfileModel? CurrentProfile
    {
        get => _currentProfile;
        private set
        {
            _currentProfile = value;
            NotifyStateChanged();
        }
    }

    public async Task UpdateProfileAsync(UserProfileModel profile)
    {
        await _semaphore.WaitAsync();
        try
        {
            CurrentProfile = profile;
        }
        finally
        {
            _semaphore.Release();
        }
    }

    public void ClearProfile()
    {
        CurrentProfile = null;
    }

    private void NotifyStateChanged() => OnChange?.Invoke();
}
```

### Component Subscription Pattern

```csharp
using Microsoft.AspNetCore.Components;

namespace Client.Components.Pages.UserProfile;

public partial class UserProfile : ComponentBase, IDisposable
{
    [Inject] protected UserProfileState ProfileState { get; set; } = default!;
    [Inject] protected UserProfileService ProfileService { get; set; } = default!;

    protected override async Task OnInitializedAsync()
    {
        // Subscribe to state changes
        ProfileState.OnChange += StateHasChanged;

        // Load initial data
        await ProfileService.LoadProfileAsync();
    }

    private async Task HandleUpdateProfile()
    {
        await ProfileService.UpdateProfileAsync(model);
        // State service will notify via OnChange event
    }

    public void Dispose()
    {
        // Unsubscribe from events
        ProfileState.OnChange -= StateHasChanged;
    }
}
```

### Business Service Pattern

```csharp
namespace Client.Services.UserProfile;

public class UserProfileService
{
    private readonly UserProfileState _state;
    private readonly HttpClient _httpClient;
    private readonly ILogger<UserProfileService> _logger;

    public UserProfileService(
        UserProfileState state,
        HttpClient httpClient,
        ILogger<UserProfileService> logger)
    {
        _state = state;
        _httpClient = httpClient;
        _logger = logger;
    }

    public async Task LoadProfileAsync()
    {
        try
        {
            var profile = await _httpClient.GetFromJsonAsync<UserProfileModel>("api/profile");
            if (profile != null)
            {
                await _state.UpdateProfileAsync(profile);
            }
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to load user profile");
            throw;
        }
    }

    public async Task UpdateProfileAsync(UserProfileModel model)
    {
        try
        {
            var response = await _httpClient.PutAsJsonAsync("api/profile", model);
            response.EnsureSuccessStatusCode();

            await _state.UpdateProfileAsync(model);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to update user profile");
            throw;
        }
    }
}
```

## Service Lifetime and Scoping Rules

### Default: Scoped Services

**Use scoped registration for most services:**

```csharp
builder.Services.AddScoped<UserProfileState>();
builder.Services.AddScoped<UserProfileService>();
builder.Services.AddScoped<BasketState>();
builder.Services.AddScoped<BasketService>();
```

**Rationale for Scoped Default:**

- **Better Memory Management**: Services are disposed when navigation occurs, preventing memory leaks
- **Automatic Cleanup**: State resets naturally on page navigation, avoiding stale data
- **Isolated Testing**: Each test can have independent service instances
- **Future SSR Compatibility**: Scoped services align with Blazor Server and future hybrid rendering
- **Natural Reset Points**: State clears on navigation, reducing complexity of manual state management

### Exception: Singleton Services

**Use singleton only for truly global state:**

```csharp
builder.Services.AddSingleton<ITelemetryService, ApplicationInsightsTelemetryService>();
builder.Services.AddSingleton<AppPreferencesState>();  // User preferences persist across navigation
builder.Services.AddSingleton<CatalogCache>();          // Reference data shared across features
```

**When to Use Singleton:**

- **Global Application State**: Settings, preferences, configuration that persists across navigation
- **Shared Reference Data**: Catalogs, lookup tables, cached data used by multiple features
- **Analytics/Telemetry**: Services that aggregate data across the entire application session
- **Performance-Critical Caches**: Data that's expensive to reload and safe to share globally

**Warning**: Singleton services in WASM persist for the entire application lifetime. Ensure they properly manage memory and don't accumulate unbounded data.

## Scoped CSS Guidelines

**Scoped CSS (.razor.css) Benefits:**

- Automatic CSS isolation prevents style bleeding
- Component-specific styling without global conflicts
- Better maintainability and debugging
- Automatic generation of unique CSS selectors

**Scoped CSS Best Practices:**

```css
/* ComponentName.razor.css */

/* Component root styling */
.component-container {
  padding: 1rem;
  border-radius: 0.375rem;
  background-color: var(--bs-light);
}

/* Use CSS custom properties for theming */
.primary-button {
  background-color: var(--bs-primary);
  border-color: var(--bs-primary);
}

/* Deep selectors for child component styling */
::deep .child-component {
  margin-bottom: 1rem;
}

/* Responsive design within component */
@media (max-width: 768px) {
  .component-container {
    padding: 0.5rem;
  }
}

/* Component state classes */
.loading-state {
  opacity: 0.6;
  pointer-events: none;
}

.error-state {
  border-left: 4px solid var(--bs-danger);
  background-color: rgba(220, 53, 69, 0.1);
}
```

## Scoped JavaScript Guidelines

**Scoped JavaScript (.razor.js) Benefits:**

- Module-based JavaScript loading
- Automatic cleanup and disposal
- Component-specific JavaScript functionality
- Better performance through lazy loading

**Scoped JavaScript Best Practices:**

```javascript
// ComponentName.razor.js

// Export initialization function
export function initialize(element, dotNetHelper) {
  // Store reference to .NET helper for callbacks
  element._dotNetHelper = dotNetHelper;

  // Initialize component-specific functionality
  const initializeTooltips = () => {
    const tooltips = element.querySelectorAll('[data-bs-toggle="tooltip"]');
    return [...tooltips].map((el) => new bootstrap.Tooltip(el));
  };

  const tooltipInstances = initializeTooltips();

  // Return disposal function
  return {
    dispose: () => {
      tooltipInstances.forEach((tooltip) => tooltip.dispose());
      if (element._dotNetHelper) {
        element._dotNetHelper.dispose();
      }
    },
  };
}

// Export utility functions
export function updateComponentState(element, state) {
  element.classList.toggle("loading-state", state.isLoading);
  element.classList.toggle("error-state", state.hasError);
}
```

## Code-Behind Structure

```csharp
using Microsoft.AspNetCore.Components;
using Microsoft.JSInterop;

namespace Client.Components.Pages.FeatureName;

public partial class ComponentName : ComponentBase, IAsyncDisposable
{
    // Injected services (direct injection)
    [Inject] protected IJSRuntime JSRuntime { get; set; } = default!;
    [Inject] protected FeatureState State { get; set; } = default!;
    [Inject] protected FeatureService Service { get; set; } = default!;

    // Parameters
    [Parameter] public string Title { get; set; } = string.Empty;
    [Parameter] public EventCallback<string> OnValueChanged { get; set; }

    // JavaScript interop
    private IJSObjectReference? _jsModule;
    private IJSObjectReference? _jsInstance;
    private DotNetObjectReference<ComponentName>? _dotNetHelper;

    protected override async Task OnInitializedAsync()
    {
        // Subscribe to state service events
        State.OnChange += StateHasChanged;

        // Initialize component data
        await Service.LoadDataAsync();
    }

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            await InitializeJavaScriptAsync();
        }
    }

    private async Task InitializeJavaScriptAsync()
    {
        try
        {
            _jsModule = await JSRuntime.InvokeAsync<IJSObjectReference>(
                "import", "./Components/Pages/FeatureName/ComponentName.razor.js");

            _dotNetHelper = DotNetObjectReference.Create(this);

            _jsInstance = await _jsModule.InvokeAsync<IJSObjectReference>(
                "initialize", _dotNetHelper);
        }
        catch (Exception ex)
        {
            // Log error - JavaScript module failed to load
            Console.WriteLine($"Failed to initialize JavaScript: {ex.Message}");
        }
    }

    // Event handlers for user interactions
    private async Task HandleSubmit()
    {
        await Service.SubmitDataAsync();
        // State service will notify via OnChange event
    }

    // JavaScript callback methods
    [JSInvokable]
    public async Task HandleJavaScriptEvent(string eventData)
    {
        // Handle callbacks from JavaScript
        await OnValueChanged.InvokeAsync(eventData);
    }

    // Disposal
    public async ValueTask DisposeAsync()
    {
        // Dispose JavaScript resources
        if (_jsInstance != null)
        {
            try
            {
                await _jsInstance.InvokeVoidAsync("dispose");
                await _jsInstance.DisposeAsync();
            }
            catch { /* Ignore disposal errors */ }
        }

        if (_jsModule != null)
        {
            try
            {
                await _jsModule.DisposeAsync();
            }
            catch { /* Ignore disposal errors */ }
        }

        _dotNetHelper?.Dispose();

        // Unsubscribe from state service events
        State.OnChange -= StateHasChanged;
    }
}
```

## Dependency Injection Guidelines

### Feature-Based Service Registration

Each feature should have its own service registration extension method to keep DI configuration organized and maintainable.

**Feature Extension Pattern (Client/Extensions/):**

```csharp
namespace Client.Extensions;

public static class AuthenticationExtensions
{
    public static IServiceCollection AddAuthenticationFeature(
        this IServiceCollection services,
        IConfiguration configuration)
    {
        // Add MSAL authentication
        services.AddMsalAuthentication(options =>
        {
            configuration.Bind("AzureAdB2C", options.ProviderOptions.Authentication);
        });

        // Add authorization policies
        services.AddAuthorizationCore(options =>
        {
            options.AddPolicy("Admin", policy =>
            {
                policy.RequireClaim("jobTitle", "Admin");
            });

            options.AddPolicy("User", policy =>
            {
                policy.RequireAuthenticatedUser();
            });
        });

        // Register state services (scoped)
        services.AddScoped<AuthenticationState>();

        // Register business services (scoped)
        services.AddScoped<AuthenticationService>();
        services.AddScoped<IClaimsService, ClaimsService>();

        // Register validators for complex validation scenarios
        services.AddTransient<IValidator<LoginRequest>, LoginRequestValidator>();
        services.AddTransient<IValidator<RegisterRequest>, RegisterRequestValidator>();

        return services;
    }
}
```

**Another Feature Example (Client/Extensions/):**

```csharp
namespace Client.Extensions;

public static class UserProfileExtensions
{
    public static IServiceCollection AddUserProfileFeature(
        this IServiceCollection services,
        IConfiguration configuration)
    {
        // Configure typed HttpClient with resilience
        services.AddHttpClient<UserProfileService>(client =>
        {
            client.BaseAddress = new Uri(configuration["ApiSettings:BaseUrl"]);
            client.DefaultRequestHeaders.Add("Accept", "application/json");
        })
        .AddPolicyHandler(HttpPolicyExtensions
            .HandleTransientHttpError()
            .WaitAndRetryAsync(3, retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt))));

        // Register state service (scoped for navigation reset)
        services.AddScoped<UserProfileState>();

        // Register business service (scoped)
        services.AddScoped<UserProfileService>();

        // Register validators for complex business rules
        services.AddTransient<IValidator<UpdateProfileRequest>, UpdateProfileRequestValidator>();

        return services;
    }
}
```

**Program.cs Registration:**

```csharp
using Client.Extensions;

var builder = WebAssemblyHostBuilder.CreateDefault(args);

// Add root components
builder.RootComponents.Add<App>("#app");
builder.RootComponents.Add<HeadOutlet>("head::after");

// Add shared infrastructure services
builder.Services.AddScoped(sp => new HttpClient
{
    BaseAddress = new Uri(builder.HostEnvironment.BaseAddress)
});

// Add singleton services for global state
builder.Services.AddSingleton<ITelemetryService, ApplicationInsightsTelemetryService>();
builder.Services.AddSingleton<AppPreferencesState>();  // Global user preferences
builder.Services.AddSingleton<CatalogCache>();          // Shared reference data

builder.Services.AddApplicationInsightsTelemetry(options =>
{
    options.ConnectionString = builder.Configuration["ApplicationInsights:ConnectionString"];
});

// Add features using extension methods (registers scoped state and business services)
builder.Services.AddAuthenticationFeature(builder.Configuration);
builder.Services.AddUserProfileFeature(builder.Configuration);
builder.Services.AddBasketFeature(builder.Configuration);

await builder.Build().RunAsync();
```

### Service Registration Pattern

**Service Lifetimes:**

- **Scoped** (Default): State services, business services, HTTP clients - Reset on navigation
- **Singleton** (Exception): Global state, telemetry, reference data caches - Persist for app lifetime
- **Transient**: FluentValidation validators, lightweight utilities

**Feature Extension Benefits:**

- **Organized DI**: Each feature manages its own dependencies
- **Testability**: Easy to register mock services for testing
- **Modularity**: Features can be added/removed independently
- **Clarity**: Clear dependencies for each feature in one place

## Hybrid Validation Strategy

### Validation Philosophy

Use a two-tier validation approach that leverages the strengths of both DataAnnotations and FluentValidation:

- **DataAnnotations**: Simple property validation with immediate UI feedback via EditForm
- **FluentValidation**: Complex business rules, async validation, cross-property validation
- **Server-Side**: Always validate with FluentValidation for defense in depth

### DataAnnotations for Simple UI Validation

**When to Use DataAnnotations:**

- Simple property rules (required, length, range, regex patterns)
- Immediate UI feedback needed in EditForm
- Client-side validation without async operations
- Basic format validation (email, phone, URL)

**Model Example with DataAnnotations:**

```csharp
using System.ComponentModel.DataAnnotations;

namespace Client.Models.UserProfile;

public class UpdateProfileRequest
{
    [Required(ErrorMessage = "Email is required")]
    [EmailAddress(ErrorMessage = "Invalid email format")]
    [StringLength(256, ErrorMessage = "Email cannot exceed 256 characters")]
    public string Email { get; set; } = string.Empty;

    [Required(ErrorMessage = "First name is required")]
    [StringLength(50, MinimumLength = 2, ErrorMessage = "First name must be 2-50 characters")]
    [RegularExpression(@"^[a-zA-Z\s]+$", ErrorMessage = "First name can only contain letters")]
    public string FirstName { get; set; } = string.Empty;

    [Required(ErrorMessage = "Last name is required")]
    [StringLength(50, MinimumLength = 2, ErrorMessage = "Last name must be 2-50 characters")]
    public string LastName { get; set; } = string.Empty;

    [Phone(ErrorMessage = "Invalid phone format")]
    public string? PhoneNumber { get; set; }

    [Range(18, 120, ErrorMessage = "Age must be between 18 and 120")]
    public int Age { get; set; }
}
```

**Component with EditForm and DataAnnotations:**

```razor
@page "/profile/edit"
@inject UserProfileState ProfileState
@inject UserProfileService ProfileService

<EditForm Model="@model" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />

    <div class="mb-3">
        <label for="email" class="form-label">Email</label>
        <InputText id="email" class="form-control" @bind-Value="model.Email" />
        <ValidationMessage For="@(() => model.Email)" />
    </div>

    <div class="mb-3">
        <label for="firstName" class="form-label">First Name</label>
        <InputText id="firstName" class="form-control" @bind-Value="model.FirstName" />
        <ValidationMessage For="@(() => model.FirstName)" />
    </div>

    <button type="submit" class="btn btn-primary">Save</button>
</EditForm>

@code {
    private UpdateProfileRequest model = new();

    protected override async Task OnInitializedAsync()
    {
        if (ProfileState.CurrentProfile != null)
        {
            model.Email = ProfileState.CurrentProfile.Email;
            model.FirstName = ProfileState.CurrentProfile.FirstName;
            // ... map other fields
        }
    }

    private async Task HandleValidSubmit()
    {
        // DataAnnotations validation passed, now call service
        await ProfileService.UpdateProfileAsync(model);
    }
}
```

### FluentValidation for Complex Business Rules

**When to Use FluentValidation:**

- Complex cross-property validation
- Async validation (database checks, API calls)
- Conditional validation based on other properties
- Reusable validation logic with service dependencies
- Business rule validation beyond simple formats

**FluentValidation Example:**

```csharp
using FluentValidation;

namespace Client.Validators.UserProfile;

public class UpdateProfileRequestValidator : AbstractValidator<UpdateProfileRequest>
{
    private readonly HttpClient _httpClient;

    public UpdateProfileRequestValidator(HttpClient httpClient)
    {
        _httpClient = httpClient;

        // Simple rules (duplicates DataAnnotations for server-side defense)
        RuleFor(x => x.Email)
            .NotEmpty().WithMessage("Email is required")
            .EmailAddress().WithMessage("Invalid email format")
            .MaximumLength(256).WithMessage("Email cannot exceed 256 characters");

        // Async validation - check email uniqueness via API
        RuleFor(x => x.Email)
            .MustAsync(async (email, cancellation) =>
            {
                var response = await _httpClient.GetAsync($"api/users/check-email?email={email}");
                var result = await response.Content.ReadFromJsonAsync<bool>();
                return result; // true if available
            })
            .WithMessage("Email is already registered")
            .When(x => !string.IsNullOrEmpty(x.Email)); // Only check if email provided

        // Cross-property validation
        RuleFor(x => x.Age)
            .GreaterThanOrEqualTo(18).WithMessage("Must be at least 18 years old")
            .LessThanOrEqualTo(120).WithMessage("Age must be realistic");

        // Conditional validation
        RuleFor(x => x.PhoneNumber)
            .Matches(@"^\+?[1-9]\d{1,14}$").WithMessage("Invalid international phone format")
            .When(x => !string.IsNullOrEmpty(x.PhoneNumber));

        // Custom business rule with multiple properties
        RuleFor(x => x)
            .Must(HaveCompleteAddress)
            .WithMessage("Address information is incomplete")
            .When(x => x.RequiresShipping);
    }

    private bool HaveCompleteAddress(UpdateProfileRequest request)
    {
        return !string.IsNullOrEmpty(request.Street)
            && !string.IsNullOrEmpty(request.City)
            && !string.IsNullOrEmpty(request.PostalCode);
    }
}
```

**Server-Side Validation in API:**

```csharp
[HttpPut("api/profile")]
public async Task<IActionResult> UpdateProfile(
    [FromBody] UpdateProfileRequest request,
    [FromServices] IValidator<UpdateProfileRequest> validator)
{
    // Always validate on server with FluentValidation
    var validationResult = await validator.ValidateAsync(request);

    if (!validationResult.IsValid)
    {
        return BadRequest(validationResult.Errors);
    }

    // Proceed with update
    await _userService.UpdateProfileAsync(request);
    return Ok();
}
```

### Validation Decision Matrix

| Scenario                  | Use DataAnnotations | Use FluentValidation | Notes                                              |
| ------------------------- | ------------------- | -------------------- | -------------------------------------------------- |
| Required field            | ✅ Yes              | ✅ Yes (server)      | DataAnnotations for UI, FluentValidation on server |
| String length             | ✅ Yes              | ✅ Yes (server)      | Simple constraint, both work                       |
| Email format              | ✅ Yes              | ✅ Yes (server)      | Built-in validators in both                        |
| Email uniqueness check    | ❌ No               | ✅ Yes               | Requires async database query                      |
| Cross-property validation | ❌ No               | ✅ Yes               | Address completeness, password confirmation        |
| Conditional validation    | ❌ Limited          | ✅ Yes               | FluentValidation `.When()` is more flexible        |
| Regex pattern             | ✅ Yes              | ✅ Yes               | Both support patterns                              |
| Range validation          | ✅ Yes              | ✅ Yes               | Both support numeric ranges                        |
| Database-dependent rules  | ❌ No               | ✅ Yes               | FluentValidation with service injection            |
| Reusable validation logic | ❌ No               | ✅ Yes               | FluentValidation allows DI and composition         |

### Validation Flow

```
User Input → DataAnnotations (UI immediate feedback via EditForm)
          ↓
    Form Valid?
          ↓ Yes
    Submit to Server
          ↓
    FluentValidation (complex rules, async checks, defense in depth)
          ↓
    All Valid?
          ↓ Yes
    Process Request
```

## Authentication with Azure AD B2C

```csharp
public interface IAuthenticationService
{
    Task<bool> IsAuthenticatedAsync();
    Task<string> GetAccessTokenAsync();
    Task<IEnumerable<string>> GetUserRolesAsync();
    Task<bool> IsInRoleAsync(string role);
}

public class AzureB2CAuthenticationService : IAuthenticationService
{
    private readonly IAccessTokenProvider _tokenProvider;
    private readonly AuthenticationStateProvider _authStateProvider;

    public AzureB2CAuthenticationService(
        IAccessTokenProvider tokenProvider,
        AuthenticationStateProvider authStateProvider)
    {
        _tokenProvider = tokenProvider;
        _authStateProvider = authStateProvider;
    }

    public async Task<bool> IsAuthenticatedAsync()
    {
        var authState = await _authStateProvider.GetAuthenticationStateAsync();
        return authState.User.Identity.IsAuthenticated;
    }

    // ... other implementations
}
```

## Component Library Promotion Criteria

### When to Promote Components to ClientComponents

Components must meet **ALL** of the following criteria before promotion from `Client/Components` to the shared `ClientComponents` library:

#### 1. Reusability Threshold

- **Rule**: Component is used in **3 or more** different features or pages
- **Rationale**: Prevents premature abstraction; ensures genuine reusability
- **Example**: A `UserAvatar` component used in navbar, user list, and profile page qualifies

#### 2. Zero Business Logic

- **Rule**: Component contains **only presentation logic** with no feature-specific business rules
- **Rationale**: Shared components must be pure UI with behavior controlled by parameters
- **Good Example**: `DataTable` component that accepts data via parameters and emits events via EventCallback
- **Bad Example**: `UserTable` component that directly injects `UserService` and loads data internally

#### 3. Stable API

- **Rule**: Component parameters are **unlikely to change** and are well-documented
- **Rationale**: Frequent parameter changes indicate component isn't ready for sharing
- **Requirements**:
  - All `[Parameter]` properties have XML documentation
  - Parameter names are descriptive and follow conventions
  - Breaking changes would affect multiple features

#### 4. Generic and Flexible

- **Rule**: Component is **generic enough** to be useful in other applications
- **Rationale**: Component library should contain truly reusable UI, not app-specific widgets
- **Good Example**: `PaginatedTable<TItem>` that works with any data type
- **Bad Example**: `InvoiceStatusBadge` that's specific to invoicing domain

#### 5. Comprehensive Documentation

- **Rule**: Component has **complete XML documentation** and usage examples
- **Requirements**:
  - Summary describing component purpose
  - Documentation for all parameters
  - At least one code example in XML remarks
  - Any special behaviors or requirements documented

#### 6. 100% Test Coverage

- **Rule**: Component has **complete bUnit test coverage** before promotion
- **Rationale**: Shared components are dependencies for multiple features; must be reliable
- **Requirements**:
  - All rendering scenarios tested
  - All EventCallback invocations tested
  - Edge cases and error states covered
  - Parameter validation tested

### Good vs Bad Patterns for Shared Components

**✅ Good: EventCallback Parameters**

```csharp
// ClientComponents/Components/Forms/ConfirmDialog.razor.cs
public partial class ConfirmDialog
{
    [Parameter] public string Title { get; set; } = "Confirm";
    [Parameter] public string Message { get; set; } = string.Empty;
    [Parameter] public EventCallback<bool> OnConfirm { get; set; }

    private async Task HandleConfirm()
    {
        await OnConfirm.InvokeAsync(true);
    }
}
```

**❌ Bad: Service Injection in Shared Component**

```csharp
// BAD - Don't inject services in ClientComponents
public partial class UserSelector
{
    [Inject] private UserService UserService { get; set; } = default!; // ❌ NO!

    // This couples the component to specific feature logic
}
```

**✅ Good: Generic Data Handling**

```csharp
// ClientComponents/Components/Tables/DataTable.razor
@typeparam TItem

<table class="table">
    <thead>
        <tr>
            @foreach (var column in Columns)
            {
                <th>@column.Header</th>
            }
        </tr>
    </thead>
    <tbody>
        @foreach (var item in Items)
        {
            <tr @onclick="() => OnRowClick.InvokeAsync(item)">
                @RowTemplate(item)
            </tr>
        }
    </tbody>
</table>

@code {
    [Parameter] public IEnumerable<TItem> Items { get; set; } = Enumerable.Empty<TItem>();
    [Parameter] public RenderFragment<TItem> RowTemplate { get; set; } = default!;
    [Parameter] public EventCallback<TItem> OnRowClick { get; set; }
}
```

### WASM-Specific Rules for ClientComponents

**Critical Constraints:**

- **Never use `localStorage` directly** in ClientComponents - state management is app-specific
- **Never inject state services** - use EventCallback to communicate with parent components
- **Never make HTTP calls** - data fetching is feature-specific, not UI responsibility
- **Scoped services auto-dispose** - components should not assume service lifetime

### Promotion Checklist

Before promoting a component to ClientComponents, verify:

- [ ] Component is used in 3+ different features/pages
- [ ] Component contains zero business logic (pure presentation)
- [ ] All parameters have stable, well-documented APIs
- [ ] Component is generic enough for use in other applications
- [ ] Complete XML documentation with usage examples
- [ ] 100% bUnit test coverage with all scenarios
- [ ] Component uses EventCallback for parent communication (no service injection)
- [ ] Component does not use localStorage, HttpClient, or state services
- [ ] Component has been reviewed and approved for promotion

## Form Validation Best Practices

### Simple Forms with DataAnnotations

```csharp
// For simple forms, DataAnnotations with EditForm provide immediate feedback
public class LoginRequest
{
    [Required(ErrorMessage = "Email is required")]
    [EmailAddress(ErrorMessage = "Invalid email format")]
    public string Email { get; set; } = string.Empty;

    [Required(ErrorMessage = "Password is required")]
    [StringLength(100, MinimumLength = 8, ErrorMessage = "Password must be 8-100 characters")]
    public string Password { get; set; } = string.Empty;
}
```

### Complex Forms with FluentValidation

```csharp
// For complex validation, use FluentValidation
public class RegisterRequestValidator : AbstractValidator<RegisterRequest>
{
    public RegisterRequestValidator(HttpClient httpClient)
    {
        RuleFor(x => x.Email)
            .NotEmpty()
            .EmailAddress();

        RuleFor(x => x.Password)
            .NotEmpty()
            .MinimumLength(8)
            .Must(ContainUppercase).WithMessage("Password must contain uppercase")
            .Must(ContainNumber).WithMessage("Password must contain number");

        RuleFor(x => x.ConfirmPassword)
            .Equal(x => x.Password).WithMessage("Passwords must match");
    }

    private bool ContainUppercase(string password) => password.Any(char.IsUpper);
    private bool ContainNumber(string password) => password.Any(char.IsDigit);
}
```

## Local Storage and State Management

```csharp
public interface IStorageService
{
    Task<T> GetItemAsync<T>(string key);
    Task SetItemAsync<T>(string key, T value);
    Task RemoveItemAsync(string key);
    Task ClearAsync();
}

public class LocalStorageService : IStorageService
{
    private readonly IJSRuntime _jsRuntime;

    public LocalStorageService(IJSRuntime jsRuntime)
    {
        _jsRuntime = jsRuntime;
    }

    public async Task<T> GetItemAsync<T>(string key)
    {
        var json = await _jsRuntime.InvokeAsync<string>("localStorage.getItem", key);
        return json == null ? default : JsonSerializer.Deserialize<T>(json);
    }

    // ... other implementations
}
```

## Performance Optimizations

### 1. Lazy Loading

- Use `@page` directive with parameters for deep linking
- Implement lazy loading for large datasets using pagination
- Use `Virtualize` component for large lists

### 2. Memory Management

- Always implement `IDisposable` in components that subscribe to events
- Dispose of ViewModels properly
- Use weak references for long-lived event subscriptions

### 3. HTTP Client Optimization with Resilience

**Advanced Configuration with Polly:**

```csharp
// Install: Microsoft.Extensions.Http.Polly package

// Configure HttpClient with base address, default headers, and resilience policies
builder.Services.AddHttpClient<IUserService, UserService>(client =>
{
    client.BaseAddress = new Uri(builder.Configuration["ApiSettings:BaseUrl"]);
    client.DefaultRequestHeaders.Add("Accept", "application/json");
    client.Timeout = TimeSpan.FromSeconds(30);
})
.AddPolicyHandler(GetRetryPolicy())
.AddPolicyHandler(GetCircuitBreakerPolicy());

static IAsyncPolicy<HttpResponseMessage> GetRetryPolicy()
{
    return HttpPolicyExtensions
        .HandleTransientHttpError()
        .OrResult(msg => msg.StatusCode == System.Net.HttpStatusCode.TooManyRequests)
        .WaitAndRetryAsync(
            retryCount: 3,
            sleepDurationProvider: retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt)),
            onRetry: (outcome, timespan, retryAttempt, context) =>
            {
                // Log retry attempt
                Console.WriteLine($"Retry {retryAttempt} after {timespan.TotalSeconds}s");
            });
}

static IAsyncPolicy<HttpResponseMessage> GetCircuitBreakerPolicy()
{
    return HttpPolicyExtensions
        .HandleTransientHttpError()
        .CircuitBreakerAsync(
            handledEventsAllowedBeforeBreaking: 5,
            durationOfBreak: TimeSpan.FromSeconds(30));
}
```

**Service Implementation with Proper Error Handling:**

```csharp
public class UserService : IUserService
{
    private readonly HttpClient _httpClient;
    private readonly ILogger<UserService> _logger;

    public UserService(HttpClient httpClient, ILogger<UserService> logger)
    {
        _httpClient = httpClient;
        _logger = logger;
    }

    public async Task<IEnumerable<UserDto>> GetUsersAsync(CancellationToken cancellationToken = default)
    {
        try
        {
            var response = await _httpClient.GetAsync("api/users", cancellationToken);
            response.EnsureSuccessStatusCode();

            return await response.Content.ReadFromJsonAsync<IEnumerable<UserDto>>(cancellationToken)
                ?? Enumerable.Empty<UserDto>();
        }
        catch (HttpRequestException ex)
        {
            _logger.LogError(ex, "Failed to retrieve users from API");
            throw new ServiceException("Unable to retrieve users. Please try again later.", ex);
        }
        catch (TaskCanceledException ex)
        {
            _logger.LogWarning(ex, "Request to retrieve users was cancelled or timed out");
            throw new ServiceException("The request timed out. Please try again.", ex);
        }
    }
}
```

## Testing Guidelines

### Testing Framework Setup

**Recommended Testing Stack:**

- **Unit Testing**: xUnit or NUnit
- **Mocking**: Moq or NSubstitute
- **Assertions**: FluentAssertions
- **Blazor Component Testing**: bUnit
- **Code Coverage**: Coverlet

**Package Installation:**

```xml
<PackageReference Include="xunit" Version="2.6.0" />
<PackageReference Include="Moq" Version="4.20.0" />
<PackageReference Include="FluentAssertions" Version="6.12.0" />
<PackageReference Include="bUnit" Version="1.26.0" />
<PackageReference Include="coverlet.collector" Version="6.0.0" />
```

### Unit Testing State Services

**Testing State Services with EventCallback Mocking:**

```csharp
[Fact]
public async Task UpdateProfileAsync_ShouldUpdateState_AndNotifySubscribers()
{
    // Arrange
    var state = new UserProfileState();
    var profile = new UserProfileModel
    {
        Email = "test@example.com",
        FirstName = "Test",
        LastName = "User"
    };

    var eventWasRaised = false;
    state.OnChange += () => eventWasRaised = true;

    // Act
    await state.UpdateProfileAsync(profile);

    // Assert
    state.CurrentProfile.Should().Be(profile);
    state.CurrentProfile.Email.Should().Be("test@example.com");
    eventWasRaised.Should().BeTrue("OnChange event should notify subscribers");
}

[Fact]
public void ClearProfile_ShouldSetProfileToNull_AndNotifySubscribers()
{
    // Arrange
    var state = new UserProfileState();
    var profile = new UserProfileModel { Email = "test@example.com" };
    state.UpdateProfileAsync(profile).Wait();

    var eventWasRaised = false;
    state.OnChange += () => eventWasRaised = true;

    // Act
    state.ClearProfile();

    // Assert
    state.CurrentProfile.Should().BeNull();
    eventWasRaised.Should().BeTrue();
}

[Fact]
public async Task UpdateProfileAsync_ShouldBeThreadSafe_WhenCalledConcurrently()
{
    // Arrange
    var state = new UserProfileState();
    var tasks = new List<Task>();

    // Act - call update from multiple threads concurrently
    for (int i = 0; i < 10; i++)
    {
        var profile = new UserProfileModel { Email = $"user{i}@example.com" };
        tasks.Add(state.UpdateProfileAsync(profile));
    }

    await Task.WhenAll(tasks);

    // Assert - should complete without exceptions
    state.CurrentProfile.Should().NotBeNull();
}
```

### Unit Testing Business Services

**Testing Services with State Service Mocking:**

```csharp
[Fact]
public async Task LoadProfileAsync_ShouldUpdateState_WhenApiReturnsData()
{
    // Arrange
    var mockHttpClient = new Mock<HttpClient>();
    var mockState = new Mock<UserProfileState>();
    var logger = Mock.Of<ILogger<UserProfileService>>();

    var profile = new UserProfileModel { Email = "test@example.com" };

    // Mock HttpClient to return profile data
    var mockHandler = new Mock<HttpMessageHandler>();
    mockHandler.Protected()
        .Setup<Task<HttpResponseMessage>>(
            "SendAsync",
            ItExpr.IsAny<HttpRequestMessage>(),
            ItExpr.IsAny<CancellationToken>())
        .ReturnsAsync(new HttpResponseMessage
        {
            StatusCode = HttpStatusCode.OK,
            Content = JsonContent.Create(profile)
        });

    var httpClient = new HttpClient(mockHandler.Object)
    {
        BaseAddress = new Uri("http://localhost")
    };

    var service = new UserProfileService(mockState.Object, httpClient, logger);

    // Act
    await service.LoadProfileAsync();

    // Assert
    mockState.Verify(s => s.UpdateProfileAsync(It.Is<UserProfileModel>(
        p => p.Email == "test@example.com")), Times.Once);
}

[Fact]
public async Task UpdateProfileAsync_ShouldCallApi_AndUpdateState()
{
    // Arrange
    var mockState = new Mock<UserProfileState>();
    var logger = Mock.Of<ILogger<UserProfileService>>();
    var profile = new UserProfileModel { Email = "updated@example.com" };

    var mockHandler = new Mock<HttpMessageHandler>();
    mockHandler.Protected()
        .Setup<Task<HttpResponseMessage>>(
            "SendAsync",
            ItExpr.IsAny<HttpRequestMessage>(),
            ItExpr.IsAny<CancellationToken>())
        .ReturnsAsync(new HttpResponseMessage
        {
            StatusCode = HttpStatusCode.OK
        });

    var httpClient = new HttpClient(mockHandler.Object)
    {
        BaseAddress = new Uri("http://localhost")
    };

    var service = new UserProfileService(mockState.Object, httpClient, logger);

    // Act
    await service.UpdateProfileAsync(profile);

    // Assert
    mockState.Verify(s => s.UpdateProfileAsync(profile), Times.Once);
}

[Fact]
public async Task LoadProfileAsync_ShouldLogError_WhenApiCallFails()
{
    // Arrange
    var mockState = new Mock<UserProfileState>();
    var mockLogger = new Mock<ILogger<UserProfileService>>();

    var mockHandler = new Mock<HttpMessageHandler>();
    mockHandler.Protected()
        .Setup<Task<HttpResponseMessage>>(
            "SendAsync",
            ItExpr.IsAny<HttpRequestMessage>(),
            ItExpr.IsAny<CancellationToken>())
        .ThrowsAsync(new HttpRequestException("Network error"));

    var httpClient = new HttpClient(mockHandler.Object)
    {
        BaseAddress = new Uri("http://localhost")
    };

    var service = new UserProfileService(mockState.Object, httpClient, mockLogger.Object);

    // Act & Assert
    await Assert.ThrowsAsync<HttpRequestException>(() => service.LoadProfileAsync());

    mockLogger.Verify(
        x => x.Log(
            LogLevel.Error,
            It.IsAny<EventId>(),
            It.Is<It.IsAnyType>((v, t) => true),
            It.IsAny<Exception>(),
            It.IsAny<Func<It.IsAnyType, Exception, string>>()),
        Times.Once);
}
```

### Testing Components with bUnit

**Testing Components with State Service Injection:**

```csharp
public class UserProfilePageTests : TestContext
{
    [Fact]
    public void UserProfilePage_ShouldDisplayProfile_WhenStateContainsData()
    {
        // Arrange
        var profile = new UserProfileModel
        {
            Email = "test@example.com",
            FirstName = "Test",
            LastName = "User",
            PhoneNumber = "123-456-7890"
        };

        var mockState = new Mock<UserProfileState>();
        mockState.Setup(s => s.CurrentProfile).Returns(profile);
        mockState.Setup(s => s.IsLoading).Returns(false);

        Services.AddScoped(_ => mockState.Object);
        Services.AddScoped(_ => Mock.Of<UserProfileService>());

        // Act
        var cut = RenderComponent<UserProfilePage>();

        // Assert
        cut.Find("h2").TextContent.Should().Contain("Test User");
        cut.Find(".email").TextContent.Should().Contain("test@example.com");
        cut.Find(".phone").TextContent.Should().Contain("123-456-7890");
    }

    [Fact]
    public void UserProfilePage_ShouldSubscribeToStateChanges_OnInitialized()
    {
        // Arrange
        var mockState = new Mock<UserProfileState>();
        var initialProfile = new UserProfileModel { Email = "initial@example.com" };
        var updatedProfile = new UserProfileModel { Email = "updated@example.com" };

        mockState.Setup(s => s.CurrentProfile).Returns(initialProfile);
        mockState.Setup(s => s.IsLoading).Returns(false);

        Services.AddScoped(_ => mockState.Object);
        Services.AddScoped(_ => Mock.Of<UserProfileService>());

        // Act - render component
        var cut = RenderComponent<UserProfilePage>();

        // Initial state
        cut.Find(".email").TextContent.Should().Contain("initial@example.com");

        // Simulate state change by raising OnChange event
        mockState.Setup(s => s.CurrentProfile).Returns(updatedProfile);
        mockState.Raise(s => s.OnChange += null);

        // Assert - component should re-render
        cut.WaitForState(() => cut.Find(".email").TextContent.Contains("updated@example.com"));
        cut.Find(".email").TextContent.Should().Contain("updated@example.com");
    }

    [Fact]
    public void UserProfilePage_ShouldShowLoadingIndicator_WhenStateIsLoading()
    {
        // Arrange
        var mockState = new Mock<UserProfileState>();
        mockState.Setup(s => s.IsLoading).Returns(true);
        mockState.Setup(s => s.CurrentProfile).Returns((UserProfileModel)null);

        Services.AddScoped(_ => mockState.Object);
        Services.AddScoped(_ => Mock.Of<UserProfileService>());

        // Act
        var cut = RenderComponent<UserProfilePage>();

        // Assert
        cut.Markup.Should().Contain("Loading");
        cut.FindAll(".profile-data").Should().BeEmpty();
    }

    [Fact]
    public async Task EditProfileButton_ShouldCallService_WhenClicked()
    {
        // Arrange
        var mockState = new Mock<UserProfileState>();
        var mockService = new Mock<UserProfileService>();
        var profile = new UserProfileModel { Email = "test@example.com" };

        mockState.Setup(s => s.CurrentProfile).Returns(profile);
        mockState.Setup(s => s.IsLoading).Returns(false);

        Services.AddScoped(_ => mockState.Object);
        Services.AddScoped(_ => mockService.Object);

        var cut = RenderComponent<UserProfilePage>();

        // Act - click edit button
        var editButton = cut.Find("button.edit-profile");
        await editButton.ClickAsync(new Microsoft.AspNetCore.Components.Web.MouseEventArgs());

        // Assert
        mockService.Verify(s => s.UpdateProfileAsync(It.IsAny<UserProfileModel>()), Times.Once);
    }

    [Fact]
    public void UserProfilePage_ShouldDisposeSubscription_WhenComponentDisposed()
    {
        // Arrange
        var mockState = new Mock<UserProfileState>();
        mockState.Setup(s => s.CurrentProfile).Returns(new UserProfileModel());
        mockState.Setup(s => s.IsLoading).Returns(false);

        Services.AddScoped(_ => mockState.Object);
        Services.AddScoped(_ => Mock.Of<UserProfileService>());

        var cut = RenderComponent<UserProfilePage>();

        // Act - dispose component
        cut.Dispose();

        // Trigger OnChange event after disposal
        mockState.Raise(s => s.OnChange += null);

        // Assert - no exceptions should be thrown
        // Component should have unsubscribed from OnChange event
    }
}
```

**Testing Forms with Validation:**

```csharp
[Fact]
public void UpdateProfileForm_ShouldValidate_AndShowErrors()
{
    // Arrange
    var mockState = new Mock<UserProfileState>();
    mockState.Setup(s => s.CurrentProfile).Returns(new UserProfileModel());

    Services.AddScoped(_ => mockState.Object);
    Services.AddScoped(_ => Mock.Of<UserProfileService>());

    var cut = RenderComponent<UpdateProfileForm>();

    // Act - submit form with invalid data
    var emailInput = cut.Find("input[type='email']");
    emailInput.Change("invalid-email"); // Invalid email format

    var submitButton = cut.Find("button[type='submit']");
    submitButton.Click();

    // Assert
    cut.FindAll(".validation-message").Should().NotBeEmpty();
    cut.Find(".validation-message").TextContent.Should().Contain("valid email");
}

[Fact]
public async Task UpdateProfileForm_ShouldCallService_WhenValidDataSubmitted()
{
    // Arrange
    var mockState = new Mock<UserProfileState>();
    var mockService = new Mock<UserProfileService>();

    mockState.Setup(s => s.CurrentProfile).Returns(new UserProfileModel
    {
        Email = "test@example.com",
        FirstName = "Test"
    });

    Services.AddScoped(_ => mockState.Object);
    Services.AddScoped(_ => mockService.Object);

    var cut = RenderComponent<UpdateProfileForm>();

    // Act - submit form with valid data
    var emailInput = cut.Find("input[type='email']");
    emailInput.Change("updated@example.com");

    var firstNameInput = cut.Find("input[name='firstName']");
    firstNameInput.Change("Updated Name");

    var submitButton = cut.Find("button[type='submit']");
    await submitButton.ClickAsync(new Microsoft.AspNetCore.Components.Web.MouseEventArgs());

    // Assert
    mockService.Verify(s => s.UpdateProfileAsync(It.Is<UserProfileModel>(
        p => p.Email == "updated@example.com" && p.FirstName == "Updated Name")), Times.Once);
}
```

### Integration Testing with HttpClient

```csharp
public class UserServiceIntegrationTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly WebApplicationFactory<Program> _factory;
    private readonly HttpClient _client;

    public UserServiceIntegrationTests(WebApplicationFactory<Program> factory)
    {
        _factory = factory;
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task GetUsersAsync_ShouldReturnUsers_WhenApiIsAvailable()
    {
        // Arrange
        var service = new UserService(_client);

        // Act
        var users = await service.GetUsersAsync();

        // Assert
        users.Should().NotBeNull();
        users.Should().AllBeOfType<UserDto>();
        users.Should().OnlyContain(u => !string.IsNullOrEmpty(u.Email));
    }
}
```

### Testing State Service EventCallback Pattern

**Verifying EventCallback Notifications:**

```csharp
[Fact]
public void StateService_ShouldNotifySubscribers_WhenStateChanges()
{
    // Arrange
    var state = new UserProfileState();
    var notificationCount = 0;

    state.OnChange += () => notificationCount++;

    // Act
    state.UpdateProfileAsync(new UserProfileModel { Email = "test@example.com" }).Wait();

    // Assert
    notificationCount.Should().Be(1);
}

[Fact]
public void StateService_ShouldNotNotify_WhenNoChangeOccurs()
{
    // Arrange
    var state = new UserProfileState();
    var notificationCount = 0;

    state.OnChange += () => notificationCount++;

    // Act - method that doesn't change state
    var profile = state.CurrentProfile; // Just reading, not updating

    // Assert
    notificationCount.Should().Be(0);
}

[Fact]
public void StateService_ShouldSupportMultipleSubscribers()
{
    // Arrange
    var state = new UserProfileState();
    var subscriber1Count = 0;
    var subscriber2Count = 0;

    state.OnChange += () => subscriber1Count++;
    state.OnChange += () => subscriber2Count++;

    // Act
    state.UpdateProfileAsync(new UserProfileModel { Email = "test@example.com" }).Wait();

    // Assert
    subscriber1Count.Should().Be(1);
    subscriber2Count.Should().Be(1);
}
```

## Best Practices Summary

1. **Separation of Concerns**: Keep state services focused on feature state and EventCallback notifications, business services focused on API calls and domain logic, components focused on UI rendering
2. **Dependency Injection**: Use scoped services by default for state services, singleton only for truly global state (app preferences, cache, telemetry), register services in feature-based extension methods
3. **Error Handling**: Implement consistent error handling across all layers with proper error boundaries and user-friendly messages
4. **Validation**: Use hybrid validation strategy - DataAnnotations for simple UI validation with EditForm, FluentValidation for complex business rules with async validation and service injection, always server-side FluentValidation
5. **State Management**: Use state services with EventCallback pattern for UI reactivity, implement IDisposable in components for event cleanup, use SemaphoreSlim for thread-safe async operations in state services
6. **Performance**: Implement proper disposal patterns with event unsubscription, lazy load features and components, use HTTP resilience policies with Polly
7. **Security**: Always validate user permissions before executing actions, use server-side validation for defense in depth
8. **Testing**: Write comprehensive tests - state services with EventCallback mocking using `mockState.Raise(s => s.OnChange += null)`, components with bUnit and state service injection, business services with HttpClient mocking, 100% coverage for ClientComponents before promotion
9. **Documentation**: Document complex business logic and API interactions, include XML comments for public APIs
10. **Code Organization**: Use feature-based organization in Client project, category-based organization in ClientComponents, scoped CSS and JavaScript per component
11. **Component Library**: Promote components to ClientComponents only when ALL 6 criteria met (3+ reuse, zero business logic, stable API, generic/flexible, documented, 100% tested)
12. **Resilience**: Implement retry policies and circuit breakers for HTTP operations using Polly, handle transient failures gracefully

## Error Boundary Pattern

**Component Location:** `Shared/Components/ErrorHandling/`

**Error Boundary Component (CustomErrorBoundary.razor):**

```razor
@inherits ErrorBoundary

<div class="error-boundary">
    @if (CurrentException != null)
    {
        <div class="alert alert-danger" role="alert">
            <h4 class="alert-heading">
                <i class="bi bi-exclamation-triangle-fill me-2"></i>
                Something went wrong
            </h4>
            <p>@GetErrorMessage()</p>

            @if (ShowDetails)
            {
                <hr>
                <details>
                    <summary>Technical Details</summary>
                    <pre class="mt-2"><code>@CurrentException.ToString()</code></pre>
                </details>
            }

            <div class="mt-3">
                <button class="btn btn-primary" @onclick="Recover">
                    <i class="bi bi-arrow-clockwise me-2"></i>
                    Try Again
                </button>
            </div>
        </div>
    }
    else
    {
        @ChildContent
    }
</div>
```

**Code-Behind (CustomErrorBoundary.razor.cs):**

```csharp
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.Web;

namespace YourApp.Shared.Components.ErrorHandling;

public partial class CustomErrorBoundary : ErrorBoundary
{
    [Inject] private ITelemetryService TelemetryService { get; set; } = default!;

    [Parameter] public bool ShowDetails { get; set; }
    [Parameter] public EventCallback OnError { get; set; }

    protected override async Task OnErrorAsync(Exception exception)
    {
        await base.OnErrorAsync(exception);

        // Log to telemetry service
        await TelemetryService.TrackExceptionAsync(exception, new Dictionary<string, string>
        {
            { "Component", "ErrorBoundary" },
            { "ErrorType", exception.GetType().Name }
        });

        await OnError.InvokeAsync();
    }

    private string GetErrorMessage()
    {
        return CurrentException switch
        {
            HttpRequestException => "Unable to connect to the server. Please check your internet connection.",
            TaskCanceledException => "The operation timed out. Please try again.",
            ServiceException se => se.Message,
            UnauthorizedAccessException => "You don't have permission to perform this action.",
            _ => "An unexpected error occurred. Please try again later."
        };
    }
}
```

**Scoped CSS (CustomErrorBoundary.razor.css):**

```css
.error-boundary {
  width: 100%;
}

.error-boundary .alert {
  margin: 1rem 0;
  border-radius: 0.5rem;
}

.error-boundary .alert-heading {
  display: flex;
  align-items: center;
  font-weight: 600;
}

.error-boundary details {
  margin-top: 0.5rem;
}

.error-boundary summary {
  cursor: pointer;
  font-weight: 500;
  color: var(--bs-danger);
}

.error-boundary summary:hover {
  text-decoration: underline;
}

.error-boundary pre {
  background-color: #f8f9fa;
  padding: 1rem;
  border-radius: 0.375rem;
  overflow-x: auto;
  font-size: 0.875rem;
}
```

**Error Boundary Usage:**

```razor
<CustomErrorBoundary ShowDetails="@IsDevelopment">
    <UserListComponent />
</CustomErrorBoundary>
```

## Telemetry Service

**Service Interface (ITelemetryService.cs):**

```csharp
namespace YourApp.Shared.Infrastructure.Telemetry;

public interface ITelemetryService
{
    Task TrackEventAsync(string eventName, Dictionary<string, string>? properties = null);
    Task TrackExceptionAsync(Exception exception, Dictionary<string, string>? properties = null);
    Task TrackMetricAsync(string metricName, double value, Dictionary<string, string>? properties = null);
    Task TrackPageViewAsync(string pageName, Dictionary<string, string>? properties = null);
}
```

**Application Insights Implementation (ApplicationInsightsTelemetryService.cs):**

```csharp
using Microsoft.ApplicationInsights;
using Microsoft.ApplicationInsights.DataContracts;

namespace YourApp.Shared.Infrastructure.Telemetry;

public class ApplicationInsightsTelemetryService : ITelemetryService
{
    private readonly TelemetryClient _telemetryClient;

    public ApplicationInsightsTelemetryService(TelemetryClient telemetryClient)
    {
        _telemetryClient = telemetryClient;
    }

    public Task TrackEventAsync(string eventName, Dictionary<string, string>? properties = null)
    {
        _telemetryClient.TrackEvent(eventName, properties);
        return Task.CompletedTask;
    }

    public Task TrackExceptionAsync(Exception exception, Dictionary<string, string>? properties = null)
    {
        var telemetry = new ExceptionTelemetry(exception);

        if (properties != null)
        {
            foreach (var property in properties)
            {
                telemetry.Properties[property.Key] = property.Value;
            }
        }

        _telemetryClient.TrackException(telemetry);
        return Task.CompletedTask;
    }

    public Task TrackMetricAsync(string metricName, double value, Dictionary<string, string>? properties = null)
    {
        _telemetryClient.TrackMetric(metricName, value, properties);
        return Task.CompletedTask;
    }

    public Task TrackPageViewAsync(string pageName, Dictionary<string, string>? properties = null)
    {
        var telemetry = new PageViewTelemetry(pageName);

        if (properties != null)
        {
            foreach (var property in properties)
            {
                telemetry.Properties[property.Key] = property.Value;
            }
        }

        _telemetryClient.TrackPageView(telemetry);
        return Task.CompletedTask;
    }
}
```

**Service Registration:**

```csharp
// Program.cs
builder.Services.AddSingleton<ITelemetryService, ApplicationInsightsTelemetryService>();

// Configure Application Insights
builder.Services.AddApplicationInsightsTelemetry(options =>
{
    options.ConnectionString = builder.Configuration["ApplicationInsights:ConnectionString"];
});
```

## Advanced Patterns

### Command Pattern for ViewModels

```csharp
public interface IAsyncCommand
{
    bool CanExecute { get; }
    Task ExecuteAsync();
    event EventHandler CanExecuteChanged;
}

public class AsyncCommand : IAsyncCommand
{
    private readonly Func<Task> _execute;
    private readonly Func<bool> _canExecute;
    private bool _isExecuting;

    public AsyncCommand(Func<Task> execute, Func<bool> canExecute = null)
    {
        _execute = execute ?? throw new ArgumentNullException(nameof(execute));
        _canExecute = canExecute ?? (() => true);
    }

    public bool CanExecute => !_isExecuting && _canExecute();

    public event EventHandler CanExecuteChanged;

    public async Task ExecuteAsync()
    {
        if (!CanExecute) return;

        _isExecuting = true;
        RaiseCanExecuteChanged();

        try
        {
            await _execute();
        }
        finally
        {
            _isExecuting = false;
            RaiseCanExecuteChanged();
        }
    }

    private void RaiseCanExecuteChanged()
    {
        CanExecuteChanged?.Invoke(this, EventArgs.Empty);
    }
}

// Usage in ViewModel
public class UserViewModel : BaseViewModel
{
    public IAsyncCommand SaveCommand { get; }
    public IAsyncCommand DeleteCommand { get; }

    public UserViewModel(IUserService userService)
    {
        SaveCommand = new AsyncCommand(
            execute: async () => await SaveUserAsync(),
            canExecute: () => !HasErrors && !string.IsNullOrEmpty(Model.Email)
        );

        DeleteCommand = new AsyncCommand(
            execute: async () => await DeleteUserAsync(),
            canExecute: () => Model.Id > 0
        );
    }
}
```

### State Management with Fluxor (Optional)

```csharp
// Install: Fluxor.Blazor.Web

// State definition
public record UserState
{
    public IEnumerable<UserDto> Users { get; init; } = Enumerable.Empty<UserDto>();
    public bool IsLoading { get; init; }
    public string ErrorMessage { get; init; }
}

// Actions
public record LoadUsersAction;
public record LoadUsersSuccessAction(IEnumerable<UserDto> Users);
public record LoadUsersFailureAction(string ErrorMessage);

// Reducer
public static class UserReducers
{
    [ReducerMethod]
    public static UserState OnLoadUsers(UserState state, LoadUsersAction action)
        => state with { IsLoading = true, ErrorMessage = null };

    [ReducerMethod]
    public static UserState OnLoadUsersSuccess(UserState state, LoadUsersSuccessAction action)
        => state with { IsLoading = false, Users = action.Users };

    [ReducerMethod]
    public static UserState OnLoadUsersFailure(UserState state, LoadUsersFailureAction action)
        => state with { IsLoading = false, ErrorMessage = action.ErrorMessage };
}

// Effects
public class UserEffects
{
    private readonly IUserService _userService;

    public UserEffects(IUserService userService)
    {
        _userService = userService;
    }

    [EffectMethod]
    public async Task HandleLoadUsers(LoadUsersAction action, IDispatcher dispatcher)
    {
        try
        {
            var users = await _userService.GetUsersAsync();
            dispatcher.Dispatch(new LoadUsersSuccessAction(users));
        }
        catch (Exception ex)
        {
            dispatcher.Dispatch(new LoadUsersFailureAction(ex.Message));
        }
    }
}
```
