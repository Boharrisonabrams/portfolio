# WikiRace: a Wikipedia race where the server checks every click

*A Wikipedia racing game I shipped as a web app, an iOS app, and an API, built on one rule: the server never takes the client's word for a move.*

## Problem

The game is old: start on one Wikipedia article and reach a target using only the links on each page. The trouble starts with a daily leaderboard. If the client reports its own path, anyone can post a two-click win, and the rankings mean nothing.

## Approach

One pnpm monorepo holds an Express 5 API, a React 19 web app, and an Expo SDK 54 iOS app, on Supabase Postgres.

The API is the only writer. On every click, `POST /game/navigate` gets the current article's links from Wikipedia, through a short server-side cache, and accepts the move only if the destination is on that page. The route's comment: "The client's word is never taken." The check lives in `validate-move.ts`, a pure function with no I/O.

Finishing a race writes the session, the streak, and the stats. The Postgres function `complete_game_session_v2` does all of it in one transaction under an advisory lock, and a retry returns the stored result instead of counting twice. The daily resets at the player's local midnight and allows one official attempt. It feeds two leaderboards: shortest path and fastest active time.

<!-- screenshot pending (batch 4): ![TODO: web race screen, 1280x800, no personal data](../images/wikirace-web-race.png) -->
## What the agents did and what I kept

The game has no AI inside it. I'm the only human committer. Claude Opus planned and reviewed, and Codex implemented. Under `AGENTS.md`, Codex rules on each review finding as ACCEPT, REJECT, or ADJUST and logs each cycle in `docs/ai/state.md`. Of the 200 commits, 186 landed in July 2026.

I kept the product calls and the releases. The daily-ritual spec lists ten product decisions and is marked "Approved by Bo on 2026-07-10." I ran the deploys and TestFlight builds, and their lessons live as rules in `CLAUDE.md` and `docs/DEPLOY.md`. OpenAI's gpt-image made the W monogram on the app icon, per the file's metadata.

## Result

The API and web app run on Railway. Play it at [wikirace.day](https://wikirace.day). The iOS build went through TestFlight internal testing. That build has since expired, and an App Store release is the next step. The source repo shows 24 merged pull requests, and the public repo has merged nine more since October 2026. Tests span 18 API files, 40 mobile files, 14 web unit files, 8 shared-client files, 5 Playwright specs, and 4 pgTAP database suites. A run on October 19, 2026 passed 138 API, 321 mobile, 32 web unit, 77 shared-client, and 4 share-intent tests, and all four pgTAP suites pass in CI.

One pgTAP suite is the proof I trust most. `atomic_completion_effects.sql` runs 28 assertions. It forces a stats write to fail, then checks that the completion rolls back and that the retry applies official effects exactly once.

<!-- screenshot pending (batch 4): ![TODO: iOS search palette, iPhone portrait, no personal data](../images/wikirace-ios-search.png) -->
## What failed and what I learned

When I audited the repo in October 2026, CI covered less than the test count suggested. GitHub Actions ran the mobile, API, and shared-lib tests plus a web typecheck and build. The web unit tests, Playwright, and pgTAP never ran there, and `game-complete-concurrency.test.ts` mocks Supabase, so the real atomicity proof was the one CI skipped. Worse, the pgTAP atomicity suite failed on a fresh database because two fixtures had drifted from the migrations. The 28 assertions above were only true again after a fixture fix. Since then a Tests workflow runs the web units and all four pgTAP suites on every push, and a Lint workflow runs ESLint, Prettier, and the typecheck. Playwright is still local only.

Find-in-page was too slow on real phones. The mobile update that turned on the server's `find` search tier was rolled back on July 27, 2026. The server then held every session at the `palette` tier behind a `SEARCH_TIER_HOLD` flag, with a documented one-line revert.

The two clients shared code by copy. The mobile and web `src/lib` folders had 17 file names in common, and `active-timer.ts` was 264 identical lines in each. Web search started as a copy while a mobile branch waited to merge, and it never moved into a shared package until October 2026, when the shared logic became `lib/game-client` with 77 tests of its own. `api.ts` and `social-auth.ts` stay separate on purpose, because the transport and auth flows differ by platform. The agent docs drifted too: `state.md` flags `task.md` and `architecture.md` as stale.

It's still a friend beta that never went wide. Product work stopped on July 28, 2026, and the October 2026 commits are cleanup, not features. The next steps are the App Store release, Playwright in CI, and deleting the re-export files now that the package exists.

## What changed since the audit

I audited this repo on October 13, 2026 and spent the next week fixing what the audit found. Coding agents made the edits in isolated clones, a second agent reviewed each branch, and I merged each pull request. Before and after, on the public repo:

| | October 13 | October 19 |
| --- | --- | --- |
| CI | API, mobile, and shared-lib tests, web typecheck and build | Plus ESLint, Prettier, root typecheck, web unit tests, and all four pgTAP suites |
| pgTAP on a fresh database | Atomicity suite stopped at 23 of 28 assertions | 51 of 51 assertions across four suites |
| Root typecheck | Three errors that no job caught | Clean, and a CI step |
| Client logic copied between apps | 17 same-named files, 784 shared lines | One package, 142 shared lines left in the re-exports |
| Four longest mobile screens | 1,133 to 1,332 lines each | 569 to 828 lines, with 54 screen tests that pass on both versions |
| Vitest tests | 485 in the last logged July runs | 572 |
| Template leftovers | `lib/db`, a hello-world script, two stub notes | Removed |

Each refactor landed its characterization tests first, and each pull request states what it did not verify: nothing ran on a device or in a browser, and CI pins Node 24 while the local runs used Node 22.

## Code to read first

If you have five minutes, read these three files in order.

- [artifacts/api-server/src/lib/game/validate-move.ts](https://github.com/Boharrisonabrams/wikirace-public/blob/main/artifacts/api-server/src/lib/game/validate-move.ts): the pure function, with no I/O, that accepts a move only if the source article links to it.
- [supabase/migrations/20260710160000_atomic_completion_effects.sql](https://github.com/Boharrisonabrams/wikirace-public/blob/main/supabase/migrations/20260710160000_atomic_completion_effects.sql): `complete_game_session_v2` writes the session, streak, and stats in one transaction under an advisory lock.
- [supabase/tests/atomic_completion_effects.sql](https://github.com/Boharrisonabrams/wikirace-public/blob/main/supabase/tests/atomic_completion_effects.sql): the pgTAP suite that forces a stats write to fail and checks that the retry applies effects exactly once.

## Links

- Live: [wikirace.day](https://wikirace.day)
- Code: [wikirace-public on GitHub](https://github.com/Boharrisonabrams/wikirace-public)
