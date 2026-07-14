---
name: blazor-component
description: For any .razor component or page making a REST call to the backend, OR debugging WASM/SSR double-fetch or PersistentState flicker: FeaturesClient (only permitted HTTP transport, never HttpClient directly), mandatory [PersistentState] on every backend-data property (SSR+WASM dual-run of OnInitializedAsync), component template with loading/error states, file locations, and build validation.
---

Load this skill whenever a `.razor` file fetches data from the backend (new page, new component, adding a new API call).

---

## Why PersistentState is mandatory in this app

This app runs in **SSR + WASM hybrid mode**. `OnInitializedAsync` executes **twice**:

1. On the **server** (SSR prerender) — data is fetched and rendered as HTML.
2. On the **client** (WASM hydration) — the component re-initializes.

Without `[PersistentState]`, the client re-fetches the data, causing visible flicker and unnecessary round trips.

**Rule: every `OnInitializedAsync` that calls the backend MUST persist its result.**

---

## FeaturesClient — the only permitted API transport

- Inject `FeaturesClient Http` — **never** `HttpClient`, `GetFromJsonAsync`, `dynamic`, or reflection.
- `FeaturesClient` and all DTOs are **auto-generated** at build time by NSwag from `openapi.procurement.json`.
- `openapi.procurement.json` is **auto-generated** by the Blazor.Server build (Swashbuckle) — **NEVER edit it manually**.
- Use only the generated typed methods and generated DTO types — **never** create matching C# classes by hand.
- **Never use `?.` (null-conditional) on generated DTO members.** The NSwag-generated DTOs already declare every member as nullable where appropriate. Access members directly (e.g. `@result.Heading`, `item.Info.Pages`), not `@result?.Heading` or `item.Info?.Pages`. Only null-check the **top-level response object** itself (which may be null if the request failed).

### When a typed method does not exist yet

The backend endpoint has not been built yet:

1. Delegate to the **`backend-cqrs`** agent — add the handler + controller endpoint.
2. Run `build-solution` — Swashbuckle regenerates the spec; NSwag generates the typed method + DTOs.
3. Then use the generated method and DTO in `.razor`.

---

## PersistentState — mandatory for every backend call

### Preferred: declarative `[PersistentState]` (public properties — ASP.NET Core 10+)

```razor
@page "/feature/route"
@inject FeaturesClient Http

@code {
    [PersistentState]
    public FeatureDto? Item { get; set; }            // single item

    [PersistentState]
    public List<FeatureItemDto>? Items { get; set; } // collection

    protected override async Task OnInitializedAsync()
    {
        // ??= means: only fetch if not already restored from SSR prerender
        Item ??= await Http.FeatureGetAsync(Id);
        Items ??= (await Http.FeatureListAsync())?.Items?.ToList();
    }
}
```

- Use `[PersistentState(AllowUpdates = true)]` when state must refresh on subsequent navigations.
- **Public properties only** — for private fields use the manual pattern below.

### Manual pattern (private fields)

```razor
@implements IDisposable
@inject PersistentComponentState AppState
@inject FeaturesClient Http

@code {
    private PersistingComponentStateSubscription _subscription;
    private List<FeatureItemDto>? _items;

    protected override async Task OnInitializedAsync()
    {
        _subscription = AppState.RegisterOnPersisting(PersistAsync);

        if (!AppState.TryTakeFromJson<List<FeatureItemDto>>(nameof(_items), out var restored) || restored == null)
            _items = (await Http.FeatureListAsync())?.Items?.ToList();
        else
            _items = restored;
    }

    private Task PersistAsync()
    {
        AppState.PersistAsJson(nameof(_items), _items);
        return Task.CompletedTask;
    }

    public void Dispose() => _subscription.Dispose();
}
```

---

## Full Component Template

```razor
@page "/feature/page-route"
@inject FeaturesClient Http
@inject NavigationManager Navigation
@inject ISnackbar Snackbar

<MudContainer>
    @if (_isLoading)
    {
        <MudProgressLinear Indeterminate="true" />
    }
    else if (Items == null)
    {
        <MudAlert Severity="Severity.Warning">Nu s-au gasit date.</MudAlert>
    }
    else
    {
        <!-- render Items here -->
    }
</MudContainer>

@code {
    private bool _isLoading;

    [PersistentState]
    public List<GeneratedItemDto>? Items { get; set; }

    [Parameter] public string? Id { get; set; }

    protected override async Task OnInitializedAsync()
    {
        if (Items != null) return;  // restored from SSR prerender — skip fetch
        await LoadAsync();
    }

    private async Task LoadAsync()
    {
        _isLoading = true;
        StateHasChanged();
        try
        {
            var result = await Http.FeatureListAsync();
            Items = result?.Items?.ToList();
        }
        catch (Exception ex)
        {
            Snackbar.Add($"Eroare: {ex.Message}", Severity.Error);
        }
        finally
        {
            _isLoading = false;
            StateHasChanged();
        }
    }

    private async Task HandleActionAsync()
    {
        try
        {
            await Http.FeatureActionAsync(new GeneratedRequestDto { /* props */ });
            Snackbar.Add("Actiune executata cu succes", Severity.Success);
            await LoadAsync();
        }
        catch (Exception ex)
        {
            Snackbar.Add($"Eroare: {ex.Message}", Severity.Error);
        }
    }
}
```

---

## Component Design Rules

- **Keep dialogs minimal**: Dialog components contain ONLY feature-specific logic (API calls unique to that dialog). Extract shared UI into reusable components.
- **Single responsibility**: each component does one thing. Split when handling multiple concerns.
- **EventCallback naming**: name after what the callback carries, not where it comes from. `OnAnchorSelected` (carries a DOM anchor `string`) is correct; `OnPageSelected` (carries a `string` that is not a page number) is misleading.
- **No co-located CSS by default**: only create a `.razor.css` file for keyframe animations or pseudo-elements that cannot be expressed via Bootstrap/MudBlazor. Delete it if empty.

---

## File Locations

```
src/Blazor/Client/
├── Features/
│   ├── <Domain>/
│   │   ├── <FeatureName>.razor             ← page
│   │   ├── Components/
│   │   │   └── <ComponentName>.razor       ← component
│   │   ├── Models/
│   │   │   └── <ModelName>.cs
│   │   └── Services/
│   │       └── <ServiceName>.cs
│   └── Shared/                             ← shared layouts, navigation
├── Services/                               ← global services
└── Components/                             ← reusable cross-feature components
```

---

## Build Validation

1. If `dotnet watch` (Blazor Server) is already running — **do not start another**. Read the watch output to check compile errors.
2. **CSS/SCSS-only changes**: do NOT trigger a build — rely on hot reload.
3. When no watch is active and Razor/C# was changed:
   ```bash
   dotnet run --project src\Blazor\Server\Blazor.Server.csproj
   ```
   This compiles Blazor Client (WASM) + Server, validates all Razor syntax, and checks generated client types.

---

## Acceptance Criteria

- Every `OnInitializedAsync` with a backend call uses `[PersistentState]` + `??=` (public props) or the manual persist pattern (private fields).
- `FeaturesClient` used exclusively — no raw `HttpClient` or manually crafted DTOs.
- `openapi.procurement.json` untouched.
- Build passes cleanly after changes.
