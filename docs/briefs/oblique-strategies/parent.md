---
description: "Parent synthesis — oblique strategies deep research campaign"
type: deep-research-parent
campaign: oblique-strategies
updated: 2026-04-15
---

# Oblique Strategies: Campaign Synthesis

## Campaign Seed

"Oblique strategies for software ideation — lateral thinking moves when structured approaches get stuck"

## Specialist Summaries

### Specialist 1: Reframing Moves

Researched: de Bono's provocation ("Po"), design thinking frame-storming, Munger's inversion, audience swapping, constraint flipping.

**Core finding:** Reframing moves share a single mechanism — they force the thinker to exit the current frame before generating solutions. Four distinct attack vectors emerged: provocation (absurd frame), frame-storming (question the question), inversion (reverse the goal), and audience swap (borrow someone else's frame). Each has a different trigger and ceiling: inversion is most universally applicable, frame-storming directly attacks the root of stuck-ness, audience swap has the lowest barrier, and provocation has the highest ceiling but requires more practice.

**Contributed moves:** Flip the Goal, Question the Question, Borrow Eyes, State the Absurd.

### Specialist 2: Constraint Moves

Researched: TRIZ contradiction resolution and 40 inventive principles, artificial constraint injection (time/scope/subtraction), Ideal Final Result, kill your darlings, scope slamming.

**Core finding:** Constraint moves work by changing the size and shape of the solution space. They come in three flavors: add constraints (force creative solutions by eliminating the comfortable zone), remove constraints (reveal which limitations are real vs. self-imposed), and resolve contradictions (refuse the trade-off entirely). TRIZ's most transferable insight for software: most "impossible trade-offs" are only impossible within the current framing. The kill-your-darlings move is the most emotionally difficult but often the most productive.

**Contributed moves:** Ship Tomorrow, Kill Your Favorite, Resolve Don't Trade Off.

### Specialist 3: Randomness and Stimulus Moves

Researched: Koestler's bisociation, de Bono's random entry technique, rubber duck debugging as reframing, analogous domain transfer, Eno's error-honoring and randomization techniques.

**Core finding:** Randomness moves work by importing information from outside the current thinking loop. They range from highly structured (analogous domain transfer) to purely random (random word entry). The spectrum matters — structured external input (domain transfer, explain-to-stranger) produces more reliably useful results, while random input (word association, card draws) has a lower hit rate but occasionally produces breakthrough insights that structured approaches never reach. The key discipline: use random input to *generate* directions, then switch to analytical thinking to *evaluate* them.

**Contributed moves:** Steal From a Stranger, Force a Connection, Explain It to a Barista.

## Cross-Specialist Themes

### 1. The Exit-and-Return Pattern

All three specialist domains converge on the same rhythm: exit the current thinking frame, explore laterally, then return to structured analysis. Reframing exits by changing the question. Constraints exit by changing the boundaries. Randomness exits by importing external input. None of these are useful as standalone techniques — they must be paired with rigorous evaluation (first-principles thinking) to produce actionable results.

### 2. The Mechanism Spectrum

The 10 moves map onto a spectrum from structured to chaotic:
- **Most structured:** Flip the Goal (inversion), Question the Question (reframing), Ship Tomorrow (constraint)
- **Semi-structured:** Borrow Eyes (perspective shift), Kill Your Favorite (subtraction), Resolve Don't Trade Off (TRIZ), Steal From a Stranger (domain transfer)
- **Least structured:** Explain It to a Barista (verbalization), Force a Connection (random stimulus), State the Absurd (provocation)

This spectrum should inform usage guidance: start with structured lateral moves; escalate to chaotic ones when structured approaches don't break the pattern.

### 3. The Complementarity with First Principles

First-principles thinking goes *deeper* within a frame. Oblique strategies go *sideways* to find a new frame. The two are complementary, not competitive. The ideal rhythm:
1. First-principles analysis produces a decomposition
2. The decomposition reveals a stuck point or a local optimum
3. Oblique strategies generate a new direction
4. First-principles analysis evaluates and develops the new direction

This rhythm should be explicit in the primer.

## Recommendations for the Primer

1. **10 moves across 3 phases** (Reframe: 4, Constrain: 3, Stimulate: 3) — this balances coverage with memorability
2. **Phase ordering matters:** Reframe first (lowest risk), Constrain second (moderate risk), Stimulate third (highest risk/reward)
3. **Each move needs a concrete software example** — abstract examples won't stick. Use product and architecture scenarios.
4. **Explicit guardrails** against over-using lateral thinking — these are tools for getting *unstuck*, not replacements for structured analysis
5. **The rhythm section** should explicitly describe the handoff between first-principles and oblique strategies
