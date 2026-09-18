# banan-tech / ai-skills

Reusable [Claude Code](https://claude.com/claude-code) skills for banan-tech
projects.

## What's inside

| Skill | What it does |
|---|---|
| `/write-issue` | Drafts a well-structured GitHub issue (context, scope, requirements, subtasks, dependencies, testing notes, definition of done) and creates it via `gh`. |
| `/implementor` | Implements a single issue or ticket from "assigned" to "PR open, ready for review" using a TDD workflow. Tracker-agnostic — works with any issue tracker. |

Both skills are repo-agnostic. `/write-issue` auto-detects the active
repository with `gh repo view`; `/implementor` detects the repo via `git
remote` and is tracker-agnostic (GitHub Issues, Linear, etc.). Both read the
host project's `CLAUDE.md` for build commands, conventions, and hard rules.

## Structure

```
ai-skills/
├── write-issue/
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
- [`gh`](https://cli.github.com/) — GitHub CLI, authenticated against the org
  whose repos you'll be working in (`gh auth login`).
- `git`.
- A repository with `CLAUDE.md` at its root (strongly recommended for
  `/implementor` — it's how the skill learns the project's conventions).
  `README.md` is a fallback if `CLAUDE.md` is absent.

## Usage

### `/write-issue`

```
/write-issue <title or free-form description of the work>
```

The skill walks through:

1. Parse the request and pull relevant context from the repo.
2. Confirm title, labels, assignee, and milestone via `AskUserQuestion`.
3. Draft the issue using a strict template (Context → Scope → Requirements →
   Dependencies → Subtasks → Testing → Definition of Done).
4. Present the draft for your approval.
5. Create the issue on GitHub and return the URL.

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
