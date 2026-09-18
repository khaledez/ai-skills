---
name: spec
description: Draft a well-structured issue or ticket — context, scope, requirements, subtasks, dependencies, testing notes, and a verifiable definition of done — and create it in the project's issue tracker. Use whenever the user wants to "create an issue", "open a ticket", "track this", "log this work", "file a bug", or similar — including when they're handing off a backlog of todo items to be turned into tickets. Also trigger when they're describing a piece of engineering work that's clearly meant to become trackable (a refactor, a feature, a bug they just found) even if they don't explicitly use the word "issue" or "ticket".
allowed-tools: Read, Grep, Glob, Bash, AskUserQuestion
argument-hint: <ticket title or description of what needs to be done>
---

# Write Ticket Spec

Draft a structured, high-quality issue or ticket from a description and create
it in the project's tracker. The ticket must be useful to the person picking
it up later — concrete enough that they can start working without
re-litigating the scope.

**Request:** $ARGUMENTS

This skill is tracker-agnostic. It never assumes a specific issue tracker
(GitHub Issues, Linear, etc.). Use whatever CLI or integration your project
uses for ticket operations — listing, fetching, labelling, assigning,
creating. Detect the active repo with `git remote -v`.

This skill assumes every author is a product-engineer: each ticket includes
the full technical detail (file paths, services, code references) AND the
product context (why the work matters). One template, one voice.

---

## Step 1: Understand the Request

Parse the request to identify:
- **What** needs to be done — feature, bug fix, refactor, investigation, chore.
- **Why** it needs to be done — business context, technical debt, incident, follow-up to recent work.
- **References** to code, files, services, or existing tickets / PRs.

If the request references code or a feature area in this repo, **read the relevant source files** to gather precise context (file paths, function signatures, schema fields). This is what makes the ticket useful when someone picks it up cold.

If the request references an existing ticket or PR (by number or URL), fetch it using your tracker's CLI or integration so the new ticket can correctly reference dependencies.

If the title isn't obvious from the arguments, derive a concise one in imperative form ("Add stale-tenant session invalidation", not "fix the bug where stale tenants 500"). Avoid the words *fix*, *improve*, *update* on their own — they're vague.

## Step 2: Gather Repo & Tracker Context

Detect the active repo (the user might have several checked out):

```bash
git remote -v
```

Then, using your tracker's CLI or integration, gather the conventions you'll
need to draft a complete ticket:

- The project / team the ticket will be filed in (confirm you're operating on
  the right project — the user may have several).
- The available **labels** (or equivalent taxonomy — Linear statuses, GitHub
  labels, etc.).
- The available **milestones / cycles / sprints** (whatever the tracker uses
  to group work into a time window).
- The available **assignees / team members**.

You'll use these when prompting the user in Step 3. If the tracker has no
labels/taxonomy, suggest creating a minimal set later, but proceed without
them.

## Step 3: Confirm Title, Labels/Status, Assignee, Milestone/Cycle

Use `AskUserQuestion` to confirm metadata up front — that way the draft you
present in Step 5 is already complete.

Ask (adapting the field names to whatever your tracker uses):

> Ticket metadata — confirm before I draft:
>
> - **Title** — *<the title you derived>* (or supply a better one)
> - **Labels / Status** — *<show the available labels or statuses here, pre-select the ones you think fit>*
> - **Assignee** — *@me* / a specific person / unassigned
> - **Milestone / Cycle / Sprint** — *<show open ones>* / none

If there are >6 labels, present the most likely 3–4 as multi-select options with "Other" as the escape hatch. Don't dump a 50-label list into the question.

Wait for the user's answer before proceeding.

## Step 4: Draft the Ticket

Compose the ticket body using **this exact template** (markdown — most
trackers render it cleanly):

```markdown
## Context

[1–3 sentences: Why does this ticket exist? What problem does it solve, what need does it address, or what incident did it follow? Link to related tickets, PRs, or design docs if they exist (`#123`, `https://...`). Frame the *why* — the *what* and *how* come below.]

## Scope

**In scope:**
- [Concrete deliverable, named.]
- [Another deliverable.]

**Out of scope:**
- [Explicitly excluded — prevents scope creep when the picker-upper asks "should I also…?"]

## Requirements

1. [Technical requirement with file/service references. E.g. "Add `clearSessionCookie(res)` helper in `src/auth/session-cookie.ts`."]
2. [Database / schema changes if any — link the migration that will land.]
3. [API contract changes if any — request/response shape, status codes.]
4. [Edge case the implementer needs to handle.]

## Dependencies

- Depends on #N — [why; what this ticket is waiting on]
- Blocks #M — [optional; what's waiting on this]
- External: [3rd-party API, infra change, another team — or remove the line if none]

(Leave "None" if there are no dependencies — but check; a ticket that touches the same files as another open ticket is a dependency in practice.)

## Subtasks

- [ ] [First chunk of work, completable in 1–2 days max]
- [ ] [Second chunk]
- [ ] [Third chunk]
- [ ] [4–6 total; if you have more, split this ticket into two]

## Testing

**Test scenarios:**
1. [Happy path: given X, when Y, then Z]
2. [Another scenario the implementer should add a test for]

**Edge cases:**
- [Empty input / nil / zero / very large]
- [Concurrent access / race condition]
- [Permission boundary / multi-tenant isolation]

**Regression areas:**
- [Adjacent flow that this change could break — what to manually click through]

## Definition of Done

- [ ] Code implemented and the project's build command passes (e.g. `make build`, `npm run build`, `cargo build`).
- [ ] Tests written or updated; the project's test command passes (e.g. `make test`, `npm test`, `pytest`).
- [ ] Linter / typecheck passes.
- [ ] All subtasks above are checked off.
- [ ] Behavior matches the requirements above on staging / locally.
- [ ] [Ticket-specific verifiable criterion — e.g. "Audit log shows the new event type with the right label."]
- [ ] PR linked to this ticket (`Closes #<this-ticket-number>` or your tracker's equivalent link syntax).
```

### Title

Imperative, concise, technical-but-readable.
- Good: "Add stale-tenant session invalidation"
- Good: "Restyle audit log with typed outcome verbs"
- Bad: "Fix audit log" (too vague)
- Bad: "Implementation of the new audit log outcome verb feature" (verbose)

### Requirements

Be specific. Reference code where you can — file paths, function names, structs. The implementer should be able to start typing without re-discovering the codebase.

### Subtasks

Break the work into 2–6 actionable subtasks. Each subtask should be:
- **Independently completable** — can be worked on, reviewed, and verified on its own.
- **Clearly scoped** — one concern per subtask.
- **Sized 1–2 days max** — if a subtask feels bigger, split it.

Subtasks are rendered as a checklist (`- [ ]`) in the body. They're light-weight — if any subtask is large enough to deserve its own tracking, mention it as "(consider splitting this into a separate ticket)" and let the user decide.

### Testing

Three buckets, all from an engineer's perspective:
- **Test scenarios** — concrete given/when/then cases the implementer should add tests for.
- **Edge cases** — boundary conditions the test scenarios might miss.
- **Regression areas** — adjacent code paths to manually verify (or add tests for).

Don't be exhaustive — pick the 2–4 most useful cases. The implementer can find the rest.

## Step 5: Present Draft for Approval

Show the user the complete ticket draft:

1. **Title** (and the project/team so they know where it's going)
2. **Labels / Status** + **Assignee** + **Milestone / Cycle**
3. **Full body** (rendered markdown — what they'd see in the tracker)

Then ask:

> Here's the ticket draft. Should I create it in `<project>`, or would you like to adjust anything?

**Do NOT create anything in the tracker until the user approves.** If they request changes, revise and present again.

Before presenting, sanity-check the draft:
- Every requirement references something concrete (file, function, schema field, API endpoint).
- Subtasks are sized for 1–2 days each.
- The DoD items are *verifiable* — not "improved" or "better" but observable outcomes.
- Dependencies actually exist (don't invent `#123` — look it up if you reference one).

## Step 6: Create the Ticket

Once approved, write the body to a temp file (so multi-line markdown survives
the shell intact) and create the ticket using your tracker's CLI or
integration:

```bash
BODY_FILE=$(mktemp -t ticket-XXXXXX.md)
cat > "$BODY_FILE" <<'EOF'
<the full ticket body from Step 4>
EOF

# Then create the ticket with your tracker's CLI, passing:
#   --title "<the title>"
#   --body-file "$BODY_FILE"   (or --body "$(cat "$BODY_FILE")")
#   the labels/status, assignee, and milestone/cycle confirmed in Step 3
# Omit any metadata flag entirely when the corresponding answer was "none" —
# passing empty strings usually errors.

rm -f "$BODY_FILE"
```

The tracker CLI prints the new ticket URL on success. **Show that URL to the
user verbatim** so they can click straight through to verify and share it.

If creation errors:
- **Authentication**: tell the user to authenticate with their tracker CLI and re-invoke the skill.
- **Permission denied**: the project might not allow the current account to create tickets — surface the error verbatim.
- **Invalid label / status / milestone**: the value drifted between Step 2 and Step 6 (race with someone editing the project). Re-list the available values, ask the user to pick again, retry.

## After Creation

If the request was part of a backlog (the user said "create tickets for all the todos" or similar), report what was created and ask whether to continue with the next item. **One ticket per invocation** keeps the skill predictable — don't batch-create from a single call.

---

## Guidelines

- **Be specific, not generic.** "Add `handleFormSubmitted` method to `EventsFormSubmittedConsumer` that maps event data to `ContactService.create()`" beats "implement the feature". Vague requirements are the #1 failure mode of tickets — they get reopened or sit stale because nobody knows where to start.

- **Explain the why.** Every Context paragraph should answer "why does this matter right now?" If you can't, the ticket might not be worth creating yet.

- **Keep subtasks small.** 1–2 days each, max. Bigger means split. Smaller (an hour of work) means consolidate — don't pad the count.

- **Definition of Done items are observable.** "Behavior is improved" is not done; "audit log shows the new event with the right label" is.

- **Flag file overlap between tickets.** If two tickets touch the same file (e.g. both modify `src/middleware/auth.ts`), note it explicitly in the Dependencies section: "⚠️ Touches `src/middleware/auth.ts` — overlaps with #N; implement on the same branch to avoid merge conflicts." This prevents parallel PRs from conflicting.

- **Don't invent references.** Never write `#123` unless you confirmed ticket #123 exists. If you're not sure, leave the field as "None" or omit the line.

- **Match the project's voice.** If recent tickets use a particular style (e.g. all titles start with `[area]`), match it. Skim 2–3 recent tickets via your tracker CLI if you're unsure.

- **Don't pad.** If a section doesn't apply (no dependencies, no schema changes), say "None" or omit the line. Empty headings make the ticket look unfinished.
