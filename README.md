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

## The model in detail

The snapshot is DefiLlama's protocol index as fetched on 2026-11-14: 758
protocols at or above $5M of TVL. Of those, 342 record no token at all, and
40 meet the house watchlist rule (at least $50M of TVL, no token on record,
not a wrapped, staked or bridged product of a token that already trades).
The board ranks all 40 watchlist protocols plus the 186 pre-token protocols
below the floor with the most TVL, 226 rows in all, worth $2.40B of implied
value together.

A token's implied value is priced the way the market already prices the
tokens that trade. For each category, the model takes the median
market-cap-to-TVL of the live protocols in that category (five or more of
them trading), and falls back to the median of all live protocols when a
category has fewer. The medians are clamped to 0.10–0.80 so one thin
category cannot set the price for the whole board. On this snapshot the
global median is 0.17. Dexs price near 0.34, RWA near 0.28, Lending near
0.13, Yield near 0.12, and Liquid Staking near 0.03 (clamped to 0.10).

The three named factors then shape each row:

- **TVL** is the size of the protocol's own record. The largest watchlist
  protocol, Steakhouse Financial, holds $2.75B of TVL; the smallest row on
  the board holds about $5M.
- **Seven-day growth** is the momentum of that TVL, clamped to -60%..+120%
  and weighted by 0.5, so a protocol growing 20% in a week scores 10% more
  than one standing still, and one shrinking 20% scores 10% less.
- **Age with no token** is the time since DefiLlama first listed the
  protocol, the best record of how long it has run without issuing a token.
  Maturity rises from 0.60 for a brand-new listing to 1.00 at three years.
  A protocol whose listing date is unknown is scored at 365 days.

The top of the board on this snapshot:

| Rank | Protocol | TVL | Implied value |
| --- | --- | --- | --- |
| 1 | Steakhouse Financial | $2.75B | $367.3M |
| 2 | Gauntlet | $1.58B | $218.7M |
| 3 | Veda | $1.56B | $210.3M |
| 4 | USDD | $1.06B | $144.7M |
| 5 | Bittensor dTAO | $376.0M | $77.3M |

No name is ever special-cased. The same function scores every row, and the
dashboard recomputes each row from the snapshot figures and checks the
number `board.json` carries, so the page's code and the board's data are
visibly the same arithmetic.

## The farm plan

Every row carries six steps, derived from the protocol's own record and
never from a hand-written list of names:

1. Set up a fresh wallet that holds only the funds you plan to farm, and
   store its seed phrase offline.
2. Take the category's one action on its main chain: supply USDC to a
   lending protocol, add liquidity to a DEX, bridge to a bridge, stake
   through a staking protocol, deposit margin on a perpetuals exchange,
   deposit into a curator's vault, and so on.
3. Do one qualifying action every week, such as a deposit, a borrow, a
   swap or a vote.
4. Hold the position for at least 30 days (60 for a protocol younger than
   six months, 45 for one younger than a year), so the activity is counted
   in more than one snapshot.
5. Before withdrawing, open the protocol's own points or rewards page from
   its official site and claim only from the link it publishes.
6. Log the wallet, the dates and every transaction.

The capital for one position follows the protocol's TVL: $250 for the
largest protocols (TVL at or above $5B), $150 above $1B, $100 above $100M,
and $50 below that. Each step states what it costs and how long it takes.

## The dashboard

The page is a dark radar terminal: a header with the snapshot date and
model version, four facts (watchlist covered, rows scored, board implied
value, live-token protocols set aside), and a filterable, sortable table.
The filters are a search box, a category select, a chain select, a
watchlist/early scope select, and pre-token/live-token tabs. Every column
sorts. Selecting a row opens its detail view: the model's inputs (TVL,
category multiple, seven-day growth, age, growth and maturity
multipliers, implied value), the recomputed check against `board.json`,
and its six-step farm plan. A methodology panel under the board shows the
formula and the full table of category multiples with their live-comp
counts, and a live-token view lists the 24 largest protocols that already
trade, with their market caps as facts, not models.

The page renders with no network. All data is in `data.js`, written beside
the page by `main.ts` from the same board, and the page's state (search,
filters, selected protocol) lives in the URL hash, so a board, a detail
view and a filtered view each have an address of their own.

## What the board does not claim

A protocol is shown as recording no token when its DefiLlama record
carries no market cap, CoinGecko id or symbol. That is the record, not
proof it has no token. Implied values are this radar's own model of what a
token would be worth if the protocol's record stayed as it is. They are
not a prediction that a token is issued, or when, and not financial
advice.
