---
name: knowledge-index
description: >
  Show all accumulated project knowledge — briefs, primers, research docs, architecture docs.
  Auto-loads at session start so the agent knows what context is available before doing work.
  Run at the start of any session, or anytime you need to find relevant project knowledge.
  Use when starting work on a project, looking for existing research, or checking what briefs exist.
user-invocable: true
allowed-tools: Read, Glob, Grep
model: sonnet
---

# Knowledge Index

You surface all accumulated project knowledge so the agent (and user) knows what context
is available before doing work. This is the "what do we already know?" check that should
happen at the start of every build session.

## When This Runs

- **Start of any session** where work will be done on a project
- **Before writing a brief** — check if one already exists
- **Before researching** — check if a primer already covers the topic
- **Anytime** you need to find relevant context

## Workflow

### Step 1: Find the Knowledge Index

Look for `docs/knowledge-index.yaml` in the project root. If it exists, read it and
present the contents.

If it doesn't exist, **scan the project** to build one:

### Step 2: Scan for Knowledge (if no index exists)

Scan these locations for knowledge documents:

```
docs/briefs/              ← domain briefs, primers
docs/architecture/        ← architecture docs, north star
docs/*/briefs/            ← module-specific briefs (e.g., docs/ds-engine/briefs/)
docs/*/architecture/      ← module-specific architecture docs
*.md in docs/             ← any markdown in the docs tree
```

For each document found, extract:
- **File path**
- **Title** (first `# heading`)
- **Description** (first paragraph or frontmatter `description` field)
- **Type**: primer, brief, architecture, north-star, roadmap, features, other
- **Last modified** (from git or file system)

### Step 3: Present the Index

Organize by type and present:

```
📋 Project Knowledge Index
==========================

North Star & Architecture:
  • north-star.md — Vision, principles, domain model
  • architecture.md — Modules, data flow, conventions
  • roadmap.md — 14 phases, Phase 3 is NEXT

Primers (domain research):
  • turn-structure.md — MTG turn phases/steps, priority loop, state machine (411 lines)
  • edh-tournament-rules.md — London mulligan, commander tax, win/loss conditions
  • meta-card-interactions.md — Trigger layering, APNAP, storm, LED timing, win cons

Briefs (implementation context):
  • meta-seed-pipeline.md — TopDeck.gg → Moxfield → Scryfall pipeline
  • data-ingestion.md — All data source access patterns
  • local-meta-tracker.md — Tournament location data, regional analytics

Commander Briefs (auto-generated):
  • 32 briefs in docs/briefs/commanders/ — tournament stats + cEDH DDB primers

Module-Specific:
  • docs/ds-engine/briefs/technique-inventory-eda.md — 12 EDA techniques
  • docs/ds-engine/briefs/technique-inventory-causal.md — 10 causal techniques
  • docs/ds-engine/briefs/decision-dag-design.md — DAG routing patterns
```

### Step 4: Recommend What to Read

Based on the current conversation context (what the user is about to work on), recommend
which documents to load:

```
For Phase 2a (game state models), I recommend reading:
  1. edh-tournament-rules.md — commander zone, mulligan, life total rules
  2. meta-card-interactions.md — card complexity the models must handle
  3. roadmap.md Phase 2a — exact scope and acceptance criteria
```

## Index File Format

When the index is generated, write it to `docs/knowledge-index.yaml`:

```yaml
# Auto-generated knowledge index. Updated by /research, /brief, and /knowledge-index.
# Last updated: 2026-04-10

documents:
  - path: docs/architecture/north-star.md
    title: "North Star: EDH Engine"
    type: north-star
    description: "Vision, principles, domain model for the EDH simulation engine"
    updated: 2026-04-08

  - path: docs/briefs/turn-structure.md
    title: "Brief: MTG Turn Structure"
    type: primer
    description: "Turn phases/steps, priority loop, state machine diagram, multiplayer specifics"
    updated: 2026-04-08
    blocks_phase: "2b"

  - path: docs/briefs/commanders/kraum-ludevics-opus_tymna-the-weaver.md
    title: "Commander Brief: Kraum/Tymna"
    type: commander-brief
    description: "Kraum Tymna Breach archetype — 131 entries, 57.1% WR, strategy + card breakdown"
    updated: 2026-04-08
```

## Keeping the Index Current

The index is updated by:
1. **`/ideate`** — adds north star entry (built into the skill)
2. **`/architecture`** — adds architecture entry (built into the skill)
3. **`/roadmap`** — adds roadmap entry (built into the skill)
4. **`/brief`** — adds brief entry (built into the skill)
5. **`/research`** — the vendored skill does NOT auto-update the index. **After running `/research`, you must manually update the index** or run `/knowledge-index` to rescan.
6. **`/knowledge-index`** — full rescan if the index is stale or missing. Use this after `/research` or any manual doc creation.

Custom skills (1-4) append to the YAML file automatically. Vendored skills (5) don't — rescan with `/knowledge-index` to pick up their output.

## Anti-Patterns

- **Don't skip the index check.** If you're about to research something, check if a primer already exists.
- **Don't present the full content of every doc.** The index is a catalog — show titles and descriptions. The user or agent reads full docs on demand.
- **Don't forget to update the index.** When any skill produces a new knowledge document, it must add an entry.
