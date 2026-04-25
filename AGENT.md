# Agent Scrum Protocol

A lightweight Scrum + Kanban layer for AI-assisted projects. Pairs with the agent handoff protocol — handoffs are chronological session journals; scrum tracks the work itself.

## Directories

Use whichever subset fits the project:

- `SPRINTS/` — One file per sprint: goal, stories, acceptance criteria, demo checkpoint
- `EPICS/` — One subdirectory per epic with state-prefixed story files
- `LEARNINGS/` — Per-sprint retros (one file per sprint, e.g. `sprint-1.md`)

Create these alongside `HANDOFF/` and `DOC/` (from the handoff protocol).

## Sprints

**File:** `SPRINTS/sprint-{n}.md`

Each sprint file should include:

- **Sprint Goal** — One-sentence outcome that proves the sprint succeeded
- **Stories** — List of story sections (`### S{n}-{m} · {title} · [ ]`), each with owner, scope, acceptance criteria
- **Demo checkpoint** — What a human runs/sees to accept the sprint
- **Acceptance** — Status, date, reviewer (filled in when the sprint is accepted)
- **Decisions made this sprint** — Optional; conventions or tradeoffs that bind future work
- **Deferred to later sprints** — Optional; what was pulled out of scope

Stories use checkbox status in their heading (`[ ]` → `[x]`). Acceptance criteria are checklists.

## Epics

**Directory:** `EPICS/epic{YYMMDD}-{slug}/` — e.g. `epic0228-theme-catalog`

Each epic directory contains:

- `epic-definition.md` — Goal, scope, approach, risks, definition of done, **backlog order**
- One file per story, named with a **state prefix** (see below)

### Story state prefix

| Prefix | State | Meaning |
|--------|-------|---------|
| `0-` | backlog | Not started |
| `1-` | in-progress | Currently being worked on |
| `2-` | finished | Done |
| (extend as needed) | blocked, cancelled | |

**Format:** `{prefix}-{state-name}-{task-slug}.md`

Examples:
- `0-backlog-api-discovery.md`
- `1-in-progress-build-scraper.md`
- `2-finished-catalog-schema.md`

**Renaming a file changes its state.** The prefix drives sort order in `ls`, making workflow visible at a glance. No metadata file to keep in sync.

### Story content

Each story file should include:

- **Story ID** — short slug (matches the filename suffix)
- **Epic** — parent epic ID
- **Status** — redundant with filename, useful inside the file
- **Points** — optional (small/medium/large or numeric)
- **Description** — what the story is
- **Tasks** — checklist of work items
- **Acceptance criteria** — definition of done
- **Dependencies** — optional; other stories that must complete first
- **References** — optional; DOC files, prior handoffs

### Backlog order

Story filenames do **not** imply priority. Maintain a `## Backlog order` section in `epic-definition.md` listing story IDs in priority order. The product owner edits this when reordering.

```markdown
## Backlog order
1. api-discovery
2. build-scraper
3. download-screenshots
4. validate-catalog
5. document-process
```

(Alternatives: numeric prefix in filename, or `Priority:` field in story content. The `epic-definition.md` list is recommended — easiest to reorder.)

## Learnings (Retros)

**File:** `LEARNINGS/sprint-{n}.md`

A bullet list of dated, durable findings from the sprint — gotchas, environment quirks, conventions that emerged. Phrased so a future agent or teammate can act on them without re-deriving.

```markdown
- 2026-03-19: On this machine, `ddev describe` uses `--json-output` instead of `--json`.
- 2026-03-19: Drush JSON output has an inconsistent `definition.arguments` field — handle with `json.RawMessage`.
```

Keep it short. If a learning becomes a load-bearing convention, promote it to `DOC/`.

## Workflow

1. **Create sprint** — `SPRINTS/sprint-{n}.md` with goal and stories
2. **(Optional) Create epic** — `EPICS/epic{YYMMDD}-{slug}/epic-definition.md` for cross-sprint work
3. **Add stories** — `0-backlog-*.md` in the epic dir; or as story sections in the sprint file
4. **Set backlog order** — update `Backlog order` in epic definition
5. **Pick up work** — rename `0-backlog-*` → `1-in-progress-*`
6. **Complete** — rename `1-in-progress-*` → `2-finished-*`; check the box in the sprint file
7. **Handoff** — session handoffs (`HANDOFF/handoff-*.md`) reference the sprint and any stories touched
8. **Sprint demo + acceptance** — human reviews demo checkpoint; fill in `## Acceptance` block
9. **Retro** — write `LEARNINGS/sprint-{n}.md`

## Relation to other protocols

- **Handoff (`HANDOFF/`)** — Session journals, chronological. Reference the active sprint/epic/story.
- **DOC (`DOC/`)** — Persistent reference docs. Promote durable conventions out of `LEARNINGS/` when they stabilize.
- **Scrum (this protocol)** — Work organization: what's planned, in flight, done, learned.

## Version History

Source: https://github.com/cellear/agent-scrum

- **1.0** (2026-04-25) — Initial extraction from theme_machine, ddev-xdebug-tui, ddev-drush-tui
