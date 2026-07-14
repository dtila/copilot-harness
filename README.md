# GitHub Copilot Harness — CQRS + Vertical Slice + Blazor (MudBlazor 9)

A structured set of agents, skills, and instructions that teach GitHub Copilot how to plan,
implement, and review changes on a .NET + Blazor stack.

> Battle-tested in production: these files drive **[Achizito](https://achizito.ro)**, a real
> commercial product I build on this stack every day. Shared openly for reuse and feedback — issues
> and PRs welcome.

---

## The stack

- **Backend** — .NET / C#, **CQRS + vertical-slice architecture**, EF Core, domain events, outbox pipeline.
- **Frontend** — **Blazor** (SSR + WASM), **MudBlazor 9** on top of Bootstrap 5.3.
- **Testing** — xUnit with isolated SQLite unit tests and PostgreSQL-container integration tests.

### Architecture — CQRS + vertical slice

This is an opinionated, subjective choice. Each feature is a self-contained slice — a **command**
(write) or a **query** (read) with its own handler, DTOs, and validation — which keeps coupling low
and lets a feature change in isolation instead of touching shared layers. Writes and reads stay
strictly separated.

MudBlazor 9 is a preference, not a requirement — swap it for any component library and adjust the
frontend skills accordingly.

---

## The three building blocks

| Block | Folder | What it does |
|-------|--------|--------------|
| **Agents** | [.github/agents/](.github/agents/) | Personas with a defined role, allowed tools, and the sub-agents/skills they may use. They orchestrate, implement, or review. |
| **Skills** | [.github/skills/](.github/skills/) | Single-concern playbooks (e.g. write a CQRS command, build a Blazor component). Agents load only the minimal relevant set. |
| **Instructions** | [.github/instructions/](.github/instructions/) | File-scoped rules auto-applied by glob (`applyTo`), e.g. every `*.cs` or `*.razor` file. |

The top-level router is [.github/copilot-instructions.md](.github/copilot-instructions.md) — repo-wide
conventions that always apply.

---

## Before you use this: replace the project placeholder

The project name is replaced throughout with the placeholder **`[projectname]`**. Replace it with
your own solution name before the harness will work. If your test project is `MyShop.Application.Tests`,
then `[projectname]` → `MyShop`:

```
[projectname].Application.Tests   →   MyShop.Application.Tests
[projectname].Domain              →   MyShop.Domain
[projectname].sln                 →   MyShop.sln
```

Find and replace it across the whole `.github/` folder:

```bash
# preview where the placeholder appears
grep -rn "\[projectname\]" .github

# replace it everywhere (use your real name instead of MyShop)
grep -rlZ "\[projectname\]" .github | xargs -0 sed -i 's/\[projectname\]/MyShop/g'
```

> It appears in namespaces, project paths, the solution file, and `dotnet test` commands. Adjust any
> folder references (`src/[projectname].Application/...`) to match your layout.

---

## Agents

Agents live in [.github/agents/](.github/agents/) — an orchestrator, implementers, and reviewers.

### Orchestration
| Agent | Purpose |
|-------|---------|
| [orchestrator](.github/agents/orchestrator.agent.md) | Plans work, classifies the task, picks the implementation agent + minimal skill set, delegates with acceptance criteria, coordinates reviews. Never writes production code. |
| [planning](.github/agents/planning.agent.md) | Explores the repository and returns findings when a request is large, ambiguous, or spans multiple domains. |

### Implementation
| Agent | Purpose |
|-------|---------|
| [backend-cqrs](.github/agents/backend-cqrs.agent.md) | CQRS commands and queries — vertical slice structure, EF Core rules, DTO conversion, naming, domain-event publishing. |
| [backend-tests](.github/agents/backend-tests.agent.md) | All backend test types: unit, saga, and integration. |
| [frontend](.github/agents/frontend.agent.md) | Blazor `.razor` components/pages — MudBlazor 9 + Bootstrap, `FeaturesClient` REST calls, mandatory `PersistentState` for SSR+WASM. |

### Review
| Agent | Purpose |
|-------|---------|
| [reviewer](.github/agents/reviewer.agent.md) | General reviewer — backend C#, Razor, SCSS, and tests against architectural rules and scope. |
| [review-backend](.github/agents/review-backend.agent.md) | Backend C# — CQRS structure, EF query rules, domain-event conventions, repository architecture. |
| [review-frontend](.github/agents/review-frontend.agent.md) | Razor and SCSS — MudBlazor 9, Bootstrap, frontend conventions. |
| [review-tests](.github/agents/review-tests.agent.md) | Unit and integration tests — fixtures, isolation, repository testing rules. |

> Reviewers only evaluate — they never implement fixes.

---

## Skills

Skills live in [.github/skills/](.github/skills/). Each is a single-concern playbook loaded on demand.

### Backend — CQRS & domain
| Skill | Use it to… |
|-------|------------|
| [backend-cqrs-command](.github/skills/backend-cqrs-command/SKILL.md) | Implement a **command**: vertical slice, handler conventions, DTOs, validation, EF Core rules, domain-event publishing. |
| [backend-cqrs-query](.github/skills/backend-cqrs-query/SKILL.md) | Implement a **query**: vertical slice, query handler conventions, DTOs, validation, EF Core rules, specifications. |
| [backend-domain-events](.github/skills/backend-domain-events/SKILL.md) | Define an `IDomainEvent`, write its handler, wire the outbox-to-integration-event flow. |
| [backend-feature-flag](.github/skills/backend-feature-flag/SKILL.md) | Declare, configure, and consume a feature flag (Development-only by default). |
| [backend-query-pagination](.github/skills/backend-query-pagination/SKILL.md) | Add pagination with total-count fields to a query. |

### Backend — testing
| Skill | Use it to… |
|-------|------------|
| [backend-unit-tests](.github/skills/backend-unit-tests/SKILL.md) | Fast isolated unit tests — `InMemorySqliteTestScope`, `Assert.*` only, `UnitTest` trait. |
| [backend-integration-tests](.github/skills/backend-integration-tests/SKILL.md) | Integration tests — `AppServerFixture`, PostgreSQL containers, outbox verification, category traits. |
| [backend-test-datafixture](.github/skills/backend-test-datafixture/SKILL.md) | `DataFixture` helpers — pure in-memory factories with fluent chaining (no DB). |

### Frontend — Blazor & styling
| Skill | Use it to… |
|-------|------------|
| [blazor-component](.github/skills/blazor-component/SKILL.md) | Any `.razor` component/page that calls the backend — `FeaturesClient`, `[PersistentState]`, loading/error states, SSR+WASM double-fetch handling. |
| [blazor-generated-client-usage](.github/skills/blazor-generated-client-usage/SKILL.md) | Add a missing typed `FeaturesClient` method — add the endpoint, then regenerate the OpenAPI client (never edit the spec by hand). |
| [frontend-design](.github/skills/frontend-design/SKILL.md) | Styling decisions in `.razor`/`.css`/`.scss` — CSS priority order, forbidden patterns, MudBlazor 9 generic rules, Bootstrap 5.3 reference. |
| [frontend-design-creativity](.github/skills/frontend-design-creativity/SKILL.md) | Premium redesigns / dashboards / card-based results — MudBlazor + Bootstrap layout patterns. |
| [mudblazor-generic-t-audit](.github/skills/mudblazor-generic-t-audit/SKILL.md) | Diagnose and fix MudBlazor generic-`T` build errors (`MudTable`, `MudSelect`, `MudChip`, `MudListItem`, …). |

---

## Instructions

Auto-applied by Copilot to files matching their `applyTo` glob. They live in
[.github/instructions/](.github/instructions/).

| Instruction | Applies to |
|-------------|-----------|
| [cs.instructions.md](.github/instructions/cs.instructions.md) | `**/*.cs` |
| [razor.instructions.md](.github/instructions/razor.instructions.md) | `**/*.razor` |
| [css.instructions.md](.github/instructions/css.instructions.md) | `**/*.css` |
| [scss.instructions.md](.github/instructions/scss.instructions.md) | `**/*.scss` |
| [backend-unit-testing.instructions.md](.github/instructions/backend-unit-testing.instructions.md) | `src/[projectname].Domain.Tests/**/*.cs`, `src/[projectname].Application.Tests/**/*UnitTests.cs` |
| [backend-integration-testing.instructions.md](.github/instructions/backend-integration-testing.instructions.md) | `src/[projectname].Application.Tests/**/*IntegrationTests.cs` |

---

## MCP servers

The agents are wired to these MCP servers (see the `tools:` lists in [.github/agents/](.github/agents/)).

| MCP server | Used by | What it provides |
|------------|---------|------------------|
| [MudMCP](https://github.com/mcbodge/MudMCP) &nbsp;(`mudmcp`) | frontend, orchestrator | Version-accurate MudBlazor API — ~85 components with parameters, events, methods, and examples, indexed from source via Roslyn. Agents query it instead of guessing markup. |
| Context7 &nbsp;(`upstash/context7`) | backend-cqrs, backend-tests, planning, frontend-design | Version-pinned docs for any library. Required by `frontend-design`. |
| Microsoft Docs &nbsp;(`microsoftdocs/mcp`) | orchestrator, planning | Authoritative .NET / C# / Blazor documentation. |
| Stitch &nbsp;(`stitch`) | frontend, orchestrator | UX / design reference for the layout the frontend generates. |

> **MudMCP setup:** point it at your exact MudBlazor version, e.g.
> `dotnet run --project src/MudBlazor.Mcp/MudBlazor.Mcp.csproj -- --version 9.0.0`, or configure it in
> `.mcp.json`. First run clones the MudBlazor repo; later starts load from cache.

---

## How it fits together

```
copilot-instructions.md        ← repo-wide conventions (always on)
        │
   orchestrator                ← plans, classifies, delegates
        │
        ├── planning           ← research (optional)
        │
        ├── implementation     ── loads ──▶ skills/
        │   (backend-cqrs,
        │    backend-tests,
        │    frontend)
        │
        └── review             ← review-backend / review-frontend / review-tests
                                  (evaluate only, never fix)

instructions/  ← file-scoped rules auto-applied by glob alongside everything above
```

Lifecycle: **Planning → Implementation → Review → User commit checkpoint**, repeated until the plan
is complete.
