---
name: frontend-design
description: For any visual or styling decision in .razor, .css, or .scss files: CSS priority order (5 levels), forbidden patterns (no inline styles, no hardcoded colors, no custom CSS class names, no ?? operator), MudBlazor 9 generic T requirements (MudTable, MudSelect, MudChip, MudListItem), Bootstrap 5.3 utility quick-reference, and AOS/_utilities.scss usage. Does NOT cover data-fetching, PersistentState, or API calls — load blazor-component for that.
---

Use this skill for any UI or styling decision. Pair with `blazor-component` for new pages or components that fetch data.

## CSS Priority — apply in order, stop at first level that works

1. **MudBlazor component props** (`Color`, `Variant`, `Class`, `Style`) — always first.
2. **Bootstrap 5.3 utility classes** — spacing, layout, color, typography, borders, shadows.
3. **`_utilities.scss` classes** — shared utilities (`animate-pulse`, `gradient-*`, `icon-badge-*`, etc.)
4. **`style="..."` inline** — only for a single atomic value with no Bootstrap equivalent (e.g. `style="z-index:1400"`). Max 1–2 properties.
5. **Scoped `.razor.css`** — absolute last resort, only for keyframe animations, pseudo-elements, or multi-property transitions that cannot be expressed any other way. Delete the file if it would be empty.

---

## Non-negotiable UI Constraints

- **NEVER** add `style="..."` inline attributes on HTML elements — use Bootstrap utility classes instead.
- **NEVER** create new CSS class names (e.g. `class="preview-section"`, `class="my-custom-chunk"`) — use existing Bootstrap/MudBlazor class names only.
- **NEVER** hard-code colors (`#f5f5f5`, `#ccc`, `#e0e0e0`, etc.) — use Bootstrap semantic classes (`bg-light`, `border`, `text-muted`) or MudBlazor `Color.*` props.
- **NEVER** use `<hr style="...">` for dividers — use `<MudDivider />` instead.
- **NEVER** use `background-color`, `border-*`, `flex:*`, `padding:*`, `margin:*` as inline styles — every one has a Bootstrap utility equivalent.
- **NEVER** use the null-coalescing operator `??` in Razor markup — use explicit `if` checks or ternary `? :` expressions.
- Do not introduce new color palettes, fonts, or bespoke visual themes.
- Keep Bootstrap variables as the source of truth for SCSS; avoid new hardcoded hex colors.

---

## Libraries

- **MudBlazor 9** — use Context7 MCP to read documentation. If Context7 tools are unavailable, do not proceed; ask the user.
- **Bootstrap 5.3** — for layout and utility classes (`d-flex`, `gap-*`, `border-*`, `bg-*-subtle`, etc.)
- **AOS** — Animate On Scroll library for scroll animations.
- **`_utilities.scss`** — shared project CSS utilities (`animate-pulse`, `gradient-*`, `icon-badge-*`, etc.)

---

## MudBlazor Generic Components — always add `T="..."`

Many MudBlazor components require an explicit type parameter to prevent "cannot be inferred" build errors:

| Component | Correct usage |
|---|---|
| `MudChip` | `<MudChip T="string" Color="Color.Primary">Text</MudChip>` |
| `MudList` | `<MudList T="string">...</MudList>` |
| `MudListItem` | `<MudListItem T="string" Icon="@Icons.Material.Filled.Star">Item</MudListItem>` |
| `MudTable` | `<MudTable T="YourDto" Items="@items">...</MudTable>` |
| `MudSelect` | `<MudSelect T="string" @bind-Value="selectedValue">...</MudSelect>` |
| `MudAutocomplete` | `<MudAutocomplete T="string" SearchFunc="@SearchAsync">...</MudAutocomplete>` |

Use the `mudblazor-generic-t-audit` skill after any `.razor` change to catch missing `T=` before build.

---

## Bootstrap Utility Quick-Reference

- **Spacing**: `m*`, `p*`, `gap-*`, `row-gap-*`, `column-gap-*`, gutters `.g-*`/`.gy-*`; center with `mx-auto`; `ms-auto` as spacer.
- **Layout**: `d-flex`/`d-grid` + `gap-*`; stacks `.hstack`/`.vstack`; width `w-25/50/75/100/auto`; `flex-fill`, `flex-grow-1`, `flex-shrink-1`.
- **Typography**: `lh-{1,sm,base,lg}`, `text-{start,center,end}`, `text-break`, link underline/offset utilities, `.mark`, `.small`, `.text-decoration-underline`.
- **Emphasis/backgrounds**: `text-bg-*`, `bg-*-subtle` + `text-*-emphasis` + `border-*-subtle`.
- **Components/forms**: `row g-*`, `mb-3`; pagination `justify-content-end`; spinners `spinner-border-sm` + `m-*`; placeholders `w-*`/`col-*`.
- **Borders/dividers**: `border`, `border-*`, `border-start-0`; vertical rule `.vr` inside stacks.
- **Sizing**: `w-*`, `h-*`, `mh-100`, viewport `vw`/`vh`; prefer utilities over inline styles.

---

## Acceptance Criteria

- UI is clearer without new design primitives.
- No new hard-coded colors or fonts.
- Zero `style="..."` attributes on elements you added or modified.
- Zero new CSS class names that are not from Bootstrap or MudBlazor.
- All MudBlazor generic components have explicit `T="..."`.
