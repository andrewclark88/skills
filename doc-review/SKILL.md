---
name: doc-review
description: >
  Review core planning documents for consistency, completeness, and alignment. Uses cascading
  passes: system-level first, then system + each module. Finds stale references, conflicting
  decisions, missing cross-references, and docs that haven't kept up with design changes.
  Run after major design changes, before starting a new phase, or at quality checkpoints.
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

## Why This Exists

Planning documents drift. Architecture decisions get made in one doc but not reflected in
others. In multi-module projects, module docs diverge from system-level docs. Cross-references
break. Frontmatter specs diverge. This skill catches all of that.

## Arguments

- No arguments: full cascading review (system-level + each module)
- Module name (e.g. `brief`): system-level + that module only
- `--system-only`: system-level docs only, skip module passes

---

## Phase 1: Discover Documents

**Primary source:** Read `docs/knowledge-index.yaml` if it exists.

**Planning docs** (`/doc-review` audits these):

| Type | What to check | Examples |
|------|--------------|---------|
| `north-star` | Vision, principles, domain model | `north-star.md`, module north stars |
| `architecture` | Modules, data flow, conventions, cross-cutting designs | `architecture.md`, `knowledge-store.md`, `tool-use-map.md` |
| `roadmap` | Phases, status, dependencies | `roadmap.md` |

**Briefs** (`knowledge/lint` checks these, NOT doc-review):

| Type | Checked by |
|------|-----------|
| `brief` | `knowledge/lint` — staleness, orphaned, contradictions |

**Also always check:** `CLAUDE.md` — project-level rules.

**Fallback (no index):** Scan `docs/architecture/`, `docs/*/architecture/`, `CLAUDE.md`.

**Classify documents into levels:**

| Level | What | Examples |
|-------|------|---------|
| **System** | Project-wide planning docs | `north-star-grimoire.md`, `architecture.md`, `roadmap.md`, `knowledge-store.md`, `tool-use-map.md` |
| **Module** | Module-specific planning docs | `docs/brief/architecture/north-star-brief.md`, `docs/ds-engine/architecture/north-star-ds-engine.md` |

**Present the inventory:**
"Found N system-level docs and M module docs across K modules. Running cascading review."

---

## Phase 2: Cascading Review Passes

Review in passes. Each pass is scoped enough for thorough checking. Launch each pass as
a parallel Agent subagent where possible.

### Pass 1: System-Level Consistency

**Scope:** Only system-level docs (north star, architecture, roadmap, cross-cutting designs).

**Check:**

**2a. Document ownership**
- Content in the wrong doc? (Architecture details in north star, phase status in architecture)
- Duplicated content across docs?
- Missing content? (Decision made but not documented anywhere)

**2b. Pipeline & phase consistency**
- Pipeline stated the same everywhere?
- Phase numbers match across roadmap and architecture?
- Phase statuses (DONE/NEXT) accurate given the codebase?
- Blocking briefs listed in phases actually exist?

**2c. Decision consistency**
- Locked decisions stated the same in every doc?
- Architecture doc matches north star's domain model?
- Roadmap phase scope matches what architecture describes?
- Open questions: resolved in one doc but still open in another?

**2d. Cross-reference integrity**
- Every doc-to-doc reference: does the target exist?
- Tool-use-map matches actual module dependencies in roadmap?
- Knowledge index matches actual docs on disk?
- Related Architecture Docs section is current?

**2e. Schema consistency**
- TypeScript interfaces (KnowledgeEntry, ModuleContribution) identical everywhere they appear?
- BQ schemas consistent across docs?
- Frontmatter field specs identical where defined?

**2f. Staleness**
- Old phase numbers after renumbering?
- Old file paths after reorganization?
- "Deferred" items that are now implemented?
- References to renamed/superseded tools or concepts?

### Pass 2+: System + Module (one pass per module)

**Discover modules dynamically.** Scan for module-level planning docs:
1. Check knowledge index for `north-star` entries that aren't the system north star
2. Scan `docs/*/architecture/` for any `north-star-*.md` files
3. Scan `modules/` directories for any that have their own docs

Each discovered module gets its own pass. No hardcoded module list — works for any project
with any number of modules.

**Example (a project with 3 modules):**
- Pass 2: System + Module A (`docs/module-a/architecture/north-star-module-a.md`)
- Pass 3: System + Module B (`docs/module-b/architecture/north-star-module-b.md`)
- Pass 4: System + Module C (`docs/module-c/architecture/north-star-module-c.md`)

**Each module pass checks:**

**Module north star vs system north star:**
- Does the module's vision align with the system vision?
- Does the module reference system-level principles correctly?
- Are module-specific principles consistent with system principles?

**Module north star vs system architecture:**
- Does the module's architecture description match what the system architecture says about it?
- Are MCP primitives listed in the module doc consistent with the system's module map?
- Does the module mention cross-cutting systems (knowledge store, tool-use-map) correctly?

**Module north star vs roadmap:**
- Do the roadmap phases for this module match what the module north star describes?
- Are scope items in the module doc reflected in the roadmap?
- Are features marked "v1" in the module doc actually scheduled in the roadmap?
- Are "deferred" items in the module doc consistently deferred in the roadmap?

**Module north star vs cross-cutting architecture docs:**
- If the project has a knowledge-store.md: does the module's knowledge contribution match?
- If the project has a tool-use-map.md: are the module's dependencies listed?

**Module internal consistency:**
- Does the module's scope table match its MCP primitives table?
- Are frontmatter specs in the module doc consistent with the system-level spec?
- Does the module reference the correct phase numbers from the roadmap?

---

## Phase 3: Compile Report

Collect all pass findings. Deduplicate (same issue found in multiple passes → report once).
Classify by severity:

| Severity | Meaning | Examples |
|----------|---------|---------|
| **Critical** | Active contradiction between docs | Architecture says BQ-first, module doc says in-memory default |
| **High** | Missing content that could mislead | Phase references brief that doesn't exist. Module scope says v1 but roadmap defers it. |
| **Medium** | Stale content that's technically wrong | Old phase number. Deferred item now implemented. |
| **Low** | Minor inconsistency | Broken cross-reference format. Missing Related Docs entry. |
| **Info** | Observation | Orphaned doc. Scope table could add phase numbers. |

**Format the report:**

```markdown
## Doc Review Report

**Project:** {project name}
**Date:** {date}
**Documents reviewed:** {count} ({system} system + {module} module)
**Passes run:** {count} (1 system + {N} module passes)
**Issues found:** {count by severity}

### Pass 1: System-Level
{findings}

### Pass 2: System + {Module Name}
{findings}

### Pass 3: System + {Module Name}
{findings}

### Clean Areas
- {What's consistent — give credit}
```

---

## Phase 4: Present & Fix

Present the report. Highlight:
- Count by severity
- Top 3 most important fixes
- Which passes found the most issues (system drift vs module drift)
- Any patterns ("module north stars haven't kept up with knowledge store changes")

**AskUserQuestion:** "Found N issues across M passes. Want me to fix Critical and High items?"

If yes: fix them, then re-run the affected passes to verify.

---

## When to Run This Skill

- **After major design changes** — new architecture decisions, new modules, restructured docs
- **Before starting a new phase** — verify the phase spec and dependencies are accurate
- **At quality checkpoints** — first skill in the checkpoint (docs before code)
- **After `/update-documentation`** — verify automated updates didn't introduce inconsistencies
- **When adding a new module** — verify it's properly integrated with system-level docs

## Anti-Patterns

- **Don't review everything in one flat pass.** The cascading approach exists because flat reviews miss module-vs-system drift. Use it.
- **Don't just check formatting.** Focus on semantic consistency — do the docs agree on what the system is?
- **Don't skip the codebase check.** If a phase is marked DONE, verify code exists.
- **Don't fix silently.** Present the report first. Some "inconsistencies" are intentional.
- **Don't check briefs for structural consistency.** That's `knowledge/lint`'s job. Doc-review checks planning docs.
- **Don't skip module passes.** Module drift is where the worst issues hide — a module doc that contradicts the system architecture can mislead an entire build phase.
