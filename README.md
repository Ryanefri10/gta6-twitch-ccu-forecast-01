# GTA VI launch-day Twitch viewers — hour-by-hour forecast

make.fun bounty entry for
[`gta6-twitch-ccu-forecast-01`](https://make.fun/bounty/gta6-twitch-ccu-forecast-01):
forecast how many people watch Grand Theft Auto VI live on Twitch in each
hour (2–24, UTC) of its release day, 2026-11-19, as five quantiles
(5%, 25%, 50%, 75%, 95%) per hour. Score is mean pinball loss on the log
scale; the entry must beat the best house entry to take a prize.

## Entry

`main.py` — Python 3.13, standard library only, no network, deterministic.
Run it the way the grader does:

```sh
mkdir -p out
MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

It reads every `*.in` case in the input directory and writes one
`<id>.out` per case: `{"hours": [{"hour": h, "q": [q05, q25, q50, q75, q95]}, ...]}`
for hours 2–24.

## Method

Log-scale diurnal-shape model, anchored on the case's hour-1 print `p1`:

1. For every comparable past big launch in the case file, take its
   release-day hourly counts and form log-ratios `r_h = ln(y_h / y_1)`.
2. Keep only launches whose shape can transfer to a rolling-midnight
   console launch: drop early-access-contaminated launches, weak
   analogues (free-to-play / slow-burn), launches with an hour-1 print
   under 10,000 viewers, and the GTA VI category's own pre-release
   history (not a launch).
3. Per hour, pool across the kept launches: `m_h` = median of `r_h`,
   `s_h` = max(1.4826·MAD, IQR/1.349, 0.30), plus the pooled empirical
   quantiles of `r_h`.
4. Forecast `q_τ(h) = p1 · exp(0.5·(m_h + z_τ·s_h) + 0.5·e_h[τ])`,
   rounded to 2 decimals half-up. The shape is the point: hour 1 is
   only the first hour after midnight UTC, so the count dips in hours
   2–4, jumps at hour 5–6 as the US east coast unlocks, and climbs
   through the evening into US west-coast prime time.
5. If a case carries no usable comparables, fall back to the house
   baseline (persist at hour 1 with a widening spread).

## Calibration

Scored locally against the four practice launches' real hourly counts
(SullyGnome), which the kit's practice cases do not include:

| model | mean pinball loss |
|---|---|
| house baseline (starter) | 0.387 |
| this entry (v2) | **0.254** |

90% coverage 0.89 (honest ≈ 0.90), typical median miss 47% (house: 83%).
Leave-one-out over 12 target-like launches (rolling-midnight, clean,
hour-1 ≥ 10k): ≈ 0.29, also well below the house baseline. Two runs on
the same case produce byte-identical output; runtime ≈ 0.1–0.3 s per
case (limit 8 s).

## License

MIT — see `LICENSE`.
