---
name: blazor-generated-client-usage
description: For when a needed FeaturesClient typed method is missing (endpoint does not exist yet) or to understand the generated OpenAPI client/DTO contract: workflow to add the backend endpoint via the backend-cqrs agent first, then run build-solution so Swashbuckle + NSwag regenerate the spec and typed client. Never edit openapi.procurement.json manually.
---

Use this skill when a `.razor` component needs to call backend APIs.

## How `FeaturesClient` is generated (read-only — never touch these files)

The generation chain is fully automatic:

1. **Backend handler + controller endpoint** → `dotnet build Blazor.Server`
2. **Swashbuckle** auto-generates `openapi.procurement.json` (do **not** edit this file manually — it is overwritten every build)
3. **NSwag** reads `openapi.procurement.json` and auto-generates `Services/FeaturesClient.cs` (namespace `ServiceReference`, class `FeaturesClient`)
4. Result: one typed `async` method per endpoint (e.g. `TeamsGetTeamSubscriptionAsync()`) and one DTO per response schema (e.g. `Team_Queries_GetTeamSubscription_Response`)

## Workflow when a typed method does not exist yet

The backend endpoint has not been implemented or built yet. Do **not** edit `openapi.procurement.json`.

1. Delegate to the `backend-cqrs` agent to add the handler + controller endpoint.
2. Run `build-solution` — Swashbuckle updates the spec; NSwag generates the typed method and DTO.
3. Use the generated method and DTO in `.razor`.

## Checklist
- Inject and call `FeaturesClient` — **never** `HttpClient`, `GetFromJsonAsync`, `dynamic`, or reflection.
- Use only generated request/response DTOs — never create matching C# classes by hand.
- Handle loading + errors (Snackbar or existing pattern).

## Acceptance criteria
- No handcrafted client DTOs.
- No raw `HttpClient` or `GetFromJsonAsync` calls.
- `openapi.procurement.json` is untouched.
- Build passes cleanly after the change.
