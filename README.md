# banan-tech / ai-skills

Reusable [Claude Code](https://claude.com/claude-code) skills for banan-tech
projects, distributed as a Claude Code plugin marketplace.

## What's inside

| Plugin | Skill | What it does |
|---|---|---|
| `write-issue` | `/write-issue` | Drafts a well-structured GitHub issue (context, scope, requirements, subtasks, dependencies, testing notes, definition of done) and creates it via `gh`. |
| `implement-issue` | `/implement-issue` | Pipeline-orchestrates one or more GitHub issues from "assigned" to "PR open, ready for review." Each issue runs in an isolated git worktree via a TDD workflow. |

Both skills are repo-agnostic. They auto-detect the active repository with
`gh repo view`, and the implementer reads the host project's `CLAUDE.md` for
build commands, conventions, and hard rules.

## Install

In any project where you want these skills:

```bash
# 1) Add this marketplace
claude plugin marketplace add banan-tech/ai-skills

# 2) Install the plugins you want
claude plugin install write-issue@banan-ai-skills
claude plugin install implement-issue@banan-ai-skills
```

That's it — `/write-issue` and `/implement-issue` are now available in that
project.

To update later:

```bash
claude plugin marketplace update banan-ai-skills
```

To list / disable / remove:

```bash
claude plugin list
claude plugin disable <plugin>@banan-ai-skills
claude plugin uninstall <plugin>@banan-ai-skills
```

## Requirements

- [Claude Code](https://claude.com/claude-code) — the CLI must be installed and
  authenticated.
- [`gh`](https://cli.github.com/) — GitHub CLI, authenticated against the org
  whose repos you'll be working in (`gh auth login`).
- `git` ≥ 2.5 (for `git worktree` support — used by `implement-issue`).
- A repository with `CLAUDE.md` at its root (strongly recommended for
  `implement-issue` — it's how the Implementer learns the project's
  conventions). `README.md` is a fallback if `CLAUDE.md` is absent.

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

### `/implement-issue`

```
# Single issue
/implement-issue #42
/implement-issue https://github.com/<owner>/<repo>/issues/42

# Multiple issues
/implement-issue #42 #43 #44

# Free-form
/implement-issue let's knock out the three open audit-log issues
```

The skill:

1. Builds a dependency-aware pipeline across the requested issues.
2. Asks you to pick a concurrency mode (sequential / pair / quad / full).
3. For each issue, spawns an **Implementer subagent in an isolated git
   worktree** that:
   - Claims the issue (label + assignment + comment)
   - Investigates the codebase (reads `CLAUDE.md`, plans before coding)
   - Implements in TDD (Red → Green → Repeat)
   - Opens the PR with a test plan and `Closes #N`
4. Surfaces a final report with PR links and merge order.

## How the marketplace is structured

```
ai-skills/
├── .claude-plugin/
│   └── marketplace.json         # marketplace catalog
└── plugins/
    ├── write-issue/
    │   ├── .claude-plugin/
    │   │   └── plugin.json
    │   └── skills/
    │       └── write-issue/
    │           └── SKILL.md
    └── implement-issue/
        ├── .claude-plugin/
        │   └── plugin.json
        └── skills/
            └── implement-issue/
                └── SKILL.md
```

References:
- [Claude Code plugin marketplaces](https://code.claude.com/docs/en/plugin-marketplaces)
- [Claude Code plugins](https://code.claude.com/docs/en/plugins)
- [Claude Code skills](https://code.claude.com/docs/en/skills)

## Contributing

Both skills are written as a single `SKILL.md` per plugin — open a PR with the
edit and a short rationale. If you find banan-platform-specific assumptions
leaking back in, that's a bug — file an issue.

## License

[MIT](LICENSE)
