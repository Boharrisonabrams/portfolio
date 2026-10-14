# How I work with coding agents

*Claude and Codex write most of my code. I write the rules they work inside, and I keep the actions that are hard to undo.*

These practices repeat across my repos. Each gets one example. Most of the source is private, so I cite file names in place of links.

## Plan, work, review, compound

My default loop for a change is a written plan, the work, a review, and a write-up of what the work taught. The write-up is the step that gets skipped, so I make it a file the next session reads. In kommu's newsletter tool, the host-volunteer refresh plan carries a five-persona document review in its frontmatter, and `docs/solutions/` holds lessons such as the ones from rendering social images in a headless browser.

## Limits are code constants

If a number caps risk, an agent can't tune it. In `noaa-arb-v2`, my weather-forecast trading bot, `src/config.ts` wraps `RISK_LIMITS` in `Object.freeze`. The comment above it says a change takes a human editing the file and a code review.

## Pre-register before the data

Pre-registration means I write the pass and kill rules and commit them before anyone sees results. After that, neither I nor the agent gets to move the rule to fit the numbers. In my equities research repo, the last idea's pre-registration and its verdict are separate commits, and the project skill names the idea-rejection rate as the product. Five ideas went through that loop, and all five were killed at their gates.

## Fail closed, and keep the hard-to-undo actions

Fail closed means a gate that can't confirm its inputs blocks the action. Sends, publishes, and pull request actions wait for me. In kommu's newsletter tool, the send function raises `SendRefused` every time and the default audience matches nobody, so I send each edition by hand.

## Agent output is input

I treat what an agent writes like data from an outside source. In kommu's founder dashboard, the promote step rejects an agent-written snapshot unless its schema, freshness, and source proof check out. `sanitizeAgentMoves` in `server-app/lib/moves.ts` caps the agent's suggested moves at five and drops any move that has no owner.

## One agent drafts, another rules

Review gets two agents with different jobs. In WikiRace, Claude writes a draft at `docs/ai/review-opus.md`, and Codex writes the authoritative `docs/ai/review.md`, marking each finding ACCEPT, REJECT, or ADJUST and adding what Claude missed. The draft stays on disk unchanged, so I can see where the two disagreed.

## Repos that brief a cold agent

I want each repo to brief an agent that has never seen it. In `noaa-arb-v2`, `AGENTS.md` is a symlink to `CLAUDE.md`, so Claude and Codex read one file. It has a read order and a section on how to check state instead of re-deriving it from memory. Dated handoffs in `docs/` and `.claude/handoffs/` carry state from one session to the next.

## What broke

The frozen limits had a gap. I froze the risk caps but left `MIN_PMODEL`, the minimum model probability for an entry, as an environment variable. One experiment dropped it from the 0.50 default to 0.02 and failed. I pre-registered a gate back at 0.50, but the running bot kept 0.02 until a readout on 2026-06-17 caught it. After the fix, the pre-registered gate found nothing that met its entry rule in its first five days. The strategy never passed a forward test, and I paused the bot.

A finding sat on a branch. A self-audit I ran on kommu's newsletter tool on 2026-07-02 ranked one bug first: the readiness gate accepts any non-empty proof file. The audit lived on an unmerged branch, so agents working on main never saw it, and the bug was still open three months later.

In WikiRace, Codex's own log, `docs/ai/state.md`, records four cycles where the plan files no longer matched the work, so it built from my direct brief. Those plan files are still stale.
