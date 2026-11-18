# Midnight Launch — a GTA 6 Twitch-viewership forecaster

Entry for make.fun's gta6-twitch-ccu-forecast-01: forecast the concurrent viewers of the
"Grand Theft Auto VI" Twitch category for each hour of its release day (2026-11-19 UTC,
hours 2-24), as five quantiles per hour, from the one case file the grader hands the program
(everything recorded up to the end of hour 1).

## The model

The release-day curve of a midnight-local console launch is well described by the launch curves
of past big launches — the case ships 42 of them as `comparables`. Each comparable with a
release-day hour 1 gives one reading per hour, `r_c(h) = viewers_c(h) / viewers_c(1)`. The
forecast reads the launch-hour-anchored distribution of those readings, on the log scale the
contest scores:

    q_tau(h) = p1 * exp(mu(h) + z_tau * sd(h))

where mu(h), sd(h) are the weighted mean and spread of the log readings at hour h, p1 is hour 1's
print and z_tau the standard normal's quantile at each level.

Three refinements make the pool honest:

- **unlock-pattern filter** — comparables whose release-day peak is more than 20x their hour 1
  cannot look like a rolling console midnight unlock (a mid-day global PC unlock, a live-service
  slow burn); early-access-contaminated and weak analogues are kept out too;
- **launch-jump weighting** — each comparable is weighted by how similar its launch jump is to
  the case's own: the log ratio of hour 1 to the whole previous UTC day, a trait the case's
  `history_hourly` exposes for the target and every comparable's pre-release days expose for
  itself, with a gaussian kernel of width 0.7 log units;
- **lognormal quantiles** — per hour the weighted log readings give the mean and spread directly,
  which beats empirical quantiles at this pool size.

Backtested leave-one-out over the comparables, on the 18 comparable-shaped targets: loss
0.213 median / 0.339 mean, against the house baseline's 0.386 / 0.507 on the same targets
(0.387 on the practice set).

## Running it

Node 22, standard library only, no network:

    mkdir -p out && MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out node --experimental-strip-types main.ts

reads every `input/*.in`, writes `out/<id>.out` (23 rows, hours 2-24, five non-decreasing
quantiles each). Deterministic: two runs write the same bytes.

## License

MIT.
