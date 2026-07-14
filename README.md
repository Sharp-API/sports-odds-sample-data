# Sports Betting Odds Sample Dataset (SharpAPI)

[![license](https://img.shields.io/badge/license-CC%20BY%204.0-06b6d4)](https://creativecommons.org/licenses/by/4.0/)
[![rows](https://img.shields.io/badge/rows-9%2C851-06b6d4)](#files)
[![sources](https://img.shields.io/badge/sources-23-06b6d4)](#files)
[![source](https://img.shields.io/badge/source-sharpapi.io-06b6d4)](https://sharpapi.io)

Real odds snapshots from [SharpAPI](https://sharpapi.io), the real-time sports betting odds API: 23 sources spanning sportsbooks, exchanges, and prediction markets. Free to use for research, coursework, backtesting experiments, and data journalism, with attribution (CC BY 4.0, see below).

## Files

| File | Rows | What it is |
|---|---|---|
| `data/worldcup_2026_odds_snapshot.csv` | 6,178 | Full odds board for the 2026 FIFA World Cup, captured 2026-07-13 (semifinals week: Argentina vs England, France vs Spain). 20 sources, 99 market types, from moneylines and Asian handicaps to correct score and outrights. |
| `data/mlb_odds_snapshot.csv` | 3,673 | Same-day MLB slate across 14 of those sources: moneylines, run lines, totals, and derivative markets. |

Both files share one schema: each row is one price on one selection at one sportsbook at capture time.

## Schema

| Column | Type | Description |
|---|---|---|
| `id` | string | Price record id from the API. Not unique across all rows; identify a price by `event_id` + `sportsbook` + `selection`. |
| `sportsbook` | string | Book slug (draftkings, fanduel, pinnacle, novig, kalshi, ...) |
| `event_id` | string | Stable event identifier, join key across books |
| `sport` / `league` | string | e.g. soccer / fifa_-_world_cup |
| `home_team` / `away_team` | string | Event participants |
| `market_type` | string | One of 99 market slugs (moneyline, asian_handicap, total_goals, ...) |
| `selection` | string | The specific outcome priced |
| `selection_type` | string | side, total, outright, ... |
| `odds_american` | int | American odds (+150, -110) |
| `odds_decimal` | float | Decimal odds |
| `odds_probability` | float | Implied probability (vig included) |
| `line` | float or empty | Handicap/total line where applicable |
| `event_start_time` | ISO 8601 | Scheduled start |
| `is_live` | bool | Whether the price was in-play at capture |
| `timestamp` | ISO 8601 | Capture time of this price |

## Quick start

Python:

```python
import pandas as pd
wc = pd.read_csv("data/worldcup_2026_odds_snapshot.csv")
ml = wc[wc.market_type == "moneyline"]
best = ml.loc[ml.groupby(["event_id", "selection"]).odds_decimal.idxmax()]
print(best[["selection", "sportsbook", "odds_american"]].head())
```

R:

```r
wc <- read.csv("data/worldcup_2026_odds_snapshot.csv")
ml <- subset(wc, market_type == "moneyline")
aggregate(odds_decimal ~ selection, ml, max)
```

Ideas: cross-book vig comparison, best-line analysis, implied-probability calibration against results, market-efficiency studies across 23 sources, arbitrage detection exercises.

## Want live or historical data?

This is a static snapshot. The live feed behind it: [SharpAPI](https://sharpapi.io) serves real-time odds from 45+ sportsbooks with sub-89ms SSE streaming, no-vig fair odds, +EV and arbitrage detection, plus historical odds and closing-line data on paid tiers. The free tier needs no credit card: [sharpapi.io/pricing](https://sharpapi.io/pricing). SDKs: [Python](https://pypi.org/project/sharpapi/) and [TypeScript](https://www.npmjs.com/package/@sharp-api/client).

## License and attribution

Data: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Use it freely, including commercially, with attribution:

> Odds data from SharpAPI (sharpapi.io), Sports Betting Odds Sample Dataset, 2026.

BibTeX:

```bibtex
@misc{sharpapi2026odds,
  title  = {Sports Betting Odds Sample Dataset},
  author = {{SharpAPI}},
  year   = {2026},
  url    = {https://sharpapi.io},
  note   = {2026 FIFA World Cup and MLB odds snapshots, 23 sources}
}
```

## Disclaimers

Odds are informational market data, not betting advice. Prices move constantly; this snapshot was accurate only at capture time. Sports betting involves financial risk, is legal only where permitted, and is restricted to those of legal age (21+ in most US states). Please wager responsibly.
