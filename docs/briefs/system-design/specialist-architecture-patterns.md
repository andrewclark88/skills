---
description: "Specialist brief — architecture patterns for system design primer"
type: deep-research/specialist
campaign: system-design
facet: architecture-patterns
updated: 2026-04-15
---

# Specialist Brief: Architecture Patterns

## Facet Scope

Decision criteria for structural architecture patterns: monolith vs modular monolith vs microservices, layered vs hexagonal/ports-and-adapters, event-driven and CQRS, module decomposition, and service boundaries. Focus on the 2024-2026 consensus shift.

## Key Findings

### The Microservices Backlash Is Real and Measurable

The industry consensus has shifted decisively between 2024-2026:

- **42% of organizations** that adopted microservices are consolidating services back toward monoliths or modular monoliths.
- **Amazon Prime Video** achieved a 90% infrastructure cost reduction by consolidating from microservices to a monolith.
- **90% of microservices teams** still batch-deploy like monoliths (per DORA metrics research) — achieving maximum complexity with minimum benefit.
- **Service mesh adoption declined** from 50% to 42% (CNCF 2025 data) — a signal of architectural fatigue.
- **Cost differential:** A modular monolith requires 1-2 ops engineers; an equivalent microservices architecture typically requires 2-4 platform engineers plus distributed operational burden.

**The emerging consensus:** Microservices are a destination, not a starting point. Benefits only materialize with teams exceeding 10-15 developers and proven independent scaling needs. Below that threshold, teams experience net productivity losses.

### The Modular Monolith as Default

DHH's position: "The majestic monolith remains undefeated for the vast majority of web apps. Replacing method calls with network calls makes everything harder, slower, and more brittle. It should be the absolute last resort."

The modular monolith organizes code as "a collection of loosely coupled, domain modules based on DDD subdomains/bounded context rather than technical layers." This gives you:

- Clean module boundaries that can evolve to service boundaries later
- Method calls instead of network calls (orders of magnitude faster, no serialization overhead)
- Single deployment unit (simpler CI/CD, simpler debugging)
- The option to extract services when there's evidence, not speculation

**Decision criteria for extraction:** Extract a module to a service only when you have (1) independent scaling requirements backed by data, (2) independent deployment cadence that the monolith blocks, or (3) a team boundary that genuinely benefits from process isolation.

### Hexagonal / Ports-and-Adapters

The core insight: business logic sits at the center and depends on nothing. Everything external — databases, APIs, UIs, message queues — connects through well-defined interfaces (ports) with concrete implementations (adapters).

**When to use:** Projects with multiple integration points, projects where you want to swap infrastructure without touching business logic, projects with meaningful test requirements (mock adapters for unit tests without real DB hits).

**When NOT to use:** Small scripts, simple CRUD apps, prototypes. The pattern adds ~20% boilerplate — that cost needs to be justified by the boundary benefits.

**Practical rule:** You don't need the full hexagonal ceremony. The core principle — dependency inversion at system boundaries — is always valuable. Apply it where you have actual boundaries (database, external APIs, message queues), not between every internal function.

### CQRS and Event Sourcing

**CQRS** (Command Query Responsibility Segregation): Separate write models from read models.

**When to use:** High volumes of reads and writes with different optimization needs, complex domains where read and write shapes diverge significantly, systems needing audit trails.

**When NOT to use:** Simple CRUD applications with modest needs. Smaller teams should delay adoption until a clear necessity arises. The eventual consistency tradeoff (reads may lag writes) adds cognitive complexity.

**Event Sourcing:** Store events, not state. Replay events to compute current state.

**When to use:** Full audit trail requirements (finance, compliance), need to rebuild state or project new views, complex domain where understanding "how we got here" matters.

**When NOT to use:** Most applications. Event sourcing adds significant complexity (event versioning, snapshot management, eventual consistency). Reserve it for domains where the audit trail is a core requirement, not a nice-to-have.

### Event-Driven Architecture

Decouple producers from consumers through events. Two flavors:

1. **Event notification:** Something happened, go look it up (lightweight, loose coupling).
2. **Event-carried state transfer:** Here's what happened plus the data (heavier payload, less coupling on reads).

**When to use:** Multiple systems need to react to the same business event, you need temporal decoupling (producer doesn't wait for consumer), you need to add new consumers without modifying producers.

**When NOT to use:** Simple request-response flows, when you need synchronous confirmation, when the added indirection makes debugging harder than the coupling it removes.

### Martin Fowler's Irreversibility Principle

Fowler identified irreversibility as a prime driver of complexity: "When something is too hard to change, past decisions cannot be reversed and constantly working around the limitations they impose makes new decisions more complex."

**Practical application:** For every architectural decision, ask: "How hard would this be to reverse?" If the answer is "very hard," invest more time in the decision. If it's "easy to reverse," decide quickly and move on.

## Distilled Design Moves for Primer

1. **Start Monolith, Earn Microservices** — Begin with a modular monolith. Extract services only when you have measured evidence (scaling data, deployment friction, team boundaries) that justifies the network boundary cost.

2. **Invert Dependencies at Real Boundaries** — Apply ports-and-adapters where you have actual infrastructure boundaries (databases, external APIs, message queues). Don't ceremony-ize internal function calls.

3. **Separate Reads from Writes (When Earned)** — CQRS and event sourcing add power but also complexity. Use them when read/write patterns genuinely diverge or audit trails are a core requirement. Default to simple CRUD.

4. **Minimize Irreversible Decisions** — Invest decision time proportional to reversal cost. Easy-to-reverse decisions should be made quickly. Hard-to-reverse decisions deserve research and challenge.

## Sources

- DORA metrics research on microservices deployment patterns
- CNCF 2025 survey data on service mesh adoption
- DHH on the majestic monolith (2024-2026)
- Martin Fowler on eliminating irreversibility
- Alistair Cockburn, hexagonal architecture (2024 book)
- Gergely Orosz, "Software Architecture is Overrated, Clear and Simple Design is Underrated"
