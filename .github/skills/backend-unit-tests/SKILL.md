---
name: backend-unit-tests
description: For writing [projectname].* unit tests for domain logic, validators, value objects, or handlers in isolation (no real DB): InMemorySqliteTestScope, Assert.* only (no FluentAssertions), UnitTest CategoryTrait, per-concern test file structure, and fast in-memory SQLite patterns.
---

Use this skill when testing domain entities, validators, value objects, or application logic that does not require database integration.

## Where unit tests live

- `src/[projectname].Domain.Tests/` — domain entity / value object tests
- `src/[projectname].Application.Tests/**/*UnitTests.cs` — validator and handler logic tests

File name **must** end with `UnitTests.cs` (triggers scoped unit-test guardrails).

## Hard Rules

- `[Trait(nameof(Category), Category.UnitTest)]` on every test class.
- **No database** — use `InMemorySqliteTestScope` for handlers; plain `new()` for domain entities and validators.
- **Fast** — unit tests must complete in milliseconds. No Docker, no PostgreSQL.
- **One concern per test** — one validation rule, one domain invariant, one business rule.
- `Assert.*` for assertions — **never** `x.Should()` FluentAssertion syntax (FluentAssertions is integration-test only).
- No `// Arrange`, `// Act`, `// Assert` comments.
- No conditionals inside test methods.
- Mock external dependencies (`IUserProvider`, `IBlobStorage`, etc.) with `Moq`.


## Stub 1 — Domain entity / value object test

```csharp
namespace [projectname].Content.Tests.[Domain];

[Trait(nameof(Category), Category.UnitTest)]
public class [Entity]Tests
{
    [Theory]
    [InlineData(/* valid input A */)]
    [InlineData(/* valid input B */)]
    public void [MethodOrBehaviour]_WithValidInput_[ExpectedOutcome](/* params */)
    {
        var entity = new [Entity](/* valid constructor args */);

        entity.[Method](/* args */);

        Assert.Equal(/* expected */, entity.[Property]);
    }

    [Fact]
    public void [MethodOrBehaviour]_WhenInvalidState_ThrowsInvalidOperationException()
    {
        var entity = new [Entity](/* args that lead to invalid state */);

        Assert.Throws<InvalidOperationException>(() => entity.[Method](/* args */));
    }
}
```

---

## Stub 2 — Value object / parsing test

```csharp
namespace [projectname].Content.Tests;

[Trait(nameof(Category), Category.UnitTest)]
public class [ValueObject]Tests
{
    [Theory]
    [InlineData("valid input", /* expected field 1 */, /* expected field 2 */)]
    public void Parse_ValidInput_ReturnsExpectedValue(string input, /* expected params */)
    {
        var result = [ValueObject].Parse(input);

        Assert.True(result.IsSuccess);
        Assert.Equal(/* expected field 1 */, result.Value.[Field1]);
        Assert.Equal(/* expected field 2 */, result.Value.[Field2]);
    }

    [Theory]
    [InlineData("")]
    [InlineData(null)]
    public void Parse_InvalidInput_ReturnsFailure(string? input)
    {
        var result = [ValueObject].Parse(input);

        Assert.False(result.IsSuccess);
    }
}
```

---

## Stub 3 — FluentValidation validator test

```csharp
namespace [projectname].Application.Tests.Features.[Domain].Commands;

[Trait(nameof(Category), Category.UnitTest)]
public class [CommandName]ValidatorUnitTests
{
    private readonly [CommandName].Validator _sut = new();

    [Fact]
    public void Validate_ValidRequest_PassesWithNoErrors()
    {
        var request = new [CommandName].Request
        {
            [Property] = /* valid value */,
        };

        var result = _sut.TestValidate(request);

        result.ShouldNotHaveAnyValidationErrors();
    }

    [Fact]
    public void Validate_MissingRequiredField_FailsOnField()
    {
        var request = new [CommandName].Request
        {
            [Property] = /* invalid / default value */,
        };

        var result = _sut.TestValidate(request);

        result.ShouldHaveValidationErrorFor(x => x.[Property]);
    }

    [Fact]
    public void Validate_DateTimeNotUtc_FailsOnDateField()
    {
        var request = new [CommandName].Request
        {
            [DateField] = DateTime.Now,   // Local kind — must fail
        };

        var result = _sut.TestValidate(request);

        result.ShouldHaveValidationErrorFor(x => x.[DateField]);
    }
}
```

---

## Stub 4 — Handler unit test (with `InMemorySqliteTestScope`)

Use this when you want to test handler business logic without PostgreSQL.

```csharp
namespace [projectname].Application.Tests.Features.[Domain].Commands;

[Trait(nameof(Category), Category.UnitTest)]
public class [HandlerName]HandlerUnitTests : IAsyncLifetime
{
    private readonly InMemorySqliteTestScope _scope = new();
    private readonly Mock<IUserProvider> _mockUserProvider = new();
    private ContentDbContext _context = null!;
    private [HandlerName].Handler _sut = null!;

    public async Task InitializeAsync()
    {
        _context = await _scope.CreateContextAsync();
        var validator = new [HandlerName].Validator();
        _sut = new [HandlerName].Handler(validator, _context, _mockUserProvider.Object /*, other deps */);
    }

    public Task DisposeAsync() => _scope.DisposeAsync().AsTask();

    [Fact]
    public async Task Handle_ValidRequest_[ExpectedOutcome]()
    {
        var userId = Guid.NewGuid();
        _mockUserProvider.Setup(x => x.ActingExternalUserId).Returns(userId);

        var entity = new [Entity](/* constructor args */);
        _context.[Entities].Add(entity);
        await _context.SaveChangesAsync();

        var request = new [HandlerName].Request { [EntityId] = entity.Id };

        var response = await _sut.Handle(request, default);

        Assert.NotNull(response);
        Assert.Equal(entity.Id, response.Id);

        var saved = await _context.[Entities].SingleAsync(x => x.Id == entity.Id);
        Assert.Equal(/* expected */, saved.[Property]);
    }
}
```

---

## What to unit test vs integration test

| Scenario | Test type |
|---|---|
| Domain entity invariant (e.g., `Team.ActivateSubscription` throws when expired) | Unit (Domain.Tests) |
| Value object parsing (`Money.Parse`, `NUTSCode.Parse`) | Unit (Domain.Tests) |
| Validator rule (each `RuleFor` clause) | Unit (Application.Tests — `*UnitTests.cs`) |
| Handler business rule (no DB query needed to check) | Unit (Application.Tests — `*UnitTests.cs`) |
| Handler EF Core query, persistence, outbox events | Integration (Application.Tests — `*IntegrationTests.cs`) |

---

## Acceptance Criteria

- `[Trait(nameof(Category), Category.UnitTest)]` present on every class.
- No PostgreSQL — SQLite (`InMemorySqliteTestScope`) or plain `new()` only.
- `Assert.*` used — no `.Should()` FluentAssertion syntax.
- No `// Arrange`, `// Act`, `// Assert` comments.
- No conditionals inside test methods.
- Tests complete in milliseconds.