# WikiRace: a Wikipedia race where the server checks every click

*A Wikipedia racing game I shipped as a web app, an iOS app, and an API, built on one rule: the server never takes the client's word for a move.*

## Problem

The game is old: start on one Wikipedia article and reach a target using only the links on each page. The trouble starts with a daily leaderboard. If the client reports its own path, anyone can post a two-click win, and the rankings mean nothing.

## Approach

One pnpm monorepo holds an Express 5 API, a React 19 web app, and an Expo SDK 54 iOS app, on Supabase Postgres.

The API is the only writer. On every click, `POST /game/navigate` gets the current article's links from Wikipedia, through a short server-side cache, and accepts the move only if the destination is on that page. The route's comment: "The client's word is never taken." The check lives in `validate-move.ts`, a pure function with no I/O.

Finishing a race writes the session, the streak, and the stats. The Postgres function `complete_game_session_v2` does all of it in one transaction under an advisory lock, and a retry returns the stored result instead of counting twice. The daily resets at the player's local midnight and allows one official attempt. It feeds two leaderboards: shortest path and fastest active time.

![TODO: web race screen, 1280x800, no personal data](../images/wikirace-web-race.png)

## What the agents did and what I kept

The game has no AI inside it. I'm the only human committer. Claude Opus planned and reviewed, and Codex implemented. Under `AGENTS.md`, Codex rules on each review finding as ACCEPT, REJECT, or ADJUST and logs each cycle in `docs/ai/state.md`. Of the 200 commits, 186 landed in July 2026.

I kept the product calls and the releases. The daily-ritual spec lists ten product decisions and is marked "Approved by Bo on 2026-07-10." I ran the deploys and TestFlight builds, and their lessons live as rules in `CLAUDE.md` and `docs/DEPLOY.md`. OpenAI's gpt-image made the W monogram on the app icon, per the file's metadata.

## Result

The API and web app ran on Railway, and the iOS build went through TestFlight internal testing. GitHub shows 24 merged pull requests. Tests span 18 API files, 38 mobile files, 15 web unit files, 5 Playwright specs, and 4 pgTAP database suites. The last logged runs passed 137 API and 299 mobile tests.

One pgTAP suite is the proof I trust most. `atomic_completion_effects.sql` runs 28 assertions. It forces a stats write to fail, then checks that the completion rolls back and that the retry applies official effects exactly once.

![TODO: iOS search palette, iPhone portrait, no personal data](../images/wikirace-ios-search.png)

## What failed and what I learned

CI covers less than the test count suggests. GitHub Actions runs the mobile, API, and shared-lib tests plus a web typecheck and build. The web unit tests, Playwright, and pgTAP never run there. `game-complete-concurrency.test.ts` mocks Supabase and never runs a real race, so the real atomicity proof is the one CI skips.

Find-in-page was too slow on real phones. The mobile update that turned on the server's `find` search tier was rolled back on July 27, 2026. The server then held every session at the `palette` tier behind a `SEARCH_TIER_HOLD` flag, with a documented one-line revert.

The two clients share code by copy. The mobile and web `src/lib` folders share 17 file names, and `active-timer.ts` is 264 identical lines in each. Web search started as a copy while a mobile branch waited to merge, and it never moved into a shared package. The agent docs drifted too: `state.md` flags `task.md` and `architecture.md` as stale.

It's still a friend beta that never went wide, and the last commit is from July 28, 2026. The next fixes are pgTAP and web tests in CI and one shared client package.

## Links

- Code: [wikirace-public on GitHub](https://github.com/Boharrisonabrams/wikirace-public)
