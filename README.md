# skills

A complete software development workflow suite for Claude Code. Custom methodology combined with adopted skills from [nklisch/skills](https://github.com/nklisch/skills).

Full methodology: [`docs/build-process.md`](docs/build-process.md)

## The Pipeline

```
/ideate           → Define the project. North star (vision, principles, domain model).
                    Identify domains that need research.

/research         → Research each domain deeply. Primers + auto-loading reference skills.
                    Repeat for every domain. Don't rush — assumptions cause rewrites.

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
│  /release           → Changelog + release.                        │
└───────────────────────────────────────────────────────────────────┘
```

## Knowledge Index

Every project accumulates knowledge — briefs, architecture docs, research findings. The knowledge index tracks all of it so you never lose context between sessions.

- **`/knowledge-index`** — run at session start to see all available project knowledge
- All doc-producing skills (`/ideate`, `/research`, `/architecture`, `/roadmap`, `/brief`) update `docs/knowledge-index.yaml` automatically after writing docs
- Before researching, check if a brief already exists. Don't duplicate work.

## How to Use

1. **Start a new conversation.** Describe your idea.
2. **Run `/knowledge-index`** to see what context already exists for this project.
3. **Type the slash command** for the current step (e.g., `/ideate`).
3. **The skill guides you.** It asks questions, researches, produces docs.
4. **You decide when to advance.** Each skill has checkpoints. Say "this is solid" to move on, or "let's research more" to go deeper.
5. **Repeat** through the pipeline until the project ships.

## Skills

### Custom (this repo)

| Skill | Step | What it produces |
|-------|------|-----------------|
| `/knowledge-index` | Session start | Shows all available project knowledge |
| `/ideate` | 1. Define | North star + research plan |
| `/research` | 2. Research | Primer doc + auto-loading reference skill. Updates knowledge index. |
| `/architecture` | 3. Design | Architecture doc (modules, data flow, conventions) |
| `/roadmap` | 4. Plan | Phased roadmap (blocking briefs, I/O/Tests) |
| `/brief` | 5. Per phase | Curated domain brief for the builder |
| `/doc-review` | Quality checkpoint + after design changes | Planning doc consistency report |

### Adopted (from [nklisch/skills](https://github.com/nklisch/skills))

| Skill | Step | What it produces |
|-------|------|-----------------|
| `/design` | 6. Per phase | Implementation spec (interfaces, types, acceptance criteria) |
| `/implement` | 7. Per phase | Code + tests (<20 files) |
| `/implement-orchestrator` | 7. Per phase | Code + tests (parallel agents, 20+ files) |
| `/update-documentation` | 9. Per phase | Docs synced to code changes |
| `/refactor-design` | Quality checkpoint | Refactor plan |
| `/extract-patterns` | Quality checkpoint | Pattern documentation |
| `/test-quality` | Quality checkpoint | Spec-driven tests |
| `/security-review` | Pre-deploy | Scored security report |
| `/cruft-cleaner` | As needed | Dead code removal |
| `/bold-refactor` | As needed | Architectural simplification |
| `/e2e-test-design` | As needed | End-to-end test suite design |
| `/feature` | As needed | Quick extension brief |
| `/expand` | As needed | Scope expansion |
| `/release` | As needed | Changelog + release |
| `/repo-eval` | As needed | Codebase scoring |

### Auto-Loading Principles

| Principle | Enforces |
|-----------|----------|
| `build-process` | The methodology. Always loaded. |
| `design-principles` | Ports & Adapters, Single Source of Truth, Generated Contracts. |
| `implementation-principles` | Fail Fast, guard clauses, data-driven extensibility. |

## Key Rules

1. **Research before design.** Don't make architecture decisions based on assumptions.
2. **Each doc has one job.** North star = vision. Architecture = how. Roadmap = phases. No duplication.
3. **Every phase ships with tests.** A phase is done when its PR merges with CI green.
4. **Never `terraform apply` locally.** CI only.
5. **Never commit secrets.**
6. **Quality checkpoint every 2-4 phases.** Refactor + patterns + test gaps.
7. **Security review before deploy.** No Critical/High findings.

## Installation

```bash
# Copy all skills to your personal skills folder
cp -r knowledge-index ~/.claude/skills/
cp -r ideate ~/.claude/skills/
cp -r architecture ~/.claude/skills/
cp -r roadmap ~/.claude/skills/
cp -r brief ~/.claude/skills/
cp -r principles/build-process ~/.claude/skills/
cp -r vendor/nklisch/* ~/.claude/skills/
```

## Attribution

Skills in `vendor/nklisch/` are from [nklisch/skills](https://github.com/nklisch/skills) by [@nklisch](https://github.com/nklisch). Vendored with attribution.
