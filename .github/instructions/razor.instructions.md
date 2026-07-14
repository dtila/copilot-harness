---
applyTo: "**/*.razor"
---

Load skills based on your task:

- **`frontend-design`** — for every `.razor` file: CSS priority order, forbidden patterns (no inline styles, no hardcoded colors, no custom CSS class names, no `??`, no `?.` on generated DTO members — the OpenAPI-generated DTOs already declare members nullable, access them directly), MudBlazor generic T requirements, Bootstrap 5.3 utility quick-reference.
- **`blazor-component`** — for any component making a REST call or debugging WASM/SSR PersistentState flicker: FeaturesClient (only HTTP transport), mandatory `[PersistentState]` on every backend-data property, component template with loading/error states, file locations, build validation.
- **`blazor-generated-client-usage`** — when the needed FeaturesClient method doesn't exist yet: delegate to backend-cqrs agent first, then run build-solution to regenerate the typed client.
- **`blazor-state-persistence`** — for advanced state restore across navigation or SSR/WASM hydration: declarative `[PersistentState]` vs manual `PersistentComponentState` + `RegisterOnPersisting` + `Dispose` patterns.

## Error handling convention

Every `catch` block that shows a snackbar **must** exclude `OperationCanceledException` using an exception filter:

```csharp
catch (Exception ex) when (ex is not OperationCanceledException)
{
    Snackbar.Add($"Eroare: {ex.Message}", Severity.Error);
}
```

`OperationCanceledException` is handled by infrastructure/middleware and must never surface as a visible error to the user.


## ENCODING & LANGUAGE REQUIREMENTS
1. Razor files (*.razor) MUST be created and saved in **UTF-8 encoding**.
2. When writing user-facing text, use correct **Romanian (ro-RO)** diacritics natively (ă, â, î, ș, ț). Never use '?' or invalid fallback characters.


## Empty state convention

Never create a standalone page-level empty-state block (custom `MudPaper` + `MudIcon` + `MudText`) to handle "no data".

Use the built-in MudBlazor template slots instead:

| Control | Slot |
|---|---|
| `MudTable<T>` | `<NoRecordsContent>` |
| `MudDataGrid<T>` | `<NoRecordsContent>` |
| `MudVirtualize` | `<NoRecordsContent>` |

If the list/table is in a child component, pass `Loading="@_loading"` and a null-safe collection (`Items="@(Items ?? [])"`) so the child component's `<NoRecordsContent>` and `<LoadingContent>` slots handle all states.

Do not add a top-level page skeleton. Delegate loading and empty state entirely to the child component's `MudTable`/`MudDataGrid` template slots (`<LoadingContent>` and `<NoRecordsContent>`).
