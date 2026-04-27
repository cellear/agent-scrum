# Project Setup Wizard

A scripted question flow for setting up a new agent-scrum project from a one-sentence goal. The Architect persona runs it; the human answers; the output is a fully-populated set of `SPRINTS/`, `EPICS/`, and `LEARNINGS/` files plus a project metadata file.

The wizard is just a prompt — no code. An agent reads this file and follows the script.

---

## How to invoke

User: "Run the project setup wizard" (or `/architect`, when the slash command exists).

Agent: Reads this file. Adopts the Architect persona for the duration. Runs the phases in order. At the end, writes all output files.

If the project already has agent-scrum content (existing sprints/epics), the wizard offers to **augment** rather than overwrite. It does not delete anything.

---

## Architect role during the wizard

- Ask questions one phase at a time. Don't dump all six phases at once.
- Within a phase, batch related questions into a single message rather than one-at-a-time.
- After each phase, summarize what was captured before moving on.
- Propose answers when the human seems unsure ("here's a draft epic list — edit it").
- Prefer editing the human's words to inventing new ones.
- Be willing to skip a phase when the human says it's already known (e.g. "the team is just me + Cody").

---

## Phase 1 — Project framing

Ask:

1. **What are we building?** (one sentence)
2. **Who is it for, and what does success look like?**
3. **Any hard constraints?** Deadlines, platform/stack requirements, budget caps, must-use or must-avoid tools.

Capture: `goal`, `audience`, `success`, `constraints`.

---

## Phase 2 — Team selection

Read `Personas.md` if present in the project. If not, use the standard roster (Architect, Product Owner, Coder, Librarian, QA, Designer, Researcher, Professor, Executive Assistant, Scrum Master, UX Specialist, Security Auditor).

**Always include:** Architect (you), Product Owner.
**Default working team:** Coder, Librarian.
**Add by project shape:**
- User-facing UI → Designer, UX Specialist, QA
- Unknowns / discovery work → Researcher
- Likely to produce learnings worth durable docs → Professor
- Security-sensitive → Security Auditor
- Multi-track work → multiple Coders

Propose a roster. Let the human edit. Note that personas are durable across the project; per-story model assignment happens later.

Capture: `team` — list of `{persona, default_model}`.

---

## Phase 3 — Epic decomposition

Propose an epic list that covers the goal end-to-end. Aim for 3–8 epics. Each epic should be a meaningful chunk of value, not a single feature.

For each epic, capture:
- `epic_id` — `epic{YYMMDD}-{slug}` (today's date for new epics)
- `name` — human-readable
- `goal` — one paragraph
- `definition_of_done` — checklist (3–6 items)

Don't write `epic-definition.md` yet — wait until phase 6 commits.

---

## Phase 4 — Sprint plotting

Propose sprint count and per-sprint goals. Map epics to sprints (an epic can span sprints; a sprint can touch multiple epics).

For each sprint, capture:
- `sprint` — number (1, 2, …)
- `goal` — one sentence
- `demo_checkpoint` — concrete: "human runs X, sees Y" or "look at Z and confirm"
- `epics_touched` — which epics this sprint advances

Don't write `SPRINTS/sprint-N.md` yet — wait for phase 6.

---

## Phase 5 — Story decomposition (all the way to the goal)

Draft every story for every sprint. Don't stop at sprint 1.

For each story, capture:

```yaml
story_id: catalog-schema       # short slug, unique within epic
title: Build catalog schema    # human-readable
epic: epic0228-theme-catalog
sprint: 2
state: 0-backlog
confidence: confirmed | planned
points: 1 | 3 | 5              # or s | m | l
persona: Cody                  # which team member picks it up
model: claude-sonnet-4-6       # suggested model for this story
depends_on: [api-discovery]    # other story_ids; empty list ok
description: >
  One short paragraph.
acceptance_criteria:
  - Observable, testable outcome
  - …
```

**Confidence rules:**
- Sprint 1 stories: `confirmed` after the human signs off in phase 6
- Later sprint stories: `planned` (written in good faith, expected to flex)

**Model selection guidance** — propose the cheapest model that can plausibly handle the story:
- Boilerplate, scaffolding, file moves → Haiku
- Routine implementation, refactors with clear scope → Sonnet
- Architectural decisions, ambiguous requirements, novel problems → Opus
- (Adapt for non-Anthropic models: Codex/Cursor/Gemini per the team's preferences)

The human can override any assignment. Show the projected token-spend ballpark when the full story list is drafted (rough: count stories per model tier).

---

## Phase 6 — Review and commit

Present the full plan in one summary message:
- Goal, audience, success, constraints
- Team
- Epic list with goals + DoD
- Sprint list with goals + demos
- Story count per sprint with model distribution

Ask the human to accept, edit, or restart. Edits go back to whichever phase they affect.

On accept, write the output files:

### Files to write

```
project.yaml                                    # project metadata
SPRINTS/sprint-1.md                             # one per sprint, from phase 4 + 5
SPRINTS/sprint-2.md
…
EPICS/{epic_id}/epic-definition.md              # one per epic, from phase 3
EPICS/{epic_id}/0-backlog-{story_id}.md         # one per story, from phase 5
LEARNINGS/                                      # empty dir, ready for retros
```

### `project.yaml` shape

```yaml
goal: One sentence.
audience: Who it's for.
success: What success looks like.
constraints:
  - Deadline: …
  - Stack: …
team:
  - persona: Architect
    default_model: claude-opus-4-7
  - persona: Cody
    default_model: claude-sonnet-4-6
  - persona: Lila
    default_model: claude-haiku-4-5
created: 2026-04-26
schema_version: 1
```

### Story file front-matter

Story files use YAML front-matter (the schema from phase 5) followed by the existing markdown body format documented in `AGENT.md` (Description, Tasks, Acceptance Criteria, Dependencies, References).

The state prefix in the filename (`0-backlog-`, `1-in-progress-`, `2-finished-`) stays the source of truth for state. Front-matter `state` field mirrors it. When a story moves between states, both rename the file *and* update the front-matter.

### Event log

If `.scrum/events.csv` doesn't exist, create it with the header. Then for each output file written, append events:

- `project_created` (one event)
- `epic_created` (one per epic)
- `sprint_created` (one per sprint)
- `story_created` (one per story)

All with `source: wizard` and `actor: Architect`. See [event-log-schema.md](../DOC/event-log-schema.md) for the column spec. *(Until that doc is moved into agent-scrum, the wizard inlines the schema — pending refactor.)*

### Final message to the human

Print:
- Where files were written
- Suggested first action ("ready to start sprint 1; run the first story?")
- Reminder: stories can be reorganized by editing files directly or asking the architect to revise

---

## Stop triggers

The architect knows it has enough to commit when, without hand-waving:
- Goal is one sentence
- Team has at least Architect + Product Owner + one doer
- Every epic has a goal + DoD
- Every sprint has a goal + demo checkpoint
- Every story has a title, persona, model, points, and at least one acceptance criterion

If the architect stalls on any of these, that's the next question to ask.

---

## What the wizard doesn't do

- Doesn't pick the human's tools (IDE, AI vendor) — agnostic
- Doesn't enforce model assignments — they're suggestions
- Doesn't run the work — it scaffolds the project; sprint execution is separate
- Doesn't generate handoffs — that's the agent-handoff protocol's job

---

*Schema version: 1. Last updated: 2026-04-26 by claude-opus-4-7.*
