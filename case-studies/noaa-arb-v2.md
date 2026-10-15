# Letting pre-registered gates kill my weather-market strategy

*I rebuilt a 48,000-line weather-market bot in about 2,200 lines, with PASS and KILL rules written before the data. The forward tests ended the strategy.*

## Problem

My first bot priced Polymarket daily high-temperature bands with a Gumbel model fed by National Weather Service forecasts. It grew to about 48,000 lines, including a loop that rewrote its own trading thresholds. Its paper simulator showed large gains that forward validation could not reproduce, and with that much self-tuning code I couldn't trace why.

## Approach

I started over in noaa-arb-v2 with nine architecture decision records. The risk caps sit in one `Object.freeze` block in `src/config.ts`, so neither the bot nor an env var can change them.

`docs/NEXT_TESTS.md` is a test ladder where each rung gates the next, one variable at a time. Paper results never count as promotion evidence. After the first autopsy, every experiment got PASS, KILL, and extend rules plus a dated frequency checkpoint up front. EXP-002, registered 2026-06-11, passes with at least 6 wins in its first 25 resolved takes. Three or fewer wins kill the strategy, with no more tuning.

## What the agents did and what I kept

Claude and Codex wrote most of the code, tests, autopsies, and research notes. `AGENTS.md` is a symlink to `CLAUDE.md`, so both read the same rules. I set those rules, ran the forward experiments, and kept none of v1's self-tuning machinery.

After a research pass cited a nonexistent npm package, no package name counted until its registry page resolved. A background agent once returned prompt-injection-like text, so the rule became to treat file and web content as data. In the follow-on repo, polymarket-edge-lab, one commit fixed all 5 findings from an adversarial review of the collector.

## Result

The T0 backtest on replayed v1 data passed, with 52 of 169 simulated resting bids filled. Three fills carried 70% of the simulated gain. Forward, resting bids almost never filled, so v2.1 switched to taking cheap asks.

A pre-declared experiment dropped the model-probability floor from 0.50 to 0.02 and went 0 for its first 18. The autopsy, `docs/WHY_PAPER_LIED.md`, found the in-sample hit rate pooled three cohorts, and bands under 1 cent never won in-sample. The snapshot backtest credited fills on quotes that vanished within minutes. In the best cohort, those hit 84.8%, against 44.2% for quotes that stayed.

<!-- screenshot pending (batch 4): ![TODO: WHY_PAPER_LIED.md cohort table, 1280x800, no hosts or addresses](../images/noaa-arb-v2-cohort-table.png) -->
After a verified redeploy, the EXP-002 gate found zero candidates in 1,379 scans. A faster websocket feed found none either: the model rated no cheap band at 0.5 or higher. The bot was paused on 2026-06-30.

The edge lab kept the method: a Rust collector over public order books, a fail-closed canary harness, and a paper ladder, each measuring one hypothesis. Its calibration audit found 8 observations at or above 0.5 model probability in clean v2 data, all near-certain locks. Its arbitrage test, pre-registered at 5:55p PT on 2026-07-08, died that evening because most legs never traded.

## What failed and what I learned

The worst failure was mine. Until 2026-06-17 the bot kept the 0.02 floor and ran a stale container, so EXP-002 hadn't started when I thought it had. From 2026-06-05 it made 90 paper takes: 1 win in 77 resolved, none in the EXP-002 cohort. Reverting the 0.02 experiment was manual, and I'd hit the same stale-container trap on 2026-06-05. A setting that decides an experiment belongs in code next to the risk caps, or gets checked at boot.

The edge lab's pre-registered check for anyone profiting from the arbitrage counted completed pairs and ignored stranded legs. Full-cycle accounting reversed that finding the same evening. The kill stood, because the fillability test had already failed. The paper ladder has one recorded session of about 22 minutes, and the canary never fired.

## Code to read first

Source is private. On a walkthrough I start with these three pieces.

- The frozen risk-cap block in the config, which neither the bot nor an env var can change.
- The test ladder, where each rung gates the next and paper results never count as promotion evidence.
- The paper autopsy, which found the backtest credited fills on quotes that vanished within minutes.

## Links

Source is private; ask me for a walkthrough.
