---
description: "Specialist brief — distributed systems and scalability for system design primer"
type: deep-research/specialist
campaign: system-design
facet: distributed-scalability
updated: 2026-04-15
---

# Specialist Brief: Distributed Systems and Scalability

## Facet Scope

CAP theorem and its real nuance, eventual consistency patterns, idempotency, stateless design, horizontal vs vertical scaling, partitioning strategies. Focus on practical application — what matters at 10-1000 user scale vs Google scale?

## Key Findings

### CAP Theorem: More Nuanced Than the Textbook

The textbook version: you can only have two of three — Consistency, Availability, Partition tolerance.

**The real nuance:**
1. **Partition tolerance isn't optional.** In any distributed system, network partitions will happen. So the real choice is between consistency and availability during a partition — not a free three-way pick.
2. **The properties are continuous, not binary.** Availability ranges from 0 to 100%, there are multiple levels of consistency, and even partitions have nuances (including disagreement about whether a partition exists).
3. **The choice is per-feature, not per-system.** A ticketing system needs strong consistency for seat bookings but can prioritize availability for viewing event details. Design consistency levels per data path, not per application.
4. **PACELC extends CAP:** Even without partitions, there's a tradeoff between latency and consistency. This is the tradeoff you'll actually face most often — true network partitions are rare in well-managed infrastructure.

**Practical takeaway:** Most systems at 10-1000 user scale don't need to think about CAP at all. If you're running a single database, you have strong consistency by default. CAP becomes relevant when you introduce replication, distributed caches, or multiple data stores. Don't distribute until you need to.

### Eventual Consistency Patterns

When you do need eventual consistency, these patterns manage the complexity:

#### Saga Pattern
A sequence of local transactions where each service performs its operation and initiates the next step through events. If a step fails, compensating transactions undo the completed steps.

Two flavors:
- **Choreography:** Each service listens for events and decides what to do next. Simpler for few services, harder to trace as complexity grows.
- **Orchestration:** A central coordinator directs the flow. Easier to understand and debug, but introduces a single point of coordination.

**When to use:** Distributed transactions across multiple services where you can't use a single database transaction. Common in payment flows, order processing, multi-service workflows.

**When NOT to use:** If you can use a single database transaction, do that instead. Sagas add significant complexity. The best saga is the one you don't need because your services share a database.

#### Outbox Pattern
Write both the data change and the event to the same database in a single transaction. A separate process reads the outbox table and publishes events. Guarantees that the data change and the event are atomic without distributed transactions.

**When to use:** Any time you need to update data and publish an event reliably. The simplest reliable pattern for event publishing.

### Idempotency

**Core principle:** The same operation executed multiple times produces the same result as executing it once.

**Why it matters:** Networks are unreliable. Clients retry. Message queues deliver at-least-once. Without idempotency, retries cause duplicates (double charges, duplicate orders, repeated side effects).

**Implementation patterns:**
1. **Idempotency keys:** Clients generate a unique key (UUID) per logical action. Server stores keys, checks for duplicates, only processes unique keys once. This is Stripe's approach and the industry standard for payment APIs.
2. **Natural idempotency:** HTTP GET, PUT, DELETE are naturally idempotent. POST is not — add idempotency key support for POST operations with side effects.
3. **Database-level:** Use upserts (INSERT ON CONFLICT) or conditional updates (UPDATE WHERE version = X) to make writes idempotent.

**Practical rule:** Any operation that creates resources or triggers side effects should be idempotent. This is not premature optimization — it's basic reliability hygiene.

### Stateless Design

**Core principle:** Don't store request-specific state on the server instance. Offload state to external stores (database, Redis, S3).

**Why it matters:** When any instance can handle any request, you get:
- Horizontal scaling (add instances behind a load balancer)
- Fault tolerance (one instance dies, others pick up the work)
- Simpler deployment (roll instances without session migration)

**What to externalize:** Session data, user preferences, in-progress work, file uploads. Anything that would be lost if the instance restarted.

**When stateful is okay:** Caches that can be rebuilt, connection pools, in-memory indexes that are populated on startup. These are "soft state" — losing them causes a performance hit, not data loss.

**Practical rule:** Make services stateless by default. If you find yourself storing request-specific data in memory or on local disk, externalize it.

### Horizontal vs Vertical Scaling

**Vertical scaling (scale up):** Bigger machine. Simpler, no distributed systems complexity. Works until you hit hardware limits or cost curves bend unfavorably.

**Horizontal scaling (scale out):** More machines. Requires stateless design, load balancing, potentially data partitioning. More complex but theoretically unlimited.

**Decision criteria:**
- **Start vertical.** At 10-1000 users, a single well-provisioned server handles most workloads. Vertical scaling is simpler and cheaper at small scale.
- **Go horizontal when:** You've hit the vertical ceiling, you need fault tolerance (one machine dying shouldn't take down the system), or you have workloads that parallelize well (web requests, batch processing).
- **Don't horizontally scale the database first.** Database scaling is the hardest part. Scale the stateless application tier horizontally first. Optimize queries, add read replicas, and cache before partitioning the database.

**Common mistake:** Scaling out doesn't fix bottlenecks. If your database is the bottleneck, adding more web servers doesn't help. Profile first, then scale the actual bottleneck.

### Partitioning (Sharding)

Splitting data across multiple databases or storage nodes.

**When to consider:** Single database can't handle the write volume, single database can't store the data volume, you need data locality (geographic sharding).

**When NOT to consider:** Most applications under 1000 users. Sharding adds massive complexity (cross-shard queries, rebalancing, referential integrity). Exhaust vertical scaling, read replicas, and caching before sharding.

**Partitioning strategies:**
- **Range-based:** Easy to understand, risk of hot spots on recent data.
- **Hash-based:** Even distribution, but range queries become expensive.
- **Directory-based:** Flexible routing, but the directory is a single point of failure/bottleneck.

## Distilled Design Moves for Primer

8. **Make Operations Idempotent** — Any operation that creates resources or triggers side effects must be safe to retry. Use idempotency keys for non-idempotent HTTP methods. Use upserts or conditional updates at the database level. This is reliability hygiene, not optimization.

9. **Keep Services Stateless** — Don't store request-specific state on server instances. Externalize session data, in-progress work, and uploads to dedicated stores. Statelessness enables horizontal scaling, fault tolerance, and simple deployments.

10. **Scale Vertically First** — Start with a single well-provisioned server. Go horizontal when you've hit vertical limits or need fault tolerance. Never shard the database until you've exhausted caching, read replicas, and query optimization.

## Sources

- Stripe engineering blog on idempotency
- PACELC theorem literature (Daniel Abadi, 2010)
- ByteByteGo on idempotency patterns
- Microsoft Azure Architecture Center on saga pattern
- CAP theorem practical nuance (multiple 2025 sources)
- CNCF surveys on service mesh and infrastructure patterns
