# Brokerage tools for an agent, and five pre-registered kills

*I gave a Claude Code session a broker's official MCP tools, with write access fenced to one account, and ran a frozen-gate research loop that killed all five of my strategy ideas.*

## Problem

I wanted an agent to research retail US-equity strategy ideas while connected to a brokerage through Robinhood's official agentic MCP server. Two problems followed: how much damage the agent could do, and me. A backtest can look good for bad reasons, and anyone who wants an idea to work will keep tuning until it does.

## Approach

For Gate A, the security gate in `docs/SECURITY-VERIFICATION.md`, the agent enumerated 22 tools and found no transfer, withdrawal, or deposit tools. It then called `review_equity_order`, the order-preview tool, against my main account. The server answered "FORBIDDEN: agent not authorized to access this account." The fence lives on the broker's side. The test used the preview call, not a live order. I never allowlist the order tools, so every order call stops for my click. Sessions with trading tools never read raw web content; subagents do that work.

The research loop: freeze the gate, commit it, run once, no rescue. Each variable gets frozen kill thresholds before the agent sees any results, and the evaluators (`backtest/gate*.py`) live apart from the strategy code. A tweaked variant is a new variable at the back of the queue.

## What the agent did and what I decided

Claude wrote the specs, the backtest code, 19 tests on synthetic data, and the reports. For variable 5, it launched evidence subagents before writing the pre-registration, and that document records that their results were still unread when the gate was frozen. The variable 5 report also has a claims audit that tags each load-bearing claim by where it came from, including the ones that stayed unverified.

I made the binding calls. I picked the first variable, approved the security, edge, and validation gates, ruled out WallStreetBets sentiment for good, banned IPO-day trades, and chose to kill rather than rescue after every verdict. The project skill puts it this way: "The product of this project is the idea-rejection rate."

## Result

All five died at their gates: earnings-reaction continuation, insider cluster buying, 12-1 price momentum, a Robintrack crowding veto, and an LLM portfolio manager built on the open-source ai-hedge-fund project.

The loop also caught two problems that would have fooled me. Variable 1's raw mean of +0.90% per event was all market beta. After the IWM benchmark adjustment and modeled costs, it was -0.11% (t-stat -0.68, n = 8,796). In the same run, a data vendor's holiday filler rows poisoned the rolling average-daily-volume windows. The tell was that adding events cut the qualified count from 7,812 to 4,845. The fix became a regression test.

Variable 5 failed all four frozen gates without ai-hedge-fund ever running. A contamination rule counts any backtest that overlaps the model's training window as zero evidence. The power math put an honest forward test at about 25 to 69 years, against a two-year limit.

![TODO: STATUS.md verdict table cropped to the five verdict rows, 1280x500, no account lines](../images/robinhood-verdicts.png)

## What failed and what's still open

The repo's git history proves gate-before-data for one verdict out of five. On 2026-06-24 I reset the history to drop 4.3 GB of committed data files, and the new history starts after the verdicts for variables 1 to 4. For those four, the order rests on what the docs say. For variable 5, git shows the frozen gate committed at 19:23 and the verdict at 20:26 that evening. I'd rather say that than imply all five carry equal weight.

![TODO: git log --oneline showing the variable 5 pre-registration and verdict commits, 1280x400, no personal data](../images/robinhood-v5-git-log.png)

The freeze is also weaker than the Gate C spec says. The spec claims the engine refuses modified parameters. No such check exists, so a docstring and my discipline enforce it. The cost model is copy-pasted into four engines.

The broker grew the tool surface from 22 to 49, including option order tools, with no change on my side. A live re-enumeration caught it, and STATUS.md still lists the Gate A re-review as open. Read access was never fenced, so the agent can read all my accounts. I accepted that in writing, but it's a real gap.

## Links

Source is private; ask me for a walkthrough.
