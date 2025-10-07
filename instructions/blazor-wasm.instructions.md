---
description: "Blazor WebAssembly development using Hybrid MVVM pattern with Feature Organization"
applyTo: "**/*.razor, **/*.razor.cs, **/*.razor.css, **/*.razor.js"
---

## Architecture Overview

Build Blazor WebAssembly applications using a Hybrid MVVM pattern with Feature Organization. This architecture prioritizes code reusability, maintainability, and team collaboration for data-driven applications.

## Project Structure

Follow the feature organization pattern:

```
src/
├── Core/
│   ├── Models/              # Shared domain models and DTOs
│   ├── Services/            # Core business services and interfaces
│   │   ├── Interfaces/      # Service contracts
│   │   └── Implementations/ # Service implementations
│   ├── Extensions/          # Extension methods
│   ├── Constants/           # Application constants
│   ├── Enums/              # Shared enumerations
│   └── Common/             # Shared utilities and helpers
├── Features/
│   ├── Authentication/
│   │   ├── Components/      # Feature-specific components (.razor, .razor.cs, .razor.css, .razor.js)
│   │   ├── ViewModels/      # Page and component ViewModels
│   │   ├── Models/          # Feature-specific models/DTOs
│   │   ├── Services/        # Feature-specific services
│   │   ├── Pages/           # Routable pages (.razor, .razor.cs, .razor.css, .razor.js)
│   │   ├── Validators/      # FluentValidation validators
│   │   ├── Extensions/      # Feature DI registration extensions
│   │   ├── _Imports.razor   # Feature-specific using statements
│   │   └── Authentication.md # Feature documentation
│   └── [FeatureName]/       # Follow same structure for each feature
│       ├── Components/
│       ├── ViewModels/
│       ├── Models/
│       ├── Services/
│       ├── Pages/
│       ├── Validators/
│       ├── Extensions/
│       ├── _Imports.razor
│       └── [FeatureName].md
├── Shared/
│   ├── Components/          # Reusable UI components
│   │   ├── Forms/           # Form components (.razor, .razor.cs, .razor.css, .razor.js)
│   │   ├── Tables/          # Data table components (.razor, .razor.cs, .razor.css, .razor.js)
│   │   ├── Layout/          # Layout components (.razor, .razor.cs, .razor.css, .razor.js)
│   │   ├── ErrorHandling/   # Error boundary components
│   │   │   ├── CustomErrorBoundary.razor
│   │   │   ├── CustomErrorBoundary.razor.cs
│   │   │   └── CustomErrorBoundary.razor.css
│   │   └── Common/          # General purpose components (.razor, .razor.cs, .razor.css, .razor.js)
│   ├── Layouts/             # Application layouts (.razor, .razor.cs, .razor.css, .razor.js)
│   ├── Infrastructure/      # Cross-cutting concerns
│   │   ├── Authentication/  # Auth helpers and services
│   │   ├── Http/            # HTTP client configurations
│   │   ├── Storage/         # Local storage services
│   │   ├── Notifications/   # Notification services
│   │   ├── Telemetry/       # Telemetry and logging services
│   │   │   ├── ITelemetryService.cs
│   │   │   └── ApplicationInsightsTelemetryService.cs
│   │   └── State/           # State management
│   └── Styles/              # Global CSS and Bootstrap customizations
├── wwwroot/
│   ├── css/                 # Custom stylesheets
│   ├── js/                  # JavaScript interop files
│   └── assets/              # Static assets
└── Tests/
    ├── Unit/                # Unit tests for ViewModels and Services
    └── Integration/         # Integration tests
```

## Component File Organization

Each Blazor component should follow this file organization pattern:

```
ComponentName/
├── ComponentName.razor          # Markup and basic data binding
├── ComponentName.razor.cs       # Code-behind with component logic
├── ComponentName.razor.css      # Scoped CSS styling
└── ComponentName.razor.js       # Scoped JavaScript (when needed)
```

## MVVM Pattern Implementation

**ViewModels should:**

- Implement `INotifyPropertyChanged` for property binding
- Handle all business logic and state management
- Be registered as scoped services in DI container
- Never directly reference Blazor components or UI elements
- Encapsulate all API calls and data operations

**Components should:**

- Be thin and focus only on UI rendering (use .razor.cs code-behind for complex logic)
- Use scoped CSS (.razor.css) for component-specific styling
- Use scoped JavaScript (.razor.js) when JavaScript interop is required
- Bind to ViewModel properties and commands
- Handle user interactions by calling ViewModel methods
- Implement `IDisposable` when subscribing to ViewModel events

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
        return [...tooltips].map(el => new bootstrap.Tooltip(el));
    };
    
    const tooltipInstances = initializeTooltips();
    
    // Return disposal function
    return {
        dispose: () => {
            tooltipInstances.forEach(tooltip => tooltip.dispose());
            if (element._dotNetHelper) {
                element._dotNetHelper.dispose();
            }
        }
    };
}

// Export utility functions
export function updateComponentState(element, state) {
    element.classList.toggle('loading-state', state.isLoading);
    element.classList.toggle('error-state', state.hasError);
}
```

## Code-Behind Structure

```csharp
using Microsoft.AspNetCore.Components;
using Microsoft.JSInterop;

namespace YourApp.Features.FeatureName.Components;

public partial class ComponentNameBase : ComponentBase, IAsyncDisposable
{
    // Injected services
    [Inject] protected IJSRuntime JSRuntime { get; set; } = default!;
    [Inject] protected ComponentViewModel ViewModel { get; set; } = default!;
    
    // Parameters
    [Parameter] public string Title { get; set; } = string.Empty;
    [Parameter] public EventCallback<string> OnValueChanged { get; set; }
    
    // JavaScript interop
    private IJSObjectReference? _jsModule;
    private IJSObjectReference? _jsInstance;
    private DotNetObjectReference<ComponentNameBase>? _dotNetHelper;
    
    protected override async Task OnInitializedAsync()
    {
        // Subscribe to ViewModel events
        ViewModel.PropertyChanged += OnViewModelPropertyChanged;
        
        // Initialize component data
        await ViewModel.InitializeAsync();
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
                "import", "./Features/FeatureName/Components/ComponentName.razor.js");
            
            _dotNetHelper = DotNetObjectReference.Create(this);
            
            _jsInstance = await _jsModule.InvokeAsync<IJSObjectReference>(
                "initialize", Element, _dotNetHelper);
        }
        catch (Exception ex)
        {
            // Log error - JavaScript module failed to load
            Console.WriteLine($"Failed to initialize JavaScript: {ex.Message}");
        }
    }
    
    // Event handlers
    private void OnViewModelPropertyChanged(object sender, PropertyChangedEventArgs e)
    {
        InvokeAsync(StateHasChanged);
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
        
        // Unsubscribe from events
        if (ViewModel != null)
        {
            ViewModel.PropertyChanged -= OnViewModelPropertyChanged;
        }
    }
}
```

## Dependency Injection Guidelines

### Feature-Based Service Registration

Each feature should have its own service registration extension method to keep DI configuration organized and maintainable.

**Feature Extension Pattern (Features/[FeatureName]/Extensions/):**

```csharp
namespace YourApp.Features.Authentication.Extensions;

public static class AuthenticationServiceExtensions
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

        // Register feature-specific services
        services.AddScoped<IClaimsService, ClaimsService>();
        services.AddScoped<IUserService, UserService>();
        
        // Register ViewModels
        services.AddScoped<LoginViewModel>();
        services.AddScoped<RegisterViewModel>();
        
        // Register validators
        services.AddTransient<IValidator<LoginRequest>, LoginRequestValidator>();
        services.AddTransient<IValidator<RegisterRequest>, RegisterRequestValidator>();

        return services;
    }
}
```

**Another Feature Example (Features/UserManagement/Extensions/):**

```csharp
namespace YourApp.Features.UserManagement.Extensions;

public static class UserManagementServiceExtensions
{
    public static IServiceCollection AddUserManagementFeature(
        this IServiceCollection services,
        IConfiguration configuration)
    {
        // Configure HttpClient for user management API
        services.AddHttpClient<IUserManagementService, UserManagementService>(client =>
        {
            client.BaseAddress = new Uri(configuration["ApiSettings:BaseUrl"]);
            client.DefaultRequestHeaders.Add("Accept", "application/json");
        })
        .AddPolicyHandler(HttpPolicyExtensions
            .HandleTransientHttpError()
            .WaitAndRetryAsync(3, retryAttempt => TimeSpan.FromSeconds(Math.Pow(2, retryAttempt))));

        // Register ViewModels
        services.AddScoped<UserListViewModel>();
        services.AddScoped<UserDetailViewModel>();
        services.AddScoped<UserEditViewModel>();
        
        // Register validators
        services.AddTransient<IValidator<CreateUserRequest>, CreateUserRequestValidator>();
        services.AddTransient<IValidator<UpdateUserRequest>, UpdateUserRequestValidator>();

        return services;
    }
}
```

**Program.cs Registration:**

```csharp
using YourApp.Features.Authentication.Extensions;
using YourApp.Features.UserManagement.Extensions;
using YourApp.Shared.Infrastructure.Telemetry;

var builder = WebAssemblyHostBuilder.CreateDefault(args);

// Add root components
builder.RootComponents.Add<App>("#app");
builder.RootComponents.Add<HeadOutlet>("head::after");

// Add shared infrastructure services
builder.Services.AddScoped(sp => new HttpClient 
{ 
    BaseAddress = new Uri(builder.HostEnvironment.BaseAddress) 
});

builder.Services.AddSingleton<ITelemetryService, ApplicationInsightsTelemetryService>();
builder.Services.AddApplicationInsightsTelemetry(options =>
{
    options.ConnectionString = builder.Configuration["ApplicationInsights:ConnectionString"];
});

// Add features using extension methods
builder.Services.AddAuthenticationFeature(builder.Configuration);
builder.Services.AddUserManagementFeature(builder.Configuration);

await builder.Build().RunAsync();
```

### Feature-Specific Imports

Each feature should have its own `_Imports.razor` file with feature-specific using statements.

**Example: Features/Authentication/_Imports.razor:**

```razor
@* Feature-specific imports for Authentication *@
@using YourApp.Features.Authentication.Components
@using YourApp.Features.Authentication.ViewModels
@using YourApp.Features.Authentication.Models
@using YourApp.Features.Authentication.Services
@using YourApp.Features.Authentication.Pages
@using Microsoft.AspNetCore.Components.Authorization
@using Microsoft.AspNetCore.Authorization
```

**Example: Features/UserManagement/_Imports.razor:**

```razor
@* Feature-specific imports for User Management *@
@using YourApp.Features.UserManagement.Components
@using YourApp.Features.UserManagement.ViewModels
@using YourApp.Features.UserManagement.Models
@using YourApp.Features.UserManagement.Services
@using YourApp.Features.UserManagement.Pages
@using FluentValidation
```

### Feature Documentation

Each feature should include a markdown documentation file explaining its purpose, architecture, and usage.

**Example: Features/Authentication/Authentication.md:**

````markdown
# Authentication Feature

## Overview

The Authentication feature handles user authentication and authorization using Azure AD B2C. It provides login, registration, and user profile management capabilities.

## Components

### Pages
- **Login.razor**: User login page with email/password authentication
- **Register.razor**: New user registration page
- **Profile.razor**: User profile management page

### ViewModels
- **LoginViewModel**: Manages login form state and authentication logic
- **RegisterViewModel**: Handles user registration flow
- **ProfileViewModel**: Manages user profile data and updates

### Services
- **IClaimsService**: Retrieves and manages user claims
- **IUserService**: User data management and API interactions

## Configuration

Required configuration in `appsettings.json`:

```json
{
  "AzureAdB2C": {
    "Authority": "https://yourb2c.b2clogin.com/yourb2c.onmicrosoft.com/B2C_1_signupsignin",
    "ClientId": "your-client-id",
    "ValidateAuthority": false
  }
}
```

## Usage

### In Program.cs

```csharp
builder.Services.AddAuthenticationFeature(builder.Configuration);
```

### In Components

```razor
@page "/secure-page"
@attribute [Authorize]

<AuthorizeView>
    <Authorized>
        <p>Welcome, @context.User.Identity?.Name!</p>
    </Authorized>
    <NotAuthorized>
        <p>You must be logged in to view this page.</p>
    </NotAuthorized>
</AuthorizeView>
```

## Authorization Policies

- **Admin**: Requires `jobTitle` claim with value "Admin"
- **User**: Requires authenticated user

## Testing

Unit tests are located in `Tests/Unit/Features/Authentication/`

## Dependencies

- Microsoft.Authentication.WebAssembly.Msal
- FluentValidation
````

**Example: Features/UserManagement/UserManagement.md:**

````markdown
# User Management Feature

## Overview

The User Management feature provides CRUD operations for managing users in the system. It includes list views, detail views, and editing capabilities.

## Components

### Pages
- **UserList.razor**: Displays paginated list of users
- **UserDetail.razor**: Shows detailed information about a specific user
- **UserEdit.razor**: Form for creating and editing users

### ViewModels
- **UserListViewModel**: Manages user list data, filtering, and pagination
- **UserDetailViewModel**: Handles user detail display and related data
- **UserEditViewModel**: Manages user creation and editing forms

### Services
- **IUserManagementService**: API client for user management operations

## API Endpoints

- `GET /api/users` - Get paginated list of users
- `GET /api/users/{id}` - Get user by ID
- `POST /api/users` - Create new user
- `PUT /api/users/{id}` - Update existing user
- `DELETE /api/users/{id}` - Delete user

## Usage

### In Program.cs

```csharp
builder.Services.AddUserManagementFeature(builder.Configuration);
```

### In Components

```razor
@page "/users"
@inject UserListViewModel ViewModel

<h1>Users</h1>

<CustomErrorBoundary>
    <UserListComponent ViewModel="@ViewModel" />
</CustomErrorBoundary>
```

## Validation Rules

User creation and editing enforce:
- Email: Required, valid email format, max 256 characters
- First Name: Required, max 50 characters, letters only
- Last Name: Required, max 50 characters, letters only
- Phone: Optional, international format

## Testing

Unit tests are located in `Tests/Unit/Features/UserManagement/`

## Dependencies

- FluentValidation
- Microsoft.Extensions.Http.Polly
````

### Service Registration Pattern

**Service Lifetimes:**

- **Scoped**: ViewModels, business services, HTTP clients
- **Singleton**: Configuration services, logging, caching, telemetry
- **Transient**: Lightweight utilities, validators

## BaseViewModel Pattern

```csharp
public abstract class BaseViewModel : INotifyPropertyChanged
{
    private bool _isLoading;
    private string _errorMessage;

    public bool IsLoading
    {
        get => _isLoading;
        set => SetProperty(ref _isLoading, value);
    }

    public string ErrorMessage
    {
        get => _errorMessage;
        set => SetProperty(ref _errorMessage, value);
    }

    public event PropertyChangedEventHandler PropertyChanged;

    protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }

    protected bool SetProperty<T>(ref T backingStore, T value, [CallerMemberName] string propertyName = "")
    {
        if (EqualityComparer<T>.Default.Equals(backingStore, value))
            return false;

        backingStore = value;
        OnPropertyChanged(propertyName);
        return true;
    }
}
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

## Form Validation with FluentValidation

```csharp
public class CreateUserRequestValidator : AbstractValidator<CreateUserRequest>
{
    public CreateUserRequestValidator()
    {
        RuleFor(x => x.Email)
            .NotEmpty().WithMessage("Email is required")
            .EmailAddress().WithMessage("Valid email address is required");

        RuleFor(x => x.FirstName)
            .NotEmpty().WithMessage("First name is required")
            .MaximumLength(50).WithMessage("First name cannot exceed 50 characters");
    }
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

### Unit Testing ViewModels

**Using FluentAssertions for Better Readability:**

```csharp
[Fact]
public async Task LoadUsersAsync_ShouldPopulateUsers_WhenServiceReturnsData()
{
    // Arrange
    var mockUserService = new Mock<IUserService>();
    var mockNotificationService = new Mock<INotificationService>();
    var users = new List<UserDto> 
    { 
        new UserDto { Id = 1, Name = "Test User", Email = "test@example.com" } 
    };
    
    mockUserService
        .Setup(s => s.GetUsersAsync())
        .ReturnsAsync(users);
    
    var viewModel = new UserListViewModel(
        mockUserService.Object, 
        mockNotificationService.Object);
    
    // Act
    await viewModel.LoadUsersAsync();
    
    // Assert
    viewModel.Users.Should().HaveCount(1);
    viewModel.Users.First().Name.Should().Be("Test User");
    viewModel.Users.First().Email.Should().Be("test@example.com");
    viewModel.IsLoading.Should().BeFalse();
    viewModel.ErrorMessage.Should().BeNullOrEmpty();
}

[Fact]
public async Task LoadUsersAsync_ShouldSetErrorMessage_WhenServiceThrowsException()
{
    // Arrange
    var mockUserService = new Mock<IUserService>();
    var mockNotificationService = new Mock<INotificationService>();
    var expectedError = "Failed to load users";
    
    mockUserService
        .Setup(s => s.GetUsersAsync())
        .ThrowsAsync(new InvalidOperationException(expectedError));
    
    var viewModel = new UserListViewModel(
        mockUserService.Object, 
        mockNotificationService.Object);
    
    // Act
    await viewModel.LoadUsersAsync();
    
    // Assert
    viewModel.Users.Should().BeEmpty();
    viewModel.ErrorMessage.Should().Contain(expectedError);
    viewModel.IsLoading.Should().BeFalse();
    
    mockNotificationService.Verify(
        s => s.ShowErrorAsync(It.IsAny<string>()), 
        Times.Once);
}

[Fact]
public async Task DeleteUserAsync_ShouldRemoveUser_WhenDeletionSucceeds()
{
    // Arrange
    var mockUserService = new Mock<IUserService>();
    var mockNotificationService = new Mock<INotificationService>();
    var userToDelete = new UserDto { Id = 1, Name = "Test User" };
    
    mockUserService
        .Setup(s => s.DeleteUserAsync(userToDelete.Id))
        .ReturnsAsync(true);
    
    var viewModel = new UserListViewModel(
        mockUserService.Object, 
        mockNotificationService.Object);
    
    viewModel.Users.Add(userToDelete);
    
    // Act
    await viewModel.DeleteUserAsync(userToDelete.Id);
    
    // Assert
    viewModel.Users.Should().NotContain(userToDelete);
    viewModel.Users.Should().BeEmpty();
    
    mockNotificationService.Verify(
        s => s.ShowSuccessAsync(It.Is<string>(msg => msg.Contains("deleted"))), 
        Times.Once);
}
```

### Testing Components with bUnit

**Component Testing Best Practices:**

```csharp
public class UserListComponentTests : TestContext
{
    [Fact]
    public void Component_ShouldRenderLoadingState_WhenViewModelIsLoading()
    {
        // Arrange
        var mockViewModel = new Mock<UserListViewModel>();
        mockViewModel.Setup(vm => vm.IsLoading).Returns(true);
        
        Services.AddSingleton(mockViewModel.Object);
        
        // Act
        var cut = RenderComponent<UserListComponent>();
        
        // Assert
        cut.Find(".loading-spinner").Should().NotBeNull();
        cut.FindAll(".user-row").Should().BeEmpty();
    }
    
    [Fact]
    public void Component_ShouldRenderUsers_WhenViewModelHasData()
    {
        // Arrange
        var mockViewModel = new Mock<UserListViewModel>();
        var users = new List<UserDto>
        {
            new UserDto { Id = 1, Name = "User 1", Email = "user1@example.com" },
            new UserDto { Id = 2, Name = "User 2", Email = "user2@example.com" }
        };
        
        mockViewModel.Setup(vm => vm.Users).Returns(users);
        mockViewModel.Setup(vm => vm.IsLoading).Returns(false);
        
        Services.AddSingleton(mockViewModel.Object);
        
        // Act
        var cut = RenderComponent<UserListComponent>();
        
        // Assert
        var userRows = cut.FindAll(".user-row");
        userRows.Should().HaveCount(2);
        
        cut.Find(".user-name").TextContent.Should().Contain("User 1");
        cut.Find(".user-email").TextContent.Should().Contain("user1@example.com");
    }
    
    [Fact]
    public async Task Component_ShouldCallDeleteMethod_WhenDeleteButtonClicked()
    {
        // Arrange
        var mockViewModel = new Mock<UserListViewModel>();
        var user = new UserDto { Id = 1, Name = "Test User" };
        
        mockViewModel.Setup(vm => vm.Users).Returns(new List<UserDto> { user });
        mockViewModel.Setup(vm => vm.DeleteUserAsync(It.IsAny<int>())).Returns(Task.CompletedTask);
        
        Services.AddSingleton(mockViewModel.Object);
        
        // Act
        var cut = RenderComponent<UserListComponent>();
        var deleteButton = cut.Find(".delete-button");
        await cut.InvokeAsync(() => deleteButton.Click());
        
        // Assert
        mockViewModel.Verify(vm => vm.DeleteUserAsync(1), Times.Once);
    }
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

### Property Change Notification Testing

```csharp
[Fact]
public void ViewModel_ShouldRaisePropertyChanged_WhenPropertyIsSet()
{
    // Arrange
    var viewModel = new UserListViewModel(
        Mock.Of<IUserService>(), 
        Mock.Of<INotificationService>());
    
    var propertyChangedRaised = false;
    viewModel.PropertyChanged += (sender, args) =>
    {
        if (args.PropertyName == nameof(viewModel.IsLoading))
            propertyChangedRaised = true;
    };
    
    // Act
    viewModel.IsLoading = true;
    
    // Assert
    propertyChangedRaised.Should().BeTrue();
    viewModel.IsLoading.Should().BeTrue();
}

[Fact]
public void ViewModel_ShouldNotRaisePropertyChanged_WhenValueIsUnchanged()
{
    // Arrange
    var viewModel = new UserListViewModel(
        Mock.Of<IUserService>(), 
        Mock.Of<INotificationService>());
    
    viewModel.IsLoading = true;
    
    var propertyChangedCount = 0;
    viewModel.PropertyChanged += (sender, args) => propertyChangedCount++;
    
    // Act
    viewModel.IsLoading = true; // Setting same value
    
    // Assert
    propertyChangedCount.Should().Be(0);
}
```

## Best Practices Summary

1. **Separation of Concerns**: Keep ViewModels focused on business logic, Components on UI rendering
2. **Dependency Injection**: Use DI extensively for loose coupling and testability
3. **Error Handling**: Implement consistent error handling across all layers with proper error boundaries
4. **Validation**: Use FluentValidation for complex form validation with comprehensive rules
5. **Performance**: Implement proper disposal patterns, lazy loading, and HTTP resilience
6. **Security**: Always validate user permissions before executing actions
7. **Testing**: Write comprehensive tests using FluentAssertions, bUnit for components, and xUnit/NUnit for unit tests
8. **Documentation**: Document complex business logic and API interactions
9. **Code Organization**: Use feature-based organization with scoped CSS and JavaScript
10. **Resilience**: Implement retry policies and circuit breakers for HTTP operations

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
