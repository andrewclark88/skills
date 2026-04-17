---
description: "Campaign self-evaluation — oblique strategies deep research"
type: deep-research-campaign
campaign: oblique-strategies
updated: 2026-04-15
---

# Campaign Evaluation: Oblique Strategies

## Campaign Metadata

- **Seed:** "Oblique strategies for software ideation — lateral thinking moves when structured approaches get stuck"
- **Specialists:** 3 (Reframing, Constraints, Randomness/Stimulus)
- **Primary output:** `/dev/skills/docs/oblique-strategies.md`
- **Campaign briefs:** `/dev/skills/docs/briefs/oblique-strategies/`

## Quality Assessment

### Coverage

| Domain | Coverage | Notes |
|--------|----------|-------|
| Brian Eno / Oblique Strategies | Strong | Cards researched, key principles extracted (honor the error, perspective shifts, randomization). Adapted from music to software. |
| Edward de Bono / Lateral Thinking | Strong | Provocation ("Po"), random stimulus, and the self-organizing pattern model all covered. Six Thinking Hats referenced but not directly included — hats are more of a group facilitation tool than an individual ideation move. |
| TRIZ | Moderate | Contradiction resolution and IFR covered well. The full 40 principles and contradiction matrix are too detailed for a primer — correctly abstracted to the key transferable insight. |
| Design Thinking / Reframing | Strong | HMW questions, frame-storming, constraint removal all covered. |
| Munger / Inversion | Strong | Full treatment in reframing specialist. Connected to first-principles primer's existing Move 4. |
| Koestler / Bisociation | Strong | Core concept well-covered, translated to software domain. |
| Rubber Duck Debugging | Strong | Reframed as an oblique strategy (explain to outsider), not just a debugging technique. |
| Kill Your Darlings | Strong | Connected to sunk cost fallacy, given concrete software examples. |

### Gaps

1. **Six Thinking Hats** — Deliberately excluded. The hats framework is a group facilitation tool, not an individual lateral thinking technique. It would be a better fit for a future "workshop facilitation" primer than for this one.
2. **Morphological analysis** — Not covered. Systematic combination of sub-solutions. Could be a future addition but sits closer to structured thinking than lateral thinking.
3. **SCAMPER method** — Not explicitly covered, though several SCAMPER operations (Substitute, Eliminate, Reverse) map to moves in the primer. Decided against including SCAMPER as a named move because its component parts are already covered by stronger individual techniques.

### Synthesis Quality

The parent synthesis correctly identified three cross-specialist themes:
1. Exit-and-return pattern (all moves share this structure)
2. Structured-to-chaotic spectrum (informs escalation guidance)
3. Complementarity with first-principles (the handoff rhythm)

These themes shape the primer's structure and guardrails. The rhythm section explicitly addresses when to switch between first-principles and oblique strategies.

### Primer Assessment

- **Move count:** 10 (target was 10-12) — appropriate for memorability
- **Line count:** ~250 (target was 200-250) — within range
- **Format match:** Follows first-principles.md structure: phases, move format (name/why/how/example), skill emphasis table, guardrails
- **Differentiation from first-principles:** Clear. First-principles goes deeper. Oblique strategies goes sideways. No move overlap (inversion appears in both but with different framing — first-principles uses it as an analytical tool, oblique strategies uses it as a creative reframing tool).
- **Actionability:** Each move has a concrete "how to apply" and a software-specific example. Tested against the criterion: "would someone actually reach for this when stuck?"

## Confidence Rating

**Overall: 8/10**

Strengths: Strong source coverage, practical software translations, clear differentiation from existing primers, well-structured escalation path (reframe -> constrain -> stimulate).

Weaknesses: TRIZ coverage is necessarily shallow (the full methodology is book-length). Some randomness/stimulus moves may feel uncomfortable to analytically-minded engineers — the guardrails try to address this but the discomfort is inherent to the technique. The primer would benefit from real-world case studies of lateral thinking in software, which would require more extensive research into specific product histories.
