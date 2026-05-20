# Setup project state management — full spec

> **Read [`project-state-setup-quickstart.md`](project-state-setup-quickstart.md) first.**
> This file is the longer reference for cases where the quickstart isn't
> sufficient (multi-repo, production apps, team settings, long-running
> projects, edge cases).
>
> Use this file when the agent should investigate deeply and adapt per
> the §6 variant table.

## §1 — What you're building (conceptual)

A sustainable single source of truth for project state, replacing the
typical sprawl of memory + plan docs + scattered issues + PR archaeology.

Four layers:

| Layer | What it holds | Drift mode | Update mechanism |
|---|---|---|---|
| **Canonical state** | "what's open / in flight / done" | High velocity drift | GitHub Issues + Project (auto-updates from PR activity) |
| **Narrative anchor** | "what am I working on, why, recent decisions" | Medium drift | `STATUS.md` at repo root, updated per-PR |
| **Explanatory docs** | "how it works, how to pitch it, interview stories" | Slow drift | `HOW_IT_WORKS.md`, `PITCH.md`, `INTERVIEW_PREP.md` (Phase 3, optional) |
| **Drift checks** | "this doc claim no longer matches reality" | Mechanical | `scripts/docs/check_doc_drift.py` + CI (Phase 2) |

This spec covers **Phase 1** (canonical state + narrative anchor). Phases
2–4 are follow-up PRs after Phase 1 settles. For production-grade
projects, also see [`DEFERRED_FOR_PRODUCTION.md`](DEFERRED_FOR_PRODUCTION.md).

## §2 — Pre-flight investigation

Before asking the user anything, the agent should run:

```bash
# 1. Is this a Git repo with a GitHub remote?
git rev-parse --is-inside-work-tree
git remote get-url origin

# 2. Does CLAUDE.md exist? Read it.
test -f CLAUDE.md && cat CLAUDE.md | head -50

# 3. Existing roadmap / plan / status file?
ls -la STATUS.md NEXT_UP.md ROADMAP.md PLAN.md docs/ 2>/dev/null
find . -maxdepth 3 -iname "*roadmap*" -o -iname "*next_up*" -o -iname "*status.md" 2>/dev/null

# 4. GitHub state
gh issue list --state open --limit 20
gh pr list --state open
gh pr list --state merged --limit 10

# 5. Project stack / shape
ls package.json pyproject.toml Cargo.toml go.mod 2>/dev/null
cat README.md | head -30 2>/dev/null

# 6. Recent velocity
gh pr list --state merged --limit 50 \
  --json mergedAt --jq '[.[] | .mergedAt[:10]] | group_by(.) | length'

# 7. Required gh scopes
gh auth status
```

Report findings to the user:
- Repo type (Python / Node / Rust / etc.)
- Existing roadmap structure and whether it's current
- Open work currently in GitHub
- Velocity bucket (low <3 PRs/week, medium 3–10, high 10+)
- Missing `gh` scopes (`project` is the common gap)

## §3 — Questions to ask the user

Ask as one batch via `AskUserQuestion` if available.

1. **Roadmap structure?** (V0–V4 phases / quarterly OKRs / linear backlog / none).
2. **Audience class?** (portfolio piece / personal tool / production app with real users / internal work). Determines Phase 3 applicability.
3. **Active investment right now?** Free-form.
4. **Next 3 priorities, in order?** Agent will create issues for committed ones; speculative stay as STATUS.md bullets.
5. **Recent decisions (last 7–14 days) worth recording?**
6. **Open strategic question?** Optional.
7. **Velocity tier?** Confirms agent's guess from PR data.
8. **Existing open issues to migrate?** Agent shows the list; user picks which to label.

## §4 — Execute in order

### Step 1: Labels

Universal:
```bash
gh label create "effort-day" --description "≤1 day of focused work" --color "c5def5"
gh label create "effort-week" --description "~1 week" --color "fbca04"
gh label create "effort-weeks" --description "Multiple weeks" --color "d93f0b"
gh label create "doc-audit" --description "Periodic audit issue" --color "006b75"
```

Tier labels — depend on roadmap structure (Q1):
- V0–V4 phases → `vX-open` per active phase
- Quarterly OKRs → `okr-q1`, `okr-q2`, etc. (current quarter only)
- Linear backlog → skip tier labels; recommend `priority-now`, `priority-next`, `priority-later`
- None → skip tier labels; use `effort-*` only

Area labels only if the project's existing PR titles already use that vocabulary (e.g., `area:ui`, `area:infra`).

### Step 2: Issues for committed open work

For each Next-3 item that's *actually committed* (not speculative):

```bash
gh issue create --title "<title>" --label "<tier>,<effort>" --body "$(cat <<'EOF'
## Context
[from user's description]

## Acceptance
[bulleted checklist]

## Effort
[hours/days/weeks]

## Files
[likely files to touch, if known]
EOF
)"
```

**Do not** create issues for speculative work.

### Step 3: STATUS.md

Use the template in §5. Populate every section from user answers.

### Step 4: CLAUDE.md

If missing, create minimal (template in §5). Either way, add the two ritual sections near the top and add `STATUS.md` as item #1 in the reading order.

### Step 5: GitHub Project

```bash
gh project create --owner <user-or-org> --title "<Project Name> Roadmap"
# Capture project number from output
gh project item-add <num> --owner <user-or-org> --url <issue-url>
```

If scope missing: report `gh auth refresh -s project` in PR body, document the manual commands, don't block.

### Step 6: PR

Single PR. Branch: `chore/pm-state-baseline`. Title: `chore(pm): introduce STATUS.md + pre-session ritual as canonical state`. Body documents what changed, what GitHub state was created, what's deferred.

## §5 — Templates

### STATUS.md

```markdown
<!--
How this file gets maintained:
- Per-PR: updated in the same commit as material work
- End-of-session: agent re-verifies against gh
- Pre-external-use: user re-reads top-to-bottom
If this file disagrees with gh, gh wins — patch in a follow-up commit.
-->

# Status — updated YYYY-MM-DD

> Canonical pointer. This file + [Project board](<url>) + issues are
> the single source of truth.

## Active focus + open question

[1–3 sentences on active investment / strategic position.]

**Open question — success criterion (by YYYY-MM-DD):**
[Strategic question as a 14-day measurable.]

## Next 3 (priority order)

1. **[#NNN — Title](issue-url)** (~effort, labels). Description.

## Blocked / waiting on

- Item — waiting for [trigger]

## Recent decisions (last 7 days)

- **YYYY-MM-DD** — Decision name. [PR link]
```

### CLAUDE.md sections (add near top)

```markdown
## Before recommending what's next

Don't rely on memory. Always run at session start:

\`\`\`bash
cat STATUS.md
gh pr list --state open
gh issue list --state open
\`\`\`

If STATUS.md disagrees with gh, GitHub wins.

## End-of-PR explanatory-doc check

For any non-trivial PR, before reporting "done":

1. Architecture changed? → update HOW_IT_WORKS.md + diagrams
2. A cited fact moved? → update CANONICAL_FACTS.md
3. STAR-story trigger? → draft in INTERVIEW_PREP.md
4. STATUS.md focus/next-3/open-question changed? → update STATUS.md

Otherwise: "no explanatory-doc impact."
```

### Minimal CLAUDE.md (if creating from scratch)

```markdown
# CLAUDE.md — Project Conventions for [PROJECT_NAME]

[ritual sections from above]

## Start here

1. `STATUS.md` — current focus + decisions (canonical state)
2. `README.md` — public framing
3. This file — agent conventions

## Project shape

[1–2 sentences: stack, purpose, audience]

## Code standards

[Linter, type hints, commit format, test command]
```

## §6 — Per-project adaptations

| Project type | STATUS.md tweak | CLAUDE.md tweak | Phase 3 | Audit cadence |
|---|---|---|---|---|
| **Portfolio piece** | Include interview-readiness in Active focus | Reference INTERVIEW_PREP | Yes — full HOW_IT_WORKS + PITCH + INTERVIEW_PREP | Monthly during build, quarterly stable |
| **Personal tool** | Active investment can be "none" indefinitely | Skip INTERVIEW_PREP | Skip — or just HOW_IT_WORKS for future-you | Quarterly |
| **Production product** | Add production-status line (incidents, SLOs) | Add runbook / on-call link | Yes — pitch for stakeholders | Monthly minimum; weekly if active incidents |
| **Internal / work** | Stakeholder updates in Recent decisions | Reference team norms | Skip | Quarterly with team-sync alignment |
| **Brand-new (greenfield)** | "Setting up infrastructure" as active | Minimal — 2 ritual sections only | Defer until v0.1 | Defer until Phase 3 |
| **Monorepo** | One per top-level project + root linking | Single CLAUDE.md at root | Per-project | Per-project, independent |

## §7 — Calibrating audit cadence (Phase 4)

| Velocity | Initial cadence | Notes |
|---|---|---|
| Low (<3 PRs/week) | Quarterly (`0 14 1 */3 *`) | Drift slow. Mostly verification. |
| Medium (3–10 PRs/week) | Monthly (`0 14 1 * *`) | Standard. Most personal projects. |
| High (10+ PRs/week) | Monthly + PR-count trigger (every 20 merges) | Build phase. Two backstops needed. |

Auto-tune after 2–3 audits:
- <5 items twice → loosen one tier
- 15+ items → tighten one tier
- Nothing → too tight, loosen

## §8 — Follow-up PRs

This spec sets up Phase 1 only. Optional follow-ups (only if pain emerges):

- **PR-B: Doc-drift checks** — `scripts/docs/check_doc_drift.py` + pre-commit + CI. ~30 min/project.
- **PR-C: Explanatory docs Phase 1** (portfolio/production only) — HOW_IT_WORKS + 5 Mermaid diagrams + PITCH + INTERVIEW_PREP with STAR stories drawn from real PRs.
- **PR-D: Audit workflow** — see [`DEFERRED_FOR_PRODUCTION.md`](DEFERRED_FOR_PRODUCTION.md).

## §9 — What NOT to do

- Don't copy project-specific labels (e.g., `path-b`) without matching to the new project's vocabulary
- Don't create speculative issues — if not committed, leave as STATUS.md bullet
- Don't propose Notion as primary state tracker (same failure mode as hand-maintained Markdown)
- Don't add custom Project fields when issue labels suffice
- Don't create the GitHub Project before `gh auth refresh -s project` — fail gracefully
- Don't assume CLAUDE.md exists or has the same structure across projects

## §10 — Success criteria

Phase 1 setup is successful when:

- [ ] `STATUS.md` exists at repo root and accurately describes current state
- [ ] `CLAUDE.md` has both ritual sections + reading-order update
- [ ] ≥1 GitHub issue exists for currently committed open work (if any)
- [ ] Labels exist: ≥ `effort-day` / `effort-week` / `effort-weeks` / `doc-audit`
- [ ] Single PR opened with file changes
- [ ] PR body documents GitHub state + deferred manual steps
- [ ] Agent time: ≤60 min existing repo, ≤30 min greenfield

If a step took materially longer, note in PR body so this spec can improve.
