# Auction replay: testing when a fantasy auction premium pays off

*I replayed my 14-team fantasy football auction in standard-library Python to test when paying above value beats waiting, and the model couldn't prove it ever does.*

## Problem

I kept making the same auction mistake: wait for bargains, miss the top tier, then pay more for second-tier players. In my 2025 Yahoo auction, four receivers I bought for $53 combined scored 308.40 season points. One receiver who went for $49 scored 310.50 from a single roster slot. (These are auction dollars from a fixed draft budget, not real money.)

Paying above a tool's value can still be right if the players left after passing cost more for less. Season totals are hindsight, though. They can't say what a bid was worth on draft night, so I wanted to replay the auction with only draft-time inputs.

## Approach

The replay walks the auction one nomination at a time. Before each sale it freezes keepers, earlier sales, and each team's cash. It never sees the closing price or any later sale. It estimates future prices only from earlier sales and remaining cash.

At every legal bid, it compares two branches, buy and pass, by the best complete, affordable 14-player roster each leaves. A branch-and-bound search finds feasible rosters, and the best one found sets a lower bound. A positional dynamic program that ignores bye weeks sets an upper bound on what any completed roster could score. Buying is supported only when its lower bound beats the pass branch's upper bound. Overlapping bounds stay "unresolved."

Two tests in `test_replay_auction.py` matter most. A brute-force oracle enumerates every roster and lineup on small cases and checks the optimizer against it. The second is an executable no-leakage check. It rewrites weekly outcomes, edits the auction file's outcome columns, and adds Week 18 values, then asserts the recommendation SHA-256 hash doesn't change.

## What I did and what the agents did

The project took seven commits on September 5 and 6, 2026, run through the Compound Engineering loop: plan, work, review, compound. I brought the problem and the league data and made the product calls. The plan records the central one as mine: compare complete rosters after buying versus passing, instead of a static fair-price list. Agents drafted the plan, including ten numbered technical decisions, and wrote the code and tests under my approvals.

On September 7, a code review of a later draft-day budget worksheet ran seven reviewer personas and returned "Ready with fixes." Its JSON receipt shows all four accepted findings resolved.

## Result

The full run covers all 95 of my eligible nominations across nine price and performance scenarios: 63,729 legal-price comparisons in 22.38 minutes.

| Verdict | Share of rows |
| --- | ---: |
| Buy supported | 5.43% |
| Pass supported | 36.48% |
| Unresolved | 58.09% |

No supported buy went above either baseline price ceiling, static or inflation-adjusted. For the $49 receiver, the neutral scenario supports bids through $23 and leaves a $50 bid unresolved. The README's third paragraph says the same: "better bidding remains unproven." An offline HTML explorer shows each branch's roster and bounds.

<!-- screenshot pending (batch 4): ![TODO: room replay showing buy and pass roster bounds for one nomination, 1280x800, pseudonymized team labels, no personal data](../images/auction-replay-bounds.png) -->
## What failed and what I learned

The premium-bidding edge I set out to find is unproven. The bounds are too loose near realistic prices. The 2025 season also shaped the model, so I need a held-out season before claiming an edge.

Review caught a bug. The positional calculation missed a FLEX quota when the candidate pool lacked one eligible position. (FLEX is a starting slot that a running back, receiver, or tight end can fill.) After the fix, a full league rerun reproduced all 942 recommendation records exactly.

The compound step, where lessons get written down, never happened, and `AGENTS.md` pointed new sessions at a lessons folder that was never created. The draft-day worksheet and its review receipt were left uncommitted. The headline numbers depend on a private data cache, so a clone runs the tests but can't reproduce them.

The search code has a note to add tighter per-node bounds only if measured unresolved rates require them. At 58.09% unresolved, I think they do, so that's next.

## Code to read first

If you have five minutes, read these three files in order.

- [src/auction_replay/replay_auction.py](https://github.com/Boharrisonabrams/auction-replay/blob/main/src/auction_replay/replay_auction.py): the replay model that freezes draft-time inputs and compares buy and pass roster bounds.
- [tests/test_replay_auction.py](https://github.com/Boharrisonabrams/auction-replay/blob/main/tests/test_replay_auction.py): the brute-force oracle and the no-leakage hash check, the two tests that matter most.
- [docs/plans/2026-09-05-1630-feat-league-specific-auction-replay-plan.md](https://github.com/Boharrisonabrams/auction-replay/blob/main/docs/plans/2026-09-05-1630-feat-league-specific-auction-replay-plan.md): the plan that records the buy-versus-pass roster comparison as my call, plus ten numbered technical decisions.

## Links

- Code: [auction-replay on GitHub](https://github.com/Boharrisonabrams/auction-replay)
