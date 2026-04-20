# Conventions

The full convention spec for `obsidian-vault-keeper`. Claude follows these when writing to the vault. You can override any of these by telling Claude once — the agent will adapt for the session (and ideally save the override to memory).

## Frontmatter

Every note begins with YAML frontmatter. Minimum keys:

```yaml
---
type: research | decision | project-status | meeting | daily | inbox
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags: []
---
```

Type-specific additions are documented per template.

## Dates

- Format: `YYYY-MM-DD` (ISO 8601, no time unless meeting is cross-timezone-sensitive).
- Always resolved via the system clock, never guessed from context.
- Weekdays and relative dates ("Thursday", "last week") are converted to absolute before writing.

## Filenames

| Note type | Pattern | Example |
|-----------|---------|---------|
| Daily | `YYYY-MM-DD.md` | `Daily/2026-04-20.md` |
| Decision | `YYYY-MM-DD-slug.md` | `Decisions/2026-04-20-adopt-postgres.md` |
| Meeting | `YYYY-MM-DD-slug.md` | `Meetings/2026-04-20-client-kickoff.md` |
| Research | `Topic.md` (PascalCase) | `Research/CompetitorLandscape.md` |
| Project MOC | `README.md` | `Projects/Datamapan/README.md` |
| Project status | `Status.md` | `Projects/Datamapan/Status.md` |
| Inbox | `YYYY-MM-DD-slug.md` | `Inbox/2026-04-20-random-idea.md` |

Slugs are lowercase, kebab-case, and ≤6 words. No accents or punctuation.

## Wikilinks

Use `[[Note Name]]` for:

- Projects — `[[Datamapan]]`
- People — `[[Ahmad Rayis]]`
- Recurring concepts that could have their own note — `[[Retention Strategy]]`

Do **not** wikilink:

- One-off mentions
- External URLs (use standard markdown `[text](url)`)
- Dates (use plain `2026-04-20`)

When a wikilink target doesn't exist yet, create a stub note in `Inbox/` with `type: stub` so the graph doesn't have dead links.

## Tags

Namespaced, slash-separated. Lowercase.

| Namespace | Purpose | Examples |
|-----------|---------|----------|
| `research/` | Research kind | `research/competitor`, `research/user`, `research/tech` |
| `decision/` | Decision domain | `decision/tech`, `decision/product`, `decision/hiring` |
| `project/` | Project slug | `project/datamapan`, `project/launch-q3` |
| `meeting/` | Meeting kind | `meeting/1on1`, `meeting/client`, `meeting/review` |
| `status/` | Project health | `status/on-track`, `status/at-risk`, `status/blocked` |
| `person/` | Named individual | `person/ahmad-rayis` |

Tags are in frontmatter arrays, not inline — Obsidian supports both but frontmatter is cleaner for search.

## Linking between notes

- **Every decision** links back to its project (frontmatter `project: [[Name]]`).
- **Every meeting** links to its project and its attendees (wikilinks).
- **Every research note** links to at least one "Why this matters" target (a project or a decision).
- **Project MOC (`Projects/<Name>/README.md`)** contains dynamic lists of related decisions, research, and meetings.

## Atomic units

Prefer many small notes over few large ones. Guidance:

- One decision = one file.
- One meeting = one file.
- One research topic = one file (even if revisited over weeks — append sections).
- One project = a folder, not a file.

## Session-summary format (appended to daily note)

```markdown
## Session Summary — <HH:MM>

**Work done**
- …

**Decisions made**
- [[2026-04-20-adopt-postgres]] — short reason

**Notes created / updated**
- [[CompetitorLandscape]]
- [[Projects/Datamapan/Status]]

**Tomorrow**
- …
```

## Overrides

Users override conventions by telling Claude once. Examples:

- "I use lowercase filenames" → Claude adapts for the session.
- "Put decisions in `adr/` not `Decisions/`" → Claude uses `adr/` and remembers.
- "I don't want daily notes, skip that" → Claude disables Scenario E for the session.

Overrides apply to the **current vault**, not globally.
