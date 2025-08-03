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
│   │   └── Validators/      # FluentValidation validators
│   └── [FeatureName]/       # Follow same structure for each feature
├── Shared/
│   ├── Components/          # Reusable UI components
│   │   ├── Forms/           # Form components (.razor, .razor.cs, .razor.css, .razor.js)
│   │   ├── Tables/          # Data table components (.razor, .razor.cs, .razor.css, .razor.js)
│   │   ├── Layout/          # Layout components (.razor, .razor.cs, .razor.css, .razor.js)
│   │   └── Common/          # General purpose components (.razor, .razor.cs, .razor.css, .razor.js)
│   ├── Layouts/             # Application layouts (.razor, .razor.cs, .razor.css, .razor.js)
│   ├── Infrastructure/      # Cross-cutting concerns
│   │   ├── Authentication/  # Auth helpers and services
│   │   ├── Http/            # HTTP client configurations
│   │   ├── Storage/         # Local storage services
│   │   ├── Notifications/   # Notification services
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

**Service Registration Pattern:**
```csharp
// Program.cs
builder.Services.AddScoped<IUserService, UserService>();
builder.Services.AddScoped<UserListViewModel>();
builder.Services.AddScoped<INotificationService, ToastNotificationService>();
```

**Service Lifetimes:**
- **Scoped**: ViewModels, business services, HTTP clients
- **Singleton**: Configuration services, logging, caching
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

### 3. HTTP Client Optimization
```csharp
// Configure HttpClient with base address and default headers
builder.Services.AddHttpClient<IUserService, UserService>(client =>
{
    client.BaseAddress = new Uri(builder.Configuration["ApiSettings:BaseUrl"]);
    client.DefaultRequestHeaders.Add("Accept", "application/json");
});
```

## Testing Guidelines

### Unit Testing ViewModels
```csharp
[Test]
public async Task LoadUsersAsync_ShouldPopulateUsers_WhenServiceReturnsData()
{
    // Arrange
    var mockUserService = new Mock<IUserService>();
    var mockNotificationService = new Mock<INotificationService>();
    var users = new List<UserDto> { new UserDto { Id = 1, Name = "Test User" } };
    
    mockUserService.Setup(s => s.GetUsersAsync()).ReturnsAsync(users);
    
    var viewModel = new UserListViewModel(mockUserService.Object, mockNotificationService.Object);
    
    // Act
    await viewModel.LoadUsersAsync();
    
    // Assert
    Assert.AreEqual(1, viewModel.Users.Count);
    Assert.AreEqual("Test User", viewModel.Users.First().Name);
    Assert.IsFalse(viewModel.IsLoading);
}
```

## Best Practices Summary

1. **Separation of Concerns**: Keep ViewModels focused on business logic, Components on UI rendering
2. **Dependency Injection**: Use DI extensively for loose coupling and testability
3. **Error Handling**: Implement consistent error handling across all layers
4. **Validation**: Use FluentValidation for complex form validation
5. **Performance**: Implement proper disposal patterns and lazy loading
6. **Security**: Always validate user permissions before executing actions
7. **Testing**: Write unit tests for ViewModels and integration tests for services
8. **Documentation**: Document complex business logic and API interactions
