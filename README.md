# Token Radar — pre-token protocols ranked by implied value

A make.fun entry for **token-radar-01**: a program (`main.ts`, Node 22,
standard library only) reads a sealed DefiLlama snapshot of 758 protocols,
scores every rankable pre-token protocol with the radar-2 model, and writes
a ranked board plus a static dashboard a reader can actually farm from.

## The model (radar-2)

```
implied = round( tvl_usd x 0.25
  x (1 + clamp(change_7d, -60, 120) / 100 x 0.5)
  x (0.6 + min(age_days, 1095) / 1095 x 0.4)
  x (1 + 0.05 x min(chains - 1, 4)) )
```

Four factors, all from the snapshot's own figures: size (TVL at a 0.25
fully-diluted-value multiple), momentum (seven-day growth, clamped),
maturity (how long the protocol has run with no token), and reach (each
extra chain adds 5%, capped at 20%). No protocol name is special-cased.

## The board

- 198 rows: all 40 watchlist protocols plus every other rankable pre-token
  protocol below the floor. Ranks 1..198, implied value non-increasing.
- 144 pre-token records are set aside, never ranked: wrapped, staked and
  bridged products of tokens that already trade, exchanges and versioned
  pools — WBTC is Bitcoin on Ethereum, not a token that does not exist yet.
- Every row carries a 7-step farm plan with costs and times, derived from
  the protocol's category, chains, TVL and age.
- Live-token protocols are shown apart with their market caps, never mixed
  into the ranking.

## The dashboard

`dashboard/` is a static page (no network, no framework): a ranked board
with search, category and chain filters, a watchlist-only toggle and
sorting; a detail view for each protocol with the model's step-by-step
breakdown and its farm plan; a live-token tab; and the model, stated in
full, so the page and `board.json` can be checked against each other.

## Run it

```
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json>
```

Reads `/input/snapshot.json` by default; writes `board.json`, `data.js`,
`dashboard/`, `screenshots/` and `dossier.json` to the output directory.

## A worked example

Rank 1, Steakhouse Financial, from the snapshot's own figures:

- TVL $2.75B x 0.25 = $687.5M base
- Seven-day growth -2.19% -> growth multiplier 0.989
- Age 1.5 years with no token -> maturity multiplier 0.7984
- 11 chains -> reach multiplier 1.20 (capped)
- Implied value = round($687.5M x 0.989 x 0.7984 x 1.20) = $652.4M

The dashboard recomputes this same arithmetic in the page for every row
it shows, and prints whether its number agrees with `board.json`.

## A farm plan, step by step

Each row's plan is seven steps, each with a cost and a time:

1. A fresh wallet that holds only the funds to farm, seed phrase offline.
2. The category's one action — a deposit, a liquidity provision, a stake,
   a bridge, a mint — sized from the protocol's TVL ($50 to $250).
3. One qualifying action a week, so the wallet is never idle a month.
4. Hold the position 30 to 60 days, so more than one snapshot counts it.
5. Repeat the main action once on a second chain, where the protocol runs
   on one.
6. Claim only from the points page the protocol's own site publishes.
7. Log the wallet, the dates and every transaction, and decide in advance
   when to unwind instead of chasing the position.

The plan never names a protocol by hand: it is derived from the row's
category, chains, TVL and age, so the same plan shape fits a curator, a
DEX, a lender and a bridge.

## Files

- `main.ts` — the entrypoint: read the snapshot, score it, write the
  board, the dashboard data, and the dossier.
- `lib/model.ts` — the radar-2 model, the farm plans, and the snapshot
  parser with its gate checks.
- `vendor/makefun-board.ts` — the kit's dossier builder, copied
  byte-for-byte, as the kit requires.
- `dashboard/` — the static page: `index.html`, `app.js`, `app.css`.
- `screenshots/` — the three images the judge sees: the board, a detail
  view, and a filtered view.

## License

MIT — see LICENSE.
