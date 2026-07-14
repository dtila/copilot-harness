---

name: backend-test-datafixture
description: Write DataFixture helper classes for integration and unit tests. Pure in-memory factories, no database access, fluent extension method chaining. Use when creating test data factories in src/[projectname].Application.Tests/Features/<Domain>/.

---

Use this skill when creating or extending a `*DataFixture.cs` class in:

```
src/[projectname].Application.Tests/Features/<Domain>/<Domain>DataFixture.cs
```

---

# Core Rules

1. **Never touch the database.** No `context.Add()`, `context.SaveChangesAsync()`, or `DbContext` references inside a fixture. The **test** is responsible for all persistence.
2. **Pure factory methods only.** Every method creates and returns domain objects in memory.
3. **Extension methods for fluent chaining.** Build object graphs fluently — `company.WithProcedure()`, `procedure.WithFile()`, `invitation.WithAward()`.
4. **Return the aggregate root.** When a method creates multiple related objects, return the root — EF Core persists the whole graph. Only use tuples when the two objects belong to different aggregate roots.
5. **Use `IDomainBus` mock internally.** Never expose the mock — it's an implementation detail.
6. **Use AutoFixture for random scalar data.** Avoid hard-coded magic strings/integers except for well-known stable constants.
7. **All methods are synchronous.** DataFixture never awaits anything.
8. **Trim scalar strings to domain length constraints** so they don't cause DB constraint violations in tests.

---

# Class Structure

```csharp
using AutoFixture;
using Moq;
using SharedKernel.Messaging;
using [projectname].Domain.<DomainNamespace>;

namespace [projectname].Application.Tests.Features.<Domain>;

public static class <Domain>DataFixture
{
    private static readonly Fixture _fixture = new();
    private static readonly Mock<IDomainBus> _bus = new();

    // Well-known stable constants (optional)
    public static readonly SomeId KnownId = SomeId.Create("...").Value;

    // Root factory
    public static DomainObject CreateDomainObject() { ... }

    // Fluent extension that mutates the parent and returns the child
    public static Child WithChild(this Parent parent, ...) { ... }

    // Fluent extension returning the aggregate root (EF persists the full graph)
    public static AggregateRoot WithChild(this Parent parent, ...) { ... }
}
```

---

# Pattern: Root Factory

Creates the aggregate root. No arguments required; all data is random.

```csharp
public static Team CreateTeam()
{
    return Team.Create(
        id: _fixture.Create<int>(),
        name: Truncate(_fixture.Create<string>(), 50)
    ).Value;
}
```

---

# Pattern: Fluent `.WithX()` Extension

Attaches a child to the parent. Mutates the parent when the domain API requires it (e.g., `procedure.AddFile()`). Returns the child.

```csharp
public static ProcedureFile WithFile(this Procedure procedure, ...)
{
    var name = _fixture.Create<string>() + ".pdf";

    var procedureFile = ProcedureFile.CreateOriginal(
        id: _fixture.Create<int>(),
        name: name,
        order: procedure.Files.Aggregate(0, (max, f) => Math.Max(max, f.Order)) + 1,
        path: $"seap/files/{name}/{Guid.NewGuid()}",
        procedure: procedure,
        bus: _bus.Object);

    procedure.AddFile(procedureFile, DateTime.UtcNow);
    return procedureFile;
}
```

---

# Pattern: Return the Aggregate Root

When a method creates multiple related objects, return the **aggregate root** that owns them. EF Core will persist the entire graph when the root is added to the context. The caller adds only the root.

`TeamProcedureFile` is the aggregate: it holds a `File` navigation property. Returning it gives the caller everything.

```csharp
public static TeamProcedureFile WithTeamFile(this Procedure procedure, int teamId)
{
    var name = _fixture.Create<string>() + ".pdf";
    var path = $"team/files/{name}/{Guid.NewGuid()}";

    var file = DomainFile.CreateOriginal(
        id: _fixture.Create<int>(),
        name: name,
        path: path,
        bus: _bus.Object);

    return TeamProcedureFile.Create(
        file: file,
        procedure: procedure,
        teamId: teamId,
        order: 1,
        bus: _bus.Object);
}
```

Caller pattern in the test:

```csharp
var team      = TeamDataFixture.CreateTeam();
var procedure = ProcedureDataFixture.CreateRandomProcedure();
scope.Context.Add(procedure.Authority.Company);
scope.Context.Add(team);
scope.Context.Add(procedure);
await scope.Context.SaveChangesAsync();

// Adding the aggregate root persists File + TeamProcedureFile in one shot
var teamFile = TeamDataFixture.WithTeamFile(procedure, team.Id);
scope.Context.Add(teamFile);
await scope.Context.SaveChangesAsync();
```

---

# Utility: String Truncation Helper

Domain entities often enforce max-length constraints. Always truncate:

```csharp
private static string Truncate(string value, int maxLength)
    => value.Length <= maxLength ? value : value[..maxLength];
```

---

# Utility: AutoFixture Random Scalars

| Data type         | Pattern                                                           |
|-------------------|-------------------------------------------------------------------|
| `int`             | `_fixture.Create<int>()`                                          |
| `string` (trimmed)| `Truncate(_fixture.Create<string>(), maxLength)`                  |
| `Guid`            | `_fixture.Create<Guid>()`                                         |
| `enum`            | `_fixture.Create<ProcedureType>()`                                |
| `DateTime` (UTC)  | `DateTime.UtcNow.AddDays(-Random.Shared.Next(1, 30)).ToUniversalTime()` |
| Positive int      | `Math.Abs(_fixture.Create<int>()) + 1`                            |

---

# Anti-Patterns — Never Do These

| Anti-pattern                                      | Why                                              |
|---------------------------------------------------|--------------------------------------------------|
| `context.Add(entity)` inside a fixture            | Couples fixture to EF; impossible to compose      |
| `await context.SaveChangesAsync()` in fixture     | Fixture must be sync and side-effect-free         |
| Hard-coded IDs like `id: 42`                      | Causes primary-key conflicts across tests         |
| `async` fixture methods                            | Fixtures are factories, not test steps            |
| Assertions or `Assert.*` inside fixture methods   | Fixtures produce data; tests verify behaviour     |
| Injecting `DbContext` as parameter                | Defeats composability; hides persistence intent   |

---

# Where to Place Fixtures

| Fixture file                                              | Owned domain      |
|-----------------------------------------------------------|-------------------|
| `Features/Companies/CompanyDataFixture.cs`                | Company           |
| `Features/Procedures/ProcedureDataFixture.cs`             | Procedure, Notice |
| `Features/Files/FilesDataFixture.cs`                      | File, ProcedureFile |
| `Features/Team/TeamDataFixture.cs`                        | Team, TeamProcedureFile |
| `Features/Account/UserDataFixture.cs`                     | User, CompanyProfile |
| `Features/<Domain>/<Domain>DataFixture.cs`                | Any new domain    |

Cross-domain extension methods belong in the fixture that **owns the returned type**, not the receiver type.
