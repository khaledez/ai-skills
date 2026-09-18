# banan-tech / ai-skills

Reusable [Claude Code](https://claude.com/claude-code) skills for banan-tech
projects.

## What's inside

| Skill | What it does |
|---|---|
| `/spec` | Drafts a well-structured issue or ticket (context, scope, requirements, subtasks, dependencies, testing notes, definition of done) and creates it in the project's issue tracker. Tracker-agnostic — works with any issue tracker. |
| `/implementor` | Implements a single issue or ticket from "assigned" to "PR open, ready for review" using a TDD workflow. Tracker-agnostic — works with any issue tracker. |

Both skills are repo-agnostic and tracker-agnostic (GitHub Issues, Linear,
etc.). They detect the active repo via `git remote -v` and use whatever
CLI or integration your project uses for ticket operations. Both read the
host project's `CLAUDE.md` for build commands, conventions, and hard rules.

## Structure

```
ai-skills/
├── spec/
│   └── SKILL.md
└── implementor/
    └── SKILL.md
```

Each skill is a single `SKILL.md` under a top-level `<name>/` directory. Drop
the directory into your project's skills location (or symlink it) and the
skill becomes available as `/<name>`.

## Requirements

- [Claude Code](https://claude.com/claude-code) — the CLI must be installed and
  authenticated.
- An issue tracker CLI or integration your project uses (e.g. `gh` for GitHub
  Issues, the Linear CLI, etc.), authenticated.
- `git`.
- A repository with `CLAUDE.md` at its root (strongly recommended for
  `/implementor` — it's how the skill learns the project's conventions).
  `README.md` is a fallback if `CLAUDE.md` is absent.

## Usage

### `/spec`

```
/spec <title or free-form description of the work>
```

The skill walks through:

1. Parse the request and pull relevant context from the repo.
2. Confirm title, labels/status, assignee, and milestone/cycle via
   `AskUserQuestion`.
3. Draft the ticket using a strict template (Context → Scope → Requirements →
   Dependencies → Subtasks → Testing → Definition of Done).
4. Present the draft for your approval.
5. Create the ticket in the tracker and return the URL.

### `/implementor`

```
/implementor #42
/implementor https://<tracker>/<org>/<project>/issue/42
/implementor 42
```

The skill works directly in your current checkout and:

1. Resolves the ticket and the active repo.
2. Claims the ticket (status/label + assignment + comment).
3. Investigates the codebase (reads `CLAUDE.md`, plans before coding).
4. Implements in TDD (Red → Green → Repeat).
5. Opens the PR with a test plan and a link that closes the ticket.

## Contributing

Both skills are written as a single `SKILL.md` — open a PR with the edit and
a short rationale. If you find banan-platform-specific assumptions leaking back
in, that's a bug — file an issue.

## License

[MIT](LICENSE)
