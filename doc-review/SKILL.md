---
name: doc-review
description: >
  Review core planning documents for consistency, completeness, and alignment. Checks that
  north star, architecture, roadmap, features, briefs, and module docs tell the same story.
  Finds stale references, conflicting decisions, missing cross-references, and docs that
  haven't kept up with design changes. Run after major design changes, before starting a
  new phase, or whenever you suspect doc drift.
  Use when user says "review docs", "check consistency", "audit planning docs", or "are our docs up to date".
user-invocable: true
allowed-tools: Read, Glob, Grep, Agent, AskUserQuestion
model: opus
---

# Doc Review

You audit a project's core planning documents for consistency, completeness, and alignment.
You find the gaps between what was decided and what's documented — the drift that accumulates
as a project evolves.

**You follow the build process at `/dev/skills/docs/build-process.md`.** Read it before starting.
The document ownership rules define what belongs where.

## Why This Exists

Planning documents drift. Architecture decisions get made in one doc but not reflected in
others. Roadmap phases evolve but the north star still describes the old design. Briefs are
written but the roadmap doesn't reference them. Cross-references break. Frontmatter specs
diverge between docs. This skill catches all of that.

## Arguments

- No arguments: audit all core planning docs in the project
- Module name (e.g. `brief`): audit docs related to that module
- `--scope <path>`: audit docs in a specific directory

## Phase 1: Discover Documents

**Primary source:** Read `docs/knowledge-index.yaml` if it exists. This is the canonical catalog
of all project knowledge, maintained by every doc-producing skill (`/ideate`, `/research`,
`/architecture`, `/roadmap`, `/brief`).

Filter entries by type to find core planning docs:

| Type | Role in review |
|------|---------------|
| `north-star` | Vision, principles, domain model — the "what and why" |
| `architecture` | Modules, data flow, conventions — the "how" |
| `roadmap` | Phases, status, dependencies — the "when and in what order" |
| `features` | User-facing capabilities — the "what it can do today" |
| `brief` | Domain knowledge for specific phases |
| `primer` | Domain research findings |

**Also always check** (even if not in the index):
- `CLAUDE.md` — project-level rules
- `docs/architecture/tool-use-map.md` — cross-module dependencies
- `docs/architecture/knowledge-store.md` — knowledge store architecture (if it exists)

**Fallback (if no knowledge index exists):** Scan directories:
```
docs/architecture/           → north star, architecture, roadmap, features
docs/*/architecture/         → module north stars
docs/briefs/                 → briefs and primers
docs/*/briefs/               → module-specific briefs
```

**Check the index itself:**
- Are there `.md` files in `docs/` that aren't in the knowledge index? (Index drift)
- Are there index entries pointing to files that don't exist? (Broken references)

**Present the document inventory** to the user:
"Found N planning docs (M from knowledge index, K from directory scan). Checking consistency across: [list]"

## Phase 2: Check Document Ownership

Per the build process, each doc has one job:

| Doc | Owns | Does NOT own |
|-----|------|-------------|
| Primers | Domain facts, external system behavior | Design decisions, implementation |
| North Star | Vision, principles, domain model | File paths, phase status, technical details |
| Architecture | Modules, data flow, conventions, deps | Vision, roadmap status, CLI usage |
| Roadmap | Phases, status, dependencies, briefs, decisions | Module internals, domain model |
| Features | User-facing capabilities, CLI reference | Architecture internals, future plans |

**Check for:**
- Content in the wrong doc (architecture details in the north star, phase status in architecture)
- Duplicated content across docs (same table in two places, same decision described differently)
- Missing content (design decisions made but not in any doc)

## Phase 3: Cross-Reference Audit

**Launch parallel Agent subagents** — one per check category:

### 3a. Pipeline & Phase Consistency
- Is the pipeline stated the same way everywhere? (`/ideate` → `/research` → `/architecture` → `/roadmap` → build loop)
- Do phase numbers match across roadmap, architecture, and any module docs that reference them?
- Are phase statuses (DONE/NEXT/blank) accurate given the codebase?
- Do "blocking briefs" listed in roadmap phases actually exist?
- Do "Read before building" docs in each phase actually exist?

### 3b. Decision Consistency
- Are locked decisions stated the same in every doc that references them?
- Does the architecture doc match the north star's domain model?
- Does the roadmap's phase scope match what the architecture describes for each module?
- Are open questions the same across docs, or resolved in one but open in another?

### 3c. Cross-Reference Integrity
- Every doc-to-doc reference: does the target exist?
- Every phase that lists a blocking brief: does the brief exist and is it marked as written?
- Every module north star: does it reference the architecture and roadmap correctly?
- Does the tool-use-map match the actual module dependencies in the roadmap?
- Does the knowledge index (if it exists) match the actual docs on disk?

### 3d. Frontmatter & Schema Consistency
- Are frontmatter fields defined identically across docs that specify them?
- Are TypeScript interfaces (KnowledgeEntry, ModuleContribution, etc.) the same everywhere they appear?
- Are BQ schemas consistent between the knowledge store doc and any SQL examples?

### 3e. Staleness Detection
- Any references to old phase numbers (e.g., Phase 3a/3b after renumbering)?
- Any references to old file paths (e.g., `/dev/docs/build-process.md` after moving to skills)?
- Any "Deferred" items in scope tables that are now actually implemented?
- Any status markers that don't match the codebase (phase marked DONE but no code exists)?
- Any references to tools/features that were renamed or superseded?

## Phase 4: Compile Report

Collect all subagent findings. Deduplicate. Classify by severity:

| Severity | Meaning | Examples |
|----------|---------|---------|
| **Critical** | Active contradiction between docs | Architecture says X, roadmap says Y for the same thing |
| **High** | Missing content that could mislead a builder | Phase references a brief that doesn't exist. Module north star missing a major feature. |
| **Medium** | Stale content that's technically wrong | Old phase number referenced. Deferred item now implemented. |
| **Low** | Minor inconsistency or missing cross-reference | Doc references another doc but link format is wrong. |
| **Info** | Observation, not an issue | Orphaned doc not referenced anywhere. Scope table could add phase numbers. |

**Format the report:**

```markdown
## Doc Review Report

**Project:** {project name}
**Date:** {date}
**Documents reviewed:** {count}
**Issues found:** {count by severity}

### Critical ({n})

#### {Issue title}
**Files:** {file1} vs {file2}
**What:** {description of the contradiction}
**Fix:** {what should change in which doc}

### High ({n})
...

### Clean Areas
- {What's consistent and correct — give credit}
```

## Phase 5: Present & Fix

Present the report. Highlight:
- Count by severity
- Top 3 most important fixes
- Any patterns (e.g., "the brief north star hasn't kept up with knowledge store changes")

**AskUserQuestion:** "Found N issues. Want me to fix the Critical and High items now?"

If yes: make the fixes, then re-run checks 3a-3e to verify nothing new was introduced.

---

## When to Run This Skill

- **After major design changes** — new architecture decisions, new modules, restructured docs
- **Before starting a new phase** — verify the phase spec, blocking briefs, and dependencies are accurate
- **At quality checkpoints** — alongside `/refactor-design` and `/extract-patterns`, run `/doc-review`
- **After `/update-documentation`** — verify the automated doc updates didn't introduce inconsistencies
- **Whenever it feels like docs are drifting** — trust the instinct

## Anti-Patterns

- **Don't just check formatting.** Formatting issues are noise. Focus on semantic consistency — do the docs agree on what the system is and how it works?
- **Don't skip the codebase check.** If a phase is marked DONE, verify code exists. If a brief is marked "written," verify the file exists.
- **Don't fix silently.** Always present the report first. Some "inconsistencies" are intentional — the user knows which ones.
- **Don't check docs that aren't core planning docs.** Source code comments, README files in module directories, and other documentation are out of scope unless they reference or are referenced by core planning docs.
