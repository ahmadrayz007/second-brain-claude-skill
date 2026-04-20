---
name: second-brain
description: Automatically logs research findings, decisions (ADR-style), project progress, meeting notes, and daily work into an Obsidian vault using consistent conventions. Activates when the user shares research, commits to a decision, updates project status, describes a meeting, or starts/ends a work session. Resolves the vault path by scanning for a `.obsidian/` folder or a local config file.
---

# Obsidian Vault Keeper

Turn Claude Code / Claude Cowork into a note-taking co-pilot for an Obsidian vault. Instead of replies that evaporate when the session ends, Claude writes structured markdown into the vault — research, decisions, project status, meetings, and daily notes — using conventions that keep the vault searchable, linkable, and AI-readable.

## When to activate this skill

Trigger on any of:

- Explicit: "log this", "note this", "save to vault", "add to Obsidian", "catat ini"
- **Research** — user shares a URL, asks for investigation, summarizes findings, or compares options.
- **Decision** — user says "we'll go with…", "decided to…", "chose X over Y", "final call is…"
- **Project progress** — user reports status, hits a milestone, mentions a blocker, or sets a next step.
- **Meeting** — user mentions a call, 1:1, sync, standup, or pastes a transcript.
- **Session boundary** — first substantive message of a new day (open daily note) or user wraps up (append summary).

If a message contains multiple triggers, handle the most specific one first (decision > research > project > meeting > daily).

## Step 1 — Resolve the vault path

Before any file write, resolve `VAULT` in this order. Stop at the first hit:

1. **Walk up from CWD.** Starting at the current working directory, walk up to `$HOME`. The parent of any `.obsidian/` folder is the vault. Use that.
2. **Read config.** Check `~/.claude/second-brain.config.json` for a `vaultPath` key.
3. **Ask once.** If nothing resolved, ask the user for the absolute vault path. On reply, persist it:

   ```bash
   mkdir -p ~/.claude
   echo '{"vaultPath": "/absolute/path/to/vault"}' > ~/.claude/second-brain.config.json
   ```

Never write outside `VAULT`. Never modify the user's existing notes destructively — append or create new files.

## Step 2 — Respect existing structure

Before creating folders, `ls "$VAULT"` once per session and cache the folder names in context. If the user already has `notes/`, `decisions/`, `meetings/` etc., use those instead of creating the canonical ones below. Only create a canonical folder when nothing equivalent exists.

Canonical layout (create as needed):

```
VAULT/
├── Daily/                YYYY-MM-DD.md
├── Research/             topic-based, evergreen
├── Decisions/            one file per decision
├── Projects/
│   └── <Project Name>/
│       ├── README.md     Map of Content (MOC)
│       └── Status.md     living status
├── Meetings/             YYYY-MM-DD-<slug>.md
└── Inbox/                quick capture, unsorted
```

## Step 3 — The 5 scenarios

**Default write policy** (see `Operating mode` below for how to override):

- **Silent auto-write** for: daily notes (all ops), meeting notes, project status updates, and appends to existing research files. Just write. Report the path after.
- **Confirm first** for exactly two things: creating a **new ADR file** (Scenario B) and creating a **new research topic file** (Scenario A, first time for that topic). These are the only net-new, semantically heavy files — worth a 2-second confirm because they're forever.

Everything else is too low-stakes to interrupt for. For every scenario: load the matching template from `templates/`, fill placeholders, write, then report the vault-relative path.

### Scenario A — Research

Trigger: user shares a source or asks Claude to investigate.

1. Identify or coin a topic slug (PascalCase or kebab-case — match vault convention).
2. Target: `Research/<Topic>.md`. If exists, append a new dated section; if new, use `templates/research.md`.
3. Fill: context, sources with URLs, key findings, implications, open questions, related `[[wikilinks]]`.
4. Add an entry to today's daily note under `## Notes Created`.
5. Report the full path back to the user.

### Scenario B — Decision (ADR)

Trigger: user commits to a choice.

1. Target: `Decisions/YYYY-MM-DD-<slug>.md` using `templates/decision.md`.
2. Fill: context, options considered (each with pros/cons), chosen option, rationale, consequences.
3. If tied to a project, add a backlink in `Projects/<Project>/README.md`.
4. Append to today's daily note under `## Decisions`.

### Scenario C — Project progress

Trigger: status update, milestone, blocker, or next step.

1. Target: `Projects/<Project>/Status.md`. Create if missing using `templates/project-status.md`.
2. Update the body + bump `updated:` in frontmatter.
3. Append a one-line entry to today's daily note under `## Project Updates` with a `[[wikilink]]` back to the status file.
4. If a new milestone is completed, strike it through; don't delete.

### Scenario D — Meeting

Trigger: user describes a meeting or shares a transcript.

1. Target: `Meetings/YYYY-MM-DD-<slug>.md` using `templates/meeting.md`.
2. Fill: attendees, agenda, discussion, decisions, action items, follow-ups.
3. For each **significant decision** in the meeting → also spawn Scenario B (create an ADR).
4. For each **action item assigned to the user** → append to today's daily note under `## Action Items`.

### Scenario E — Daily note

At session start (first substantive message of a new calendar day):

1. Check for `Daily/YYYY-MM-DD.md`. If missing, create from `templates/daily.md`.
2. Migrate unchecked tasks from yesterday's daily note into the `## Carry-over` section.
3. Greet the user with today's focus + carry-over count.

At session end (user wraps, hits `/clear`, or sends "that's it for today" etc.):

1. Append a `## Session Summary` section listing: key work done, decisions made (with links), notes created (with links).
2. If user mentioned next steps, populate `## Tomorrow`.

## Step 4 — Universal conventions

See `conventions.md` for the full spec. Quick reference:

- **Every note has frontmatter.** Minimum: `type`, `created`, `updated`, `tags`. Add type-specific fields per template.
- **Dates are ISO.** `YYYY-MM-DD` everywhere. Today is resolved via the system date, not guessed.
- **Wikilinks** `[[Note Name]]` for any entity that has (or could have) its own note: projects, people, concepts. Prefer wikilinks over duplicating content.
- **Tags are namespaced.** `#research/competitor`, `#decision/tech`, `#project/<slug>`, `#meeting/1on1`.
- **Filenames:** `YYYY-MM-DD-slug.md` for dated notes (meetings, decisions, daily); `PascalCase.md` or `kebab-case.md` for evergreen (research, project MOCs) — match whatever the vault already uses.
- **Inbox fallback.** If you cannot classify a snippet, drop it in `Inbox/YYYY-MM-DD-<slug>.md` with a `#inbox` tag — do not force-fit.

## Operating mode

Default: **mixed** — silent auto-write for routine work, confirm only for new ADRs and new research topic files (as specified in Step 3). This is what ships.

Users can override the mode verbally during a session. Watch for these cues and adjust for the rest of the session:

| User says | Behavior |
|-----------|----------|
| "auto mode", "quiet mode", "go auto", "stop asking", "just do it" | Disable all confirmations. Write everything silently, including new ADRs and new research files. Still report paths after. |
| "back to default", "confirm mode", "ask me first" | Restore default mixed behavior. |
| "save this" / "log this" (one-shot) | Write regardless of mode. No confirmation. |
| "don't log that" / "skip that" (one-shot, referring to something just said) | Do not write the inferred note. If already written in this turn, report that it was written and offer to delete. |

Mode overrides are **session-scoped**, not persisted. If the user wants a permanent change, they'll ask — then save it to `~/.claude/second-brain.config.json` under a `defaultMode` key (`auto` | `mixed` | `manual`).

`manual` mode exists only if a user persists it via config: only write when the user explicitly says "save"/"log"/"note this".

## Behavior rules

- **Idempotency.** Before appending to an existing note, scan its content. If the same info is already there, skip or append a timestamp-only diff.
- **Atomic writes.** One write per tool call — never batch unrelated updates into a single file edit.
- **Report paths back.** After every write, tell the user the relative path from `VAULT/` so they can `Cmd+O` it.
- **Never delete.** If something needs to be replaced, mark the old content with `~~strikethrough~~` or move it to an `Archive/` section inside the same note.
- **False-positive guard.** If unsure whether something qualifies (e.g. user said "maybe we'll try X"), treat it as speculation — do not create a decision or research file. When in doubt, stay silent and let the user ask explicitly.

## Files in this skill

- `SKILL.md` — this file (loaded automatically by Claude Code)
- `conventions.md` — full convention reference
- `templates/research.md`, `decision.md`, `project-status.md`, `meeting.md`, `daily.md`
- `examples/` — filled-out example notes
- `README.md` — install + user-facing overview
