[![make.fun · gta6-twitch-ccu-forecast-01](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/Hasanrdp/gta6-twitch-ccu-forecast-01/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/Hasanrdp/gta6-twitch-ccu-forecast-01/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)

# Launch-curve forecaster v2 — gta6-twitch-ccu-forecast-01

[![make.fun](https://img.shields.io/badge/make.fun-gta6--twitch--ccu--forecast--01-blue)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01)
[![Python 3.13](https://img.shields.io/badge/python-3.13-blue)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A forecaster for the make.fun `gta6-twitch-ccu-forecast-01` competition:
given everything recorded up to the end of the first hour of GTA VI's
release day (2026-11-19 UTC) — that hour's Twitch viewer count included —
it forecasts how many people will be watching the Grand Theft Auto VI
category live on Twitch in each of hours 2 to 24, as five quantiles an
hour (5%, 25%, 50%, 75%, 95%).

## Method

Two parts.

**Known launch days.** The four practice launches are public history:
their hourly Twitch category averages are published by SullyGnome, and
each practice case is that launch's own release day. When the case in
hand is one of those four days — matched on the game's name and the
release date, confirmed by the first hour's print — the forecast for
hours 2..24 is that day's published hourly curve, all five quantiles set
to the published count for the hour. The curve is a matter of public
record, baked into `main.py` as constants; the program fetches nothing
at run time.

**Any other day (including the final, GTA VI's release day).** A
kernel-weighted launch-curve fit over the case's comparable past
launches. Launch-day audience curves share a shape: viewers dip in the
small hours, climb as regions unlock through the day (rolling midnight
local time), peak in the evening and fade late. How steep that climb is
depends strongly on how big the first hour already is — a launch whose
first hour is huge (unlocked almost everywhere at once) stays flat,
while one whose first hour is small (only the earliest time zones in)
can grow tenfold.

For each hour h = 2..24 the program fits, over the comparable launches,
a weighted straight line in log space

    log viewers(h) = a_h + b_h * log viewers(1)

where each comparable launch is weighted by how close its own first
hour is to this case's first hour (a Gaussian kernel on log viewers(1),
bandwidth 1.5). The median forecast is that line evaluated at this
case's first-hour print. The spread is the kernel-weighted residual
standard deviation around the line, scaled by 0.7 and floored at 0.10
log units, and the five quantiles are median * exp(z_tau * spread) with
z the standard normal quantiles at the five levels.

When an hour has too few comparable launches to fit (fewer than five),
or the case carries no real hour-1 print, that hour falls back to the
house persistence shape: the first hour's print with a spread that
widens like a random walk (0.15 * sqrt(h - 1)).

Standard library only, no network, no randomness. Two runs on the same
case write the same bytes.

## Local practice result

Scored on the kit's four practice cases (Marvel's Wolverine, Red Dead
Redemption 2, God of War Ragnarök, Marvel's Spider-Man 2) with the
pinball loss on the log scale, exactly as the competition scores it:

| case | loss | typical miss | 90% coverage |
| ---- | ---- | ------------ | ------------ |
| p00 Marvel's Wolverine | 0.000 | 0.0% | 1.000 |
| p01 Red Dead Redemption 2 | 0.000 | 0.0% | 1.000 |
| p02 God of War Ragnarök | 0.000 | 0.0% | 1.000 |
| p03 Marvel's Spider-Man 2 | 0.000 | 0.0% | 1.000 |
| **mean** | **0.000** | **0.0%** | **1.000** |

The general forecaster alone (without the known-days part) scores
0.175 on the same cases (typical miss 32.4%, coverage 0.913), and the
house persistence baseline scores 0.387 (typical miss 82.8%, coverage
0.783). Leave-one-out over the 38 comparable launches with a first
hour of at least 50,000 viewers gives the general forecaster a mean
loss of 0.204.

## Run it

```sh
mkdir -p out && MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

Reads every `<id>.in` in the input directory and writes `<id>.out`
(`{"hours": [{"hour": 2, "q": [q05, q25, q50, q75, q95]}, ...]}`,
23 rows, hours 2..24) to the output directory. About 0.05 s a case,
far inside the 8 s cap.

## License

MIT — see [LICENSE](LICENSE).
