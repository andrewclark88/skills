---
description: "Specialist brief — interface and data design for system design primer"
type: deep-research/specialist
campaign: system-design
facet: interface-data-design
updated: 2026-04-15
---

# Specialist Brief: Interface and Data Design

## Facet Scope

API contract patterns (REST, GraphQL, gRPC), schema evolution and backward compatibility, versioning strategies, normalization vs denormalization tradeoffs, consistency models, and caching strategies. Focus on tradeoffs — what do you give up with each choice?

## Key Findings

### API Contract Patterns: Decision Framework

#### REST
- **Strengths:** Universal tooling, HTTP caching, simple mental model (resources + verbs), mature debugging ecosystem, browser-native.
- **Weaknesses:** Over-fetching (get more fields than needed) and under-fetching (need multiple calls to assemble a view). Designing good resource boundaries is harder than it looks.
- **When to use:** Public APIs, CRUD-heavy domains, when caching matters, when consumers have uniform data needs.
- **When NOT to use:** When clients have wildly different data shape needs (mobile vs desktop vs admin).

#### GraphQL
- **Strengths:** Clients request exactly what they need, strong type system as a contract between frontend/backend, single endpoint, introspection.
- **Weaknesses:** N+1 query problems (requires DataLoader patterns), caching is significantly harder than REST, clients can construct pathologically expensive queries without rate limiting and query complexity analysis, steeper learning curve.
- **When to use:** Multiple client types with different data needs, rapidly evolving frontends, when the type system contract pays for itself in reduced integration bugs.
- **When NOT to use:** Single client with uniform data shapes, simple CRUD APIs, when REST caching is a key requirement.

#### gRPC
- **Strengths:** Binary protocol (faster than JSON), strongly typed contracts via Protocol Buffers, supports streaming (unary, client, server, bidirectional), code generation across languages.
- **Weaknesses:** Limited browser support, harder to debug (binary format), requires protobuf tooling, overkill for simple APIs.
- **When to use:** Service-to-service communication, high-performance internal systems, polyglot environments, real-time streaming requirements.
- **When NOT to use:** Public-facing APIs, browser clients (without a gateway), simple request-response patterns.

#### MCP and Protocol-Level Patterns
For tool/agent communication (like MCP), the pattern is closer to RPC with capability negotiation — the protocol defines what operations are available, not what resources exist. This is a distinct category from REST/GraphQL/gRPC and suits scenarios where the consumer discovers capabilities at runtime.

**Meta-principle:** Choose the API pattern that matches your consumer profile. If you have one consumer type with predictable needs, REST is simplest. If you have many consumer types with divergent needs, GraphQL pays for itself. If you have high-throughput service-to-service calls, gRPC wins.

### Schema Evolution and Backward Compatibility

**Core principle:** New versions of a schema must be able to handle data created with older versions without breaking systems.

**Compatibility modes:**
- **Backward compatible:** New consumers can read old data. Achieved by making new fields optional with defaults. Upgrade consumers before producers.
- **Forward compatible:** Old consumers can read new data. Achieved by ignoring unknown fields. Upgrade producers before consumers.
- **Full compatible:** Both directions. Maximum safety, maximum constraint on changes.

**Practical rules:**
1. Only add optional fields with defaults — never remove or rename required fields.
2. Use semantic versioning for schemas.
3. Automate compatibility checks in CI/CD to catch breaking changes before they ship.
4. Prefer evolution (additive changes) over versioning (new endpoints) when possible.

**API versioning strategies:**
- **URL versioning** (`/v1/users`): Most visible, easiest to understand, but pollutes URLs.
- **Header versioning** (`Accept: application/vnd.api+json;version=2`): Cleaner URLs, but less discoverable.
- **Query parameter** (`?version=2`): Simple but mixes concerns.
- **Evolution without versioning:** Best when possible — add fields, deprecate old ones, never break consumers. GraphQL naturally supports this pattern.

### Normalization vs Denormalization

**The guiding principle:** "Normalize for correctness, denormalize for performance, and never do either blindly."

**Normalization (fewer tables, no redundancy):**
- Ideal for OLTP systems where consistency and accuracy are critical.
- Reduces data anomalies during insert, update, delete operations.
- Better for write-heavy workloads.
- Tradeoff: Reads require joins, which can be slower at scale.

**Denormalization (wider tables, controlled redundancy):**
- Ideal for OLAP, data warehousing, read-heavy dashboards.
- Faster query response (fewer joins).
- Tradeoff: Write complexity increases, data consistency requires application-level enforcement, schema changes become harder.

**Practical strategy:** Start normalized. Measure ruthlessly. Denormalize surgically — only where profiling shows actual bottlenecks, not where you imagine future ones.

### Consistency Models

**Strong consistency:** Every read returns the most recent write. Simple mental model but requires coordination (slower).

**Eventual consistency:** Reads may temporarily return stale data, but all replicas converge. Higher availability and lower latency.

**Practical nuance (PACELC theorem):** Even without network partitions, there's a tradeoff between latency and consistency. Most systems need both models for different features — strong consistency for financial transactions, eventual consistency for activity feeds.

**Useful middle grounds:**
- **Causal consistency:** Related events appear in order (good for comment threads).
- **Read-your-own-writes:** Users always see their own updates immediately (good for profile edits, form submissions).

**Decision criteria:** Default to strong consistency unless you have measured evidence that the latency cost is unacceptable. Eventual consistency is an optimization, not a default.

### Caching Strategies

**Cache-Aside (Lazy Loading):** Application checks cache, fetches from DB on miss, populates cache. Most common, simplest mental model. Best for read-heavy workloads with infrequent updates.

**Write-Through:** Every write goes to both cache and database synchronously. Strong consistency between cache and DB. Higher write latency. Use when you cannot afford stale reads after writes.

**Write-Behind (Write-Back):** Writes go to cache first, then asynchronously to database. Fastest writes but risk of data loss if cache fails before async write completes. Use for high-volume writes where some risk is acceptable.

**Practical default:** Start with cache-aside. It's the simplest, most transparent, and easiest to reason about. Move to write-through or write-behind only when you have specific consistency or throughput requirements that cache-aside can't meet.

## Distilled Design Moves for Primer

5. **Design Contracts Before Implementations** — Define your API contract (types, schemas, error shapes) before writing code. The contract is the interface between teams and between present and future. Use additive-only changes to evolve without breaking consumers.

6. **Normalize First, Denormalize With Evidence** — Start with normalized data. Denormalize only where profiling reveals actual read bottlenecks. The cost of premature denormalization (write complexity, consistency bugs) almost always exceeds the cost of a slow query you haven't measured yet.

7. **Cache Deliberately** — Start with cache-aside. Add caching at measured bottlenecks, not speculative ones. Every cache introduces a consistency question — make sure you can answer it before adding the cache.

## Sources

- Stripe on API idempotency design
- Confluent documentation on schema evolution compatibility modes
- AWS whitepapers on caching strategies
- REST vs GraphQL vs gRPC decision frameworks (2025-2026)
- PACELC theorem literature
