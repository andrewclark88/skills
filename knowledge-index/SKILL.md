---
name: knowledge-index
description: >
  Show all accumulated project knowledge — briefs, architecture docs, and other project knowledge.
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
- **Before researching** — check if a brief already covers the topic
- **Anytime** you need to find relevant context

## Workflow

### Step 1: Find the Knowledge Index

Look for `docs/knowledge-index.yaml` in the project root. If it exists, read it and
present the contents.

If it doesn't exist, **scan the project** to build one:

### Step 2: Scan for Knowledge (if no index exists)

Scan these locations for knowledge documents:

```
docs/briefs/              ← domain briefs
docs/architecture/        ← architecture docs, north star
docs/*/briefs/            ← module-specific briefs (e.g., docs/ds-engine/briefs/)
docs/*/architecture/      ← module-specific architecture docs
*.md in docs/             ← any markdown in the docs tree
```

For each document found, extract these fields. **Frontmatter is the source of truth — fall back to body content only when frontmatter is missing.**

| Field | Source (in priority order) |
|-------|---------------------------|
| **path** | The file path |
| **title** | Frontmatter `title` → first `# heading` |
| **description** | Frontmatter `description` → first paragraph |
| **type** | Frontmatter `type` → inferred from path (`docs/architecture/` → architecture, `docs/briefs/` → brief, files starting with `north-star-` → north-star) |
| **updated** | Frontmatter `updated` → git last-modified date → file mtime |
| **blocks_phase** | Frontmatter `blocks_phase` (optional) |

### Indexable Doc Frontmatter Convention

Doc-producing skills (`/ideate`, `/research`, `/architecture`, `/brief`, `/roadmap`) and any hand-written planning doc should include this minimum frontmatter:

```yaml
---
description: One-line summary used in the knowledge index and search results
type: north-star | architecture | roadmap | brief | primer | features | ideate | workon
updated: 2026-04-13
# Optional fields:
title: Override for the title field (defaults to first # heading)
blocks_phase: "5b"           # if this brief gates a specific roadmap phase
---
```

`description` and `type` and `updated` are required for clean indexing. The other fields enrich the index entry. If a doc lacks this frontmatter, the scan falls back to inference — but inferred values are lower-quality, so authoring tools should always emit the frontmatter.

### Step 3: Present the Index

Organize by type and present:

```
📋 Project Knowledge Index
==========================

Planning Docs:
  • north-star.md — Vision, principles, domain model
  • architecture.md — Modules, data flow, conventions
  • roadmap.md — 14 phases, Phase 3 is NEXT
  • knowledge-store.md — Knowledge store architecture

Briefs (domain knowledge):
  • turn-structure.md — MTG turn phases/steps, priority loop, state machine (411 lines)
  • edh-tournament-rules.md — London mulligan, commander tax, win/loss conditions
  • meta-card-interactions.md — Trigger layering, APNAP, storm, LED timing, win cons
  • meta-seed-pipeline.md — TopDeck.gg → Moxfield → Scryfall pipeline
  • data-ingestion.md — All data source access patterns
  • knowledge-retrieval-patterns.md — LLM Wiki, MemPalace, MCP ecosystem research

Commander Briefs (auto-generated):
  • 32 briefs in docs/briefs/commanders/ — tournament stats + cEDH DDB data

Module-Specific Briefs:
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
    type: brief
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

The index is updated by every doc-producing skill:
1. **`/ideate`** — adds north star entry
2. **`/research`** — adds brief entry
3. **`/architecture`** — adds architecture entry
4. **`/roadmap`** — adds roadmap entry
5. **`/brief`** — adds brief entry
6. **`/knowledge-index`** — full rescan if the index is stale or missing. Use after manual doc creation or if entries are missing.

Each skill appends to the YAML file after writing its doc.

## Anti-Patterns

- **Don't skip the index check.** If you're about to research something, check if a brief already exists.
- **Don't present the full content of every doc.** The index is a catalog — show titles and descriptions. The user or agent reads full docs on demand.
- **Don't forget to update the index.** When any skill produces a new knowledge document, it must add an entry.
