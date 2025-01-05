# Decorator Pattern Documentation

The Decorator Pattern is a structural design pattern that allows behavior to be added to individual objects, dynamically and independently, without affecting the behavior of other objects from the same class. This pattern is especially useful for adhering to the OpenClosed Principle, as it allows for the extension of functionality without modifying existing code.

## Key Concepts of the Decorator Pattern

1. Interface The pattern involves an interface that both the core class and its decorators implement. This enables interchangeability.
2. Component This is the core functionality to which additional behaviors will be added.
3. Decorator Class A decorator class wraps the core component to add additional behavior dynamically.
4. Usage Ideal for cases where behavior needs to be added to objects individually or to satisfy different requirements without altering the class structure.

### Benefits

- Flexible Decorators can be used to stack behaviors on top of each other dynamically.
- Adheres to OpenClosed Principle Allows extending functionality without modifying existing code.
- Enhanced Reusability Individual decorators can be reused across multiple contexts.

### Common Use Cases

- Logging, authorization, caching, and validation are often implemented as decorators in complex systems.
- Wrapping specific behavior such as adding validation layers in service-oriented architectures.

## Example Seed Data Decorator in a .NET Project

In this project, we use the Decorator Pattern to manage seeding different types of data (e.g., general, documents, sales, supplier) for a tenant dynamically. Each seeder class implements the `ISeedData` interface, and a `SeedDataDecorator` class wraps and iterates over the seeders to execute them in sequence.

### Code Example

Here's how the Decorator Pattern is applied in the code

```csharp
 Interface for seeding data
public interface ISeedData
{
    Task SeedAsync(Tenant tenant, Employee employee, ServicesDbContext dbContext);
}

 General data seeder implementing ISeedData
public class GeneralSeeder  ISeedData
{
    private readonly RoleManagerRole _roleManager;
    public GeneralSeeder(RoleManagerRole roleManager)
    {
        _roleManager = roleManager;
    }

    public Task SeedAsync(Tenant tenant, Employee employee, ServicesDbContext dbContext)
    {
        return Task.Run(async () =
        {
            await SeedTaxes(tenant, dbContext);
            await SeedRoles(tenant, dbContext);
            await SeedLocations(tenant, dbContext);
            await SeedOperationUnits(tenant, dbContext);
            await SeedDepartments(tenant, dbContext);
            await SeedPaymentTerms(tenant, dbContext);
            await SeedProductUOMs(tenant, dbContext);
            await SeedProblemTypes(tenant, dbContext);
        });
    }
}

 Additional seeders (DocumentsSeeder, SalesSeeder, SupplierSeeder) implementing ISeedData
public class DocumentsSeeder  ISeedData
{
    public async Task SeedAsync(Tenant tenant, Employee employee, ServicesDbContext dbContext)
    {
        await SeedCorrectiveActionCodes(tenant, dbContext);
        await SeedRootCauseCodes(tenant, dbContext);
        await SeedDefectCodes(tenant, dbContext);
        await SeedDispositionTypes(tenant, dbContext);
        await SeedDocumentClasses(tenant, dbContext);
        await SeedAutoNumber(tenant, dbContext);
        await SeedDispositionMailSettings(tenant, dbContext);
    }
}

public class SalesSeeder  ISeedData
{
    public async Task SeedAsync(Tenant tenant, Employee employee, ServicesDbContext dbContext)
    {
        await SeedDeliveryTerms(tenant, dbContext);
        await SeedWarehousePriority(tenant, employee, dbContext);
        await SeedSOPDFSettingDoc(tenant, employee, dbContext);
        await SeedInvoiceSettings(tenant, dbContext);
    }
}

public class SupplierSeeder  ISeedData
{
    public async Task SeedAsync(Tenant tenant, Employee employee, ServicesDbContext dbContext)
    {
        await SeedAttributes(tenant, dbContext);
        await SeedCertifications(tenant, dbContext);
        await SeedSupplierServices(tenant, dbContext);
        await SeedRatingParameters(tenant, dbContext);
        await SeedAuditCategories(tenant, dbContext);
        await SeedTrainingTypes(tenant, dbContext);
        await SeedWorkCenters(tenant, dbContext);
        await SeedEquipmentTypes(tenant, dbContext);
        await SeedIncidentTypes(tenant, dbContext);
        await SeedInjuryTypes(tenant, employee, dbContext);
    }
}

 Decorator class to manage and execute all seeders
public class SeedDataDecorator
{
    private readonly IServiceProvider _serviceProvider;
    public SeedDataDecorator(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }

    public async Task SeedAllAsync(Tenant tenant, Employee employee, ServicesDbContext dbContext)
    {
        var seeders = _serviceProvider.GetServicesISeedData();
        foreach (var seeder in seeders)
        {
            await seeder.SeedAsync(tenant, employee, dbContext);
        }
    }
}

 Example Usage in Tenant Repository
public class TenantRepositoryEF  ITenantRepository
{
    private SeedDataDecorator _seedDataDecorator { get; set; }
    private readonly IServiceProvider _serviceProvider;
    private readonly ServicesDbContext _dbContext;

    public TenantRepositoryEF(
        ServicesDbContext dbContext,
        IServiceProvider serviceProvider
    )
    {
        _serviceProvider = serviceProvider;
        _dbContext = dbContext;
    }

    public TaskResultbool AddTenant(TenantVM tenantVM)
    {
        return Task.Run(async () =
        {
             Code for adding tenant
            _seedDataDecorator = new SeedDataDecorator(_serviceProvider);
            await _seedDataDecorator.SeedAllAsync(tenant, admin, _dbContext);
        });
    }
}
```

## Explanation of the Example

### Interface `ISeedData`
- Definition The `ISeedData` interface defines a `SeedAsync` method, which each specific seeder must implement.

### Concrete Seeders
- Classes Seeders like `GeneralSeeder`, `DocumentsSeeder`, `SalesSeeder`, and `SupplierSeeder` implement the `ISeedData` interface to perform specific data seeding operations.

### Decorator Class `SeedDataDecorator`
- Function This class aggregates all seeders, using dependency injection (`IServiceProvider`) to retrieve each implementation of `ISeedData`.
- Method The `SeedAllAsync` method iterates over the seeders and calls `SeedAsync` on each one.

### Usage in `TenantRepositoryEF`
- Integration The `TenantRepositoryEF` class utilizes `SeedDataDecorator` to call `SeedAllAsync`, ensuring that all relevant data is seeded when a new tenant is added.

## Benefits in this Context

### Modular Data Seeding
- Structure Each seeder handles a specific type of data, making it easy to extend or modify data seeding without affecting other components.

### Flexible and Scalable
- Expansion New seeders can be added by implementing the `ISeedData` interface, and they will automatically be picked up by `SeedDataDecorator`.

### Adherence to SOLID Principles
- Principles This pattern adheres to the OpenClosed Principle and Single Responsibility Principle by decoupling the seeding logic into separate classes.
