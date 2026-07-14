---
name: backend-query-pagination
description: Implement backend query pagination with total fields display.
---

# Skill: Backend Pagination + Item Metadata

## Quick Reference

Implement **page-based pagination** (skip/take) + **item metadata** in a query handler using the `IPageableRequest<Response>` + `IPageableResponse` contract with `PredicateBuilder` filters.

---

## Core Interfaces

```csharp
public interface IPageable
{
    int PageIndex { get; }   // 0-based
    int PageSize { get; }    // 0-100
}

public interface IPageableRequest<out TResponse> : IPageable
    where TResponse : IPageableResponse { }

public interface IPageableResponse
{
    int Total { get; }  // Total matching items BEFORE pagination
}
```

---

## Implementation Flow

### 1. Request & Response Classes

```csharp
public class GetMyItems
{
    public class Request : IRequest<Response>, IPageableRequest<Response>
    {
        public int? FilterCriteria { get; set; }
        public MyFilterType Filter { get; set; } = MyFilterType.All;
        public required int PageIndex { get; set; }
        public required int PageSize { get; set; }
    }

    public class Response : IPageableResponse
    {
        public IEnumerable<ItemDto> Items { get; }
        public int Total { get; }

        public Response(IEnumerable<ItemDto> items, int total)
        {
            Items = items ?? throw new ArgumentNullException(nameof(items));
            Total = total;
        }
    }
}
```

### 2. Item DTO (With Metadata)

```csharp
public class ItemDto
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int TotalCount { get; set; }
    public int ProcessedCount { get; set; }
    public string Status { get; set; }

    public ItemDto(int id, string name, int totalCount, int processedCount)
    {
        Id = id;
        Name = name ?? throw new ArgumentNullException(nameof(name));
        TotalCount = totalCount;
        ProcessedCount = processedCount;
        Status = $"{processedCount}/{totalCount}";
    }
}
```

### 3. Handler Implementation

**Pattern**: `var query` → `var filter` (PredicateBuilder) → `query.Where(filter)` → `CountAsync` → `Skip/Take` → `Select ItemDto` → `ToListAsync` → return

```csharp
public async Task<Response> Handle(Request request, CancellationToken cancellationToken)
{
    _validator.ValidateAndThrow(request);

    // Step 1: Base query
    var query = from item in _context.MyItems.AsNoTracking()
                select item;

    // Step 2: Build filters using PredicateBuilder
    var filter = PredicateBuilder.New<MyItem>(true);

    if (request.FilterCriteria.HasValue)
        filter = filter.And(item => item.CriteriaValue == request.FilterCriteria.Value);

    filter = request.Filter switch
    {
        MyFilterType.Pending => filter.And(item =>
            item.SubItems.Count(sub => sub.Status == null) > 0),
        MyFilterType.Completed => filter.And(item =>
            item.SubItems.All(sub => sub.Status != null)),
        _ => filter
    };

    query = query.Where(filter);

    // Step 3: Count BEFORE pagination
    var totalItems = await query.CountAsync(cancellationToken);

    // Step 4: Apply pagination
    var pagedQuery = query
        .Skip(request.PageIndex * request.PageSize)
        .Take(request.PageSize);

    // Step 5: Project to DTO directly (no anonymous type)
    var itemsQuery = from item in pagedQuery
                     orderby item.Id
                     select new ItemDto(
                         item.Id,
                         item.Name,
                         item.SubItems.Count(),
                         item.SubItems.Count(s => s.Status != null)
                     );

    var items = await itemsQuery.ToListAsync(cancellationToken);

    return new Response(items, totalItems);
}
```

### 4. Validator

```csharp
public class Validator : AbstractValidator<Request>
{
    public Validator()
    {
        RuleFor(r => r.PageSize)
            .GreaterThanOrEqualTo(0)
            .LessThanOrEqualTo(100)
            .WithMessage("PageSize must be 0-100");

        RuleFor(r => r.PageIndex)
            .GreaterThanOrEqualTo(0)
            .WithMessage("PageIndex must be >= 0");
    }
}
```

---

## Frontend Integration

### Generated Client Types (NSwag)

```csharp
public class Items_Queries_GetMyItems_Request
{
    public int? FilterCriteria { get; set; }
    public int Filter { get; set; }
    public int PageIndex { get; set; }
    public int PageSize { get; set; }
}

public class Items_Queries_GetMyItems_Response
{
    public List<Items_Queries_GetMyItems_ItemDto> Items { get; set; }
    public int Total { get; set; }
}

public class Items_Queries_GetMyItems_ItemDto
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int TotalCount { get; set; }
    public int ProcessedCount { get; set; }
    public string Status { get; set; }
}
```

### Razor Component

```csharp
private int _pageIndex = 0;
private int _pageSize = 10;
private int _totalItems = 0;
private List<Items_Queries_GetMyItems_ItemDto> _items = [];
private bool _loading = false;

// Computed helpers
private int TotalPages => (_totalItems + _pageSize - 1) / _pageSize;
private bool CanPreviousPage => _pageIndex > 0;
private bool CanNextPage => _pageIndex < TotalPages - 1;

private async Task LoadPage(int pageIndex = 0)
{
    _pageIndex = pageIndex;
    _loading = true;
    try
    {
        var response = await Http.ItemsGetMyItemsAsync(
            new Items_Queries_GetMyItems_Request
            {
                FilterCriteria = FilterCriteria,
                Filter = (int)_filter,
                PageIndex = _pageIndex,
                PageSize = _pageSize
            }
        );
        _items = response.Items?.ToList() ?? [];
        _totalItems = response.Total;
    }
    finally
    {
        _loading = false;
    }
}

private async Task OnFilterChanged()
{
    await LoadPage(0);  // Reset to page 1 on filter change
}
```

### Razor Markup

```razor
<!-- Pagination info -->
<MudText Typo="Typo.body2" Color="Color.Secondary" class="mb-3">
    Page @(_pageIndex + 1) of @TotalPages | @_totalItems total items
</MudText>

<!-- Items -->
@if (_items.Any())
{
    @foreach (var item in _items)
    {
        <MudCard class="mb-3">
            <MudCardContent>
                <MudText Typo="Typo.h6">@item.Name</MudText>
                <MudText Typo="Typo.body2">@item.Status</MudText>
            </MudCardContent>
        </MudCard>
    }
}
else
{
    <MudAlert Severity="Severity.Info">No items</MudAlert>
}

<!-- Pagination controls -->
<div class="d-flex justify-content-between">
    <MudButton OnClick="async () => await LoadPage(_pageIndex - 1)" 
               Disabled="!CanPreviousPage || _loading">
        Previous
    </MudButton>
    <MudButton OnClick="async () => await LoadPage(_pageIndex + 1)" 
               Disabled="!CanNextPage || _loading">
        Next
    </MudButton>
</div>

@if (_loading) { <MudProgressLinear Indeterminate /> }
```

---

## API Example

```powershell
POST https://localhost:7123/api/items/get-my-items
{
    "filterCriteria": 123,
    "filter": 0,
    "pageIndex": 0,
    "pageSize": 10
}

Response:
{
    "total": 47,
    "items": [
        {
            "id": 1,
            "name": "Item 1",
            "totalCount": 6,
            "processedCount": 3,
            "status": "3/6 processed"
        },
        ...
    ]
}
```

---

## Implementation Checklist

### Backend
- [ ] Create `Request` with `IPageableRequest<Response>`
- [ ] Create `Response` with `IPageableResponse` (constructor-based)
- [ ] Create `ItemDto` with metadata fields + constructor
- [ ] Handler: Build query → filters (PredicateBuilder) → CountAsync → Skip/Take → Project to DTO
- [ ] Validator: PageSize 0-100, PageIndex ≥ 0
- [ ] Build: `dotnet build`

### Frontend
- [ ] Build triggers NSwag client generation
- [ ] Add `_pageIndex`, `_pageSize`, `_totalItems` to razor component
- [ ] Implement `LoadPage(int pageIndex)` method
- [ ] Add pagination UI (Previous/Next buttons)
- [ ] Display metadata: `item.Status`

---

## Critical Details

| Aspect | Rule |
|--------|------|
| **Count Timing** | `CountAsync()` BEFORE `.Skip().Take()` |
| **Filter Pattern** | Use `PredicateBuilder` + `.And()` chains, apply with `.Where(filter)` |
| **DTO Projection** | Project directly to `ItemDto` in SELECT (no anonymous type) |
| **Response** | Constructor-based only (never `required` properties) |
| **Frontend State** | Update `_totalItems = response.Total` in LoadPage |

---

## Related Patterns

- **SearchInitiated.cs**: Complex multi-filter + sorting + full-text search
- **SearchAssigned.cs**: Pagination + sorting pattern
- **GetProcedureEvents.cs**: Pagination + sorting pattern

All use `IPageableRequest<Response>` + `IPageableResponse`.

---

## Troubleshooting

| Problem | Cause | Fix |
|---------|-------|-----|
| Wrong total count | CountAsync after filters not applied | Move CountAsync immediately after Where(filter) |
| Pagination skips items | Skip/Take reversed | Order: CountAsync → Skip → Take |
| Metadata shows 0 | SubItems.Count() in wrong scope | Evaluate counts in SELECT before pagination |
| Previous/Next always disabled | _totalItems not assigned | Assign `_totalItems = response.Total` in LoadPage |
| NSwag types not updated | Client generation didn't run | Run `dotnet build` explicitly |
