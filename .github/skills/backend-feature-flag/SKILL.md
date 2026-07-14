---
name: backend-feature-flag
description: How to declare, configure, and consume a feature flag to enable or disable a feature per environment. By default a new feature flag is enabled only in Development; Staging and Production have it disabled.
---

Use this skill whenever you need to gate a feature (job, endpoint, service registration, or code path) behind a flag that can be toggled per environment without redeploying.

---

## Overview

Feature flags in this repo are **plain boolean configuration values** read from the `FeatureManagement` section of `appsettings.json`. They follow the `Microsoft.FeatureManagement` naming convention (`FeatureManagement:<FlagName>`) so the configuration section is compatible with the library without requiring the library to be wired in manually.

**Default policy: enabled in Development only.**

| Environment | Default state |
|-------------|---------------|
| Development | **enabled** (set explicitly to `true`) |
| Staging     | disabled (key absent = `false`) |
| Production  | disabled (key absent = `false`) |

---

## 1. Declare the flag constant

Create (or add to) a `<Domain>FeatureFlags` static class **in the same namespace as the feature that needs the gate**.

**File location:**
```
src/[projectname].Application/Features/<Domain>/[<SubFolder>/]<Domain>FeatureFlags.cs
```

**Example** (`[Domain]` domain, `[SubFolder]` sub-folder):
```csharp
namespace [projectname].Application.Features.[Domain].[SubFolder];

public static class [Domain]FeatureFlags
{
    public const string [FeatureName] = "[Domain].[FeatureName]";
}
```

**Naming rules:**
- Constant value format: `"<Domain>.<FeatureName>"` (PascalCase, dot-separated).
- Class name: `<Domain>FeatureFlags`.
- One class per domain — add multiple constants to the same class if the domain grows.

---

## 2. Configure in appsettings

### `appsettings.Development.json` — **add the flag set to `true`**

```json
{
  "FeatureManagement": {
    "[Domain].[FeatureName]": true
  }
}
```

### `appsettings.Staging.json` and `appsettings.Production.json`

**Do NOT add the key.** Absence evaluates to `false`. Only add an explicit `false` entry in Staging/Production if you need to document an intentional rollout decision.

---

## 3. Consume the flag

Inject `IConfiguration` wherever the decision is made (service registration, job scheduling, a code path at startup, or a request handler).

```csharp
private static bool IsFeatureEnabled(IConfiguration configuration, string featureFlagName) =>
    configuration.GetValue<bool?>($"FeatureManagement:{featureFlagName}") is true;
```

### Service registration (startup / DI)

```csharp
var is[FeatureName]Enabled = IsFeatureEnabled(context.Configuration, [Domain]FeatureFlags.[FeatureName]);

if (is[FeatureName]Enabled)
{
    services.AddHostedService<[FeatureName]Job>();
}
```

### Quartz job scheduling

```csharp
var is[FeatureName]Enabled = IsFeatureEnabled(context.Configuration, [Domain]FeatureFlags.[FeatureName]);

if (is[FeatureName]Enabled)
{
    options.AddTrigger(opts => opts
        .ForJob([FeatureName]Job.Key)
        .WithIdentity([FeatureName]Job.IdleTrigger)
        .WithCronSchedule(dep.Value.ActiveCronExpression));
}
```

### Inside a request handler (runtime gate)

Inject `IConfiguration` as a constructor dependency and call `IsFeatureEnabled` at the start of `Handle()`. Prefer startup-time gating (service registration) when possible — it makes the feature invisible rather than returning an error at runtime.

---

## 4. Full example end-to-end

**Flag declaration** — `src/[projectname].Application/Features/[Domain]/[SubFolder]/[Domain]FeatureFlags.cs`
```csharp
namespace [projectname].Application.Features.[Domain].[SubFolder];

public static class [Domain]FeatureFlags
{
    public const string [FeatureName] = "[Domain].[FeatureName]";
}
```

**appsettings.Development.json**
```json
{
  "FeatureManagement": {
    "[Domain].[FeatureName]": true
  }
}
```

**appsettings.Staging.json** — no `FeatureManagement` key added.

**appsettings.Production.json** — no `FeatureManagement` key added.

**Usage in `QuartzExtensions.cs`**
```csharp
var is[FeatureName]Enabled = IsFeatureEnabled(context.Configuration, [Domain]FeatureFlags.[FeatureName]);

if (is[FeatureName]Enabled)
{
    options.AddTrigger(opts => opts
        .ForJob([FeatureName]Job.Key)
        .WithIdentity([FeatureName]Job.IdleTrigger)
        .WithCronSchedule(dep.Value.ActiveCronExpression));
}
```

---

## Rules summary

- **Never hard-code `true`/`false`** in source for a feature flag value — always read from configuration.
- **Never add the key to Staging or Production** unless intentionally enabling there; absence = disabled.
- **Flag constants live in the domain** they protect, not in a shared infrastructure constants file.
- **Naming**: `"<Domain>.<FeatureName>"` — never use generic names like `"Enabled"` or `"NewFeature"`.
- **One static class per domain** (`<Domain>FeatureFlags`) — add new constants to the existing class; don't create a new class per flag.
- **Prefer startup-time gating** (conditional service/job registration) over runtime `if` checks in handlers.
