---
description: "Specialist brief: Constraint moves — techniques that change the BOUNDARIES of the problem"
type: deep-research-specialist
campaign: oblique-strategies
updated: 2026-04-15
---

# Specialist Brief: Constraint Moves

## Research Scope

Techniques that change the BOUNDARIES of the problem. "Ship in one day," "remove the feature you love most," "make it 10x simpler," "make it 10x more ambitious," TRIZ contradiction resolution. Sources: TRIZ (Altshuller), creative constraints literature, "less is more" in product design, Eno's constraint cards.

## Key Findings

### 1. TRIZ Contradiction Resolution

TRIZ's core insight: inventive breakthroughs come from *eliminating* contradictions rather than trading one parameter off against another. When improving A makes B worse, the standard approach is to compromise. TRIZ says: find a way to resolve the contradiction entirely.

**Mechanism:** Altshuller analyzed 40,000+ patents and found that inventive solutions fall into 40 recurring principles. The Contradiction Matrix maps pairs of conflicting parameters to the principles most likely to resolve them. The key insight for software: most "impossible trade-offs" are only impossible within the current framing.

**Relevant TRIZ principles for software:**
- **Segmentation** (Principle 1): Divide a monolithic thing into independent parts. (Microservices, but only when earned.)
- **Extraction** (Principle 2): Separate the interfering part from the useful part. (Extract the slow operation into an async job.)
- **The Other Way Around** (Principle 13): Invert the action. Instead of the server pushing, have the client pull. Instead of validating input, generate valid input.
- **Preliminary Action** (Principle 10): Perform the required action in advance. (Pre-compute, cache, pre-warm.)
- **Intermediary** (Principle 24): Use an intermediate object to transfer or carry out an action. (Message queues, adapters, API gateways.)

**Software translation:** "We need the API to be both fast AND flexible." TRIZ says: don't compromise. Can you serve a fast fixed response *and* a slower flexible one? (GraphQL + REST side by side. Cached summary + on-demand detail.)

### 2. Artificial Constraint Injection

Deliberately add a constraint that doesn't exist to force creative solutions. The constraint acts as a forcing function — it eliminates the option space you're stuck in and forces you into unexplored territory.

**Mechanism:** Paradoxically, more freedom creates more paralysis (the paradox of choice). Constraints reduce the search space, which makes the remaining space easier to explore deeply. Research consistently shows that moderate constraints enhance creativity while their absence leads to conventional solutions.

**Canonical constraints for software ideation:**
- **The 1-Day Constraint:** "What if you had to ship this tomorrow?" Strips away nice-to-haves and exposes the essential kernel.
- **The 10x Constraint:** "What if this needed to serve 10x the users?" or "What if this needed to cost 1/10th?" Forces structural rethinking rather than incremental optimization.
- **The Subtraction Constraint:** "Remove the feature you love most. What's left?" Reveals whether the remaining product still has value — and often the answer is "yes, and it's simpler."
- **The Zero-UI Constraint:** "What if there were no interface?" Forces the system to be so smart or well-integrated that it just works.

### 3. Ideal Final Result (TRIZ)

Imagine the ideal solution — the problem solves itself with no cost, no mechanism, no added complexity. Work backward from there.

**Mechanism:** TRIZ's Ideal Final Result (IFR) asks: "What would the solution look like if it appeared by magic?" This is not wishful thinking — it establishes a target. The gap between the IFR and reality reveals the actual obstacles, which are often fewer and more specific than the problem felt before this exercise.

**Software translation:** "The ideal deployment has zero downtime, zero manual steps, and zero configuration." That's the IFR. Working backward: what's preventing that? A manual approval step? A config file that differs per environment? Each obstacle is now a specific, attackable problem rather than a vague "deployment is hard."

### 4. Kill Your Darlings

Remove the thing you're most attached to. The feature you love, the architecture pattern you're proud of, the abstraction you spent a week building. If the product/system is still good without it, it was excess. If it collapses, you've found the load-bearing element.

**Mechanism:** Sunk cost fallacy and endowment effect make us overvalue things we've invested in. "Kill your darlings" originated in writing (attributed to Faulkner, Quiller-Couch) and means: the piece you're most reluctant to cut is often the piece that's dragging the whole work down, because you're protecting it from the scrutiny you apply to everything else.

**Software translation:** "We spent three sprints building the plugin architecture. But do we actually have plugins?" Kill it. "The real-time sync is elegant engineering. But users check the dashboard once a day." Kill it, ship a cron job. The exercise reveals whether your design is serving users or serving your ego.

### 5. Scope Slam

Radically change the scope — either collapse it to the absolute minimum or expand it to the absurdly ambitious. Both moves break the incremental thinking trap.

**Scope down:** "What's the smallest possible version that still solves the core problem?" This isn't MVP thinking (which often includes too much). This is: one feature, one user, one use case. What is it?

**Scope up:** "What if this were 100x more ambitious?" Not "how do we scale the current thing" but "what would a completely different thing look like at that scale?" The 100x version often reveals a fundamentally different approach that, when scaled back down, is better than the incremental approach at 1x.

## Synthesis for Primer

Constraint moves work by changing the *size and shape* of the solution space. They come in three flavors:

1. **Add constraints** (artificial constraint injection) — force creative solutions by eliminating the comfortable option space
2. **Remove constraints** (kill your darlings, IFR) — reveal which constraints are real and which are self-imposed
3. **Resolve contradictions** (TRIZ) — refuse the trade-off and find solutions that satisfy both sides

**Recommended moves for the primer:**
5. **Ship Tomorrow** (artificial time constraint) — simplest, most immediately useful
6. **Kill Your Favorite** (kill your darlings) — attacks sunk cost directly
7. **Resolve, Don't Trade Off** (TRIZ contradiction) — highest-ceiling constraint move, requires identifying the specific contradiction
