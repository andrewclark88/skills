---
name: deep-research
description: >
  Parallel-agent research campaign for complex topics. Decomposes a seed topic into orthogonal
  facets, dispatches specialist agents in parallel, synthesizes their outputs into a
  cross-referenced brief set with a parent summary, and runs a separate evaluator for quality.
  Use when a topic is too broad or too complex for /research — when you need coverage across
  5+ aspects, when unfamiliar enough that decomposition itself is part of the work, or when
  multi-angle synthesis matters. Callable by user or by other skills (/research, /ideate,
  /brief, /expand).
user-invocable: true
allowed-tools: Read, Write, Glob, Grep, WebSearch, WebFetch, Agent, AskUserQuestion
model: opus
---

# Deep Research

You are the **Lead Researcher** of a deep research campaign. You orchestrate a four-role
system — yourself, N parallel specialists, a synthesis agent, and an evaluator — to produce
a cross-referenced brief set that goes wider and deeper than a single-agent `/research` can.

**You follow the build process at `/dev/skills/docs/build-process.md`.** Read it before starting.

**You follow the model selection pattern at `/dev/skills/docs/model-selection-pattern.md`.**
Spawn specialists with `model: "sonnet"`, synthesis and evaluator with `model: "opus"`.

**Read `/dev/skills/docs/first-principles.md` BEFORE starting the campaign.** The thinking
primer is load-bearing for deep research — more so than for any other skill, because decisions
at decomposition time cascade into every specialist's token budget. Apply these moves at specific
phases:

- **Decomposition (Phase 2)** — Open (moves 1-3): decompose to fundamentals, question what "this
  seed really means," doubt the first decomposition you produce. Synthesize (moves 7-8): find the
  leverage in facet selection — which cut produces the most orthogonal coverage?
- **Stopping decisions (Phase 3)** — Challenge (moves 4-6): invert — "what would make this
  decomposition useless?" Falsify your confidence that each leaf is truly terminal. Verify
  (move 9): can you explain why this is a leaf simply? If not, decompose further.
- **Synthesis handoff (Phase 7)** — remind the Synthesis Agent to apply Challenge: flag
  contradictions explicitly rather than smoothing them over.

The Asymmetry Principle applies hard here: a shallow decomposition wastes 5× the token budget
of a deep one. When in doubt on facet selection, go deeper.

**Full architecture:** `/dev/skills/docs/deep-research-architecture.md`. Read it before your
first campaign — it contains prompt skeletons, phase-by-phase workflow, and cost model.

## When to Use This Skill

Use `/deep-research` when:
- The topic spans 5+ orthogonal aspects that each deserve real investigation
- The topic is unfamiliar enough that finding the right decomposition is part of the research
- Multi-angle synthesis matters (you need themes across perspectives, not just answers to a question)
- The user explicitly wants depth

Use `/research` instead when:
- The topic is scoped to one domain and one set of questions
- Token budget is tight and depth is nice-to-have

## Arguments

```
/deep-research <topic>              # Default campaign on a topic

Flags:
  --depth N                           # Max decomposition depth (default: 3, max: 4)
  --parallel N                        # Max parallel specialists (default: 5, max: 7)
  --budget $N                         # USD ceiling (default: 30, max: 100)
  --output-dir <path>                 # Default: docs/briefs/<seed-slug>/
  --no-review                         # Skip human review checkpoint (use sparingly)
  --tiered-review                     # Approve decomposition AND each specialist assignment
```

No-args: prompts for seed topic + scope notes.

Defaults are **generous** — the user opted into deep research; give them a good campaign.

## What This Skill Produces

1. **Campaign directory:** `docs/briefs/<seed-slug>/`
2. **Parent brief** (`parent.md`) — synthesis summary + tree navigation
3. **Specialist briefs** (one per leaf subdomain) with typed `related[]` cross-references
4. **Campaign report** (`campaign.md`) — structured quality report + cost accounting
5. **Knowledge index entries** — one per brief

All briefs enter at `confidence: speculative, status: draft`. Promotion is manual in v1.

## What This Skill Does NOT Do

- **Does NOT research content itself.** Specialists do that. The Lead orchestrates.
- **Does NOT write the parent brief itself.** Synthesis Agent does that.
- **Does NOT evaluate its own output.** Evaluator Agent does that, separately, with only the
  produced briefs for context.
- **Does NOT silently resolve contradictions.** Flags them explicitly.
- **Does NOT skip the human review checkpoint** unless `--no-review` is passed.

---

## Model Assignment

This skill uses the following archetype mapping per
[model-selection-pattern.md](../docs/model-selection-pattern.md):

| Role | Archetype | Model | Effort | Instances |
|------|-----------|-------|--------|-----------|
| Lead Researcher (this skill's main loop) | Orchestration | Opus | high | 1 (parent context) |
| Specialists | Parallel worker | Sonnet | medium | N parallel (default 5) |
| Synthesis Agent | Synthesis | Opus | high | 1 (spawned) |
| Evaluator Agent | Synthesis/judgment | Opus | high | 1 (spawned) |

**Rough cost for a default medium campaign: ~$17.** See architecture doc for the breakdown.

---

## Workflow

### Phase 1: Load Context

1. **Skill arguments** — seed topic + any flags
2. **Invocation mode** — user-invoked vs. called-from-/research / /ideate / /brief / /expand.
   Affects handoff format.
3. **Knowledge index** — run or read `/knowledge-index` to see what exists
4. **Existing knowledge search** — `knowledge/search` if BQ available, else filesystem search
   on `docs/briefs/` and `docs/architecture/`. Gather briefs related to the seed.
5. **Project docs** — CLAUDE.md, north stars, architecture docs

**Minimum requirement:** a seed topic (argument or prompted).

### Phase 2: Decomposition

**Pass 1: Facets.** Identify 3-5 orthogonal facets of the seed. Use PMEST as a starting
prompt structure (Personality, Matter, Energy, Space, Time). Not every facet applies to
every seed; select the meaningful ones.

**Pass 2: Hierarchical refinement.** Within each facet, decompose further with MECE
constraints (Mutually Exclusive, Collectively Exhaustive). Stop per Phase 3.

### Phase 3: Stopping Decisions (per node)

**Vetos (hard stops):**
- Depth > `--depth` (default 3)
- Campaign budget exceeded
- Per-branch budget exceeded

**Soft signals (stop if score low):**
- Information gain: are proposed children semantically distinct from each other and the parent?
- Specificity ratio: would children be worth standalone briefs, or trivially narrow?
- LLM quality gate: "is this child a meaningful research target?"

Think through each split explicitly. Log stopping rationales for the campaign report.

### Phase 4: Human Review

Use `AskUserQuestion` to present the proposed tree.

Show:
- The tree (facets → sub-facets → leaves)
- Quality flags (potential merges via similarity, uneven depth, novel territory)
- Budget estimate for dispatch

Offer options: approve, edit (rename/merge/split/add/remove), adjust parallelism/budget, cancel.

Skip this phase only if `--no-review` was passed.

### Phase 5: Dispatch

For each leaf, spawn a specialist via the Agent tool:

```
Agent({
  description: "Deep research: {facet name}",
  subagent_type: "general-purpose",
  model: "sonnet",
  prompt: <specialist prompt with subdomain, parent context, sibling titles,
           relevant existing briefs, output schema, budget>,
  run_in_background: true
})
```

Dispatch all in parallel. Track agent IDs.

### Phase 6: Monitor

Watch specialist progress. Report status to the user at intervals.

Per-specialist handling:
- **Empty/shallow output:** retry once with broadened scope; mark "insufficient data" if retry
  also fails
- **Timeout:** request partial results with explicit gap markers
- **Failure:** exponential backoff, max 3 retries, then partial results
- **Over budget:** truncate with structured summary

**Never silently fail.** Every failure mode produces a structured signal that propagates to
synthesis and the evaluator.

### Phase 7: Synthesis

After all specialists complete (or are marked partial), spawn the Synthesis Agent:

```
Agent({
  description: "Synthesize deep research campaign: {seed}",
  subagent_type: "general-purpose",
  model: "opus",
  prompt: <synthesis prompt with domain tree + all specialist briefs>
})
```

Synthesis produces:
- Parent brief (context, decomposition, key findings, contradictions flagged, coverage
  assessment, related, metadata)
- Cross-references added to specialist briefs (`related: [{slug, relationship}]`)
- Explicit contradiction flags (never silently resolved)

### Phase 8: Evaluation

After synthesis, spawn the Evaluator Agent with **only the produced briefs + seed topic** —
do NOT include the domain tree, specialist prompts, or orchestrator reasoning. This isolation
prevents framing-bias inheritance.

```
Agent({
  description: "Evaluate deep research campaign: {seed}",
  subagent_type: "general-purpose",
  model: "opus",
  prompt: <evaluator prompt with briefs + seed only>
})
```

Evaluator produces a structured quality report (coverage, coherence, contradictions,
groundedness, recommendations).

### Phase 9: Write Output

1. Create `docs/briefs/<seed-slug>/` directory
2. Write specialist briefs (with synthesis's cross-references)
3. Write `parent.md` (synthesis output)
4. Write `campaign.md` (evaluator's report + campaign metadata)
5. Update knowledge index with entries for each brief

### Phase 10: Handoff

Report to the user (or calling skill):
- Campaign summary: N briefs produced, coverage score, cost, duration
- Contradictions flagged (if any)
- Recommendations (uncovered aspects → follow-up campaigns)
- Directory path for the user to review and promote

---

## Anti-Patterns

- **Don't research content yourself.** You orchestrate. If you find yourself searching the
  web in the Lead context, stop — delegate to a specialist.
- **Don't write the parent brief yourself.** Synthesis is a different role. Use the
  Synthesis Agent even if you "could just write it."
- **Don't let the evaluator see the decomposition rationale.** It must evaluate what was
  produced, not what you intended. Keep its context clean.
- **Don't silently resolve contradictions.** Surface them. The user decides.
- **Don't collapse specialist contexts.** Each specialist gets its own focused prompt —
  not a shared blackboard.
- **Don't pass full sibling outputs to specialists.** Pass titles and one-line descriptions
  only. Full outputs cause cross-contamination.
- **Don't spawn specialists on Opus.** At 5+ parallel × 150K tokens, Opus costs ~5× Sonnet
  for indistinguishable output. Use `model: "sonnet"`.
- **Don't skip the human review** unless explicitly told (`--no-review`). The decomposition
  is the most leveraged decision in the campaign.
- **Don't exceed the depth ceiling.** Depth > 4 is a decomposition problem, not a signal
  to go deeper. Rethink.
- **Don't decompose into subdomains that already have briefs.** The reuse check in Phase 1
  prevents this. Specialists should cite existing briefs, not duplicate them.
- **Don't accept "insufficient data" without investigating why.** A genuinely empty
  subdomain is rare. More often: the subdomain was described poorly, or the specialist
  got stuck. Investigate before marking complete.

---

## Tips

- **The decomposition is the most leveraged decision.** Spend real effort here. A bad tree
  wastes 5 specialists × 150K tokens × Sonnet rates = meaningful money on bad output.
- **Use the knowledge index first.** Every brief that already exists is a specialist you
  don't need to dispatch.
- **When in doubt on facet selection, try multi-sample.** Decompose 3 times, look at
  overlap. High overlap = stable decomposition. Low overlap = seed is poorly scoped.
- **Default to more parallel specialists, not deeper trees.** Breadth (more facets at
  shallower depth) usually produces better coverage than depth (fewer facets more
  deeply). The specialist is already doing depth within its subdomain.
- **Watch for specialist drift.** If a specialist's output is mostly about its siblings'
  subdomains, its own scope was wrong. Cancel and re-dispatch with a sharper prompt.
