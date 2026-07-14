---
name: backend-domain-events
description: For defining a domain event (IDomainEvent implementation), writing its handler, or wiring the outbox-to-integration-event flow: event definition, change-tracking pattern that raises events during SaveChanges, and the outbox publishing pipeline.
---

Use this skill when a business action in an aggregate needs to trigger a side-effect in another service or component.

## What are domain events in this repo?

Domain events are **in-memory only** — they implement `IDomainEvent` (a MediatR `INotification`) and are dispatched by the EF Core interceptor within the same database transaction. They are **not serialized** and can carry full entity/aggregate references.

**Key characteristics:**
- Not serializable — can carry full aggregate references and value objects
- Live in `src/[projectname].Domain/[Domain]/Events/`
- Raised inside aggregate methods via `bus.Add(new [Event](...))`
- Consumed by `IDomainEventHandler<T>` in `src/[projectname].Application/Features/[Domain]/DomainHandlers/`
- Handlers publish integration events to the outbox (cross-service, serialized, via RabbitMQ)

## Event flow

```
Aggregate method
  → bus.Add(new SomethingChangedDomainEvent(this, oldValue))
      ↓
  SaveChangesAsync() — EF interceptor fires
      ↓
  IDomainEventHandler<SomethingChangedDomainEvent>
      → _busProvider.IntegrationBus.Add(() => new SomethingChangedIntegrationEvent(...))
          ↓ (after transaction commits)
      Outbox table → RabbitMQ
```

---

## Change-tracking pattern (most common)

The dominant pattern in this repo: the event carries the **current entity** (new state is on the entity itself) plus the **old value** for what changed. The ID is accessed via `entity.Id` — never duplicated as a separate property.

**File:** `src/[projectname].Domain/[Domain]/Events/[Something]ChangedDomainEvent.cs`

```csharp
namespace [projectname].Domain.[Domain].Events;

public class [Something]ChangedDomainEvent : IDomainEvent
{
    public [Aggregate] [Aggregate] { get; }      // current state — new value is on the entity
    public [ValueType] Old[Property] { get; }   // previous value for change tracking

    public [Something]ChangedDomainEvent([Aggregate] aggregate, [ValueType] oldProperty)
    {
        [Aggregate] = aggregate ?? throw new ArgumentNullException(nameof(aggregate));
        Old[Property] = oldProperty;
    }
}
```

Real examples from the codebase:
```csharp
// Company: carries the full Company entity + old registration number
public class UpdatedCompanyRegistrationNumberDomainEvent : IDomainEvent
{
    public Company Company { get; }
    public CompanyRegistrationNumber? OldRegistrationNumber { get; }

    public UpdatedCompanyRegistrationNumberDomainEvent(Company company, CompanyRegistrationNumber? oldRegistrationNumber)
    {
        Company = company;
        OldRegistrationNumber = oldRegistrationNumber;
    }
}

// Procedure: carries the full Procedure entity + old status
public class ProcedureStatusChangedDomainEvent : IDomainEvent
{
    public Procedure Procedure { get; }
    public ProcedureStatus OldStatus { get; }

    public ProcedureStatusChangedDomainEvent(Procedure procedure, ProcedureStatus oldStatus)
    {
        Procedure = procedure ?? throw new ArgumentNullException(nameof(procedure));
        OldStatus = oldStatus;
    }
}
```

---

## Scalar record pattern (creation events, no entity reference needed)

Use a record when the event carries only scalar data (IDs, enums, primitives) — typically for creation events or events where you do not need the full entity in the handler.

**File:** `src/[projectname].Domain/[Domain]/Events/[Something]CreatedDomainEvent.cs`

```csharp
namespace [projectname].Domain.[Domain].Events;

public record [Something]CreatedDomainEvent(
    int [AggregateId],
    int [RelatedId],
    DateTime OccurredAt) : IDomainEvent;
```

---

## Aggregate method that raises the event

**File:** `src/[projectname].Domain/[Domain]/[Aggregate].cs`

```csharp
public void [BusinessAction](IDomainBus bus /*, other args */)
{
    if (/* guard condition */)
        throw new InvalidOperationException("[Business rule violated]");

    var old[Property] = [Property];   // capture old value before mutating
    [Property] = /* new value */;

    bus.Add(new [Something]ChangedDomainEvent(this, old[Property]));
}
```

---

## Domain event handler (publishes integration event to outbox)

**File:** `src/[projectname].Application/Features/[Domain]/DomainHandlers/[Something]ChangedDomainEventHandler.cs`

```csharp
namespace [projectname].Application.Features.[Domain].DomainHandlers;

internal class [Something]ChangedDomainEventHandler : IDomainEventHandler<[Something]ChangedDomainEvent>
{
    private readonly IBusProvider _busProvider;

    public [Something]ChangedDomainEventHandler(IBusProvider busProvider)
    {
        _busProvider = busProvider ?? throw new ArgumentNullException(nameof(busProvider));
    }

    public Task Handle([Something]ChangedDomainEvent e, CancellationToken cancellationToken)
    {
        // project only scalar/serializable data into the integration event
        _busProvider.IntegrationBus.Add(() => new [Something]ChangedIntegrationEvent(
            e.[Aggregate].Id,
            e.Old[Property],
            e.[Aggregate].[NewProperty]
        ));
        return Task.CompletedTask;
    }
}
```

---

## How the command handler wires it together

```csharp
var item = await query.SingleOrDefaultAsync(cancellationToken);

if (item == null)
    throw new NotFoundException(nameof([Aggregate]));

// aggregate captures old value internally, mutates, then calls bus.Add(...)
item.[Aggregate].[BusinessAction](_busProvider.Bus /*, other args */);

// ALWAYS call SaveChangesAsync — EF interceptor dispatches domain events here
await _context.SaveChangesAsync(cancellationToken);
```

---

## Hard Rules

- Domain events live in `src/[projectname].Domain/[Domain]/Events/` — never in Application layer.
- Domain event handlers are `internal` — never `public`.
- `IntegrationBus.Add(...)` only inside `IDomainEventHandler<T>` — never in a command handler.
- `bus.Add(...)` only inside aggregate methods — never called directly from a handler.
- Change-tracking pattern: carry `this` (the aggregate, current state) + the **old** value, not the new one — the new state is already on the entity.
- Scalar record pattern: use only when handler does not need entity references.
- Naming: `[Something]ChangedDomainEvent` / `[Something]CreatedDomainEvent` — past tense.

## Acceptance Criteria

- Domain event is in `src/[projectname].Domain/[Domain]/Events/`.
- Aggregate method captures old value before mutation, then calls `bus.Add(...)`.
- `IDomainEventHandler<T>` is `internal`, in `DomainHandlers/` folder.
- Integration event carries only scalar/serializable data.
- `IntegrationBus.Add(...)` appears only in the handler, not in any command handler.
- `SaveChangesAsync()` called in the command handler.