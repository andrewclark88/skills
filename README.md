# skills

**A complete, opinionated software development workflow suite for Claude Code — 27 skills covering the full lifecycle, two auto-loading principle packs, three foundational primers (thinking, design, lateral), a model-selection framework, a three-scale research family, and cross-project knowledge patterns.**

Use this when you want agents that ship real projects: ideation → research → architecture → roadmap → per-phase build, with doc-review, refactor, security, and release baked in. Every step is a skill. Every skill is opinionated about what it produces. Every doc is indexed so future sessions never start blind.

Full methodology: [`docs/build-process.md`](docs/build-process.md)

---

## What's In Here

| Layer | What it is | Where to look |
|-------|-----------|---------------|
| **27 skills** | Full software lifecycle — 19 directly invocable as slash commands, 8 invoked programmatically from other skills | Top-level directories (`ideate/`, `research/`, etc.) |
| **Project template** | Canonical scaffold dropped into new projects — `docs/` folders, `knowledge-index.yaml`, lean `CLAUDE.md`, portable `.claude/rules` | [`templates/project/`](templates/project/) (applied by `/init-project`) |
| **Research skills family** | Three scales of the same fractal pattern: `/research` (question) → `/deep-research` (domain) → `/research-program` (megatopic) | [`docs/research-skills-overview.md`](docs/research-skills-overview.md) |
| **Thinking layer** | First-principles primer loaded by thinking-heavy skills | [`docs/first-principles.md`](docs/first-principles.md) |
| **System design layer** | 15 design moves loaded by design-heavy and refactor skills | [`docs/system-design.md`](docs/system-design.md) |
| **Oblique strategies** | 10 lateral thinking moves loaded when ideation gets stuck | [`docs/oblique-strategies.md`](docs/oblique-strategies.md) |
| **Model selection** | Four-archetype framework for picking Opus/Sonnet/Haiku + effort per role | [`docs/model-selection-pattern.md`](docs/model-selection-pattern.md) |
| **Knowledge patterns** | Cross-project architectural patterns for building knowledge layers | [`docs/knowledge-layer-overview.md`](docs/knowledge-layer-overview.md) and three pattern docs |
| **Build process** | The full methodology that ties skills, gates, and infrastructure safety together | [`docs/build-process.md`](docs/build-process.md) |

---

## The Pipeline

```
SESSION START (every non-fresh session)
│
└─ /knowledge-index → Load existing project knowledge before anything else.
                      Essential infrastructure. Skip only on truly brand-new projects.

PROJECT START (truly new project)
│
/init-project     → Scaffold the knowledge layer. Drops the canonical template
                    (docs/ folders, knowledge-index.yaml, lean CLAUDE.md,
                    portable .claude/rules) into an empty project directory.
                    Run once per new project, before /ideate.

/ideate           → Define the project. North star (vision, principles, domain model).
                    Identify domains that need research.
                    Auto-calls /scout for prior art discovery.

/scout            → Breadth-first prior art discovery. Adjacent projects, approaches,
                    patterns, lessons. Produces landscape brief + research recommendations.
                    Auto-called from /ideate; also callable standalone.

┌── RESEARCH FAMILY (three scales, same fractal pattern) ──────────┐
│                                                                   │
│  /research         → Question scale. Single-agent with 3-5 Sonnet │
│                      sub-agents. One domain, clear questions.     │
│                      Cost ~$3-5.                                  │
│                                                                   │
│  /deep-research    → Domain scale. Lead + 3-7 Sonnet specialists  │
│                      (parallel, isolated) + spawned Opus Synthesis│
│                      + spawned Opus Evaluator. Topic spans 5+     │
│                      orthogonal facets. Cost ~$6-15.              │
│                      `--continue-from` for chain mode (extending  │
│                      a leaf of a prior campaign).                 │
│                                                                   │
│  /research-program → Megatopic scale. Planner + 3-7 Campaigns     │
│                      (each a full /deep-research tree) +          │
│                      Cross-Campaign Synthesizer + Program         │
│                      Evaluator. Topic spans multiple distinct     │
│                      domains. Cost ~$35-75.                       │
│                                                                   │
│  Escalation (user-gated): /research → /deep-research →            │
│  /research-program. All three produce briefs the rest of the      │
│  pipeline consumes. Reuse check cites existing work; isolation    │
│  prevents framing bias at every scale.                            │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘

/architecture     → Design the system. Modules, data flow, conventions.
                    Grounded in north star + domain briefs. No unresearched assumptions.

/roadmap          → Plan the build. Phases with blocking briefs, I/O/Tests, dependencies.
                    Each phase scoped for one session.

┌── PER PHASE ──────────────────────────────────────────────────────┐
│                                                                   │
│  /brief          → Write the blocking brief (if one is listed).   │
│  /design         → Detailed implementation spec (interfaces,      │
│                    types, file paths, acceptance criteria).        │
│  /implement      → Build it. Code + tests. (Or /implement-        │
│                    orchestrator for large phases — Opus           │
│                    orchestrator + parallel Sonnet workers.)        │
│  PR + CI         → Branch → PR → CI green → merge.               │
│  /update-documentation → Sync docs to what was built.             │
│  /update-roadmap → Revisit phases after building (scope shifts,   │
│                    new blockers, splits/merges). Every 2-4 phases │
│                    or when the plan no longer fits reality.       │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘

┌── EVERY 2-4 PHASES (quality checkpoint) ─────────────────────────┐
│  /doc-review        → Audit planning docs for consistency         │
│                        (cascading: system → modules).              │
│  /refactor-design   → Find duplication, missing abstractions.     │
│  /extract-patterns  → Document reusable code patterns.            │
│  /test-quality      → Spec-driven test gap analysis.              │
└───────────────────────────────────────────────────────────────────┘

┌── PRE-DEPLOY ────────────────────────────────────────────────────┐
│  /security-review   → Comprehensive audit. Address Critical/High. │
└───────────────────────────────────────────────────────────────────┘

┌── AS NEEDED ─────────────────────────────────────────────────────┐
│  /cruft-cleaner     → Remove dead code, AI-accumulated bloat.    │
│  /bold-refactor     → Find architectural simplifications.         │
│  /feature           → Quick extension outside core roadmap.       │
│  /expand            → Scope expansion for subsystems.             │
│  /repo-eval         → Multi-dimensional codebase scoring.         │
│  /e2e-test-design   → End-to-end test suite design.               │
│  /release           → Changelog + release notes.                  │
└───────────────────────────────────────────────────────────────────┘
```

---

## What's Special

### 1. The knowledge index ties everything together

Every project has a `docs/knowledge-index.yaml` that catalogs every brief, architecture doc, roadmap, and research finding. Every session starts by loading it. No session ever flies blind.

**Doc-producing skills** update the index automatically. **Doc-consuming skills** check it first. The indexed frontmatter convention (`description`, `type`, `updated`, `research_method`) is standardized across all skills — `research_method` is auto-stamped by `/brief`, `/research`, `/deep-research`, and `/research-program`, and `/doc-review` audits the corpus by tool of origin to surface implicitly deprecated briefs. See [`docs/build-process.md#knowledge-index`](docs/build-process.md).

### 2. First-principles thinking layer

Thinking-heavy skills load a concrete thinking primer before starting: 10 moves organized as **Open → Challenge → Synthesize → Verify**. Each skill declares which moves to emphasize.

Loaded by: `/research`, `/deep-research`, `/research-program`, `/ideate`, `/architecture`, `/brief`, `/roadmap`.

See [`docs/first-principles.md`](docs/first-principles.md).

### 3. Oblique strategies (lateral thinking)

When structured thinking gets stuck — circular ideation, local optima, over-constrained problems — oblique strategies provide lateral moves: 10 techniques across **Reframe → Constrain → Stimulate**. Inspired by Brian Eno, Edward de Bono, TRIZ, and design thinking.

The key: use lateral moves to FIND new directions, then switch back to first-principles to EVALUATE them. The two primers work as a pair.

Loaded by: `/ideate` (primary), `/architecture` (when options seem equal), `/expand` (when scope feels forced).

See [`docs/oblique-strategies.md`](docs/oblique-strategies.md).

### 4. System design layer

Design-heavy skills and design-sensitive refactor/expansion skills load a system design primer before starting: 15 moves organized across **Structure → Interfaces → Data → Scale → Reliability**. The unifying principle is "earn your complexity" — 12 moves are design-in (cheap now, expensive to retrofit), 3 are earn-in (add only with measured evidence). Each skill declares which moves to emphasize.

| Skill | Emphasis | Why |
|-------|----------|-----|
| `/architecture` | Structure + Interfaces | Highest-leverage decisions that cascade |
| `/design` | Interfaces + Data | Bridges architecture to implementation |
| `/implement` | Data + Scale + Reliability | Where design meets reality |
| `/brief` | All (when topic is system design) | Curate toward the decisions builders face |
| `/refactor-design` | Interfaces + Structure | Surfacing the abstractions that *should* exist |
| `/bold-refactor` | Structure | Collapsing premature boundaries, undoing irreversible decisions |
| `/feature` | Interfaces + Minimize Irreversible | Extend contracts additively, avoid premature lock-in |
| `/expand` | Structure + Contracts | Defining new boundaries and their contracts before building |

Loaded by: `/architecture`, `/design`, `/implement`, `/brief`, `/refactor-design`, `/bold-refactor`, `/feature`, `/expand`.

See [`docs/system-design.md`](docs/system-design.md).

### 5. Model-selection framework

Claude has two levers — **model** (Opus/Sonnet/Haiku) and **effort** (thinking budget). Used well, they save 3-5× cost with no quality loss. Used poorly, they waste money on easy tasks and shortchange hard ones.

The pattern doc defines four archetypes — every role inside every skill maps to one:

| Archetype | Model + Effort | Typical role |
|-----------|---------------|-------------|
| **Orchestration** | Opus high | Few calls, cascading consequences (`/architecture`, `/design`, `/deep-research` Lead, `/research-program` Planner) |
| **Parallel worker** | Sonnet medium | Many parallel, scoped (research sub-agents, deep-research specialists). At program scale, campaigns themselves are the workers — each a full Opus+Sonnet tree. |
| **Synthesis / judgment** | Opus high | One call, integrates N inputs (synthesis agents, `/brief`, cross-campaign synthesizers + evaluators) |
| **Volume / structured extraction** | Haiku low | Many calls, well-defined task (entity extraction, citation verification) |

Every SKILL.md that spawns sub-agents declares its archetype mapping in a **Model Assignment** section. See [`docs/model-selection-pattern.md`](docs/model-selection-pattern.md).

### 6. The Research Skills Family — three scales of the same fractal pattern

The research tools (`/research`, `/deep-research`, `/research-program`) are a single architectural pattern applied at three scales. Every scale has the same four roles: an **Orchestrator** that decomposes and dispatches, **Workers** that investigate in parallel with isolated contexts, a **Synthesizer** that reconciles their outputs, and an **Evaluator** that judges quality independently. What changes across scales is what each role orchestrates and how much is delegated.

| Scale | Skill | Orchestrator | Workers | Unit of output | Cost (calibrated) |
|-------|-------|-------------|---------|---------------|-------------------|
| Question | `/research` | Researcher (Opus) | 3-5 Sonnet sub-agents | One domain brief + reference skill | ~$3-5 |
| Domain | `/deep-research` | Lead (Opus) | 5-7 Sonnet specialists | Campaign dir (parent + N briefs + quality report) | ~$6 scoped, ~$12-15 default |
| Megatopic | `/research-program` | Planner (Opus) | 3-7 Campaigns (full `/deep-research` runs) | Program dir (plan + super-parent + N campaigns + program evaluation) | ~$35-75 |

**Isolation is the load-bearing principle.** Workers never see each other's full output (only sibling titles). Synthesizer sees everything. Evaluator sees only the produced briefs + seed — not the orchestrator's reasoning, not the decomposition tree. This prevents framing-bias inheritance: the Evaluator judges *what was produced*, not *what was intended*. The same rule applies at every scale.

**Composition:** `/research` escalates to `/deep-research` when topic is broad; `/deep-research` escalates to `/research-program` when seed is megatopic. `/deep-research --continue-from` extends a leaf of a prior campaign via chain mode. All three share reuse checks — existing briefs and campaigns are cited rather than re-run.

**Demo-validated (2026-04-15/16):** 4 `/deep-research` campaigns (coverage 0.83-0.92, coherence 4.0-4.5/5) + 1 `/research-program` (5 campaigns, coverage 0.86, coherence 4.2/5). Key findings: severity reassessment happens in ~50% of campaigns; child synthesis mis-reports resolution status ~50% of the time in the direction of optimism (hence "Evaluator verifies, doesn't trust" as a core principle). **Known limitation:** nested agent spawning is blocked — `/research-program` campaign Leads run as single-agent research rather than the designed four-role tree. Quality is unaffected; parallelism benefits don't apply.

Grounded in Anthropic's multi-agent research pattern: 90% time reduction, 90.2% quality improvement over single-agent Opus on internal research eval.

See [`docs/research-skills-overview.md`](docs/research-skills-overview.md) (family view) · [`docs/deep-research-north-star.md`](docs/deep-research-north-star.md) + [`docs/deep-research-architecture.md`](docs/deep-research-architecture.md) · [`docs/research-program-architecture.md`](docs/research-program-architecture.md).

### 7. Cross-project knowledge patterns

Four pattern docs abstract the knowledge-layer design used by any project with a knowledge base (a cloud-native MCP server, a local simulation engine, whatever). Storage/retrieval/generation are three independent concerns; adopt them in any order.

| Pattern | What it covers |
|---------|---------------|
| [Overview](docs/knowledge-layer-overview.md) | The LLM Wiki pattern; three concerns; scale ladder |
| [Storage](docs/knowledge-storage-pattern.md) | Frontmatter schema, typed cross-references, knowledge graph, backends |
| [Retrieval](docs/knowledge-retrieval-pattern.md) | Tiered retrieval (L0-L3), metadata-first filtering, search ladder (v1-v3) |
| [Generation](docs/knowledge-generation-pattern.md) | Ingest, refresh, lint, LLM enrichment, provenance, lifecycle |

### 8. Quality gates, not ceremony

- **`/doc-review`** catches cross-doc drift with cascading passes (system-level → per-module). Triggered after `/architecture`, after `/roadmap`, at quality checkpoints, and when `/update-documentation` changes planning docs.
- **`/security-review`** produces a scored report before deploy. Address Critical/High first.
- **Every phase ships tests.** Phase is done when its PR merges with CI green.
- **Infrastructure safety is non-negotiable.** `terraform apply` runs in CI only. Resources are prefixed. State is remote. `prevent_destroy` on critical resources. See [`docs/build-process.md#infrastructure-safety-practices`](docs/build-process.md).

---

## How to Use

1. **Start a new conversation.** Describe your idea or task.
2. **Run `/knowledge-index`** to see what context already exists for this project.
3. **Type the slash command** for the current step (e.g., `/ideate` for a new project).
4. **The skill guides you.** It asks questions, researches, produces docs.
5. **You decide when to advance.** Each skill has checkpoints. Say "this is solid" to move on, or "let's research more" to go deeper.
6. **Repeat** through the pipeline until the project ships.

---

## Skills Catalog

All skills are first-party. Invocable as slash commands once installed.

### Pipeline & Planning

| Skill | Step | What it produces |
|-------|------|-----------------|
| [`/knowledge-index`](knowledge-index/SKILL.md) | Session start | Catalog of all available project knowledge |
| [`/init-project`](init-project/SKILL.md) | 0. Scaffold | Drops the canonical template (`docs/` folders, `knowledge-index.yaml`, lean `CLAUDE.md`, portable `.claude/rules`) into a new project directory |
| [`/ideate`](ideate/SKILL.md) | 1. Define | North star + research plan. Auto-calls `/scout`. Classifies domains as `/research` vs `/deep-research`. |
| [`/scout`](scout/SKILL.md) | 1.5 Scout | Landscape brief + research recommendations (Opus orchestrator + Sonnet workers) |
| [`/research`](research/SKILL.md) | 2. Research (question scale) | Domain brief + auto-loading reference skill. Updates knowledge index. |
| [`/deep-research`](deep-research/SKILL.md) | 2. Research (domain scale) | N cross-referenced briefs + parent synthesis + quality report. Parallel-agent campaign (Lead + Specialists + Synthesis + Evaluator). `--continue-from` for chain mode. |
| [`/research-program`](research-program/SKILL.md) | 2. Research (megatopic scale) | Program directory: plan + super-parent + N campaigns (each a full `/deep-research` tree) + program evaluation. For initiatives spanning multiple distinct domains. |
| [`/architecture`](architecture/SKILL.md) | 3. Design | Architecture doc (modules, data flow, conventions) |
| [`/roadmap`](roadmap/SKILL.md) | 4. Plan | Phased roadmap (blocking briefs, I/O/Tests) |

### Per Phase

| Skill | Step | What it produces |
|-------|------|-----------------|
| [`/brief`](brief/SKILL.md) | 5. | Curated domain brief for the builder |
| [`/design`](design/SKILL.md) | 6. | Implementation spec (interfaces, types, acceptance criteria) |
| [`/implement`](implement/SKILL.md) | 7. | Code + tests (<20 files) |
| [`/implement-orchestrator`](implement-orchestrator/SKILL.md) | 7. | Code + tests (Opus orchestrator + parallel Sonnet workers, 20+ files) |
| [`/update-documentation`](update-documentation/SKILL.md) | 9. | Docs synced to code changes; frontmatter verified. Triggers `/doc-review` if planning docs changed. |
| [`/update-roadmap`](update-roadmap/SKILL.md) | After phases | Revised roadmap reflecting what was learned |

### Quality Checkpoints (every 2-4 phases)

| Skill | What it produces |
|-------|-----------------|
| [`/doc-review`](doc-review/SKILL.md) | Planning doc consistency report (cascading: system → modules); auto-fix loop iterates Critical/High until clean (cap: 5 iterations) |
| [`/refactor-design`](refactor-design/SKILL.md) | Refactor plan |
| [`/extract-patterns`](extract-patterns/SKILL.md) | Reusable code pattern documentation |
| [`/test-quality`](test-quality/SKILL.md) | Spec-driven tests (gap analysis) |

### Pre-Deploy

| Skill | What it produces |
|-------|-----------------|
| [`/security-review`](security-review/SKILL.md) | Scored security report — address Critical/High before deploying |

### As-Needed

| Skill | When | What it produces |
|-------|------|-----------------|
| [`/cruft-cleaner`](cruft-cleaner/SKILL.md) | Codebase has accumulated AI bloat | Dead-code / stale-comment removal with parallel cleanup agents |
| [`/bold-refactor`](bold-refactor/SKILL.md) | Architectural simplification opportunity | Refactor proposal |
| [`/feature`](feature/SKILL.md) | Quick extension outside the roadmap | Feature brief |
| [`/expand`](expand/SKILL.md) | Scope expansion for a subsystem | Expansion doc |
| [`/repo-eval`](repo-eval/SKILL.md) | Want a multi-dimensional codebase view | Scorecard across 9 dimensions |
| [`/e2e-test-design`](e2e-test-design/SKILL.md) | Designing E2E test coverage | Golden-path + adversarial test design |
| [`/release`](release/SKILL.md) | Cutting a release | Changelog + release notes |

### Auto-Loading Principles (not slash-invocable)

| Principle | What it enforces | When loaded |
|-----------|-----------------|-------------|
| [`build-process`](build-process/) | The full methodology | Always loaded |
| [`engineering-principles`](engineering-principles/) | Ports & Adapters, Single Source of Truth, Generated Contracts, Fail Fast | Loaded by `/architecture`, `/design`, `/implement`, `/refactor-design` |

---

## Cross-Cutting Docs

These are not skills — they're patterns and primers that skills reference.

| Doc | What it covers | Who uses it |
|-----|---------------|-------------|
| [`docs/first-principles.md`](docs/first-principles.md) | 10 thinking moves (Open / Challenge / Synthesize / Verify) + per-skill emphasis table | `/research`, `/deep-research`, `/research-program`, `/ideate`, `/architecture`, `/brief`, `/roadmap` |
| [`docs/oblique-strategies.md`](docs/oblique-strategies.md) | 10 lateral thinking moves (Reframe / Constrain / Stimulate) for when structured approaches get stuck | `/ideate`, `/architecture`, `/expand` |
| [`docs/system-design.md`](docs/system-design.md) | 15 design moves across 5 concerns (Structure / Interfaces / Data / Scale / Reliability) + per-skill emphasis table | `/architecture`, `/design`, `/implement`, `/brief`, `/refactor-design`, `/bold-refactor`, `/feature`, `/expand` |
| [`docs/model-selection-pattern.md`](docs/model-selection-pattern.md) | Four archetypes (orchestration, parallel-worker, synthesis, volume-extraction) with model + effort recommendations | Every skill that spawns sub-agents |
| [`docs/knowledge-layer-overview.md`](docs/knowledge-layer-overview.md) + 3 pattern docs | Cross-project knowledge-layer architecture (storage, retrieval, generation) | Any project building a knowledge layer |
| [`docs/build-process.md`](docs/build-process.md) | Full methodology: pipeline, doc ownership, PR/CI gates, infrastructure safety | Always loaded |

### Skill-level architecture docs

Some skills have their own north-star + architecture docs for deeper reference:

| Skill | Docs |
|-------|------|
| `/scout` | [north-star](docs/scout-north-star.md) + [architecture](docs/scout-architecture.md) |
| Research family (overview) | [overview](docs/research-skills-overview.md) — unifying view of `/research` + `/deep-research` + `/research-program` |
| `/deep-research` | [north-star](docs/deep-research-north-star.md) + [architecture](docs/deep-research-architecture.md) |
| `/research-program` | [architecture](docs/research-program-architecture.md) |
| first-principles primer | [north-star](docs/first-principles-north-star.md) + [architecture](docs/first-principles-architecture.md) + [research brief](docs/brief-first-principles-frameworks.md) |
| knowledge-edge creator (planned) | [north-star](docs/knowledge-edge-north-star.md) |

---

## Key Rules

1. **Load the knowledge index first.** Every non-fresh session starts with `/knowledge-index`.
2. **Research before design.** Don't make architecture decisions based on assumptions.
3. **Each planning doc has one job.** North star = vision. Architecture = how. Roadmap = phases. No duplication.
4. **Every phase ships with tests.** A phase is done when its PR merges with CI green.
5. **Never `terraform apply` locally.** CI only, on merge to main.
6. **Never commit secrets.**
7. **Quality checkpoint every 2-4 phases.** Refactor + patterns + test gaps + doc-review.
8. **Security review before deploy.** No Critical/High findings.
9. **Every SKILL.md declares its Model Assignment.** Explicit model choice per role — don't default to Opus everywhere.
10. **Thinking-heavy skills load `first-principles.md` before starting.** Shallow thinking propagates downstream.
11. **Design-heavy skills load `system-design.md` before starting.** 15 moves across Structure, Interfaces, Data, Scale, Reliability. Earn your complexity.

---

## Installation

**Recommended — symlink the repo as your skills folder.** Zero drift, edits visible immediately:

```bash
git clone https://github.com/andrewclark88/skills.git ~/dev/skills
ln -s ~/dev/skills ~/.claude/skills
```

Claude Code discovers every top-level directory with a `SKILL.md` and ignores `README.md`, `docs/`, and other siblings. Pull to update.

**Alternative — copy skills individually.** Use this if you don't want the whole repo under `~/.claude/skills`:

```bash
git clone https://github.com/andrewclark88/skills.git
cd skills
for d in */; do
  [ -f "$d/SKILL.md" ] && cp -r "$d" ~/.claude/skills/
done
```

The cross-cutting docs under `docs/` are referenced by absolute path from SKILL.md files (e.g., `/dev/skills/docs/first-principles.md`). Keep this repo checked out at a stable path so those references resolve.

---

## Repo Layout

```
skills/
├── README.md                     ← you are here
├── docs/                         ← cross-cutting patterns + primers
│   ├── build-process.md          ← the methodology (source of truth)
│   ├── first-principles.md       ← thinking layer primer (10 moves)
│   ├── oblique-strategies.md     ← lateral thinking primer (10 moves)
│   ├── system-design.md          ← system design primer (15 moves)
│   ├── model-selection-pattern.md
│   ├── knowledge-layer-overview.md
│   ├── knowledge-storage-pattern.md
│   ├── knowledge-retrieval-pattern.md
│   ├── knowledge-generation-pattern.md
│   ├── research-skills-overview.md        ← three-scale research family
│   ├── scout-north-star.md + scout-architecture.md
│   ├── deep-research-north-star.md + deep-research-architecture.md
│   ├── research-program-architecture.md   ← megatopic-scale research
│   ├── first-principles-north-star.md + first-principles-architecture.md
│   ├── brief-first-principles-frameworks.md
│   └── knowledge-edge-north-star.md
├── knowledge-index/              ← skills (each has SKILL.md)
├── ideate/
├── scout/
├── research/
├── deep-research/                ← parallel-agent campaigns (+ chain mode)
├── research-program/             ← multi-campaign megatopic research
├── architecture/
├── roadmap/
├── brief/
├── update-roadmap/
├── design/
├── implement/
├── implement-orchestrator/
├── update-documentation/
├── doc-review/
├── refactor-design/
├── extract-patterns/
├── test-quality/
├── security-review/
├── cruft-cleaner/
├── bold-refactor/
├── feature/
├── expand/
├── repo-eval/
├── e2e-test-design/
├── release/
├── engineering-principles/       ← auto-loaded design + code principles
└── principles/
    └── build-process/            ← auto-loaded methodology skill
```

---

## Design Philosophy

- **Opinionated over flexible.** Each skill has a single job and strong defaults. Less configuration, more convergence.
- **Ship through PRs.** Nothing merges without CI. Every phase done when the PR merges, not when it "works locally."
- **Deep knowledge over shallow retrieval.** Briefs are compiled knowledge, not chunked raw docs. The knowledge layer uses the LLM Wiki pattern.
- **Good results over token savings.** Defaults favor quality; fences exist but start generous. Model selection saves cost where quality isn't impacted, not where it is.
- **Data-driven over hand-curated.** When there's a data source, build a pipeline.
- **Explicit over implicit.** Every sub-agent dispatch names its model. Every doc has frontmatter. Every phase has acceptance criteria.

---

## Acknowledgments

A handful of the pipeline skills were originally inspired by the open suite from [nklisch/skills](https://github.com/nklisch/skills). This suite has evolved substantially since then — knowledge-index integration across all skills, the thinking layer, the model-selection framework, the three-scale research family (`/research`, `/deep-research`, `/research-program`), `/scout`, the cross-project knowledge patterns, cascading `/doc-review`, and the full infrastructure-safety methodology are first-party additions. To compare against upstream, clone fresh from the source repo and diff.
