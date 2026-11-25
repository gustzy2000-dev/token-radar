# Token Radar

[![token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/gustzy2000-dev/token-radar/rank.svg)](https://make.fun/bounty/token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/gustzy2000-dev/token-radar/reward.svg)](https://make.fun/bounty/token-radar-01)

A radar for token launches: every pre-token protocol DefiLlama tracks, ranked
by the implied value of the token it may one day issue, with a farm plan for
each. Entry for the make.fun bounty
[Build the best token launch radar](https://make.fun/bounty/token-radar-01).

## What it does

`main.ts` reads the sealed snapshot (`/input/snapshot.json`), scores every
pre-token protocol with the comps model in `lib/model.ts`, writes the ranked
board to `/output/board.json`, and copies `dashboard/` and `screenshots/` to
`/output` with a `dossier.json` built by `vendor/makefun-board.ts`.

The board covers all 40 watchlist protocols plus the 200 pre-token protocols
below the $50M floor with the most TVL (240 rows). The 24 live-token
protocols are set aside with their market caps.

## The model (radar-2)

```
implied = round( tvl_usd x comps(category)
               x (1 + clamp(change_7d, -60, 120) / 100 x 0.5)
               x (0.6 + min(age_days, 1095) / 1095 x 0.4) )
```

`comps(category)` is the median market-cap-to-TVL of the live protocols in
the same category (five or more of them), otherwise the median of all live
protocols, clamped to 0.10–0.80. No protocol name is special-cased: the same
function scores every row, and the dashboard shows the multiple each
category was priced at.

## The dashboard

A static page, no network: `dashboard/index.html`, `dashboard/app.css` and
`dashboard/app.js` read `data.js`, written beside the page by `main.ts`.
Search, category and chain filters, a watchlist/early scope, sortable
columns, a detail view for every protocol with its model breakdown and farm
plan, and a live-token view.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json|directory>
```

Node 22.18 or later, standard library only. Then open `out/dashboard/` from
a local server, for example `python3 -m http.server --directory out`.
