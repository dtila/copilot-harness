---
name: backend-integration-tests
description: For writing or updating [projectname].* integration tests (*IntegrationTests.cs): AppServerFixture, DefaultAppServerFixture, TransactionScopedFixture, PostgreSQL container patterns, outbox verification, happy path + key negative paths, and CategoryTrait attribute usage.
---

Use this skill whenever backend behavior changes.

## Where tests live

- Integration tests: `src/[projectname].Application.Tests/`
- Feature tests grouped by domain: `src/[projectname].Application.Tests/Features/<Domain>/`
- File name **must** end with `IntegrationTests.cs` (triggers scoped guardrails)
- `<Domain>` matches the handler folder: handler in `Features/Team/Commands/` → test in `Features/Team/`

## Hard Rules

- `[Trait(nameof(Category), Category.IsolatedIntegrationTest)]` on every test class.
- `[Collection(DatabaseCollection.DatabaseCollectionName)]` on every test class.
- **Real PostgreSQL** via `AppServerFixture` or `DefaultAppServerFixture` — no in-memory DB.
- **Positive paths only** — no error, validation-failure, or not-found scenarios (those are unit tests).
- **No AAA comments** — never write `// Arrange`, `// Act`, `// Assert`.
- **No conditionals** inside a test method — split into separate `[Fact]` / `[Theory]` instead.
- Use `TransactionScopedFixture.CreateAsync(_factory)` — every test runs in a rolled-back transaction.
- Create test data directly via `scope.Context` — never query a live DB.
- `scope.Mediator.Send(...)` to invoke handlers — never instantiate Handler directly.
- `scope.Context` for direct DB verification after the handler runs.


## Pattern 1 — With custom mocks (`IUserProvider`, `IBlobStorage`, etc.)

Use `AppServerFixture` + **must implement `IDisposable`** to dispose the factory.

```csharp
namespace [projectname].Application.Tests.Features.[Domain].[Commands|Queries];

[Trait(nameof(Category), Category.IsolatedIntegrationTest)]
[Collection(DatabaseCollection.DatabaseCollectionName)]
public class [HandlerName]IntegrationTests : IDisposable
{
    private readonly ProcurementJobWebApplication _factory;
    private readonly Mock<IUserProvider> _mockUserProvider = new();

    public [HandlerName]IntegrationTests(AppServerFixture fixture, ITestOutputHelper output)
    {
        _mockUserProvider.Setup(x => x.IsAuthenticated).Returns(true);

        _factory = fixture.Create(builder => builder
            .ConfigureTestServices(services => services
                .AddScoped<IUserProvider>(_ => _mockUserProvider.Object)
            )
        );
        fixture.ConfigureLogging(output);
    }

    [Theory]
    [InlineData(/* variation A */)]
    [InlineData(/* variation B */)]
    public async Task Handle_[Scenario]_[ExpectedOutcome](/* params */)
    {
        await using var scope = await TransactionScopedFixture.CreateAsync(_factory);

        var userId = Guid.NewGuid();
        _mockUserProvider.Setup(x => x.ActingExternalUserId).Returns(userId);

        var entity = await CreateTestEntityAsync(scope.Context, userId);

        var request = new [HandlerName].Request { [EntityId] = entity.Id /*, other props */ };

        var response = await scope.Mediator.Send(request);

        Assert.NotNull(response);
        response.Id.Should().Be(entity.Id);

        // For Commands: verify persistence
        var saved = await scope.Context.[Entities]
            .AsNoTracking()
            .SingleAsync(x => x.Id == entity.Id);
        saved.[ChangedProp].Should().Be(/* expected */);

        // For Commands with domain events: verify outbox
        var outbox = scope.Context.OutboxEvents
            .Where(e => e.AggregateId == entity.Id.ToString())
            .ToList();
        outbox.Should().HaveCountGreaterThan(0);
    }

    private static async Task<[Entity]> CreateTestEntityAsync(ContentDbContext context, Guid ownerId)
    {
        var entity = new [Entity](/* constructor args */);
        context.[Entities].Add(entity);
        await context.SaveChangesAsync();
        return entity;
    }

    public void Dispose() => _factory.Dispose();
}
```


## Pattern 2 — Without custom mocks (default configuration)

Use `IClassFixture<DefaultAppServerFixture>` — **no `IDisposable`** needed (xUnit handles it).

```csharp
namespace [projectname].Application.Tests.Features.[Domain].[Commands|Queries];

[Trait(nameof(Category), Category.IsolatedIntegrationTest)]
[Collection(DatabaseCollection.DatabaseCollectionName)]
public class [HandlerName]IntegrationTests : IClassFixture<DefaultAppServerFixture>
{
    private readonly ProcurementJobWebApplication _factory;

    public [HandlerName]IntegrationTests(DefaultAppServerFixture fixture, ITestOutputHelper output)
    {
        _factory = fixture.Factory;
        fixture.ConfigureLogging(output);
    }

    [Theory]
    [InlineData(/* variation A */)]
    [InlineData(/* variation B */)]
    public async Task Handle_[Scenario]_[ExpectedOutcome](/* params */)
    {
        await using var scope = await TransactionScopedFixture.CreateAsync(_factory);

        var entity = await CreateTestEntityAsync(scope.Context);

        var request = new [HandlerName].Request { [EntityId] = entity.Id };

        var response = await scope.Mediator.Send(request);

        Assert.NotNull(response);
        response.[Field].Should().Be(/* expected */);

        var saved = await scope.Context.[Entities].AsNoTracking().SingleAsync(x => x.Id == entity.Id);
        saved.[ChangedProp].Should().Be(/* expected */);
    }

    private static async Task<[Entity]> CreateTestEntityAsync(ContentDbContext context)
    {
        var entity = new [Entity](/* constructor args */);
        context.[Entities].Add(entity);
        await context.SaveChangesAsync();
        return entity;
    }
}
```


## Choosing Pattern 1 vs Pattern 2

| Need | Pattern |
|---|---|
| Test requires authenticated user (`IUserProvider`) | Pattern 1 |
| Test requires blob storage, email, or other external service mock | Pattern 1 |
| Simple command/query with no user context | Pattern 2 |


## Test Method Naming

`Handle_[WhatHappened]_[WhatIsExpected]`

Examples:
- `Handle_ValidSubscription_PersistsWithCorrectDates`
- `Handle_WithRelatedEntity_ReturnsJoinedData`
- `Handle_PolicyApplied_OutboxEventPublished`


## Acceptance Criteria

- Tests use real PostgreSQL (no in-memory).
- Every test class has `[Trait]` + `[Collection]`.
- Pattern 1 classes implement `IDisposable` and dispose `_factory`.
- Pattern 2 classes use `IClassFixture<DefaultAppServerFixture>` and have no `IDisposable`.
- No `// Arrange`, `// Act`, `// Assert` comments.
- No conditionals inside test methods.
- Test data created via `scope.Context` fixture helpers — no live DB assumptions.
- `scope.Mediator.Send(...)` used to invoke handlers.