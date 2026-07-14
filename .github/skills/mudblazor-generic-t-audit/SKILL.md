---
name: mudblazor-generic-t-audit
description: Audit and fix MudBlazor generic build errors (MudTable, MudSelect, MudChip, MudListItem, etc.) 
---

Use this skill when you see "cannot be inferred" build errors, or when reviewing `.razor` changes.

Workflow
1) Search for common generic MudBlazor components.
2) Ensure `T="..."` is explicitly set where required.
3) Prefer the most specific type (DTO type, not `object`).

Acceptance criteria
- Razor compiles without generic type inference errors.
- Changes are minimal and consistent.
