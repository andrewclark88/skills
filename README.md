# skills

A complete software development workflow suite for Claude Code.

Full methodology: [`docs/build-process.md`](docs/build-process.md)

## The Pipeline

```
SESSION START (every non-fresh session)
│
└─ /knowledge-index → Load existing project knowledge before doing anything else.
                      Essential infrastructure — skip only on truly brand-new projects.

PROJECT START (truly new project)
│
/ideate           → Define the project. North star (vision, principles, domain model).
                    Identify domains that need research.
                    Auto-calls /scout for prior art discovery.

/scout            → Breadth-first prior art discovery. Adjacent projects, approaches,
                    patterns, lessons learned. Produces landscape brief + research
                    recommendations. Also callable standalone.

/research         → Research each domain deeply. Primers + auto-loading reference skills.
                    Repeat for every domain. Don't rush — assumptions cause rewrites.

/architecture     → Design the system. Modules, data flow, conventions.
                    Grounded in north star + domain briefs. No unresearched assumptions.

/roadmap          → Plan the build. Phases with blocking briefs, I/O/Tests, dependencies.
                    Each phase scoped for one session.

┌── PER PHASE ──────────────────────────────────────────────────────┐
│                                                                   │
│  /brief          → Write the blocking brief (if one is listed).   │
│  /update-roadmap → Revisit phases after building (scope shifts,   │
│                    new blockers, splits/merges). Run every 2-4     │
│                    phases or when the plan no longer fits reality.  │
│  /design         → Detailed implementation spec (interfaces,      │
│                    types, file paths, acceptance criteria).        │
│  /implement      → Build it. Code + tests. (Or /implement-        │
│                    orchestrator for large phases.)                 │
│  PR + CI         → Branch → PR → CI green → merge.               │
│  /update-documentation → Sync docs to what was built.             │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘

┌── EVERY 2-4 PHASES ──────────────────────────────────────────────┐
│  /doc-review        → Audit planning docs for consistency.        │
│  /refactor-design   → Find duplication, missing abstractions.     │
│  /extract-patterns  → Document reusable code patterns.            │
│  /test-quality      → Spec-driven test gap analysis.              │
└───────────────────────────────────────────────────────────────────┘

┌── PRE-DEPLOY ────────────────────────────────────────────────────┐
│  /security-review   → Comprehensive audit. Address Critical/High. │
└───────────────────────────────────────────────────────────────────┘

┌── AS NEEDED ─────────────────────────────────────────────────────┐
│  /cruft-cleaner     → Remove dead code, AI bloat.                 │
│  /bold-refactor     → Find architectural simplifications.         │
│  /feature           → Quick extension outside core roadmap.       │
│  /expand            → Scope expansion for subsystems.             │
│  /repo-eval         → Multi-dimensional codebase scoring.         │
│  /e2e-test-design   → End-to-end test suite design.               │
│  /release           → Changelog + release.                        │
└───────────────────────────────────────────────────────────────────┘
```

## Knowledge Index

The knowledge index (`docs/knowledge-index.yaml` per project) is **essential infrastructure, not optional polish**. It tracks every brief, architecture doc, roadmap, and research finding so future sessions never start blind.

- **`/knowledge-index`** — runs at the start of every non-fresh session. Skip only on truly brand-new projects.
- All doc-producing skills (`/ideate`, `/research`, `/architecture`, `/roadmap`, `/brief`, `/refactor-design`, `/feature`, `/extract-patterns`, `/expand`) update the index after writing.
- All doc-consuming skills (`/design`, `/implement`, `/implement-orchestrator`, `/update-documentation`, `/doc-review`) check the index FIRST before doing work.
- All audit/review skills (`/security-review`, `/repo-eval`, `/bold-refactor`, `/test-quality`, `/cruft-cleaner`, `/e2e-test-design`) load the index for architectural context.

**Doc frontmatter convention:** Every indexable planning doc declares `description`, `type`, `updated` in YAML frontmatter at the top. See `knowledge-index/SKILL.md` → "Indexable Doc Frontmatter Convention" for the spec.

## How to Use

1. **Start a new conversation.** Describe your idea or task.
2. **Run `/knowledge-index`** to see what context already exists for this project.
3. **Type the slash command** for the current step (e.g., `/ideate` for a new project, or whatever the pipeline says is next).
4. **The skill guides you.** It asks questions, researches, produces docs.
5. **You decide when to advance.** Each skill has checkpoints. Say "this is solid" to move on, or "let's research more" to go deeper.
6. **Repeat** through the pipeline until the project ships.

## Skills

All skills are first-party. Install/run from the top-level skill directories.

### Pipeline & Planning

| Skill | Step | What it produces |
|-------|------|-----------------|
| `/knowledge-index` | Session start | Loads all available project knowledge |
| `/ideate` | 1. Define | North star + research plan. Auto-calls `/scout`. |
| `/scout` | 1.5 Scout | Landscape brief + research recommendations |
| `/research` | 2. Research | Brief + auto-loading reference skill. Updates knowledge index. |
| `/architecture` | 3. Design | Architecture doc (modules, data flow, conventions) |
| `/roadmap` | 4. Plan | Phased roadmap (blocking briefs, I/O/Tests) |

### Per Phase

| Skill | Step | What it produces |
|-------|------|-----------------|
| `/brief` | 5. | Curated domain brief for the builder |
| `/update-roadmap` | After building phases | Revised roadmap reflecting what was learned |
| `/design` | 6. | Implementation spec (interfaces, types, acceptance criteria) |
| `/implement` | 7. | Code + tests (<20 files) |
| `/implement-orchestrator` | 7. | Code + tests (parallel agents, 20+ files) |
| `/update-documentation` | 9. | Docs synced to code changes; frontmatter verified |

### Quality Checkpoints

| Skill | When | What it produces |
|-------|------|-----------------|
| `/doc-review` | Every 2-4 phases + after design changes | Planning doc consistency report (cascading: system → modules) |
| `/refactor-design` | Every 2-4 phases | Refactor plan |
| `/extract-patterns` | Every 2-4 phases | Pattern documentation |
| `/test-quality` | Every 2-4 phases | Spec-driven tests |

### Pre-Deploy

| Skill | What it produces |
|-------|-----------------|
| `/security-review` | Scored security report — address Critical/High before deploying |

### As-Needed

| Skill | When | What it produces |
|-------|------|-----------------|
| `/cruft-cleaner` | Codebase has accumulated AI bloat | Dead-code/comment removal |
| `/bold-refactor` | Architectural simplification opportunity | Refactor proposal |
| `/feature` | Quick extension outside the roadmap | Feature brief |
| `/expand` | Scope expansion for a subsystem | Expansion doc |
| `/repo-eval` | Want a multi-dimensional view | Codebase scorecard |
| `/e2e-test-design` | Building test coverage | E2E test design |
| `/scout` | Need to know what exists before deciding | Landscape brief + research recommendations |
| `/release` | Cutting a release | Changelog + release notes |

### Auto-Loading Principles

| Principle | Enforces |
|-----------|----------|
| `build-process` | The methodology. Always loaded. |
| `engineering-principles` | Ports & Adapters, Single Source of Truth, Generated Contracts, Fail Fast. Design-time and implementation-time in one skill. |

### Thinking Layer

| Doc | What it does |
|-----|-------------|
| [`docs/first-principles.md`](docs/first-principles.md) | 10 thinking moves for deep reasoning. Loaded by `/research`, `/ideate`, `/architecture`, `/brief`, `/roadmap`. |

### Knowledge Patterns

Cross-project patterns for building knowledge systems. Used by any project with a knowledge layer.

| Doc | What it covers |
|-----|---------------|
| [`docs/knowledge-layer-overview.md`](docs/knowledge-layer-overview.md) | Overview — storage, retrieval, generation as three concerns |
| [`docs/knowledge-storage-pattern.md`](docs/knowledge-storage-pattern.md) | Frontmatter schema, typed cross-references, knowledge graph, backends |
| [`docs/knowledge-retrieval-pattern.md`](docs/knowledge-retrieval-pattern.md) | Tiered retrieval (L0-L3), metadata-first filtering, search ladder (v1-v3) |
| [`docs/knowledge-generation-pattern.md`](docs/knowledge-generation-pattern.md) | Ingest, refresh, lint, LLM enrichment, provenance, lifecycle |

## Key Rules

1. **Load the knowledge index first.** Every non-fresh session starts with `/knowledge-index`.
2. **Research before design.** Don't make architecture decisions based on assumptions.
3. **Each doc has one job.** North star = vision. Architecture = how. Roadmap = phases. No duplication.
4. **Every phase ships with tests.** A phase is done when its PR merges with CI green.
5. **Never `terraform apply` locally.** CI only.
6. **Never commit secrets.**
7. **Quality checkpoint every 2-4 phases.** Refactor + patterns + test gaps.
8. **Security review before deploy.** No Critical/High findings.

## Installation

```bash
# Copy all skills to your personal skills folder
for d in knowledge-index ideate scout research architecture roadmap brief doc-review \
         update-roadmap design implement implement-orchestrator \
         update-documentation refactor-design extract-patterns test-quality \
         security-review cruft-cleaner bold-refactor feature expand \
         repo-eval e2e-test-design release engineering-principles; do
  cp -r "$d" ~/.claude/skills/
done
cp -r principles/build-process ~/.claude/skills/
```

## Acknowledgments

A handful of these skills were originally inspired by the open suite from [nklisch/skills](https://github.com/nklisch/skills). Our versions have evolved independently — most notably with knowledge-index integration baked in across the suite — and we maintain them as our own. To compare against upstream, clone fresh from the source repo and diff against our top-level skills.
