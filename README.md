# second-brain-claude-skill

A Claude Code / Claude Cowork skill that turns Claude into a note-taking co-pilot for your [Obsidian](https://obsidian.md) vault. Instead of chat replies that evaporate when the session ends, Claude writes structured markdown into your vault — research, decisions, project progress, meetings, and daily notes — using conventions that keep everything searchable, linkable, and AI-readable.

Built for product managers, marketers, researchers, engineers, and anyone using Claude as a second brain.

## What it does

When you're working with Claude and one of these happens, Claude automatically logs it to your vault:

| Scenario | What gets written | Where |
|----------|-------------------|-------|
| You share research or ask Claude to investigate | Topic note with context, sources, findings, implications | `Research/<Topic>.md` |
| You commit to a decision ("we'll go with X") | ADR with context, options, rationale, consequences | `Decisions/YYYY-MM-DD-slug.md` |
| You update project status or hit a milestone | Living status doc + entry in daily note | `Projects/<Name>/Status.md` |
| You describe a meeting or paste a transcript | Structured meeting note + action items + spawned ADRs | `Meetings/YYYY-MM-DD-slug.md` |
| Start / end of a work session | Daily note with focus, carry-over, summary, tomorrow | `Daily/YYYY-MM-DD.md` |

Everything uses consistent frontmatter, wikilinks between related notes, and namespaced tags — so Obsidian's graph view and full-text search stay useful as the vault grows.

## Prerequisites

1. **Install Obsidian** — free, cross-platform. Download at [obsidian.md](https://obsidian.md).
2. **Create a vault** — in Obsidian: `Create new vault` → pick any folder on disk. That folder is your vault (e.g. `~/Documents/MyVault`). The vault can be empty or already in use — the skill appends, never overwrites.
3. **Install Claude Code or Claude Cowork** — see [claude.com/claude-code](https://claude.com/claude-code).
4. **Git** — needed for installing the skill via `git clone`.

## Install

### Recommended — Claude Code plugin (two commands)

In Claude Code, run:

```
/plugin marketplace add https://github.com/ahmadrayz007/second-brain-claude-skill
/plugin install second-brain-claude-skill
```

That's it. The skill auto-activates on context (no slash command needed). Updates via `/plugin update second-brain-claude-skill`.

### Alternative — manual install (git clone)

If you prefer to manage the skill files yourself:

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/ahmadrayz007/second-brain-claude-skill.git /tmp/sbcs
cp -r /tmp/sbcs/skills/second-brain ~/.claude/skills/second-brain
```

Restart Claude Code. Works in both Claude Code and Claude Cowork.

### Per-project install

Swap `~/.claude/skills/` for `.claude/skills/` in the commands above to scope the skill to one repo/vault instead of globally.

## First run

On the first trigger in a new project, the skill will:

1. Walk up from the current directory looking for a `.obsidian/` folder. If found, that's the vault.
2. If not found, it checks `~/.claude/second-brain.config.json` for a saved path.
3. If still not found, it asks you once for the absolute path — and saves it to the config so it never asks again.

You can change the saved path any time by editing `~/.claude/second-brain.config.json`.

## Try it

Open a chat in a folder next to (or inside) your Obsidian vault and try:

- **Research:** "Summarize these three competitors and save to the vault: …"
- **Decision:** "We're going with Postgres over Firestore for Datamapan — log this as a decision."
- **Project:** "Datamapan is moving to at-risk because of the May migration freeze. Update the status."
- **Meeting:** "Just finished a 1:1 with Farah. We covered: …"
- **Daily:** "What's on my plate today?" (opens today's daily note and carries tasks over)

Claude will confirm the filename and preview before creating any new file.

## Conventions

Every note follows the same minimal contract: YAML frontmatter with `type`, `created`, `updated`, `tags`; wikilinks `[[like this]]` for entities that have or could have their own note; namespaced tags like `#research/competitor`, `#decision/tech`, `#project/datamapan`. Full spec in [`skills/second-brain/conventions.md`](skills/second-brain/conventions.md).

You can override any convention by telling Claude once — the agent adapts for the session.

## What's in this repo

```
second-brain-claude-skill/
├── .claude-plugin/
│   └── plugin.json              # plugin manifest (enables /plugin install)
├── skills/
│   └── second-brain/
│       ├── SKILL.md             # main skill file (triggers + instructions)
│       ├── conventions.md       # full convention reference
│       ├── templates/
│       │   ├── research.md
│       │   ├── decision.md
│       │   ├── project-status.md
│       │   ├── meeting.md
│       │   └── daily.md
│       └── examples/            # filled-out example notes
├── README.md                    # this file
└── LICENSE                      # MIT
```

## Philosophy

Three principles guide the skill:

1. **Findable** — a note from two months ago should be reachable in under 30 seconds. Structured frontmatter + consistent filenames + tags make this possible.
2. **Linkable** — one note points to another, automatically. Wikilinks build the graph as you work, not as a separate curation task.
3. **AI-readable** — Claude can read your entire vault as context. The more structured your notes, the better the recall.

Folders alone give you findable. Obsidian + this skill gives you all three.

## Contributing

Issues and PRs welcome. Especially helpful:

- Additional templates (1:1 agendas, retros, product briefs, campaign briefs)
- Alternative conventions for specific workflows (academia, journalism, engineering)
- Translations of the templates

## License

MIT — see [LICENSE](LICENSE).

## Credits

Inspired by the second-brain workflow of product folks who live in Obsidian + Claude. Built by [@ahmadrayz007](https://github.com/ahmadrayz007).
