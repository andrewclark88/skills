---
description: Parking lot for skill/harness ideas that aren't ready to build yet. Each entry stands alone — feel free to extract one into its own design doc once it earns implementation. Add new entries at the top.
type: notes
updated: 2026-05-18
---

# Skill / Harness Ideas — Parking Lot

## CLI as the harness-development surface (vs. IDE for feature work)

**Pattern observed:** When you're writing skills, hooks, scheduled agents, or anything that lives in `~/.claude/` rather than in a project, the VS Code extension adds chrome between you and the substrate without making it easier to reason about. Edits to `settings.json`, hook scripts, and skill markdown are all filesystem-resident — the IDE just slows the feedback loop.

**Proposed guidance (lightweight):**

- Add a section to the skills repo's `README.md` (or a new `docs/development-environment.md`) recommending a split:
  - **VS Code / IDE extension** for project-feature work — click-through file refs, side-by-side editing, screenshots, PDFs.
  - **CLI (`claude` from terminal)** for harness-development work — building skills, debugging hooks, testing wake-up triggers, iterating on `~/.claude/keybindings.json`. You see what fires when; tmux/screen multiplexing makes parallel sessions easier.
- Both can run simultaneously — they share the same `~/.claude/` config, memory, and skills.

**Proposed tooling (heavier — open question):**

A "skill dev mode" CLI command that:
1. Watches a skill markdown file + its referenced docs for changes
2. Hot-reloads the skill registration on save
3. Surfaces hook firings in real-time (a tail of which hook ran with what args)
4. Optionally re-runs a fixed test prompt against the in-flight skill to verify behavior didn't regress

This would dramatically tighten the build-test-iterate loop for skill authoring. Today the loop is: edit file → start new session → invoke skill → observe → repeat. With watch+reload+test-replay the loop drops to: edit file → see test-replay result.

**Building blocks already in place:**

- `claude` CLI exists and is feature-equivalent to the IDE for skill invocation
- Skills are markdown files — trivially watchable
- The hook firing system already emits structured events (settings.json hooks log to stdout/stderr)

**Hardest open design questions:**

- Hot-reload semantics — does the model see the new skill mid-conversation, or does it require a fresh session? If the latter, "hot reload" is really "fast-restart" and the value drops.
- Test-replay — how do you encode the verification? A YAML fixture (`prompt:` + `expected_tools:` + `expected_files_touched:`)? Or a literal "run /myskill foo and check that it wrote X"?
- Onboarding gap — most users come in through the IDE. The "drop to CLI" recommendation needs a 5-minute path from "I've never used the CLI" to "I'm iterating on a skill." Probably a short tutorial: `claude` invocation, where files live, basic keybindings.

**Decision triggers (when to revisit):**

- If you start building skills/hooks more than once a week, the IDE friction will surface itself.
- If the auto-handoff idea above gets built, that work alone will probably motivate the CLI split — it's pure harness work.

---

## Auto-handoff: monitor context window → write seed → clear → re-seed next session

**Pattern observed:** Long multi-PR sessions hit context exhaustion mid-arc. The workaround we keep doing manually: ask Claude to "write a seed for the next session," clear, paste the seed in next time. The seed compresses ~500K tokens of conversation into a ~40-line paste-ready prompt that primes the next session with everything load-bearing.

**Proposed automation:**

1. **Detection** — context-window usage signal from the harness. The model itself doesn't see usage; this has to come from a hook or settings.json trigger. Threshold ~80%.
2. **Trigger heuristic** — fire the handoff only when BOTH conditions hold:
   - Soft signal: context usage > threshold
   - Natural break: no in-progress todos, last commit is on `main` (not a feature branch), no active background tasks
   - If only the soft signal fires, surface a notification ("context full — handoff suggested") but don't force-trigger.
3. **Seed-writing skill** — a new `/handoff` skill (or built into `/clear`) that:
   - Reads the conversation summary
   - Walks recent commits + memory updates + todo state
   - Writes a paste-ready seed to a known location (e.g., `~/.claude/projects/<project>/handoff/last-seed.md`)
   - Updates project memory with a pointer to the seed
4. **Next-session pickup** — SessionStart hook reads the handoff seed if recent (< 24h?) and auto-injects it as the first user message, OR surfaces a notification "prior session left a seed — paste it?" so the human stays in the loop.

**Building blocks already in place:**

- SessionStart hooks (proven: the knowledge-index nav hook auto-loads at session start)
- Memory system (persistent across sessions; entries already point at prior session-end snapshots)
- Skills can read/write files freely
- `/loop` + ScheduleWakeup primitive for scheduled-agent semantics

**Hardest open design questions:**

- Where does the context-window signal come from? Probably needs to be a harness-level value exposed to hooks (currently not visible to skills).
- "Natural break" detection — is it heuristic (`git status` clean + no todos) or model-asserted ("am I at a stopping point")? Both have failure modes.
- Auto-paste vs notification-only on next session start. Auto-paste is faster but removes human oversight; notification-only preserves agency but adds friction.
- Multi-project users — handoffs are per-project, but SessionStart fires before the project is known (or is it? the cwd hint reveals it). Need to scope seeds by project root.

**Inspiration / prior art:**

- The manual seed pattern ds-engine sessions use — every 1-2 sessions a seed gets written and pasted.
- ChatGPT's "memory" feature (cross-session, but coarser than a structured handoff).
- Aider's `--restore-chat-history` flag (closest existing CLI thing — but doesn't compress).

**Decision triggers (when to revisit):**

- If you do 3+ more manual seed cycles, build the v1 of this.
- If a project starts hitting context exhaustion mid-arc more than ~once a week.
