# Build Process

Standard methodology for building projects. Follow this process for all new projects and modules. Skills (slash commands) automate each step — see the Pipeline section for the full flow.

## Where This Lives

This document is the source of truth. It's enforced through three layers:

| File | Scope | Purpose |
|------|-------|---------|
| `~/.claude/CLAUDE.md` | Global — always loaded in every conversation | Hard guardrails. Cannot be dropped even in long sessions. |
| `/dev/CLAUDE.md` | Loaded when working in any project under `/dev/` | Points to this document. |
| `/dev/skills/docs/build-process.md` | This file | Full methodology, reference, and checklists. Human-readable and portable. |

If the rules here conflict with a project-level doc, this document wins.

---

## The Pipeline

Every project follows this pipeline. Skills in `code blocks` are slash commands.

```
PROJECT START
│
├─ 1. Define the Project       /ideate
│     └─ produces: north-star.md (vision, principles, domain model)
│     └─ produces: research plan (domains needing briefs)
│
├─ 2. Research the Domain      /research (for each domain identified)
│     └─ produces: domain briefs + auto-loading reference skills
│     └─ repeat until all critical domains are understood
│     └─ don't rush this — assumptions here cause rewrites later
│
├─ 3. Design the Architecture  /architecture
│     └─ produces: architecture.md (modules, data flow, conventions)
│     └─ grounded in north star + domain briefs — no unresearched assumptions
│
├─ 4. Plan the Build           /roadmap
│     └─ produces: roadmap.md (blocking briefs, Input/Output/Tests per phase)
│
│  ┌── PER PHASE ──────────────────────────────────────────────┐
│  │                                                           │
│  ├─ 5. Brief (if listed)     /brief                          │
│  │     └─ produces: curated domain brief for the builder     │
│  │                                                           │
│  ├─ 6. Design                /design                         │
│  │     └─ produces: design doc (interfaces, types,           │
│  │        acceptance criteria, file paths)                    │
│  │                                                           │
│  ├─ 7. Build                 /implement or /implement-orchestrator
│  │     └─ produces: code + tests                             │
│  │                                                           │
│  ├─ 8. PR + CI               (branch → PR → CI green → merge)
│  │     └─ phase done when PR merges                          │
│  │                                                           │
│  ├─ 9. Update Docs           /update-documentation           │
│  │     └─ syncs all docs to code changes                     │
│  │                                                           │
│  └───────────────────────────────────────────────────────────┘
│
│  ┌── EVERY 2-4 PHASES ──────────────────────────────────────┐
│  │                                                           │
│  ├─ Quality Checkpoint                                       │
│  │  ├─ /doc-review         → audit planning docs for          │
│  │  │                        consistency and drift             │
│  │  ├─ /refactor-design    → find duplication, missing       │
│  │  │                        abstractions → implement fixes   │
│  │  ├─ /extract-patterns   → document reusable patterns      │
│  │  └─ /test-quality       → spec-driven test gap analysis   │
│  │                                                           │
│  └───────────────────────────────────────────────────────────┘
│
│  ┌── PRE-DEPLOY ────────────────────────────────────────────┐
│  │                                                           │
│  ├─ /security-review       → comprehensive audit, scored     │
│  │                           report, address Critical/High   │
│  │                           before deploying                │
│  └───────────────────────────────────────────────────────────┘
│
├─ /update-documentation maintains the Features section of architecture.md
│     └─ what the tool can do today — updated after each phase, not written once
│
├─ Deploy (final phase)
│     └─ Terraform via CI only — see Infrastructure Safety
│
│  ┌── ONGOING / AS-NEEDED ───────────────────────────────────┐
│  │                                                           │
│  ├─ /cruft-cleaner         → remove dead code, AI bloat      │
│  ├─ /bold-refactor         → find architectural              │
│  │                           simplifications                  │
│  └───────────────────────────────────────────────────────────┘

KNOWLEDGE INDEX (maintained automatically):
  Every skill that produces a doc (/ideate, /research, /architecture,
  /brief, /roadmap) updates docs/knowledge-index.yaml.
  Run /knowledge-index at session start to see all available context.
```

---

## Knowledge Index

Every project accumulates knowledge — briefs, architecture docs, research findings. The knowledge index (`docs/knowledge-index.yaml`) tracks all of it so future sessions know what context is available.

**How it works:**
- `/knowledge-index` — scans the project and presents all available knowledge. Run at the start of any session.
- Every skill that writes a doc (`/ideate`, `/research`, `/architecture`, `/brief`, `/roadmap`) appends an entry to the index after writing.
- The index is a YAML file listing each doc's path, title, type, description, and last updated date.

**The rule:** Before researching or writing a brief, check if one already exists. Before starting a phase, check what context is available. The index prevents duplicate work and ensures accumulated knowledge is used.

---

## Step Details

### 1. Define the Project (`/ideate`)

Interactive workshop. Explore the idea, refine it, produce a north star. Also identify what domains need research before architecture can be designed.

**Produces:** `north-star.md` (vision, principles, domain model) + a research plan.

**Does NOT produce architecture.** Architecture requires domain research first. Don't design how to build something until you understand the domain it operates in.

### 2. Research the Domain (`/research`)

Deep investigation of each domain identified in step 1. Use for external systems, protocols, APIs, analytical methods, hardware, regulations — anything the project builds on.

**Produces:** Domain briefs + auto-loading reference skills.

**Be aggressive about research.** It's far cheaper to spend a session researching than to discover mid-build that the domain works differently than assumed. If you're not sure whether something needs research, it does.

**Run `/research` multiple times** — once per domain. Don't try to research everything in one pass.

### 3. Design the Architecture (`/architecture`)

Technical design, informed by the north star AND domain briefs. Now you know enough to make real decisions.

**Produces:** `architecture.md` (modules, data flow, conventions, dependencies).

**Prerequisites:** North star and relevant domain briefs MUST exist. If you discover during architecture that a domain isn't understood, stop and run `/research` first.

**Ground every decision in research.** Not "we'll use BigQuery" but "we'll use BigQuery because the domain brief confirmed it's the existing platform."

### Document Types & Ownership

Three planning doc types + one knowledge type. Each has one job. No duplication across types.

When content belongs in two places, put it in one and reference the other.

#### North Star (`type: north-star`)

**Owns:** Vision, principles, domain model — the "what and why."
**Does NOT own:** File paths, phase status, technical implementation, module structure.
**Produced by:** `/ideate`

**Required sections:**
- **Vision** — 1-2 sentences
- **Problem** — what's broken today
- **Principles** — 5-7 rules that guide every decision
- **Domain Model** — core concepts and their relationships
- **Related Documents** — links to architecture + roadmap

#### Architecture (`type: architecture`)

**Owns:** How the system is built — modules, data flow, conventions, dependencies, cross-cutting design decisions.
**Does NOT own:** Vision/principles (north star), phase status (roadmap).
**Produced by:** `/architecture`. Maintained by `/update-documentation` after each phase.

**Required sections:**
- **System Overview** — diagram
- **Module Map** — file/directory → responsibility → external dependency
- **Data Flow** — how data moves through the system
- **Conventions** — code org, API patterns, error handling, testing
- **Dependencies** — table
- **What's Deferred** — and why

**Optional sections (as the project needs them):**
- **Features** — what the tool can do today, organized by user-facing capability. Maintained by `/update-documentation`.
- **Knowledge Store** — if the project has searchable knowledge (knowledge-store.md)
- **Tool/Dependency Map** — cross-module dependencies (tool-use-map.md)
- **Development Workflow** — local dev, CI, deployment model
- **Other cross-cutting design docs** — whatever the project needs

All architecture-type docs live in `docs/architecture/`. There can be multiple files — one main `architecture.md` plus supplementary docs like `knowledge-store.md`, `tool-use-map.md`, etc. All typed as `architecture` in the knowledge index.

#### Roadmap (`type: roadmap`)

**Owns:** Phases, status, dependencies, blocking briefs, decisions, open questions — the "when and in what order."
**Does NOT own:** Module internals, domain model, architecture details.
**Produced by:** `/roadmap`

**Required sections:**
- **Acceptance gate** — how a phase is considered done (PR + CI + tests)
- **Phase table** — each phase with: blocking briefs, read before building, build, output, done-when (automated vs manual)
- **Quality checkpoints** — every 2-4 phases
- **Security review** — pre-deploy
- **Dependency graph** — ASCII diagram
- **Decisions (locked)** — choices made and rationale
- **Open questions** — unresolved items

#### Briefs (`type: brief`)

**Owns:** Curated domain knowledge — facts, research findings, implementation context.
**Does NOT own:** Design decisions, system architecture, phase status.
**Produced by:** `/research` (domain investigation), `/brief` (phase-specific context)
**Checked by:** `knowledge/lint` (staleness, orphans, contradictions) — NOT `/doc-review`

### 4. Plan the Build (`/roadmap`)

Produces a roadmap where each phase is a self-contained build spec, completable in one session.

**Every phase has:**
- **Blocking briefs** — domain knowledge that must be written before the phase begins
- **Read before building** — docs and code to load into context
- **Build** — what to implement (specific files, endpoints, features)
- **Output** — files produced
- **Tests** — acceptance criteria, split into automated and manual
- **Dependencies** — what must exist before this phase starts

Status tracking: DONE / NEXT / blank.

### 5. Brief-Driven Development (`/brief`)

Before each implementation phase, **write the brief that unblocks it.** Briefs are curated domain knowledge optimized for agent consumption — not raw research, not architecture, not tutorials. They answer: "what does the builder need to know to implement this phase correctly?"

**The rule:** If a roadmap phase lists a blocking brief, run `/brief` before the phase begins. The skill reads the roadmap phase, researches the domain, and produces a structured brief.

**`/research` vs `/brief`:** Research investigates broadly and produces a reference. Brief takes research (or does its own) and distills it into implementation-ready context for a specific phase. Research feeds briefs; briefs feed builds.

### 6. Design (`/design`)

Produces a detailed design document from the roadmap phase spec. Contains:
- Exact file paths
- Full interfaces and types in the project's language
- Function signatures
- Implementation notes for non-obvious logic
- Acceptance criteria (testable assertions, not subjective)
- Implementation order (resolves dependencies)

**The design doc is the bridge between the roadmap (what to build) and implementation (building it).** An implementer agent can write code from it without asking questions.

### 7. Build (`/implement` or `/implement-orchestrator`)

Implement the design document.

- **`/implement`** — single agent, best for phases with <20 files
- **`/implement-orchestrator`** — Opus orchestrator spawns parallel Sonnet agents, best for large phases or parallelizable work

Output includes code AND tests. Tests are the contract — a phase is done when its tests pass.

### 8. PR + CI

All code goes through a PR. The phase is done when:

1. Automated tests pass
2. Docker build succeeds (if applicable)
3. Human confirms manual checks (if any)
4. PR is opened, CI passes, and PR is merged to main

**A phase is not done until its PR is merged.** Working locally is not enough.

### 9. Update Documentation (`/update-documentation`)

After each phase, sync all docs to the code changes just made. Catches drift between what was planned and what was built.

### Quality Checkpoints (every 2-4 phases)

After 2-4 implementation phases, pause and run a quality pass:

- **`/doc-review`** — audit planning docs for consistency. Uses **cascading passes**: system-level first (north star ↔ architecture ↔ roadmap), then system + each module (does the module doc match the system docs?). Run this first — doc issues reveal code issues.
- **`/refactor-design`** — find duplication, missing abstractions, structural improvements. Produces a refactor plan. Implement the plan.
- **`/extract-patterns`** — document reusable code patterns for consistency across future work. Other skills read these patterns.
- **`/test-quality`** — spec-driven test gap analysis. Derives tests from behavioral contracts (specs, designs, interfaces), not from reading implementation code. Writes the tests.

**Also run `/doc-review` after major design changes**, not just at quality checkpoints. If you changed the architecture, added a module, or resolved an open question — check that all docs agree. For multi-module projects, the cascading approach catches module-vs-system drift that flat reviews miss.

### Pre-Deploy Security Review

Before any production deployment:

- **`/security-review`** — comprehensive audit. Discovers stack, lets you choose focus domains (auth, injection, secrets, dependencies, API, infra, crypto, data protection, error handling). Researches current best practices. Produces a scored markdown report with severity-classified findings.
- **Address all Critical and High findings before deploying.** Medium findings should be tracked. Low/Info are best-effort.
- The security report feeds into `/design` for remediation planning.

### Cleanup (as needed)

- **`/cruft-cleaner`** — remove dead code, stale comments, AI-accumulated bloat. Run when the codebase feels heavy.
- **`/bold-refactor`** — find beautiful cross-cutting simplifications. Run when you suspect the architecture can be fundamentally simplified.

---

## Working Principles

- **Data-driven over hand-curated.** When there's a data source, build a pipeline. Don't manually curate what can be automated.
- **Repeatable processes.** Pipelines should re-run (new data, new releases, meta shifts). One-off scripts become recurring commands.
- **Auto-generate, then enrich.** Generate what you can from existing data, layer in external sources, leave TODOs only for things requiring human judgment.
- **Don't hand-write what can be researched.** Use `/research`, web search, API exploration, and data analysis before asking for manual input.

---

## Infrastructure Safety Practices

These apply whenever a project deploys to shared cloud infrastructure (GCP, AWS, etc.). The goal: **never destroy or interfere with resources you don't own.**

### Terraform Rules

**Never apply locally.** Local `terraform apply` has no review, no audit trail, and no guardrails against destroying someone else's resources.

| Action | Where it runs | When |
|--------|--------------|------|
| `terraform init` | Local or CI | Anytime (safe, idempotent) |
| `terraform plan` | Local or CI | Anytime — review the plan carefully |
| `terraform apply` | **CI only, on merge to main** | After plan is reviewed and approved in PR |
| `terraform destroy` | **Never without explicit approval** | Requires manual confirmation + team review |

**Remote state is mandatory.** Terraform state must be stored in a remote backend (GCS bucket, S3, Terraform Cloud) with state locking enabled. Never commit `.tfstate` files to git.

```hcl
terraform {
  backend "gcs" {
    bucket = "<project>-terraform-state"
    prefix = "<service-name>"
  }
}
```

**State lock prevents concurrent applies.** If two CI jobs try to apply at the same time, the lock prevents conflicts. Never use `-lock=false`.

### Shared Project Safety

When deploying into a GCP project (or AWS account) that other teams use:

**Prefix all resources.** Every resource your project creates should be namespaced:
- GCS buckets: `<project-name>-<purpose>` (e.g., `grimoire-briefs`)
- Service accounts: `<project-name>-<role>` (e.g., `grimoire-server`)
- Secrets: `<project-name>-<secret>` (e.g., `grimoire-bearer-token`)
- Cloud Run services: `<project-name>` (e.g., `grimoire`)

**Never use wildcard IAM.** Don't grant project-level `roles/owner` or `roles/editor` to service accounts. Use the narrowest role on the specific resource.

**Import before managing.** If a resource already exists (someone created it manually), `terraform import` it into your state before writing the `.tf` file.

**Use `prevent_destroy` on critical resources:**

```hcl
resource "google_storage_bucket" "data" {
  lifecycle {
    prevent_destroy = true
  }
}
```

**Tag everything.** Add labels to all resources so ownership is clear:

```hcl
labels = {
  managed_by = "terraform"
  project    = "grimoire"
  team       = "data"
}
```

### Secrets

**Never in code.** Secrets (API keys, tokens, passwords) never appear in:
- Source code (`.ts`, `.py`, `.tf` files)
- Terraform state committed to git
- Environment variable files committed to git (`.env`)
- CI/CD logs

**Where secrets live:**
- GCP Secret Manager (for deployed services)
- Local environment variables (for development)
- CI/CD secret store (GitHub Actions secrets, etc.)

### Pre-Flight Checklist (Before First `terraform apply`)

- [ ] Remote state backend exists (GCS bucket for state)
- [ ] State bucket has versioning enabled
- [ ] CI/CD pipeline is configured for plan-on-PR, apply-on-merge
- [ ] All resources are prefixed with project name
- [ ] IAM bindings are scoped to specific resources (not project-wide)
- [ ] Secrets are in Secret Manager (not in code or `.tfvars` committed to git)
- [ ] `prevent_destroy` is set on any resource that would be catastrophic to lose
- [ ] All resources have ownership labels
- [ ] Team has reviewed the initial `terraform plan` output

---

## PR & CI Checkpoints

**All code goes through PRs.** No direct pushes to main for application code or infrastructure.

### Application Code PR Pipeline

```
PR opened/updated:
  → Build (compile, lint)
  → Run full test suite
  → Docker build (verify the deployable image still builds)
  → All checks must pass before merge is allowed

Merge to main:
  → Same checks run again
  → (After deploy phase) Deploy
```

### Infrastructure PR Pipeline

```
PR opened/updated:
  → terraform init
  → terraform plan
  → Post plan output as PR comment
  → Require approval before merge

Merge to main:
  → terraform init
  → terraform plan (again, to catch drift)
  → terraform apply -auto-approve
```

### Branch Protection Rules

| Rule | Required |
|------|----------|
| PRs required before merge to main | Yes |
| At least 1 approval (or self-review for solo projects) | Yes |
| CI checks must pass | Yes |
| No direct pushes to main | Yes |
| No force pushes to main | Yes |

### What CI Checks Per Phase

Each roadmap phase ships tests. CI runs them all, not just the new phase's tests. The full suite is the regression gate.

| Check | When | Why |
|-------|------|-----|
| Build/compile | Every PR | Catch type errors, syntax issues |
| Unit tests | Every PR | Phase acceptance + regression |
| Docker build | Every PR | Verify the deployable image works |
| Terraform plan | Infra PRs only | Review before apply |
| Lint (if configured) | Every PR | Code quality |

---

## Git Practices

- **Commits are small and focused.** One logical change per commit.
- **PRs before merge.** No direct pushes to main. Code review required.
- **Don't commit generated files.** Data caches, build outputs, `.env` files, `.tfstate` — all gitignored.
- **Don't commit secrets.** Ever. Use pre-commit hooks or CI checks to catch accidental secret commits.

---

## Skill Reference

| Skill | When | What it produces |
|-------|------|-----------------|
| `/knowledge-index` | Start of every session | Shows all available project knowledge (briefs, docs) |
| `/ideate` | Step 1 (project start) | North star + research plan |
| `/research` | Step 2 (domain research) + before any phase using unfamiliar tech | Domain brief + auto-loading reference skill |
| `/architecture` | Step 3 (after north star + domain briefs) | Architecture doc: modules, data flow, conventions |
| `/roadmap` | Step 4 (after architecture) | Phased roadmap with blocking briefs, I/O/Tests |
| `/brief` | Step 5 (per phase, when blocking brief listed) | Curated domain brief optimized for the builder |
| `/design` | Step 6 (per phase) | Design doc: interfaces, types, file paths, acceptance criteria |
| `/implement` | Step 7 (per phase, <20 files) | Code + tests |
| `/implement-orchestrator` | Step 7 (per phase, 20+ files) | Code + tests (parallel agents) |
| `/update-documentation` | Step 9 (after each phase) | Updated docs synced to code |
| `/doc-review` | Quality checkpoint + after major design changes | Doc consistency report |
| `/refactor-design` | Quality checkpoint (every 2-4 phases) | Refactor plan |
| `/extract-patterns` | Quality checkpoint | Pattern documentation |
| `/test-quality` | Quality checkpoint | Spec-driven tests |
| `/security-review` | Pre-deploy | Scored security report |
| `/cruft-cleaner` | As needed | Cleanup report + fixes |
| `/bold-refactor` | As needed | Architectural simplification plan |
| `/repo-eval` | As needed | Multi-dimensional codebase scoring |
