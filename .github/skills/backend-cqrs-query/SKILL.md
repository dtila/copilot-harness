---
name: backend-cqrs-query
description: To implement a query - action that will read state in the system. Defines vertical slice structure, query handler conventions, DTO usage, validation, EF Core querying rules, and documentation co-location.
---

Use this skill when implementing any **CQRS query handler (read operation)** in:

```
src/[projectname].Application/Features/<Domain>/Queries/
```

Example:

```
src/[projectname].Application/Features/<Domain>/Queries/<VerbNoun>.cs
```

---

# Handler Structure

Each query must be implemented in **a single `.cs` file** containing:

* `Request`
* `Response`
* `Handler`
* `Validator` (optional)

Example structure:

```
<VerbNoun>.cs
```

```csharp
namespace [projectname].Application.Features.<Domain>.Queries;

public class <VerbNoun>
{
    public class Request : IRequest<Response>
    {
        public int <EntityId> { get; }

        public Request(int entityId)
        {
            <EntityId> = entityId;
        }
    }

    public class Response
    {
        public int Id { get; }
        public string Name { get; }
        public string? OptionalField { get; }

        public Response(int id, string name, string? optionalField)
        {
            Id = id;
            Name = name ?? throw new ArgumentNullException(nameof(name));
            OptionalField = optionalField;
        }
    }

    [ExposureFeature(ExposureAccess.API)]
    public class Handler : IRequestHandler<Request, Response>
    {
        private readonly ContentDbContext _context;

        public Handler(ContentDbContext context)
        {
            _context = context ?? throw new ArgumentNullException(nameof(context));
        }

        public async Task<Response> Handle(Request request, CancellationToken cancellationToken)
        {
            var query =
                from entity in _context.<Entities>.AsNoTracking()
                join related in _context.<RelatedEntities>.AsNoTracking()
                    on entity.<ForeignKey> equals related.Id into relatedGroup
                from related in relatedGroup.DefaultIfEmpty()
                where entity.Id == request.<EntityId>
                select new Response(
                    entity.Id,
                    entity.Name,
                    related == null ? null : related.<Field>
                );


            var result = await query.SingleOrDefaultAsync(cancellationToken);

            if (result == null)
                throw new NotFoundException(nameof(<Entity>));

            return result;
        }
    }

    public class Validator : AbstractValidator<Request>
    {
        public Validator()
        {
            RuleFor(x => x.<EntityId>).GreaterThan(0);
        }
    }
}
```


# Query Rules

## Response and DTO Classes

`Response` and nested DTO classes (e.g. `DocumentItem`, `DocumentInfo`) **must use explicit constructors** with **readonly getters** (no setters). Never use `required` properties or object initializer syntax on Response or DTO classes.

> **Rule: `required` is only permitted on `Request` classes.**
> Response and DTO classes always use explicit constructors. `required` keyword must never appear on `Response` or any nested DTO.

Correct — `Request` uses `required`, `Response` uses explicit constructor:

```csharp
public class Request : IRequest<Response>
{
    public required int ProcedureId { get; set; }  // ✅ required is fine on Request
}

public class Response
{
    public IEnumerable<DocumentItem> Documents { get; }  // ✅ readonly — no setter

    public Response(IEnumerable<DocumentItem> documents)
    {
        Documents = documents ?? throw new ArgumentNullException(nameof(documents));
    }
}

public class DocumentItem
{
    public int Id { get; }
    public string Name { get; }

    public DocumentItem(int id, string name)  // ✅ explicit constructor
    {
        Id = id;
        Name = name ?? throw new ArgumentNullException(nameof(name));
    }
}
```

Incorrect:

```csharp
// WRONG - required properties on Response
public class Response
{
    public required IEnumerable<DocumentItem> Documents { get; set; }
}

// WRONG - object initializer instead of constructor call
return new Response { Documents = documents };  // fails if no setter
```

When constructing a `Response` in the handler, always call the constructor directly:

```csharp
// ✅ correct
return new Response(offers);

// WRONG - object initializer is invalid when there is no setter
return new Response { Offers = offers };
```

---

## Queries Are Read-Only

Queries must never mutate entities.
Queries must never raise domain events.
Queries must never call `SaveChanges`.

Query handlers should stay thin:

* build one base `query`
* compose one `filter`
* keep filtering and ordering in SQL whenever possible
* materialize only the rows needed for the response
* perform in-memory grouping only when it is strictly response-shaping work that cannot be expressed cleanly in SQL

Do not sort, filter, or reduce in memory after `ToListAsync()` when the same work can be done by the database with `Where`, `OrderBy`, `ThenBy`, joins, correlated subselects, or projection.

---

## Team-Scoped Query Rule

When a query needs the current team, never trust `IUserProvider.ActingUser.TeamId`.

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

## AsNoTracking

All queries must use:

```
.AsNoTracking()
```

Example:

```csharp
from entity in _context.<Entities>.AsNoTracking()
```


## Query Variable Pattern

Always define a variable named `query` containing the LINQ query.

Use `from ... select` query syntax. **Never** inline the query inside `await (...)`. **Never** use method-chain syntax as the top-level query expression.

Correct:

```csharp
var query =
    from entity in _context.<Entities>.AsNoTracking()
    where entity.Id == request.<EntityId>
    select entity;

var result = await query.SingleOrDefaultAsync(cancellationToken);
```

Incorrect:

```csharp
// WRONG - never inline query in await
var result = await (
    from entity in _context.<Entities>.AsNoTracking()
    ...
).SingleOrDefaultAsync(cancellationToken);
```

Do not repeatedly reshape an existing `query` variable with patterns like:

```csharp
query =
    from item in query
    where ...
    select item;
```

That pattern makes the query harder to reason about and leads to noisy generated SQL. Keep one base `query` variable and compose a separate `filter` expression, then apply it once with `.Where(filter)`.

Correct dynamic-filter pattern:

```csharp
var query =
    from entity in _context.<Entities>.AsNoTracking()
    select new QueryFilter
    {
        Id = entity.Id,
        Status = entity.Status,
        Entity = entity
    };

var filter = PredicateBuilder.New<QueryFilter>(true);

if (request.Status != null)
    filter = filter.And(x => x.Status == request.Status);

var result = await query
    .Where(filter)
    .Select(x => new Response(x.Id))
    .ToListAsync(cancellationToken);
```

---

## Left Joins

Optional relationships (may or may not exist) **must** use left joins via `into ... from ... DefaultIfEmpty()`. Never use correlated subqueries with `let` and `.FirstOrDefault()`.

Correct — single SQL query:

```csharp
var query =
    from entity in _context.<Entities>.AsNoTracking()
    where entity.Id == request.<EntityId>
    join related in _context.<RelatedEntities>.AsNoTracking()
        on (int?)entity.Id equals (int?)related.<FK> into relatedJoin
    from related in relatedJoin.DefaultIfEmpty()
    join other in _context.<OtherEntities>.AsNoTracking()
        on (int?)related.<FK2> equals (int?)other.Id into otherJoin
    from other in otherJoin.DefaultIfEmpty()
    select new
    {
        entity.Id,
        entity.Name,
        RelatedField = (string?)related.<Field>,
        OtherField = (DateTime?)other.<Field>
    };
```

Incorrect — N+1 pattern:

```csharp
// WRONG - correlated subquery inside let causes a query per row
var query =
    from entity in _context.<Entities>.AsNoTracking()
    let related = _context.<RelatedEntities>
        .Where(r => r.<FK> == entity.Id)
        .FirstOrDefault()
    select new { entity, related };
```


## Correlated Subselect for Child Collections

When projecting a **child collection** directly inside a `select new` constructor, use a correlated subselect (not a left join). EF Core translates this to a SQL correlated subquery.

Use correlated subselects when the child rows belong exclusively to the parent row and you want to project them as a typed `List<T>` inside the parent DTO.

Correct — correlated subselect inside `select new`:

```csharp
var query =
    from offer in _context.SupplierOffers.AsNoTracking()
    where offer.ProcedureId == request.ProcedureId && offer.TeamId == teamId
    select new OfferItem(
        offer.Id,
        offer.ProcedureId,
        offer.Price == null
            ? null
            : new PriceVM(offer.Price.LowestAmount, offer.Price.HighestAmount, offer.Price.Currency),
        offer.CreatedAt,
        _context.SupplierOfferParticipants
            .Where(li => li.OfferId == offer.Id)
            .Select(li => new CompanyIdName(li.CompanyId, li.Company.Name))
            .ToList()
    );

var offers = await query.ToListAsync(cancellationToken);
return new Response(offers);
```

Rules for correlated subselects:

* The subselect must be **inline inside the `select new` constructor argument** — never in a `let` clause.
* The outer sequence uses `from ... AsNoTracking()`.
* The inner sequence uses `_context.<ChildEntities>` (no `.AsNoTracking()` needed on inner — it inherits the outer tracking mode).
* Use `.ToList()` at the end of the inner chain so EF Core materialises it as `List<T>`.
* Do **not** use left joins when a correlated subselect is the better fit for projecting a child collection.

Use left joins (see Left Joins section) when selecting a single optional related entity or flattening a 1-to-0-or-1 relationship.
Use correlated subselects when projecting a 1-to-many child collection into a typed list.

Incorrect — N+1 pattern:

```csharp
// WRONG - correlated subquery inside let causes a query per row
var query =
    from entity in _context.<Entities>.AsNoTracking()
    let related = _context.<RelatedEntities>
        .Where(r => r.<FK> == entity.Id)
        .FirstOrDefault()
    select new { entity, related };
```


## Projection Rule

Always project directly inside the query expression.
Correct:

```csharp
select new Response(...)
```

Incorrect:

```
Load entity then map afterwards
```


## Single Result Rule

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


## Filtering (PredicateBuilder)

Queries that support dynamic filtering must use **PredicateBuilder** to compose conditions dynamically. Filtering must occur **before the final projection**.

### Filter Object

Introduce an **intermediate filter model** containing fields used for filtering.

Example:

```csharp
select new QueryFilter
{
    Id = entity.Id,
    Name = entity.Name,
    Status = entity.Status,
    Price = price.Value,
    PublishedDate = entity.PublishedDate,
    Entity = entity
};
```

The filter object:

* contains properties needed for filtering
* may include the original entity if required
* is **not returned by the handler**

---

### Building the Predicate

Start with:

```csharp
var filter = PredicateBuilder.New<QueryFilter>(true);
```

Add conditions only when request parameters exist:

```csharp
if (request.Status != null)
    filter = filter.And(x => x.Status == request.Status);

if (request.MinPrice != null)
    filter = filter.And(x => x.Price >= request.MinPrice);

if (request.MaxPrice != null)
    filter = filter.And(x => x.Price <= request.MaxPrice);
```


### Applying the Filter

Apply the predicate before projection:

```csharp
var result = await query
    .Where(filter)
    .Select(x => new Response(
        x.Id,
        x.Name,
        x.Price
    ))
    .ToListAsync(cancellationToken);
```

### Constraints

Filtering must:

* remain **SQL-translatable**
* avoid client-side evaluation
* avoid `ToList()` before filtering
* operate on **IQueryable**

---

## Authorization

Queries that return team-owned resources **must** enforce access at the query level using `IUserProvider`.

Inject `IUserProvider` in the handler and read `ActingUser.TeamId`. Then start the query from the ownership root (e.g. `SupplierOffers`) filtered by `teamId`. Join down to the actual data from there.

This means unauthorized users get `NotFoundException` (or an empty list) — never a data leak.

Correct:

```csharp
var teamId = _userProvider.ActingUser.TeamId
    ?? throw new BadRequestException("User must belong to a team");

var query =
    from offer in _context.SupplierOffers.AsNoTracking()
    where offer.Id == request.OfferId && offer.TeamId == teamId
    join offerFile in _context.OfferFiles.AsNoTracking() on offer.Id equals offerFile.OfferId
    join file in _context.Files.AsNoTracking() on offerFile.FileId equals file.Id
    ...
    select new { ... };
```

Incorrect — ownership is not checked, any authenticated user can see any team's data:

```csharp
// WRONG - no team filter
var query =
    from offerFile in _context.OfferFiles.AsNoTracking()
    where offerFile.OfferId == request.OfferId
    ...
```

---

## Specifications

When an entity requires a named, reusable filter condition (especially on joined tables), define an `IExpressionSpecification<T>` instead of writing inline predicates.

Specifications live in:

```
src/[projectname].Application/Features/<Domain>/Specifications/
```

Interface:

```csharp
public interface IExpressionSpecification<T>
{
    Expression<Func<T, bool>> Criteria { get; }
}
```

### When to Use

Use a specification when:

* the same filter condition appears on more than one query
* the condition has a meaningful domain name (e.g. `WaitingSpecification`, `ProcessableDocumentAnalysisSpecification`)
* the condition is complex and should be unit-testable in isolation

### How to Apply in a Join

Pass `.Criteria` directly to `.Where()` on the source before joining:

```csharp
var waitingSpec = new WaitingSpecification();

var query =
    from entity in _context.<Entities>.AsNoTracking()
    where entity.Id == request.<EntityId>
    join analysis in _context.DocumentAnalyses.Where(waitingSpec.Criteria)
        on entity.Id equals analysis.DocumentId into analysisJoin
    from analysis in analysisJoin.DefaultIfEmpty()
    select new { entity, IsAnalyzing = analysis != null };
```

Never write the conditions inline as a `let` + `.Any()` or multiple `where` predicates:

```csharp
// WRONG - inline correlated subquery, not reusable, not named
let isAnalyzing = _context.DocumentAnalyses
    .Any(a => a.DocumentId == entity.Id
        && a.StartedAt != null
        && a.CompletedAt == null
        && a.ErrorMessage == null)
```

