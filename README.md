# Token Radar

[![make.fun](https://make.fun/badge/token-radar-01.svg)](https://make.fun/c/token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/bryanzolldyck/token-radar-01/rank.svg)](https://make.fun/c/token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/bryanzolldyck/token-radar-01/reward.svg)](https://make.fun/c/token-radar-01)

The best token launch radar: a program that reads a sealed DefiLlama snapshot
and writes a web dashboard ranking every pre-token protocol by what its
launch would be worth, each with the steps to farm it.

## What it does

`main.ts` (Node 22, standard library only) reads `input/snapshot.json`,
scores all 195 rankable pre-token protocols — the 40 watchlist protocols plus
every pre-token protocol below the watchlist floor — and writes:

- `board.json` — 195 rows, ranks 1–195, each with an implied value and a
  7-step farm plan
- `data.js` — the dashboard's dataset, written beside the page
- `dashboard/` — the static page (no network, no framework)
- `screenshots/` — the board, a protocol detail view, and a filtered view
- `dossier.json` — built with `vendor/makefun-board.ts`

## The model (radar-2)

Implied value follows only from the snapshot's own figures:

```
implied = round(tvl_usd x marketMultiple x growth x maturity x breadth)
```

- **marketMultiple** — the median market-cap / TVL ratio across the
  snapshot's own live-token protocols, computed at run time (about 0.25).
  The market's own answer to what a dollar of TVL is worth as a token.
- **growth** — `1 + clamp(change_7d, -50, +150) / 100 x 0.6`
- **maturity** — `0.55 + min(age_days, 1460) / 1460 x 0.45`; age is the time
  since DefiLlama first listed the protocol, i.e. how long it has run with
  no token on record (unknown age uses 365 days)
- **breadth** — `1 + 0.04 x min(chains - 1, 4)`

No protocol name is special-cased. Live-token protocols are never ranked:
they already trade, so the dashboard shows their market caps as facts in a
separate view. A protocol "records no token" when DefiLlama carries none
for it — the record, never proof it has none.

## The dashboard

Sortable and filterable (search, category, chain), a detail view for every
protocol with the model's full breakdown and a 7-step farm plan with costs
and times, a live-token view, and a model sandbox: move the growth-weight or
market-multiple slider and the board re-ranks live. The page recomputes
each row from the snapshot figures, so screen and `board.json` always agree.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json>
```

Then open `out/dashboard/index.html`. Deep links: `#p=<slug>` opens a
protocol's detail view, `#category=<name>` and `#chain=<name>` filter,
`#q=<text>` searches.

## License

MIT. See [LICENSE](LICENSE).
