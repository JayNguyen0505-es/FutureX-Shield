<p align="center">
  <img src="assets/logo-icon.png" alt="Future X Shield logo" width="72">
</p>

<h1 align="center">Future X Shield</h1>

<p align="center">
  <b>Every Deal looks safe. Is your account?</b><br>
  An account-level risk view for VN30F futures traders on DNSE Future X.
</p>

<p align="center">
  <a href="https://future-x-shield.pages.dev/"><b>Live demo</b></a> ·
  <a href="#run-it-locally">Run locally</a> ·
  <a href="#how-it-works">How it works</a> ·
  <a href="#the-evidence">Evidence</a>
</p>

<p align="center">
  🏆 <b>Champion, Future Business Analyst (FBA) Season 6, 2026</b> · Team <b>ROAD TO 30</b> · DNSE Securities case
</p>

![Future X Shield overview](assets/hero.jpg)

> **Disclaimer.** Future X Shield is a competition concept. It is **not** a DNSE product or an official DNSE interface, and nothing on the page is investment advice. Figures marked **[M]** are simulated, **[A]** are assumptions awaiting DNSE confirmation, and **[F]** are sourced from Future X rules or Vietnamese law.

---

## The problem

On Future X, DNSE's VN30F derivatives product, every position a trader opens is a separate **Deal** with its own margin. All Deals draw on one shared pool of **unallocated margin**, and **Auto Deal Top-up** quietly refills any weak Deal from that pool.

The result: every Deal keeps showing a healthy margin ratio while the shared pool drains underneath. When the pool runs out, the system force-closes the weakest Deal at whatever price the market is at. No existing screen shows that shared margin running down.

**Shield does.** It shows how much of the shared margin each Deal is consuming, and how far VN30F can move before the first Deal is force-closed.

## How it works

Shield runs a single rules-based calculation engine (no machine learning) built on Future X rules, and answers three questions, each only when the trader needs it:

| # | Question | Tool | When it appears |
|---|---|---|---|
| 1 | *Am I in danger?* | **Shield Score**: a two-way buffer gauge showing how far VN30F can still fall *and* rise before the first force-close | Always on: at order time and in the background |
| 2 | *Which Deal is responsible?* | **Two matrices** built from three per-Deal measures (A · risk contribution, B · P&L since opening, C · reserve draw) | Tap the gauge |
| 3 | *What happens if I continue?* | **Scenario cube**: an interactive A × B × C 3D view where the trader chains decisions and price moves | Scenario mode |

- **Shield Score.** The engine steps the price 0.01% at a time in each direction, applies the top-up rule at every step, and stops when the first Deal hits the force-close threshold (search capped at ±30%). Green ≥ 5%, caution 3–5%, danger < 3%.
- **Perception–Risk Matrix (A × B)**, shown while the account is green or caution: does the P&L the trader watches match the risk the Deal carries?
- **Attribution Matrix (A × C)**, shown when the account is in danger: which Deal drained the reserve, and which will push the account over the line?
- **Guardrails.** Shield describes and never recommends (Securities Law 54/2019), requires opt-in consent (Law 91/2025, Decree 356/2025), is never used to cut limits or score customers, and reads thresholds from DNSE systems in real time rather than hard-coding them.

Future X thresholds used in the demo (from the DNSE derivatives FAQ example, to be confirmed by DNSE **[A]**):

| Margin ratio | Meaning |
|---|---|
| 18.48% | Required to open a Deal |
| 17.71% | Auto Deal Top-up refills the Deal back to 18.48% |
| 17.35% | The Deal is force-closed |

## The evidence

Without DNSE customer data, the team replayed **real VN30F1M daily prices** on simulated accounts that follow Future X rules: 12 rolling six-month runs (one starting each month, Apr 2025 – Mar 2026), 120 million VND each.

| Question | Result |
|---|---|
| Do traders holding several Deals get force-closed more? | **38 vs 0** force-closed Deals (up to 3 Deals vs 1 Deal, same entry signals, top-up on) |
| Is Auto Deal Top-up the cause? | With top-up off, both groups are force-closed at **0.96–1.08×** the same rate |
| Is it the number of Deals or of contracts? | Contracts: one single Deal still gives **35 vs 38** force-closed contracts |
| Does Shield need to run in real time? | Computed only at each close, it would have flagged red in advance for just **16%** (6/38) of force-closes |
| Do the matrices tell Deals apart? | Across 479 multi-Deal snapshots: different cells **29%** of the time (A × B) and **94%** (A × C) |

Every figure above is reproduced in this repo by `tools/l10_sim.js` and checked row by row against the team's original Python output in `tests/test_sim.js`.

## The demo

The page is a long-form proposal (Overview, Problem, Evidence, Solution, Audience, Guardrails, Data) plus a **Demo** tab with four interactive demos, all driven by the same engine:

1. **At order time.** An order ticket with a Shield card that recalculates the score before the order is placed, lists Deals closest to force-close, and shows three "if… then…" ways back to green.
2. **Two-way Shield.** The buffer gauge and both matrices, before and after opening one more contract.
3. **Scenarios.** Chain opens, closes, deposits and price shocks, and watch every Deal move inside the 3D cube. The cube auto-rotates to the Attribution face when the score turns red.
4. **Real portfolio.** Two sample traders replayed session by session on 411 sessions of real VN30F1M prices (Jan 2025 – Aug 2026). Insert your own decisions at any session and see what changes.

Bilingual (Vietnamese / English), deep-linkable (e.g. `#/demo/atl`), responsive, and works offline.

## Tech stack

- **Vanilla JavaScript, HTML and CSS.** No framework, no backend, no runtime dependencies.
- **Custom SVG 3D renderer** (`src/cube3d.js`): turntable rotation with inertia, pinch / Ctrl-scroll zoom, hover cards with axis guides, full-screen mode, and respect for `prefers-reduced-motion`.
- **Single-file build.** `build.js` inlines scripts, styles, fonts and images into one self-contained `dist/index.html` (~2.6 MB) that opens straight from disk.
- **Strict Content-Security-Policy** with a sha256 hash of the built script, so a hand-edited `dist/` file will not run.
- **Testing:** Node test scripts plus Playwright end-to-end tests in a real browser.
- **Hosting:** Cloudflare Workers (static assets only); CI on GitHub Actions.

## Project structure

```
engine/
  engine.js          Shield calculation engine (port of the product team's shield_engine.py). Every number on the page comes from here.
  timeline.js        Multi-session account simulation on real prices (intraday top-up, force-close, end-of-day settlement, trader orders).
src/
  index.html         Page skeleton with injection points for scripts and styles.
  app.js             Rendering only: routing, language switching, the four demos. Computes nothing itself.
  cube3d.js          The SVG scenario cube.
  i18n.js            Every UI string, in both vi and en.
  style.css          Styles.
data/
  vn30f1m.csv        Daily VN30F1M prices, 411 sessions (2 Jan 2025 – 28 Aug 2026).
  sim/               The team's original simulation outputs, used as golden files.
tools/
  l10_sim.js         Line-by-line JS port of the team's simulation scripts. `node tools/l10_sim.js` prints a summary.
  vn30f_xlsx_to_csv.py  Regenerates data/vn30f1m.csv from the source spreadsheet.
  wrapscan.js        Detects awkward line wraps (orphaned last words) across pages, widths and languages.
tests/
  test_engine.js     51 checks: golden values and matrix cell boundaries.
  test_timeline.js   17 checks: multi-session simulation.
  test_sim.js        18 checks: re-runs Simulations 1 and 2 and compares every row with the original output.
  test_build.js      59 checks: build output, content and configuration.
  e2e.js             Playwright tests in a real browser.
assets/, fonts/      Logo, favicon, hero image and Be Vietnam Pro (SIL OFL), all inlined at build time.
build.js             Builds dist/index.html and dist/_headers.
wrangler.jsonc       Cloudflare Workers config: serves dist/ as static assets.
RUNBOOK.md           Deploy, update and rollback steps (Vietnamese).
```

## Run it locally

Requires **Node.js 18+** (Node 20 recommended, see `.nvmrc`).

```bash
npm ci                            # installs Playwright (needed only for e2e)
npm test                          # unit, simulation and build tests: 51 / 17 / 18 / 59 pass
node build.js                     # writes dist/index.html and dist/_headers
npx playwright install chromium   # first time only
npm run e2e                       # builds, then runs the browser tests
```

Then open `dist/index.html` in any browser. No server needed.

If Playwright's Chromium cannot be downloaded, use an installed browser instead: `E2E_CHANNEL=msedge` or `E2E_CHANNEL=chrome`.

No environment variables are required. `.env.example` documents every variable the code reads (build metadata injected by CI, and optional e2e settings), and `tests/test_build.js` checks that list stays in sync with the code.

## Build safeguards

`build.js` stamps the commit, branch and build time into `window.__BUILD__` and a `<meta name="build">` tag, shown in the page footer. Uncommitted changes are marked `+dirty`, and `--require-clean` makes the build fail on them.

The build **refuses to write output** if:

- the output contains anything that looks like a secret (tokens, API keys, private keys);
- `engine/engine.js` differs from the prototype copy at `../prototype_engine/engine.js` (checked only when that folder exists);
- `src/index.html` contains inline `onclick=` handlers or a second `<script>` tag, which the CSP would block.

## Deployment

The live site is served by **Cloudflare Workers Builds**, which pulls from GitHub and runs:

```bash
npm test && node build.js --require-clean   # build command
npx wrangler deploy                         # deploy command (serves dist/ only)
```

If tests fail or the build is not from a clean commit, Cloudflare keeps the previous version live. `dist/_headers` adds security headers a `<meta>` tag cannot set (`frame-ancestors`, `X-Content-Type-Options: nosniff`). GitHub Actions runs the unit and e2e tests on every push to `main`; its optional GitHub Pages deploy stays off unless the repo variable `DEPLOY_PAGES` is set to `true`.

Step-by-step deploy, rollback and takedown instructions are in [`RUNBOOK.md`](RUNBOOK.md).

## Privacy and security

- No sign-in, cookies, analytics or network requests. Every calculation runs in the browser.
- No form posts anywhere and there is no server, so there is no server-side attack surface.
- `localStorage` stores only the chosen language, wrapped in `try/catch` so the page still works when storage is blocked.

## Known limitations

- **Thresholds** (18.48% / 17.71% / 17.35%) come from a DNSE FAQ example and need DNSE confirmation.
- **Intraday price order** is assumed, and VN30F2M is assumed to trade at the VN30F1M price in the portfolio demo.
- **The Real portfolio demo differs from Simulation 1 in three ways:** it checks at the opening price (so gap opens force-close at the open), walks intraday prices in 0.01% steps (force-closing exactly at 17.35% rather than at the session's high or low), and keeps Deals open until closed rather than auto-closing after 5 sessions. The page states this under the demo.
- **No margin call step** yet when unallocated margin goes negative (e.g. Trader B after 3 Apr 2025).
- The **"5-session peak scale"** state (FR-23) is not implemented, as the demo account has no data for it.
- **Matrix cell boundary:** A and C count as "high" above an equal share (1/n) of the total. This is a design choice.
- Tested on Chromium and Edge; **not yet verified on Safari for iOS**.
- The product name "Future X Shield" is a working title.

## Credits

Built by team **ROAD TO 30** for Future Business Analyst Season 6 (2026), on the DNSE Securities case.

- Calculation rules and `shield_engine.py`: the team's Product group
- Simulations 1–3 and the VN30F1M dataset: the team's Simulation group

Font: [Be Vietnam Pro](https://github.com/bettergui/BeVietnamPro), licensed under the SIL Open Font License (see `fonts/OFL.txt`).

## License

Copyright © 2026 [Nguyen Truong Thinh](https://github.com/JayNguyen0505-es) and [Nguyen Hai Phong](https://github.com/Xx-Pumpkin-xX). All rights reserved.

This repository is published for portfolio and reference purposes; no license is granted to copy, modify or redistribute it without the authors' permission. Future X and DNSE are trademarks of DNSE Securities JSC, used here only to describe the competition case.
