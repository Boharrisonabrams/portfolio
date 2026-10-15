# kommu founder dashboard: an agent as the data pipeline

*A scheduled agent pulls data from seven business tools into one snapshot, and the app treats what the agent writes as untrusted input.*

## Problem

kommu is a home-swap marketplace I founded. To see how it was doing, I had to open seven tools: subscriptions, payments, product analytics, install attribution, a reporting layer over the production database, the issue tracker, and the support inbox. Their days start at different hours, and "active" can mean a paying subscriber or any user. I wanted one screen that never shows a number it can't back up.

## Approach

At refresh time, a scheduled agent session follows a runbook, `REFRESH.md`. It reads each source read-only over MCP (Model Context Protocol, which lets an agent call outside tools) and writes one JSON snapshot. That snapshot holds metrics, plain-language insights, and "moves": a number, why it matters, one action, and an owner. The main rule: a metric is a real number or `null`. If a source fails, its metrics stay null and the gap is written down.

At view time, a Next.js app with role-based sign-in reads the snapshot and checks it:

- `server-app/lib/snapshot-validation.ts` rejects a missing or future timestamp, an age over 36 hours, an unknown source status, period labels that don't match the timestamp, and ambiguous booking-value wording in agent text.
- `server-app/lib/moves.ts` drops any move missing one of its four fields and keeps at most five.
- The promote route fails closed. It needs an agent token or an admin session. A missing or misaligned source proof, or a failed validation, gets an HTTP 422 and nothing goes live.
- Past 36 hours, the dashboard shows a stale banner.

Later I added direct API adapters so a plain cron job can do the same pull. Each one is enabled only after its output matches an agent-produced baseline exactly, and `source-adapter-evidence.json` records the command and approval per source.

## What the agents did and what I kept

Agents typed most of the code, per the commit trailers. The repo's agent rules split the work: one agent plans and reviews, a second implements, and they hand off through tracked files in `docs/ai/`.

I kept the rules that decide what counts as true: null over a guess, fail-closed promotion, the 36-hour limit, and the build order (agent first, adapters only after they match it). I also kept the handoff log.

## Result

The hosted app went live in July 2026 with sign-in roles, Postgres snapshot storage, a refresh-run audit log, CSV export, and a daily cron. On August 4, 2026, the cron ran its first unattended morning refresh and passed with all seven direct adapters enabled.

<!-- screenshot pending (batch 4): ![TODO: dashboard overview rendered from synthetic data with the stale banner visible, 1280x800, no real kommu numbers or names](../images/kommu-dashboard-synthetic.png) -->
## What failed and what I learned

In August the agent refresh froze for four days and nothing said so. An earlier session left uncommitted files on main, and the scheduled task stops on a dirty tree, so every morning run aborted. It was the second time that had happened. A safety stop with no alarm is an outage you find by accident. Around then, auto-deploy also died: zero builds for five days while data kept flowing, so the screen looked fine. Now a deploy claim has to cite the deployment list.

One sentence in `scripts/assemble.mjs` was hardcoded prose with only the date filled in. When the fact behind it changed, the dashboard republished it as freshly checked for 18 days. The fix added a release gate that runs the real assembler over test fixtures and fails against the old code.

The git mirror of the snapshot has commits on 71 of the 125 days from June 11 to October 13, 2026, and none from September 10 to 28. A watchdog route exists, but my hosting plan allows two cron slots and both run the refresh. So during a freeze the only signal is the stale banner, and someone has to be looking at it. Fixes for a refresh hang and a deadline race sit on unmerged branches, and no app code has reached main since August 10, 2026.

Most of `verify-policies.mjs` (about 1,100 `includes(` calls) pins source text instead of testing behavior. Nothing grades whether the agent's insights are any good. The checks cover shape and wording only.

## Code to read first

Source is private. On a walkthrough I start with these three pieces.

- The refresh runbook: the scheduled agent writes a real number or `null`, and a failed source's gap is written down.
- The snapshot validator: rejects a missing or future timestamp, an age over 36 hours, and period labels that don't match.
- The promote route: fails closed, so a missing source proof or a failed validation gets an HTTP 422 and nothing goes live.

## Links

Source is private; ask me for a walkthrough.
