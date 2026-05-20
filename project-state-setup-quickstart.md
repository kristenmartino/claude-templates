# Setup project state management — quickstart

> **How to use**: open Claude Code in the target repo and paste this
> entire file into the chat with the message:
>
> *"Set up project state management per this spec. Investigate the repo
> first, ask me the questions, then ship a single PR."*

## Investigate first (agent runs these before asking anything)

```bash
git rev-parse --is-inside-work-tree && git remote get-url origin
test -f CLAUDE.md && head -50 CLAUDE.md
ls STATUS.md NEXT_UP.md ROADMAP.md PLAN.md docs/ 2>/dev/null
gh issue list --state open --limit 20
gh pr list --state open
gh pr list --state merged --limit 50 --json mergedAt --jq '[.[] | .mergedAt[:10]] | group_by(.) | length'
gh auth status   # check for `project` scope
```

Report a brief findings summary (repo type, existing roadmap, open work,
recent velocity bucket, missing `gh` scopes) before asking questions.

## Ask the user

1. **Roadmap structure?** (V0–V4 phases / quarterly OKRs / linear backlog / none)
2. **Audience class?** (portfolio piece / personal tool / production app / internal-work)
3. **Active investment right now?**
4. **Next 3 priorities, in order?**
5. **Recent decisions (last 7–14 days) worth recording?**
6. **Open strategic question?** (optional)
7. **Velocity tier?** (low <3 PRs/wk, medium 3–10, high 10+)

## Then ship a single PR

### Files

- `STATUS.md` at repo root using the template below
- `CLAUDE.md` (create if missing) with two sections from the template
- Reading-order list in CLAUDE.md updated so `STATUS.md` is item #1

### GitHub state

- Labels: `effort-day`, `effort-week`, `effort-weeks`, `doc-audit`,
  plus per-project tier labels from question 1 (e.g., `path-b`,
  `v3-open`, `okr-q1`)
- Issues created for committed items in Next 3 (skip speculative —
  those stay as bullets in STATUS.md until committed)
- GitHub Project created at user-level (cross-repo capable):
  `gh project create --owner <user> --title "<Repo> Roadmap"`
- Add issues to Project: `gh project item-add <num> --owner <user> --url <issue-url>`

### PR body documents

- Files changed and why
- GitHub state created (labels, issues, project URL)
- Anything deferred (e.g., gh scope missing)
- Net effect for the user ("your where-am-I ritual is now: cat STATUS.md / gh pr list / gh issue list")

Time budget: **60 min existing repo, 30 min greenfield.** Anything over → flag.

## STATUS.md template

```markdown
<!--
How this file gets maintained:
- Per-PR: updated in the same commit as material work that changes
  active focus, next-3, blocked-on, or recent decisions
- End-of-session: agent re-verifies against gh issue list / gh pr list
- Pre-external-use: user re-reads top-to-bottom
If this file disagrees with gh, gh wins — patch in a follow-up commit.
-->

# Status — updated YYYY-MM-DD

> Canonical pointer for "where am I, what's next." This file + [GitHub
> Projects board](<project-url>) + the issue tracker are the single
> source of truth for project state.

## Active focus + open question

**[1–3 sentences on the active investment or strategic position.]**

**Open question — success criterion (by YYYY-MM-DD):**
[The unresolved strategic question framed as a 14-day measurable.]

## Next 3 (priority order)

1. **[#NNN — Title](issue-url)** (~effort, labels). Description.
2. ...
3. ...

## Blocked / waiting on

- Item — waiting for [trigger condition]

## Recent decisions (last 7 days)

- **YYYY-MM-DD** — Decision name. [PR link]
- ...
```

## CLAUDE.md sections (insert near the top)

````markdown
## Before recommending what's next

Don't rely on memory or stale docs. Always run a state check at session
start before suggesting work:

```bash
cat STATUS.md                # active focus + recent decisions + open question
gh pr list --state open      # in-flight work
gh issue list --state open   # committed queue
```

If `STATUS.md` contradicts what `gh` reports, **GitHub wins** — patch
STATUS.md in the same session.

## End-of-PR explanatory-doc check

For any non-trivial PR, before reporting "done":

1. **Architecture changed?** → update HOW_IT_WORKS.md + diagrams (if they exist)
2. **A cited fact moved?** → update CANONICAL_FACTS.md (if it exists)
3. **STAR-story trigger hit?** (trade-off, debugging, surprising decision)
   → draft in INTERVIEW_PREP.md (if it exists)
4. **STATUS.md active focus, next-3, or open question changed?**
   → update STATUS.md in the same PR

Otherwise report: "no explanatory-doc impact."
````

## What NOT to do

- Don't copy specific labels from other projects (e.g., GridPulse's
  `path-b`) without matching them to this project's actual roadmap vocabulary
- Don't create speculative issues — if not committed to, leave as STATUS.md text
- Don't propose Notion or similar SaaS as primary state tracker
- Don't add custom Project fields when issue labels suffice
- Don't ship the GitHub Project before `gh auth refresh -s project` — surface the manual step

## When to escalate to the full spec

See [`project-state-setup-full.md`](project-state-setup-full.md) if any of:
- Multi-repo / monorepo
- Production application with real users
- Team setting (more than one human committer)
- Continuously updated long-term (>6 months active development expected)
- The 7 questions above don't fit cleanly
