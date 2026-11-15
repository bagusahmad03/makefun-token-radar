# Token Radar

A token launch radar: it reads a sealed snapshot of the protocol universe, ranks the
protocols that record no token by what a launch would be worth, and hands in a web
dashboard showing the board, a detail view for every protocol, and the steps to farm it.

## What it is

- `main.ts` — the program. It reads the snapshot, scores the board, writes the
  dashboard's data, copies the dashboard and screenshots to the output directory, and
  writes `dossier.json` last with the vendored library.
- `lib/model.ts` — the model, the board builder, the farm plans, and the validation
  the program runs before it writes anything.
- `vendor/makefun-board.ts` — the dossier library, shipped with the kit, copied
  byte for byte. It is the only file in this repository that is not original work.
- `dashboard/` — a static page (no framework, no network): the ranked board, filters,
  a detail view per protocol, and the model stated with its constants.
- `screenshots/` — three renders of the dashboard: the board, a detail view, and a
  filtered view.

## The model, in words

A protocol is pre-token when its snapshot record carries no token: market cap,
CoinGecko id and symbol are all null. The board covers every slug on the snapshot's
watchlist, then adds pre-token protocols below the watchlist floor by TVL, up to 160
early rows. Live-token protocols are kept apart and shown with the market's own values.

Every row is scored by one function of the snapshot's own figures, and no protocol
name is ever special-cased:

    implied = round(TVL x 0.30
                    x (1 + clamp(7d growth, -50%, +100%) / 100 x 0.40)
                    x (0.70 + min(age in days, 730) / 730 x 0.30)
                    x (1 + 0.02 x min(chain count, 4)))

TVL is the size of the claim, seven-day growth is its direction, age is how long the
protocol has run with no token on record (365 days when unlisted), and breadth is how
many chains carry its TVL. The dashboard states the formula and every constant, and
recomputes each row so the page and the board visibly agree.

Farm plans are derived from each protocol's own record — its category, its chains,
its TVL and its age — and every step carries what it costs and how long it takes.
A plan never invents a link, a token name, a date, or a rewards program, and it says
a protocol records no token rather than that it has none: the record is not proof.

## How to run it

Node 22 or later, standard library only, nothing to install:

    mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json|directory>

Outside the grader, `MAKEFUN_OUTPUT_DIR` stands in for `/output`; `--output` overrides
both, and `--input` defaults to the sealed `/input`. The program exits non-zero with a
clear message when the snapshot is unreadable, the board fails validation, or the
screenshots are missing. It is deterministic: two runs write the same bytes.

## The output

- `board.json` — the ranked board: rank, slug, name, the snapshot figures, the
  model's multipliers, the implied value, and the farm plan, one row per protocol.
- `data.js` — the same board in the compact form the dashboard renders.
- `dashboard/` — the static page, copied from this repository.
- `screenshots/` — the three renders, copied from this repository.
- `dossier.json` — the dossier the library writes from the finished output directory.
