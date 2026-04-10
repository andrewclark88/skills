# skills

A complete software development workflow suite for Claude Code. Combines custom skills with adopted skills from [nklisch/skills](https://github.com/nklisch/skills).

## Methodology

This suite implements the build process documented in `docs/build-process.md`:

```
/research → /ideate → /roadmap → [per phase: /brief → /design → /implement → PR → /update-documentation]
                                  ↕ every 2-4 phases: /refactor-design + /extract-patterns + /test-quality
                                  ↕ pre-deploy: /security-review
                                  ↕ as-needed: /cruft-cleaner, /bold-refactor
```

## Skills

### Custom Skills

Built for a specific build methodology: primers → north star → architecture → phased roadmap → brief-driven development → build + test → features doc.

| Skill | Purpose |
|-------|---------|
| `/ideate` | Interactive project definition. Produces north-star.md + architecture.md + domain primers. |
| `/roadmap` | Phased roadmap from foundation docs. Blocking briefs, Input/Output/Tests, quality checkpoints. |
| `/brief` | Curated domain briefs that give build sessions the context they need. |

### Adopted Skills (from nklisch/skills)

Skills adopted from [nklisch/skills](https://github.com/nklisch/skills) by [@nklisch](https://github.com/nklisch). These are vendored copies — updates are pulled manually.

| Skill | Purpose |
|-------|---------|
| `/design` | Detailed implementation units with interfaces, types, acceptance criteria. |
| `/implement` | Single agent code writing (<20 files). |
| `/implement-orchestrator` | Opus orchestrator spawning parallel Sonnet agents (20+ files). |
| `/research` | Investigate libraries/APIs. Produces research doc + auto-loading reference skill. |
| `/security-review` | Comprehensive security audit with scored report. |
| `/refactor-design` | Find duplication, missing abstractions. Produces refactor plan. |
| `/extract-patterns` | Document reusable code patterns for consistency. |
| `/test-quality` | Spec-driven test gap analysis. Writes missing tests. |
| `/update-documentation` | Sync documentation to code changes. |
| `/cruft-cleaner` | Remove dead code, AI-accumulated bloat. |
| `/bold-refactor` | Find architectural simplifications. |
| `/e2e-test-design` | Design end-to-end test suites. |
| `/feature` | Quick extensions outside core roadmap. |
| `/expand` | Handle scope expansion for subsystems or architectural shifts. |
| `/release` | Changelog drafting and release execution. |
| `/repo-eval` | Multi-dimensional codebase evaluation and scoring. |

### Auto-Loading Principles (from nklisch/skills)

| Principle | Enforces |
|-----------|----------|
| `build-process` | The build methodology from `docs/build-process.md`. Always loaded. |
| `design-principles` | Ports & Adapters, Single Source of Truth, Generated Contracts. |
| `implementation-principles` | Fail Fast, guard clauses, data-driven extensibility. |

## Attribution

Skills in `vendor/nklisch/` are from [nklisch/skills](https://github.com/nklisch/skills) by [@nklisch](https://github.com/nklisch). Vendored with attribution. Check the upstream repo for updates and new skills.
