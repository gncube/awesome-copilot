---
description: 'SQLite with Entity Framework Core best practices and patterns for secure, efficient database operations'
applyTo: '**/*.cs, **/DbContext.cs, **/Migrations/*.cs, **/*Repository.cs'
---

# SQLite with Entity Framework Core

## Overview

SQLite with Entity Framework Core requires specific considerations due to platform limitations, data type constraints, and migration complexities. This guide establishes secure, efficient patterns for database operations while working within SQLite's constraints.

## Core SQLite Limitations and Workarounds

### Data Type Limitations

SQLite doesn't natively support certain .NET data types. Always configure proper conversions:

```csharp
// DateTimeOffset - Store as TEXT (ISO 8601)
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    foreach (var entityType in modelBuilder.Model.GetEntityTypes())
    {
        foreach (var property in entityType.GetProperties())
        {
            if (property.ClrType == typeof(DateTimeOffset) || property.ClrType == typeof(DateTimeOffset?))
            {
                property.SetColumnType("TEXT");
            }
        }
    }
}

// Decimal - Use double for better performance or value converter for precision
modelBuilder.Entity<MyEntity>()
    .Property(e => e.DecimalProperty)
    .HasConversion<double>();

// TimeSpan - Store as ticks (long)
modelBuilder.Entity<MyEntity>()
    .Property(e => e.TimeSpanProperty)
    .HasConversion(
        v => v.Ticks,
        v => new TimeSpan(v));
```

### Concurrency Token Configuration

SQLite doesn't support database-generated concurrency tokens. Use manual RowVersion handling:

```csharp
public abstract class BaseEntity
{
    [Timestamp]
    public byte[] RowVersion { get; set; } = null!;
    
    public DateTimeOffset CreatedAt { get; set; } = DateTimeOffset.UtcNow;
    public DateTimeOffset UpdatedAt { get; set; } = DateTimeOffset.UtcNow;
}

// In DbContext configuration
modelBuilder.Entity<MyEntity>()
    .Property(e => e.RowVersion)
    .IsRowVersion()
    .HasDefaultValue(new byte[] { 0, 0, 0, 0, 0, 0, 0, 1 });
```

## DbContext Design Patterns

### Base Context for Common Configuration

Create a shared base context for SQLite-specific configurations:

```csharp
using Microsoft.EntityFrameworkCore;

/// <summary>
/// Base DbContext with SQLite-specific configurations and common patterns.
/// </summary>
public abstract class BaseDbContext : DbContext
{
    protected BaseDbContext(DbContextOptions options) : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Configure DateTimeOffset for SQLite
        ConfigureDateTimeOffsetProperties(modelBuilder);
        
        // Configure RowVersion for all entities
        ConfigureRowVersionProperties(modelBuilder);
        
        // Configure common indexes for performance
        ConfigureCommonIndexes(modelBuilder);
        
        base.OnModelCreating(modelBuilder);
    }
    
    private static void ConfigureDateTimeOffsetProperties(ModelBuilder modelBuilder)
    {
        foreach (var entityType in modelBuilder.Model.GetEntityTypes())
        {
            foreach (var property in entityType.GetProperties())
            {
                if (property.ClrType == typeof(DateTimeOffset) || property.ClrType == typeof(DateTimeOffset?))
                {
                    property.SetColumnType("TEXT");
                }
            }
        }
    }
    
    private static void ConfigureRowVersionProperties(ModelBuilder modelBuilder)
    {
        foreach (var entityType in modelBuilder.Model.GetEntityTypes())
        {
            var rowVersionProperty = entityType.FindProperty("RowVersion");
            if (rowVersionProperty?.ClrType == typeof(byte[]))
            {
                modelBuilder.Entity(entityType.ClrType)
                    .Property("RowVersion")
                    .IsRowVersion()
                    .HasDefaultValue(new byte[] { 0, 0, 0, 0, 0, 0, 0, 1 });
            }
        }
    }
    
    private static void ConfigureCommonIndexes(ModelBuilder modelBuilder)
    {
        // Configure indexes for common query patterns
        foreach (var entityType in modelBuilder.Model.GetEntityTypes())
        {
            // Index on CreatedAt for time-based queries
            var createdAtProperty = entityType.FindProperty("CreatedAt");
            if (createdAtProperty != null)
            {
                modelBuilder.Entity(entityType.ClrType)
                    .HasIndex("CreatedAt")
                    .HasDatabaseName($"IX_{entityType.GetTableName()}_CreatedAt");
            }
            
            // Index on UserId for user-scoped queries
            var userIdProperty = entityType.FindProperty("UserId");
            if (userIdProperty != null)
            {
                modelBuilder.Entity(entityType.ClrType)
                    .HasIndex("UserId")
                    .HasDatabaseName($"IX_{entityType.GetTableName()}_UserId");
            }
        }
    }
}
```

### Application-Specific DbContext

```csharp
using Microsoft.EntityFrameworkCore;

/// <summary>
/// Main application DbContext with all entities and relationships.
/// </summary>
public class ApplicationDbContext : BaseDbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options)
    {
    }

    public DbSet<Product> Products { get; set; } = null!;
    public DbSet<User> Users { get; set; } = null!;
    public DbSet<Order> Orders { get; set; } = null!;

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);

        // Configure specific entity relationships and constraints
        ConfigureProduct(modelBuilder);
        ConfigureUser(modelBuilder);
        ConfigureOrder(modelBuilder);
    }

    private static void ConfigureProduct(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Product>(entity =>
        {
            entity.ToTable("Products");
            entity.HasKey(e => e.Id);

            entity.Property(e => e.Name)
                .IsRequired()
                .HasMaxLength(200);

            entity.Property(e => e.Description)
                .HasMaxLength(1000);

            // Performance: Index for common queries
            entity.HasIndex(e => e.CategoryId)
                .HasDatabaseName("IX_Products_CategoryId");

            // Performance: Composite index for complex queries
            entity.HasIndex(e => new { e.CategoryId, e.IsActive })
                .HasDatabaseName("IX_Products_Category_Active");
        });
    }

    private static void ConfigureUser(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<User>(entity =>
        {
            entity.Property(e => e.Email)
                .IsRequired()
                .HasMaxLength(255);

            // Unique constraint for data integrity
            entity.HasIndex(e => e.Email)
                .IsUnique()
                .HasDatabaseName("IX_Users_Email_Unique");
        });
    }

    private static void ConfigureOrder(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Order>(entity =>
        {
            // Configure relationships
            entity.HasOne(o => o.User)
                .WithMany(u => u.Orders)
                .HasForeignKey(o => o.UserId)
                .OnDelete(DeleteBehavior.Restrict);

            // Performance: Index for user queries
            entity.HasIndex(e => e.UserId)
                .HasDatabaseName("IX_Orders_UserId");
        });
    }
}
```

## Migration Best Practices

### Understanding SQLite Migration Limitations

SQLite requires table rebuilds for many schema changes. Be aware of operations that trigger rebuilds:

```csharp
// Operations that require rebuilds (slower):
// - AddForeignKey, DropForeignKey
// - AddPrimaryKey, DropPrimaryKey  
// - AlterColumn
// - AddCheckConstraint, DropCheckConstraint

// Operations that are fast:
// - AddColumn (at end of table)
// - CreateIndex, DropIndex
// - CreateTable, DropTable
// - RenameTable, RenameColumn
```

### Migration Naming and Organization

```bash
# Clear, descriptive migration names
dotnet ef migrations add InitialDatabaseSchema
dotnet ef migrations add AddUserTable
dotnet ef migrations add AddProductIndexes
dotnet ef migrations add FixRowVersionForSQLite

# Organize migrations in project structure
src/Data/Migrations/
├── 20250724_InitialDatabaseSchema.cs
├── 20250724_AddUserTable.cs
└── 20250724_FixRowVersionForSQLite.cs
```

### Safe Migration Patterns

```csharp
public partial class AddProductIndexes : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        // Add columns with defaults to avoid rebuild
        migrationBuilder.AddColumn<bool>(
            name: "IsActive",
            table: "Products",
            type: "INTEGER",
            nullable: false,
            defaultValue: true);

        // Use check constraints for validation
        migrationBuilder.AddCheckConstraint(
            name: "CK_Products_Price",
            table: "Products",
            sql: "Price >= 0");

        // Create indexes after table modifications
        migrationBuilder.CreateIndex(
            name: "IX_Products_CategoryId_IsActive",
            table: "Products",
            columns: new[] { "CategoryId", "IsActive" });
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        // Always provide rollback path
        migrationBuilder.DropCheckConstraint(
            name: "CK_Products_Price",
            table: "Products");

        migrationBuilder.DropIndex(
            name: "IX_Products_CategoryId_IsActive",
            table: "Products");

        migrationBuilder.DropColumn(
            name: "IsActive",
            table: "Products");
    }
}
```

## Connection String and Configuration Patterns

### Development Configuration

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Data Source={yourapp}.db;Cache=Shared;Foreign Keys=True;"
  },
  "Database": {
    "Provider": "SQLite",
    "EnableSensitiveDataLogging": false,
    "CommandTimeout": 30
  }
}
```

### Production Configuration

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=/app/data/{yourapp}.db;Cache=Shared;Foreign Keys=True;Journal Mode=WAL;"
  },
  "Database": {
    "Provider": "SQLite",
    "EnableSensitiveDataLogging": false,
    "CommandTimeout": 60,
    "EnablePooling": true,
    "MaxPoolSize": 100
  }
}
```

### Service Configuration

```csharp
public static IServiceCollection AddDatabaseServices(
    this IServiceCollection services, 
    IConfiguration configuration)
{
    var connectionString = configuration.GetConnectionString("DefaultConnection");
    var databaseConfig = configuration.GetSection("Database");

    services.AddDbContext<ApplicationDbContext>(options =>
    {
        options.UseSqlite(connectionString, sqliteOptions =>
        {
            sqliteOptions.MigrationsAssembly("{YourProject}.Data");
            sqliteOptions.CommandTimeout(databaseConfig.GetValue<int>("CommandTimeout", 30));
        });

        // Development settings
        if (configuration.GetValue<bool>("Database:EnableSensitiveDataLogging"))
        {
            options.EnableSensitiveDataLogging();
            options.EnableDetailedErrors();
        }

        // Performance settings
        if (configuration.GetValue<bool>("Database:EnablePooling", true))
        {
            options.EnableServiceProviderCaching();
            options.EnableSensitiveDataLogging(false); // Security: Disable in production
        }
    });

    // Configure connection pool for better performance
    services.AddDbContextPool<ApplicationDbContext>(options =>
        options.UseSqlite(connectionString), 
        poolSize: configuration.GetValue<int>("Database:MaxPoolSize", 100));

    return services;
}
```

## Performance Optimization Patterns

### Indexing Strategy

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // User-scoped queries (most common pattern)
    modelBuilder.Entity<Order>()
        .HasIndex(e => e.UserId)
        .HasDatabaseName("IX_Orders_UserId");

    // Composite indexes for complex queries
    modelBuilder.Entity<Product>()
        .HasIndex(e => new { e.CategoryId, e.IsActive, e.CreatedAt })
        .HasDatabaseName("IX_Products_Category_Active_Created");

    // Unique constraints for data integrity
    modelBuilder.Entity<User>()
        .HasIndex(e => e.Email)
        .IsUnique()
        .HasDatabaseName("IX_Users_Email_Unique");

    // Partial indexes for SQLite 3.8.0+ (if needed)
    modelBuilder.Entity<Product>()
        .HasIndex(e => e.ExpiresAt)
        .HasDatabaseName("IX_Products_ExpiresAt")
        .HasFilter("ExpiresAt IS NOT NULL");
}
```

### Query Optimization

```csharp
// Efficient user-scoped queries
public async Task<List<Order>> GetUserOrdersAsync(int userId, CancellationToken cancellationToken = default)
{
    return await _context.Orders
        .Where(o => o.UserId == userId && o.IsActive)
        .OrderBy(o => o.CreatedAt)
        .Select(o => new Order
        {
            Id = o.Id,
            UserId = o.UserId,
            TotalAmount = o.TotalAmount,
            CreatedAt = o.CreatedAt,
            // Select only needed properties for performance
        })
        .AsNoTracking() // Better performance for read-only queries
        .ToListAsync(cancellationToken);
}

// Optimized existence checks
public async Task<bool> EmailExistsAsync(string email, CancellationToken cancellationToken = default)
{
    return await _context.Users
        .AnyAsync(u => u.Email == email, cancellationToken);
}

// Batch operations for better performance
public async Task UpdateProductsLastAccessedAsync(IEnumerable<int> productIds, CancellationToken cancellationToken = default)
{
    var now = DateTimeOffset.UtcNow;
    
    await _context.Products
        .Where(p => productIds.Contains(p.Id))
        .ExecuteUpdateAsync(
            p => p.SetProperty(x => x.LastAccessedAt, now),
            cancellationToken);
}
```

### Connection Management

```csharp
// Repository pattern with proper disposal
public class ProductRepository : IProductRepository, IDisposable
{
    private readonly ApplicationDbContext _context;
    private bool _disposed = false;

    public ProductRepository(ApplicationDbContext context)
    {
        _context = context ?? throw new ArgumentNullException(nameof(context));
    }

    public async Task<Product> CreateAsync(Product product, CancellationToken cancellationToken = default)
    {
        if (product == null)
            throw new ArgumentNullException(nameof(product));

        product.CreatedAt = DateTimeOffset.UtcNow;
        product.UpdatedAt = DateTimeOffset.UtcNow;

        _context.Products.Add(product);
        await _context.SaveChangesAsync(cancellationToken);

        return product;
    }

    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed && disposing)
        {
            _context?.Dispose();
        }
        _disposed = true;
    }

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }
}
```

## Security and OWASP Compliance

### Input Validation and Sanitization

```csharp
public class SecurityValidationService
{
    // Validate against SQLite injection patterns
    public static bool IsValidIdentifier(string identifier)
    {
        if (string.IsNullOrWhiteSpace(identifier))
            return false;

        // SQLite identifiers: alphanumeric, underscore, max 64 chars
        return identifier.Length <= 64 && 
               identifier.All(c => char.IsLetterOrDigit(c) || c == '_') &&
               char.IsLetter(identifier[0]);
    }

    // Parameterized queries prevent injection
    public async Task<Product?> GetProductByIdAsync(int id, int userId)
    {
        // EF Core automatically parameterizes this query
        return await _context.Products
            .FirstOrDefaultAsync(p => p.Id == id && p.UserId == userId);
    }

    // Never use raw SQL with user input
    // BAD: _context.Database.ExecuteSqlRaw($"SELECT * FROM Products WHERE UserId = '{userId}'");
    // GOOD: Use parameterized queries or LINQ
}
```

### Encryption at Application Level

```csharp
public class DataProtectionService
{
    private readonly IDataProtectionProvider _dataProtectionProvider;
    private readonly IDataProtector _protector;

    public DataProtectionService(IDataProtectionProvider dataProtectionProvider)
    {
        _dataProtectionProvider = dataProtectionProvider;
        _protector = _dataProtectionProvider.CreateProtector("{YourApp}.SensitiveData");
    }

    public string EncryptData(string plainData)
    {
        if (string.IsNullOrEmpty(plainData))
            throw new ArgumentException("Data cannot be null or empty");

        return _protector.Protect(plainData);
    }

    public string DecryptData(string encryptedData)
    {
        if (string.IsNullOrEmpty(encryptedData))
            throw new ArgumentException("Encrypted data cannot be null or empty");

        return _protector.Unprotect(encryptedData);
    }
}
```

## Error Handling and Logging

### Database Operation Error Handling

```csharp
public async Task<Result<Product>> CreateProductAsync(Product product, CancellationToken cancellationToken = default)
{
    try
    {
        // Validate input
        var validationResult = ValidateProduct(product);
        if (!validationResult.IsValid)
        {
            return Result<Product>.Failure(validationResult.ErrorMessage);
        }

        // Encrypt sensitive data if needed
        if (!string.IsNullOrEmpty(product.SensitiveData))
        {
            product.SensitiveData = _dataProtectionService.EncryptData(product.SensitiveData);
        }
        
        _context.Products.Add(product);
        await _context.SaveChangesAsync(cancellationToken);

        _logger.LogInformation("Product created with ID {ProductId}", product.Id);
        return Result<Product>.Success(product);
    }
    catch (DbUpdateException ex) when (ex.InnerException is SqliteException sqliteEx)
    {
        // Handle SQLite-specific errors
        return sqliteEx.SqliteErrorCode switch
        {
            SqliteErrorCode.Constraint => Result<Product>.Failure("Data constraint violation"),
            SqliteErrorCode.Corrupt => Result<Product>.Failure("Database corruption detected"),
            SqliteErrorCode.Full => Result<Product>.Failure("Storage space insufficient"),
            _ => Result<Product>.Failure("Database operation failed")
        };
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Failed to create product");
        return Result<Product>.Failure("An unexpected error occurred");
    }
}
```

### Migration Error Handling

```csharp
public static async Task<bool> MigrateDatabaseAsync(IServiceProvider serviceProvider, ILogger logger)
{
    try
    {
        using var scope = serviceProvider.CreateScope();
        var context = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
        
        // Check for migration lock table (SQLite EF9+ protection)
        await CheckMigrationLockAsync(context, logger);
        
        await context.Database.MigrateAsync();
        logger.LogInformation("Database migration completed successfully");
        return true;
    }
    catch (Exception ex)
    {
        logger.LogError(ex, "Database migration failed");
        return false;
    }
}

private static async Task CheckMigrationLockAsync(ApplicationDbContext context, ILogger logger)
{
    try
    {
        // Check if migration lock table exists and has stuck locks
        var lockExists = await context.Database.ExecuteScalarAsync<int>(
            "SELECT COUNT(*) FROM sqlite_master WHERE type='table' AND name='__EFMigrationsLock'") > 0;
            
        if (lockExists)
        {
            var lockCount = await context.Database.ExecuteScalarAsync<int>(
                "SELECT COUNT(*) FROM __EFMigrationsLock");
                
            if (lockCount > 0)
            {
                logger.LogWarning("Migration lock detected. Manual intervention may be required.");
                // Consider implementing automatic lock cleanup with caution
            }
        }
    }
    catch (Exception ex)
    {
        logger.LogWarning(ex, "Could not check migration lock status");
    }
}
```

## Testing Patterns

### In-Memory Database for Unit Tests

```csharp
public class ProductRepositoryTests : IDisposable
{
    private readonly ApplicationDbContext _context;
    private readonly ProductRepository _repository;

    public ProductRepositoryTests()
    {
        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseSqlite("Data Source=:memory:")
            .Options;

        _context = new ApplicationDbContext(options);
        _context.Database.OpenConnection(); // Keep connection open for in-memory DB
        _context.Database.EnsureCreated();

        _repository = new ProductRepository(_context);
    }

    [Fact]
    public async Task CreateProductAsync_ValidInput_ReturnsSuccess()
    {
        // Arrange
        var product = new Product
        {
            Name = "Test Product",
            Description = "Test Description",
            Price = 99.99m,
            CategoryId = 1
        };

        // Act
        var result = await _repository.CreateAsync(product);

        // Assert
        Assert.NotNull(result);
        Assert.True(result.Id > 0);
        Assert.Equal("Test Product", result.Name);
    }

    public void Dispose()
    {
        _context?.Dispose();
    }
}
```

### Integration Testing with Real SQLite

```csharp
public class DatabaseIntegrationTests : IClassFixture<DatabaseFixture>
{
    private readonly DatabaseFixture _fixture;

    public DatabaseIntegrationTests(DatabaseFixture fixture)
    {
        _fixture = fixture;
    }

    [Fact]
    public async Task MigrationApplies_Successfully()
    {
        using var scope = _fixture.ServiceProvider.CreateScope();
        var context = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();

        // Ensure database is created and migrations applied
        await context.Database.MigrateAsync();

        // Verify table structure
        var tableExists = await context.Database.ExecuteScalarAsync<int>(
            "SELECT COUNT(*) FROM sqlite_master WHERE type='table' AND name='Products'");
            
        Assert.Equal(1, tableExists);
    }
}
```

## Common Issues and Solutions

### Issue: "NOT NULL constraint failed: Entity.RowVersion"

**Root Cause**: SQLite doesn't auto-generate RowVersion values like SQL Server

**Solution**: Configure default values in OnModelCreating

```csharp
modelBuilder.Entity<MyEntity>()
    .Property(e => e.RowVersion)
    .IsRowVersion()
    .HasDefaultValue(new byte[] { 0, 0, 0, 0, 0, 0, 0, 1 });
```

### Issue: DateTimeOffset queries return incorrect results

**Root Cause**: SQLite stores DateTimeOffset as TEXT, affecting comparisons

**Solution**: Always use UTC for comparisons and convert in application layer

```csharp
// Store all dates as UTC
entity.CreatedAt = DateTimeOffset.UtcNow;

// Query with UTC conversion
var results = await _context.Entities
    .Where(e => e.CreatedAt >= searchDate.ToUniversalTime())
    .ToListAsync();
```

### Issue: Migration fails with "table rebuild" error

**Root Cause**: SQLite limitations require table rebuilds for certain operations

**Solution**: Use migrations that avoid problematic operations or implement manual rebuild

```csharp
// Instead of AlterColumn (requires rebuild)
migrationBuilder.AddColumn<string>("NewColumn", "MyTable", nullable: true);
// Then copy data and drop old column in subsequent migration
```

### Issue: Database locked during concurrent access

**Root Cause**: SQLite file locking in multi-threaded scenarios

**Solution**: Use connection pooling and WAL mode

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Data Source={yourapp}.db;Cache=Shared;Journal Mode=WAL;"
  }
}
```

### Issue: Poor query performance

**Root Cause**: Missing indexes or client-side evaluation

**Solution**: Add appropriate indexes and analyze query execution

```csharp
// Add indexes for common query patterns
modelBuilder.Entity<Product>()
    .HasIndex(e => new { e.CategoryId, e.IsActive })
    .HasDatabaseName("IX_Products_Category_Active");

// Use AsNoTracking() for read-only queries
var products = await _context.Products
    .AsNoTracking()
    .Where(p => p.CategoryId == categoryId)
    .ToListAsync();
```

## Compliance and Security

This guide ensures:

- **OWASP A03 (Injection)**: Parameterized queries prevent SQL injection
- **OWASP A02 (Cryptographic Failures)**: Application-level encryption for sensitive data
- **OWASP A05 (Security Misconfiguration)**: Secure database configuration patterns
- **Performance**: Optimized indexing and query patterns
- **Maintainability**: Clear separation of concerns and error handling
- **Testing**: Comprehensive unit and integration testing patterns

## References

- [SQLite EF Core Provider Limitations](https://learn.microsoft.com/en-us/ef/core/providers/sqlite/limitations)
- [EF Core Migrations](https://learn.microsoft.com/en-us/ef/core/managing-schemas/migrations/)
- [SQLite Documentation](https://www.sqlite.org/docs.html)
- [Entity Framework Core Best Practices](https://learn.microsoft.com/en-us/ef/core/miscellaneous/configuring-dbcontext)
