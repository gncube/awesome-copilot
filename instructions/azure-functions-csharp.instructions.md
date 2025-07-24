---
description: 'C# patterns for Azure Functions with .NET 9'
applyTo: '**/*.cs, **/*.csproj, **/*.json'
---

# Azure Functions C# Development Guidelines

## Guidance for Code Generation

- Generate modern C# code using **.NET 9+ features** and C# 13 language capabilities
- Use **`async/await`** for all I/O operations and long-running tasks
- Follow **Clean Architecture** principles and **SOLID design patterns**
- Apply **Domain-Driven Design** where appropriate for complex business logic
- Always use **dependency injection** for services and configuration management
- Ask before adding any extra dependencies to the project
- The API is built using **Azure Functions using .NET 9 Isolated Worker Process**
- Use **`FunctionsApplication.CreateBuilder`** as the hosting model for optimal performance and developer experience
- Each endpoint should have its own function class, using the naming convention: `src/API/Functions/<ResourceName>Functions.cs`
- When making changes to the API, make sure to update OpenAPI documentation and `README.md` accordingly

## Azure Functions Hosting Model

- **Always use `FunctionsApplication.CreateBuilder`** instead of generic `HostBuilder` for new Azure Functions projects
- This provides **faster cold starts** and **memory efficiency** optimized for serverless scenarios
- Benefits include **reduced boilerplate code**, better **IntelliSense support**, and **future-proof** alignment with Microsoft's recommendations
- **Pre-configured services**: Application Insights, logging, and configuration are automatically set up with appropriate defaults
- **Optimized for Functions runtime**: Purpose-built for Azure Functions with minimal resource footprint
- **Clean Architecture alignment**: Simpler configuration reduces complexity in Clean Architecture implementation
- **Better debugging experience**: Enhanced integration with Azure Functions Core Tools and local development

## Function Development Standards

- Keep functions **focused and single-purpose** following the Single Responsibility Principle
- Use **meaningful and descriptive function names** that clearly indicate their purpose
- Implement **proper error handling** with structured exception management and logging
- Make functions **stateless** to improve reliability during scale operations
- Follow **defensive coding practices** to handle unexpected inputs and edge cases
- Use **file-scoped namespaces** for cleaner code organization
- Implement **cancellation token support** for proper request termination and resource cleanup

## Hosting & Configuration Best Practices

- Use **Flex Consumption plan** for optimal cost efficiency and dynamic scaling
- Use **Premium plan** for consistent performance when cold-start latency is critical
- Configure proper **CORS settings** when accessed from Blazor WebAssembly client
- Set appropriate **function timeout** values (default maximum is 5 minutes)
- Consider using **deployment slots** for zero-downtime deployments in production
- Implement **warmup triggers** for Premium plan to reduce cold starts
- Configure **worker process count** using `FUNCTIONS_WORKER_PROCESS_COUNT` for CPU-intensive workloads

## Security Implementation Requirements

- Use **managed identities** for Azure resource access instead of connection strings
- Implement **comprehensive input validation** on all function parameters using DataAnnotations or FluentValidation
- Store secrets in **Azure Key Vault** rather than in application settings
- Apply **function-level authorization** using `AuthorizationLevel.Function` by default
- Configure **network isolation** using VNet integration when handling sensitive data
- Follow **OWASP security guidelines** for password management and user data protection
- Implement **structured audit logging** for security-sensitive operations

## Performance Optimization Guidelines

- Use **`FunctionsApplication.CreateBuilder`** for optimized cold start performance and memory efficiency
- Configure appropriate **batch sizes** for supported triggers in `host.json`
- Use **connection pooling** for database and external service connections
- Enable **compression** for larger payloads using response compression middleware
- Implement **output caching** for repetitive read operations using `IMemoryCache`
- Optimize **response serialization** using System.Text.Json with appropriate converters
- **Avoid long-running functions** - consider Durable Functions for extended processes
- Use **Polly** for implementing retry patterns with exponential backoff for transient failures
- Leverage **pre-configured Functions services** to reduce initialization overhead

## Data & Storage Management

- **Configure storage correctly** with appropriate connection strings in application settings
- Place storage account in **same region** as function app to reduce latency
- Use **separate storage accounts** for production function apps to improve performance
- Implement proper **connection management** to avoid connection exhaustion
- **Dispose of resources** properly using `using` statements or `IAsyncDisposable`
- Use **Entity Framework Core** with proper connection pooling for database operations
- Implement **repository pattern** for data access abstraction and testability

## Monitoring & Logging Standards

- Use **Application Insights** integration for comprehensive monitoring and telemetry
- Implement **structured logging** using `ILogger<T>` with appropriate log levels
- Configure **sampling** to manage telemetry volume and costs effectively
- Set up **alerts** for critical failures, performance thresholds, and security events
- Use **correlation IDs** to track requests across system boundaries and microservices
- Remove deprecated **`AzureWebJobsDashboard`** setting to improve performance
- Implement **custom metrics** for business-specific monitoring requirements

## Testing & Validation Requirements

- Write **unit tests** using **xUnit** with **Fluent Assertions** for readable test assertions
- Implement **integration tests** using `WebApplicationFactory` for end-to-end testing
- Use **local.settings.json** for development configuration with environment-specific values
- Create **mock data files** in a standard `/TestData` location for consistent test scenarios
- Test **scale and performance** for production workloads using load testing tools
- Create **`.http` files** for each HTTP-triggered function to enable quick manual testing
- Use **TestContainers** for integration tests requiring external dependencies

## HTTP Request Files Organization

- Maintain a **`/http`** folder in your Azure Functions project root
- Create one `.http` file per function class or logical group of related endpoints
- Include comprehensive sample requests for all endpoints with appropriate headers and JSON payloads
- Use **environment variables** in `.http` files to handle different environments (local, dev, prod)
- Document **expected responses** and **error scenarios** in comments within the file
- Organize requests with **descriptive names** and clear separators for readability
- Include **authentication headers** and **request validation examples**

## Code Organization Patterns

- Organize functions **by feature or domain** to maintain separation of concerns
- Use **feature folders** structure: `/Features/{FeatureName}/{Functions|Services|Models}`
- Implement **service layer** patterns for business logic separation from function triggers
- Create **shared models** and **DTOs** in separate projects for reusability
- Use **dependency injection extensions** for clean service registration with `FunctionsApplication.CreateBuilder`
- Follow **namespace conventions**: `ProjectName.{Layer}.{Feature}`
- Leverage **Functions-specific APIs** for better IntelliSense support and reduced configuration errors
- Minimize **configuration boilerplate** by using the Functions builder's sensible defaults

## Error Handling & Resilience

- Implement **global exception handling** middleware for consistent error responses
- Use **problem details** (RFC 7807) for standardized error response format
- Configure **retry policies** using Polly for transient failure scenarios
- Implement **circuit breaker** patterns for external service dependencies
- Log **security events** and **audit trails** for compliance and debugging
- Handle **timeout scenarios** gracefully with appropriate user feedback

## Development Productivity Guidelines

- **Faster Development**: Leverage `FunctionsApplication.CreateBuilder` for reduced configuration overhead and faster feature delivery
- **Reduced Learning Curve**: Use Functions-specific patterns that are simpler for team members new to Azure Functions
- **Fewer Configuration Errors**: Rely on sensible defaults to minimize opportunity for misconfiguration
- **Better IDE Experience**: Take advantage of Functions-specific IntelliSense and tooling support
- **Future-Proof Architecture**: Align with Microsoft's long-term support strategy for .NET 9+ Azure Functions
- **Community Alignment**: Follow mainstream patterns for better community support and documentation availability
- **Maintenance Efficiency**: Minimize configuration surface area to reduce long-term maintenance costs
