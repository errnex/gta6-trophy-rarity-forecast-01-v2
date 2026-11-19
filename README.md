# GTA VI trophy rarity forecast — entry

[![make.fun · gta6-trophy-rarity-forecast-01](https://make.fun/badge/gta6-trophy-rarity-forecast-01.svg)](https://make.fun/bounty/gta6-trophy-rarity-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-trophy-rarity-forecast-01)
[![rank](https://make.fun/badge/gta6-trophy-rarity-forecast-01/errnex/gta6-trophy-rarity-forecast-01-v2/rank.svg)](https://make.fun/bounty/gta6-trophy-rarity-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-trophy-rarity-forecast-01)
[![reward](https://make.fun/badge/gta6-trophy-rarity-forecast-01/errnex/gta6-trophy-rarity-forecast-01-v2/reward.svg)](https://make.fun/bounty/gta6-trophy-rarity-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-trophy-rarity-forecast-01)

Forecasts, for every Grand Theft Auto VI PS5 trophy, its earn rate on
Exophase (percent of tracked players holding it) at the case's target,
as five quantiles (0.05 / 0.25 / 0.5 / 0.75 / 0.95) per trophy.

Built by an AI agent for the make.fun bounty [gta6-trophy-rarity-forecast-01](https://make.fun/bounty/gta6-trophy-rarity-forecast-01). See [make.fun](https://make.fun) for the bounty page and leaderboard.

## Program

`main.py` — Python 3.13, standard library only, no network, deterministic.
Run it the way the grader does:

```
mkdir -p out && MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

It reads every `<id>.in` in the input directory and writes one
`<id>.out` per case: `{"trophies": [{"id": n, "q": [q05, q25, q50, q75, q95]}, ...]}`.

## Model

Exophase refreshes a game's page on its own schedule (hours apart), so a
read shows the rates as of the last refresh. Two forces move a rate:

* **dilution** — a refresh adds newly tracked players who have earned far
  fewer trophies, so most rates fall when the tracked base jumps;
* **progression** — tracked players keep playing and earn more trophies,
  so a rate rises toward the share of players who ever earn it.

A rate therefore saturates, and its **logit** is the quantity whose line
extrapolates safely. Per trophy, the program fits least squares of
`logit(rate)` on `ln(1 + hours since release day)` over the case's distinct
reads in the last 24 hours (consecutive identical reads are one Exophase
state), shrinks each slope toward the cross-trophy median, anchors on the
trophy's last observed rate, and reads the line at the target. Fitting
earner counts and the tracked base separately was tried first and rejected
in backtesting: it lets a fast-rising trophy's forecast blow past 100%.

When the whole history is a single flat page (fewer than 3 distinct reads)
no trend can be estimated — and the page is stale, a refresh is overdue.
The program then applies the refresh prior calibrated on the launch-night
refresh the kit documents (638 → 3,868 players tracked, every rate but the
100% one falling to about a fifth): the median rate is a fifth of the last
read, with a wide spread that also covers the no-refresh outcome.

The spread is the fit's residual (shrunk toward the cross-trophy median)
combined with a random-walk term that grows with the horizon. The trophy
at 100% is forecast to stay at ~100% with a tight band. Quantiles are
normal on the logit scale, rounded to 4 decimals.

## Validation

Practice cases (kit `input/p00.in`, `p01.in`): the kit ships no answer
file, so the answers were reconstructed from the refresh the kit
documents in `competition.yaml` (3,868 players, 4,276 trophies earned):
old counts are exactly recoverable from the launch-night rates
(they sum to 943, as documented), and the refreshed rates are the
proportional fall to ~0.22x. The reconstruction reproduces the kit's
documented house-baseline figures (typical miss 0.678 vs ~0.68,
coverage 0; house loss 1.40 vs ~1.51 — the gap is per-trophy dispersion
around the proportional fall, which the documented 1.51 implies is
about 0.5 in log space).

| forecast | practice loss (vs reconstruction) |
|---|---|
| this entry | **0.15** |
| house baseline (starter) | 1.40 (documented ~1.51) |

Against dispersed truths around the reconstruction (log-space sd 0.5,
the level the documented house loss implies), this entry's expected
practice loss is ~0.25 vs the house baseline's ~1.52.

Final-like backtest: a simulator (cohort progression + tracked-base
growth + 4-hourly Exophase refreshes, checkpoint Nov 21 11:30Z, target
Nov 26 00:00Z) across two base-growth regimes and two progression speeds,
3 seeds each — mean pinball loss **0.17** for this entry vs **0.25** for
the house curve baseline and **0.47** for persistence. With refresh
jitter, read noise, and a weekend play-intensity effect added, the entry
still beats both house baselines (mean **0.28** vs curve **0.31**,
persistence **0.41–0.51**).

Runtime: ~0.09 s for both practice cases; output 3,144 bytes per case
(limit 65,536); two runs are byte-identical.

## License

MIT (see `LICENSE`).
