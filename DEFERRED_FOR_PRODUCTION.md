# Deferred for production / continuously-updated projects

> Items intentionally cut from [`project-state-setup-quickstart.md`](project-state-setup-quickstart.md)
> because they're over-engineered for portfolio pieces and personal
> tools — but become necessary as a project graduates to production,
> gains team members, or becomes continuously updated long-term.
>
> Reach for these when the **revisit criteria** in each section fire.

## Why they were cut

The 2026-05-20 GridPulse wider replan (after multi-perspective review)
trimmed the original 4-PR plan because too much PM infrastructure was
being built ahead of demonstrated need. The cuts below were the right
call for a single-developer portfolio project — but each has a clear
trigger condition where reintroducing it becomes worth the build cost.

---

## §1 — Doc-drift CI workflow

**What it is**: A `scripts/docs/check_doc_drift.py` + pre-commit hook + GitHub Actions workflow that greps the repo for known-stale phrases (e.g., outdated counts, deprecated config keys) and fails the PR if any match.

**Why cut**: At single-developer velocity with the agent doing per-PR updates, mechanical grep checks catch ~60% of drift cases that the agent would catch anyway. The remaining 40% catches don't justify the 30-minute build cost yet.

**Revisit when**:
- A second human (team member, contractor) starts committing
- Drift is observed twice in the same month despite the per-PR `CLAUDE.md` ritual
- The project's doc surface area exceeds ~10 files actively cross-referencing each other
- The CANONICAL_FACTS.md file accumulates >20 facts that consumers cite

**Implementation pointer**: ~30 lines of Python plus a `.pre-commit-config.yaml` and `.github/workflows/doc-drift.yml`. Pattern list is project-specific — populate from observed stale-string PRs (e.g., GridPulse's `wattcast:`, `~64`, `8 BAs`).

---

## §2 — Audit workflow (monthly cron + PR-count trigger)

**What it is**: A `.github/workflows/quarterly-doc-audit.yml` (or monthly/weekly based on velocity) that creates a GitHub issue on a schedule. A companion `.github/workflows/pr-count-audit-trigger.yml` that creates an issue after every N merged PRs at high velocity. The user invokes `/audit-docs` in Claude Code to work the issue.

**Why cut**: At a single developer's session cadence, the agent's `End-of-PR explanatory-doc check` in CLAUDE.md catches most drift inline. The audit becomes valuable when there are weeks between sessions or when multiple sessions per day diverge on doc state.

**Revisit when**:
- Two consecutive months pass without the agent ever flagging a doc-impact check failure (means inline triggers aren't firing — need backstop)
- Multi-day gaps between Claude Code sessions become routine (audit is the only thing that catches drift during silence)
- Project reaches production: real users mean stale docs cost more (support load, onboarding errors, incident misrouting)
- Multi-repo cross-references accumulate (one repo's CANONICAL_FACTS gets cited from another)

**Implementation pointer**: GitHub Actions YAML in §4 of [`project-state-setup-full.md`](project-state-setup-full.md). Includes the open-audit-issue de-dup guard so vacation periods don't create duplicate issues.

---

## §3 — STATUS.md schema validation in CI

**What it is**: A `scripts/validate_status.py` that parses STATUS.md, asserts every referenced issue exists and is open, every PR link resolves, the YYYY-MM-DD date is parseable and not >14 days stale, and the required sections exist. Wired into CI on every PR.

**Why cut**: Untrusted multi-actor environments need integrity enforcement. Single-developer + agent-assisted updates have lower drift risk because the agent maintains the file as part of its workflow.

**Revisit when**:
- A second human starts committing (untrusted from the file's perspective — they might write malformed content)
- STATUS.md is shared with stakeholders outside the immediate dev loop (recruiters, investors, team leads who read it directly)
- After observing the first instance of a broken issue reference or wrong-format date in STATUS.md (one instance is forgivable, two is a pattern)

**Implementation pointer**: ~60 lines of Python parsing the Markdown headings + extracting `#NNN` issue refs and PR URLs, calling `gh issue view` / `gh pr view` to verify each. Add to CI as a check that gates PR merge.

---

## §4 — Pre-push hook for STATUS.md freshness

**What it is**: A `.git/hooks/pre-push` (or pre-commit) that fails if the branch has commits touching code AND STATUS.md hasn't been touched in this session (or in the same commit as material work).

**Why cut**: This was the most aggressive enforcement option discussed during the wider replan. Single-developer + agent-driven model has high alignment without hard enforcement — the agent self-imposes via CLAUDE.md. A hook adds friction (false positives on trivial PRs) for a problem we don't yet have.

**Revisit when**:
- Multiple humans commit to the same branch — they may not see CLAUDE.md or follow its instructions
- "STATUS.md is stale" has been called out 2+ times in code review
- The project reaches a maturity where freshness matters (production app with on-call rotation, customer-facing release notes derived from STATUS.md)

**Implementation pointer**: Shell hook. Look at `git diff --name-only HEAD~..HEAD` for material changes (non-test, non-docs); fail if STATUS.md not in the changed set. Allow override via `--no-verify` for genuine docs-only PRs.

---

## §5 — GitHub Project advanced configuration

**What it is**: Custom views per tier (e.g., "Path B Open", "V3 Backlog"), board grouping by labels, custom fields for `Effort` and `Strategic Tier` (in addition to labels), saved filters, README on the Project itself, automation rules (e.g., "PR closed → issue moves to Done").

**Why cut**: At 2 items on the board (GridPulse 2026-05-20 state), default Status field + labels is sufficient. Configuring views is busywork until there are enough items to navigate.

**Revisit when**:
- Project board has ≥10 items (the threshold where flat list becomes painful to navigate)
- Multiple roadmap tiers are simultaneously active (e.g., V3 + V4 Path B + maintenance items all open at once)
- Stakeholders/team members are using the board for status visibility (not just the solo developer)

**Implementation pointer**: GitHub Projects v2 UI for view config. Custom fields via `gh project field-create`. Automation rules via Projects v2 workflows tab. ~30 min to set up once the board is rich enough.

---

## §6 — `make status` / tool-agnostic fallback

**What it is**: A `Makefile` target (or shell alias in repo `README.md`) that runs the same three commands as the CLAUDE.md pre-session ritual: `cat STATUS.md && gh pr list && gh issue list`. Lets you run the ritual without Claude Code.

**Why cut**: Coupled to a hypothetical: "what if you switch off Claude Code." If you're still using Claude Code, the ritual fires automatically via the agent reading CLAUDE.md. Building `make status` ahead of need is speculative.

**Revisit when**:
- You start using a different agent regularly (Cursor, Aider, plain ChatGPT) for the project
- You want a non-agent path to the ritual (e.g., reviewing project state on mobile via SSH)
- Team members need the ritual but don't use Claude Code

**Implementation pointer**: 5 lines in a Makefile, plus a README snippet documenting the `make status` command. Trivial — defer until needed.

---

## §7 — Multi-tenant / cross-project linking

**What it is**: A user-level GitHub Project that pulls issues from multiple repos. Visual board showing "what's open across all my projects." Maintained at the user level via `gh project link-repo` (when available).

**Why cut**: Only one repo (GridPulse) has the system as of 2026-05-20. Cross-linking is meaningless until at least 2 repos have their own STATUS.md.

**Revisit when**:
- ≥2 repos have STATUS.md + GitHub Projects setup
- You catch yourself manually checking multiple Project boards in succession
- Cross-project dependencies emerge (e.g., portfolio-v2 case study depends on GridPulse architecture state)

**Implementation pointer**: GitHub Projects v2 supports cross-repo items natively. Either create a single user-level Project that includes all repos' issues, or link existing per-repo Projects via a custom dashboard view.

---

## §8 — Incident response for the PM system itself

**What it is**: A documented runbook for: "STATUS.md got corrupted / wrong content / merge conflict." Recovery steps, rollback to last-known-good, log of past corruptions to spot patterns.

**Why cut**: Zero incidents observed. Building a runbook ahead of any incident is over-engineering.

**Revisit when**:
- First incident (STATUS.md committed with wrong content, merged STATUS.md from a stale branch overwriting truth, etc.) happens
- Project reaches a maturity where the PM system itself has SLOs

**Implementation pointer**: 1-page runbook in `docs/RUNBOOKS/pm-system.md` with sections: symptoms, immediate response, recovery, post-incident review. Patterns from `engineering:incident-response` skill apply.

---

## How to use this file

When considering reintroducing a deferred item, the agent should:

1. Check the **revisit criteria** for that item — has the trigger condition actually fired?
2. If yes, scope a small standalone PR to add just that item. Don't bundle.
3. Update this file: move the item to a new `## Reintroduced` section with the PR link and the trigger condition that prompted it.
4. Verify the trigger condition is documented in STATUS.md or NEXT_UP.md so future-you knows why this exists.

If reaching for an item that's NOT in this list, that's a signal to update this list — append a new section documenting what you're considering and why, even before implementing.
