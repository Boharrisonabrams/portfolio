# Bo Abrams

I founded kommu, a home-swap marketplace, and I work across its product and engineering. I build with coding agents, and I keep the approvals.

## How I build

Claude and Codex write most of my code. I write the rules they work inside. Risk caps live in code where an agent can't tune them, and I commit pass or kill rules before any results come in. Sends, publishes, and pull request actions wait for me. [How I work with coding agents](methods/how-i-work-with-agents.md) gives one example of each practice and a section on what broke.

## Case studies

Every case study has a section on what failed. In two of them, rules I wrote before seeing the data killed my strategy ideas.

| Title | In one line |
| --- | --- |
| [WikiRace: a Wikipedia race where the server checks every click](case-studies/wikirace.md) | A Wikipedia racing game I shipped as a web app, an iOS app, and an API, built on one rule: the server never takes the client's word for a move. Play it at [wikirace.day](https://wikirace.day). |
| [kommu founder dashboard: an agent as the data pipeline](case-studies/kommu-dashboard.md) | A scheduled agent pulls data from seven business tools into one snapshot, and the app treats what the agent writes as untrusted input. |
| [Letting pre-registered gates kill my weather-market strategy](case-studies/noaa-arb-v2.md) | I rebuilt a 48,000-line weather-market bot in about 2,200 lines, with pass and kill rules written before the data. The forward tests ended the strategy. |
| [Brokerage tools for an agent, and five pre-registered kills](case-studies/robinhood-agentic.md) | I gave a Claude Code session a broker's official MCP tools, with write access fenced to one account, and ran a frozen-gate research loop that killed all five of my strategy ideas. |
| [Auction replay: testing when a fantasy auction premium pays off](case-studies/auction-replay.md) | I replayed my 14-team fantasy football auction in standard-library Python to test when paying above value beats waiting, and the model couldn't prove it ever does. |

## Public code

- [wikirace-public](https://github.com/Boharrisonabrams/wikirace-public): WikiRace's API, web app, and iOS app in one pnpm monorepo. Play it at [wikirace.day](https://wikirace.day).
- [auction-replay](https://github.com/Boharrisonabrams/auction-replay): the auction replay in standard-library Python, with its tests.

Case studies without a public code link describe private or company work. I'll walk you through the source on request.

## Contact

The best way to reach me is [LinkedIn](https://www.linkedin.com/in/bo-abrams).
