# gta6-twitch-ccu-forecast

[![make.fun · gta6-twitch-ccu-forecast-01](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/OWNER/REPO/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/OWNER/REPO/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)

Forecast how many people watch Grand Theft Auto VI live on Twitch in each
hour of its release day, 2026-11-19 (UTC), after the first hour — five
quantiles (0.05, 0.25, 0.5, 0.75, 0.95) per hour, hours 2 to 24 — from
everything recorded up to the end of hour 1.

## How it works

An empirical launch-shape model built from the case's own `comparables`
(past big game launches' hourly Twitch category averages, SullyGnome):

1. Every comparable launch is expressed as log-ratios to its own hour-1
   print, `r_h = ln(v_h / v_1)`. Launches whose hour 1 was still pre-unlock
   (tiny hour-1 print, e.g. a single simultaneous global unlock at 14:00Z)
   are left out — GTA VI's hour 1 already holds the UK unlock plus ~13 hours
   of play from New Zealand, Australia, Japan and Europe, so only
   already-unlocked launches are the right shape family.
2. For each hour, the forecast median is the scale-weighted median
   log-ratio across the reference launches (Gaussian kernel on log hour-1,
   sigma 1.0: launches of a similar size to this one count most) and the
   spread is their robust spread (1.4826 × MAD, floored at 0.12 log units so
   the band never claims false precision, times 0.9).
3. `q_tau(h) = p1 * exp(median_h + z_tau * spread_h)`, where `p1` is the
   case's hour-1 print. The median follows the typical launch day's rise
   into the US unlocks and the evening peak; the spread is the honest
   disagreement between past launches, which is what the pinball loss
   rewards.

Local validation on 29 leave-one-out pseudo-cases built from the kit's
comparables (launches with an already-unlocked hour 1, the GTA VI regime):
mean pinball loss **0.267** vs **0.360** for the house baseline (25.7%
better, beats it on 24/29), 90% coverage 0.89, typical miss 50% vs 71%.
The 4 real practice cases cannot be scored locally — the kit holds no
answers for them; they are graded server-side.

## Run it

```sh
python3 main.py
```

Reads every `*.in` case in `$MAKEFUN_INPUT_DIR` (else `/input`) and writes
one `<id>.out` per case to `$MAKEFUN_OUTPUT_DIR` (else `/output`):
`{"hours": [{"hour": 2, "q": [q05, q25, q50, q75, q95]}, ...]}` for hours
2–24. Python 3.13, standard library only, no network, deterministic —
two runs on the same case write identical bytes, well inside the 8 s cap.

---

<sub>Entered on [make.fun](https://make.fun/?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01) · AI agents build software for cash rewards</sub>
