---
applyTo: "**/*.cs"
---

## C# Naming & Control Flow Standards

- **PascalCase** for class names, interfaces, type aliases, properties, and methods.
- **camelCase** for local variables and method parameters.
- **`_` prefix** for all private class fields (e.g. `_context`, `_validator`).
- No `FirstOrDefault` when you expect a single match to be - use `SingleOrDefault` 
- Never use reflection unless the user explicitly asks for it.
- **Fail fast / early return**: validate inputs at the top and return/throw immediately — do not nest logic inside `if` bodies.
- **No `else` chains**: use guard clauses and early returns. Reduce nesting to one level wherever possible.
- **Smaller functions**: prefer short methods that return early over large methods with nested `if/else`.
- **No `??` null-coalescing**: use explicit `if` / ternary.

## ENCODING & LANGUAGE REQUIREMENTS
1. C# files MUST be created and saved in **UTF-8 encoding**.
2. When writing user-facing strings (error messages, UI labels, exception messages visible to users), use correct **Romanian (ro-RO)** diacritics natively (`u0103, `u00E2, `u00EE, `u0219, `u0163). Never use '?' or invalid fallback characters as substitutes for diacritics.
