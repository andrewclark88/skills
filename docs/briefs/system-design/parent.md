---
description: "Parent synthesis — system design deep research campaign"
type: deep-research/parent
campaign: system-design
updated: 2026-04-15
---

# System Design Primer: Research Synthesis

## Campaign Overview

**Seed:** System design principles and patterns for software engineers
**Output:** A curated, opinionated primer of 12-18 design moves organized by concern, analogous to the first-principles thinking primer.
**Specialists:** 4 parallel research facets — architecture patterns, interface/data design, distributed systems/scalability, reliability/observability.

## Cross-Facet Themes

### Theme 1: The Pragmatism Shift (2024-2026)

Every facet surfaced the same meta-pattern: the industry is swinging from "use every pattern" to "earn every pattern." This is not anti-pattern or anti-architecture — it's a maturity shift toward evidence-based complexity.

- **Architecture:** 42% of microservices adopters consolidating back. Modular monolith as default.
- **Interfaces:** REST as default unless multiple client types with divergent needs justify GraphQL.
- **Scalability:** Vertical first. Don't shard until you've exhausted simpler options.
- **Reliability:** Circuit breakers and tracing are powerful but premature for systems without external dependencies or service boundaries.

**Synthesis:** The strongest unifying principle across all four facets is **"earn your complexity."** Every pattern has a cost. The cost is justified only by measured evidence, not by anticipated scale or theoretical purity.

### Theme 2: Boundaries Are the Highest-Leverage Decision

Across all facets, the decision about where to draw boundaries dominates:

- **Architecture:** Module boundaries determine whether you can extract services later. Get them wrong and everything downstream suffers.
- **Interfaces:** API contracts are the boundaries between teams and between present and future. Schema evolution is boundary management.
- **Data:** Normalization is about data boundaries (where does each fact live?). Denormalization deliberately crosses those boundaries for performance.
- **Reliability:** Bulkheads isolate failure boundaries. Defense in depth layers boundaries.

**Synthesis:** Most design effort should go into deciding where boundaries are and what crosses them. The internal implementation of any component is cheaper to change than its external interface.

### Theme 3: Reversibility as a Design Principle

Fowler's irreversibility principle appeared explicitly in architecture but implicitly everywhere:

- Choosing a database is hard to reverse. Choosing a caching strategy is easy to reverse.
- Choosing an API contract shape is hard to reverse. Choosing the internal implementation behind it is easy to reverse.
- Choosing to distribute (microservices) is hard to reverse. Staying monolithic keeps options open.

**Synthesis:** Invest decision time proportional to reversal cost. The primer should help implementers identify which decisions are expensive to reverse.

### Theme 4: Consistency Is a Spectrum, Not a Toggle

The naive model of "strong consistency vs eventual consistency" is wrong:

- CAP theorem properties are continuous, not binary.
- Most systems need different consistency levels for different features.
- Read-your-own-writes and causal consistency are practical middle grounds.
- Caching introduces its own consistency dimension (every cache is a consistency question).

**Synthesis:** The primer should present consistency as a per-feature decision with multiple options, not a system-wide toggle.

## Contradictions and Tensions

### Tension: Simplicity vs Preparedness
The pragmatism shift says "start simple." But some things are much cheaper to build in from the start:
- Structured logging is easy to add on day one, expensive to retrofit.
- Idempotency is easy to design in, hard to bolt on.
- API contracts are easy to get right initially, expensive to fix after consumers depend on them.

**Resolution:** The primer distinguishes between "design-in patterns" (always include: idempotency, structured logging, boundary validation, contract-first APIs) and "earn-in patterns" (add with evidence: CQRS, circuit breakers, caching, horizontal scaling, distributed tracing).

### Tension: Ports-and-Adapters Everywhere vs Pragmatic Boundaries
Hexagonal architecture purists want ports and adapters for everything. Pragmatists say the 20% boilerplate overhead isn't justified for simple projects.

**Resolution:** Apply dependency inversion at real infrastructure boundaries (database, external APIs, message queues). Don't ceremony-ize internal function calls. The principle is always right; the ceremony is situational.

## Primer Structure

Based on the synthesis, the primer organizes into 5 concerns with 15 moves:

1. **Structure** (moves 1-3): How you organize code and draw boundaries
2. **Interfaces** (moves 4-6): How components communicate and evolve
3. **Data** (moves 7-9): How you store, cache, and keep data consistent
4. **Scale** (moves 10-12): How you handle growth without premature complexity
5. **Reliability** (moves 13-15): How you handle failure and maintain visibility

Each move is categorized as either **design-in** (include from the start) or **earn-in** (add when evidence justifies it).

## Specialist Briefs

| Brief | Key Contribution |
|-------|-----------------|
| [Architecture Patterns](specialist-architecture-patterns.md) | Modular monolith default, earned extraction, irreversibility principle |
| [Interface & Data Design](specialist-interface-data-design.md) | Contract-first design, schema evolution, caching decision framework |
| [Distributed & Scalability](specialist-distributed-scalability.md) | Idempotency as hygiene, statelessness, vertical-first scaling |
| [Reliability & Observability](specialist-reliability-observability.md) | Essential-from-day-one vs earned patterns, three pillars prioritization |
