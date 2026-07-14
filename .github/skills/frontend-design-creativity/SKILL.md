---
name: frontend-design-creativity
description: Actionable playbook for redesign, premium dashboard, analysis card, visual quality uplift, detailed results, card-based results, reference-driven UI, and landing-page-like Blazor requests. Encodes a concrete premium layout pattern for MudBlazor + Bootstrap and lists explicit anti-patterns to avoid.
---

This skill is a production playbook, not abstract inspiration. Load it when the user wants redesign, polish, visual quality uplift, premium dashboard treatment, detailed results, or a screen that should feel closer to a strong reference image than to default MudBlazor scaffolding.

Always pair this skill with `frontend-design`. Load `blazor-component` as soon as the screen fetches backend data.

## 1. Inspect Before Designing

Before writing markup:

1. Read the target component in full.
2. Read at least two sibling components in the same feature area.
3. Decide what must be preserved, what is structurally weak, and where hierarchy is missing.

Do not inherit a bad layout just because it already exists.

## 2. Canonical Premium Pattern

For detail views, analysis screens, requirement cards, result summaries, and similar data-rich interfaces, start from this structure and adapt it to the actual data.

```
┌─────────────────────────────────────────────────────────────────────┐
│ Hero / Detail Region                          Status / Action Rail │
│ Title, identifier chips, short descriptor     Score, status, CTA   │
│                                                                     │
├───────────────────────────────────────────────┬─────────────────────┤
│ Attribute Grid                                │ Insight summary     │
│ Dense 2-4 column grid of readable values      │ or secondary stats  │
├───────────────────────────────────────────────┴─────────────────────┤
│ Suggestions / Related Results / Search Strip                        │
│ Search, filter chips, ranked result cards                           │
└─────────────────────────────────────────────────────────────────────┘
```

The goal is a screen with clear primary, secondary, and tertiary regions, not a stack of unrelated blocks.

## 3. Region-by-Region Guidance

### Hero / Detail Region

- Use one strong title with `MudText Typo="Typo.h5"` or `Typo.h4` and `fw-bold lh-sm`.
- Put identifiers and small metadata in `Typo.caption` with `text-uppercase` and muted emphasis.
- Use one or two compact `MudChip T="string"` elements for status, category, or contextual labeling.
- Keep the supporting description short and directly beneath the title.
- Use whitespace and typography for hierarchy first. Do not solve hierarchy with loud borders.

### Status / Action Rail

- Use a dedicated right-side `MudPaper` or visually distinct column for action and status.
- This region should contain the current status, a score or confidence indicator, and the main CTA.
- Use semantic emphasis: `Color.Success`, `Color.Warning`, `Color.Info`, `Color.Error` where the data actually supports it.
- The main action should be obvious and singular. Secondary actions should be quieter.
- Keep this rail compact. It should feel like an operational summary, not a second page.

### Attribute Grid

- Present dense data in a `row g-2` or `row g-3` grid.
- Each cell should have a small uppercase label and a stronger value line.
- Use `MudPaper` with low or zero elevation and Bootstrap background utilities for subtle separation.
- Aim for 2-4 columns depending on viewport and data density.
- Keep cells readable and uniform. Do not create decorative boxes for every value.

### Suggestions / Results Region

- Treat this as a secondary region beneath the hero block.
- Add a clear section title and supporting count or summary.
- Search and filter controls should sit at the top of this region, not scattered around the page.
- Use compact chips or toggles for field filters and a clean ranked list or card list for results.
- Result cards should emphasize the primary label, source or context, score, and 4-6 top attributes. Do not dump every field.

## 4. Practical MudBlazor + Bootstrap Composition

Use the existing stack only: MudBlazor plus Bootstrap utilities.

Preferred composition patterns:

- Outer wrapper: `MudPaper` or `MudContainer` with spacing utilities, not nested anonymous `<div>` blocks everywhere.
- Main layout: `row g-3` or `row g-4` with `col-lg-8` and `col-lg-4` split for the core detail plus action rail.
- Section headings: `MudText` with controlled typography instead of custom ornamental wrappers.
- Status chips: `MudChip T="string" Size="Size.Small" Variant="Variant.Filled"` or `Variant.Outlined` based on emphasis.
- Search/filter strip: `MudTextField`, `MudChipSet`, `MudCheckBox`, `MudButton`, or Bootstrap utility groups arranged as one coherent bar.

Prefer restrained backgrounds such as Bootstrap subtle backgrounds and MudBlazor elevation changes. The screen should feel premium because the hierarchy is good, not because the palette is loud.

## 5. Scoring, Status, and Results Hierarchy

- If the screen contains analysis or matching, expose the best score in the status/action rail.
- Status labels should be short and operational: `Pending Match`, `Analysis Complete`, `Needs Review`, `Confirmed`.
- Show one primary metric prominently and keep secondary metrics smaller.
- Use score chips or emphasized `MudText` blocks for ranking. Do not repeat the same score in three places.
- If a result list exists, sort it visually by confidence and make the best result immediately scannable.

## 6. Reusability Rules

- If hero header, status rail, or result cards are reused or obviously reusable, extract them into parameterized components.
- Child components should expose clear inputs and callbacks. Do not bury page orchestration inside presentational fragments.
- Reusable components should own their internal layout, but not invent a separate visual language.

## 7. Anti-Patterns

Avoid these patterns unless the user explicitly asks for them:

- Bland stacked alerts used as the final layout.
- Flat white cards with identical spacing and no clear hierarchy.
- Random gradients or decorative effects that are unrelated to the screen's purpose.
- Purple-on-white defaults or generic SaaS palette clichés.
- Excessive left border accents such as `border-start border-4` status markers.
- Large empty hero areas with no operational value.
- Skeleton-heavy layouts that never resolve into a strong loaded state.
- Attribute sections rendered as long paragraphs instead of compact analytical cells.
- Throwing every possible badge, chip, and icon into the header.

## 8. Repo Constraints Still Apply

- Do not introduce a bespoke design system.
- Do not hardcode colors.
- Do not create arbitrary CSS class names when Bootstrap or MudBlazor can solve the layout.
- Do not use inline styles unless `frontend-design` explicitly allows a last-resort exception.
- Do not ignore SSR/WASM persistence rules when the screen fetches backend data.

## 9. Quality Bar

When you finish, the loaded screen should read like this:

- The user knows immediately what the item is.
- The user can identify current status and next action without searching.
- The important attributes are dense but easy to scan.
- Related suggestions or results feel intentionally connected to the main card.
- The page looks designed, not merely assembled.
