---
name: backend-cqrs-command
description: To implement a command - action that will change state in the system. Defines vertical slice structure, command handler conventions, DTO usage, validation, EF Core querying rules, in-memory domain event publishing, and documentation co-location.
---

Use this skill when implementing any **CQRS command handler (write operation)** in:

```
src/[projectname].Application/Features/<Domain>/Commands/
```

Example:

```
src/[projectname].Application/Features/<Domain>/Commands/<VerbNoun>.cs
```

---

# Handler Structure

Each command must be implemented in **a single `.cs` file** containing:

* `Request`
* `Response` (optional)
* `Handler`
* `Validator`

Example structure:

```
<VerbNoun>.cs
```

```csharp
namespace [projectname].Application.Features.<Domain>.Commands;

public class <VerbNoun>
{
    public class Request : IRequest<Response>
    {
        public required int <EntityId> { get; set; }
        public required string <InputValue> { get; set; }
    }

    public class Response
    {
        public int Id { get; }

        public Response(int id)
        {
            Id = id;
        }
    }

    [ExposureFeature(ExposureAccess.API)]
    public class Handler : IRequestHandler<Request, Response>
    {
        private readonly Validator _validator;
        private readonly ContentDbContext _context;
        private readonly IUserProvider _userProvider;
        private readonly IBusProvider _busProvider;

        public Handler(Validator validator, ContentDbContext context, IUserProvider userProvider, IBusProvider busProvider)
        {
            _validator = validator ?? throw new ArgumentNullException(nameof(validator));
            _context = context ?? throw new ArgumentNullException(nameof(context));
            _userProvider = userProvider ?? throw new ArgumentNullException(nameof(userProvider));
            _busProvider = busProvider ?? throw new ArgumentNullException(nameof(busProvider));
        }

        public async Task<Response> Handle(Request request, CancellationToken cancellationToken)
        {
            _validator.ValidateAndThrow(request);

            var query =
                from entity in _context.<Entities>
                where entity.Id == request.<EntityId>
                      && entity.<OwnerId> == _userProvider.ActingExternalUserId
                select entity;

            var entity = await query.SingleOrDefaultAsync(cancellationToken);

            if (entity == null)
                throw new NotFoundException(nameof(<Entity>));

            entity.<DomainMethod>(request.<InputValue>, _busProvider.Bus);

            await _context.SaveChangesAsync(cancellationToken);

            return new Response(entity.Id);
        }
    }

    public class Validator : AbstractValidator<Request>
    {
        public Validator()
        {
            RuleFor(x => x.<EntityId>).GreaterThan(0);
            RuleFor(x => x.<InputValue>).NotEmpty();
        }
    }
}
```

---

# Command Rules

## Commands Modify State

Commands perform write operations and may modify aggregates.

Commands must execute domain logic through aggregate methods.

Handlers should orchestrate operations but must not contain business logic.

Handlers should stay thin:

* validate input
* load the required data with one named `query` when practical
* keep filtering and ordering in SQL whenever the database can do it
* delegate non-trivial grouping, matching, validation, or transformation to a domain or application service
* save changes and return

Do not move ordering, bucketing, or reduction into memory after `ToListAsync()` when the same work can be expressed in SQL with `Where`, `OrderBy`, `ThenBy`, joins, or projections.

Prefer this shape:

```csharp
var query =
    from entity in _context.<Entities>.AsNoTracking()
    where ...
    orderby entity.Id
    select new LoadedItem(...);

var items = await query.ToListAsync(cancellationToken);
var result = await _service.ExecuteAsync(items, cancellationToken);
return new Response(result);
```

---

## Domain Events (In-Memory)

Domain events in this system are **in-memory only**.

They are **not serialized** and **not persisted to the database**.

Their purpose is to decouple domain behavior within the application.

Flow:

```
Aggregate raises domain event
↓
Domain event handler executes
↓
Integration event may be published
```

Command handlers must **not publish integration events directly**.

Instead, handlers pass the bus to aggregates that raise domain events.

Example:

```csharp
entity.<DomainMethod>(..., _busProvider.Bus);
```

---

## SaveChanges Rule

Command handlers must always call:

```
await _context.SaveChangesAsync(cancellationToken);
```

This ensures entity updates and domain event side effects are flushed.

---

## Query Pattern for Aggregate Loading

Commands must load aggregates using a query variable named `query`.

Always define the query as a **named variable** using `from ... select` query syntax. Never inline the query inside `await (...)`. Never use method-chain syntax as the top-level query.

Example:

```csharp
var query =
    from entity in _context.<Entities>
    where entity.Id == request.<EntityId>
    select entity;

var entity = await query.SingleOrDefaultAsync(cancellationToken);
```

When additional related data is needed in the same query, use **left joins** via `into ... from ... DefaultIfEmpty()` instead of separate queries or correlated subqueries:

```csharp
var query =
    from entity in _context.<Entities>
    where entity.Id == request.<EntityId>
    join related in _context.<RelatedEntities> on entity.<FK> equals related.Id into relatedJoin
    from related in relatedJoin.DefaultIfEmpty()
    select new { entity, Related = related };

var result = await query.SingleOrDefaultAsync(cancellationToken);
```

Never use `let` to embed correlated subqueries (e.g. `.FirstOrDefault()` inside a `let` clause) — this produces N+1 queries.

---

## Team-Scoped Command Rule

When a command needs the current team, never trust `IUserProvider.ActingUser.TeamId`.

Always resolve the team by querying the current user record with `IUserProvider.ActingExternalUserId`:

```csharp
var query =
    from user in _context.Users.AsNoTracking()
    where user.Id == _userProvider.ActingExternalUserId
    select user.TeamId;

var teamId = await query.SingleOrDefaultAsync(cancellationToken);
if (teamId == null)
    throw new MissingCompanyProfileException();
```

Assume one user belongs to exactly one team.

---

## EF Core Rules

Use:

```
SingleOrDefault
Single
```

Never use:

```
First
FirstOrDefault
```

Reason: duplicates must fail explicitly instead of being silently ignored.

---

## DTO and Request Rules

Requests may contain primitives or DTO objects.

All request properties must use the `required` keyword to enforce compile-time safety.

Example:

```csharp
public required int <EntityId> { get; set; }
```

DTO → domain conversion must occur inside the handler using domain extension methods.

---

## Validation Rules

Validation should verify **input correctness only**.

Examples:

```
Id greater than zero
Required values present
Enum values valid
```

Business rules must be implemented in **domain aggregates**, not validators.

---

# Documentation Co-Location

Every `.cs` file must have a sibling `.cs.md` file.

Example:

```
<VerbNoun>.cs
<VerbNoun>.cs.md
```

The `.cs.md` file should briefly describe:

* purpose of the command
* workflow
* integration points
