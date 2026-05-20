# claude-templates

Reusable Claude Code setup specs and per-project templates. Cross-cutting
infrastructure for personal projects — pasted into Claude Code in a new
or existing repo to bootstrap project-state management, doc maintenance
discipline, and explanatory-doc scaffolding consistently across projects.

## Contents

| File | Use when | Reading time |
|---|---|---|
| [`project-state-setup-quickstart.md`](project-state-setup-quickstart.md) | New or existing repo. Want STATUS.md + GitHub Projects + CLAUDE.md ritual. **Default — start here.** | 2 min |
| [`project-state-setup-full.md`](project-state-setup-full.md) | Same goal but want the full spec with project-type variants, calibration tables, and edge cases. Use for projects that will be production apps or continuously updated long-term. | 10 min |
| [`DEFERRED_FOR_PRODUCTION.md`](DEFERRED_FOR_PRODUCTION.md) | A project graduates from "portfolio piece" to "production app" or starts being continuously updated. Revisit these items that were intentionally cut from the lightweight setup. | 5 min |

## How to use

In any repo where you want to bootstrap project-state management, open
Claude Code and paste the contents of `project-state-setup-quickstart.md`
into the chat. The agent investigates the repo, asks ~7 questions, and
ships a single PR.

## Why this repo exists

Personal projects are scattered: GridPulse (energy forecasting), sift-news
(news aggregator), portfolio-v2 (portfolio site), prioritize-repo, plus
new projects spinning up. Each had been getting project-management
ad-hoc — memory, scattered docs, GitHub issues that fell stale, plan
files that drifted from reality.

This repo holds the shared infrastructure for that problem. Patterns
proven on one project (GridPulse, 2026-05-19 → 2026-05-20) get
generalized here so subsequent projects don't reinvent them.

## Provenance

Patterns originally established for [GridPulse](https://github.com/kristenmartino/gridpulse).
See [GridPulse PR #123](https://github.com/kristenmartino/gridpulse/pull/123)
for the first instance and the wider-replan review that shaped what
ended up in `project-state-setup-full.md` vs `DEFERRED_FOR_PRODUCTION.md`.
