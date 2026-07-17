# Sports Betting Odds Sample Dataset (SharpAPI)

[![license](https://img.shields.io/badge/license-CC%20BY%204.0-06b6d4)](https://creativecommons.org/licenses/by/4.0/)
[![rows](https://img.shields.io/badge/rows-9%2C805-06b6d4)](#files)
[![sources](https://img.shields.io/badge/sources-23-06b6d4)](#files)
[![source](https://img.shields.io/badge/source-sharpapi.io-06b6d4)](https://sharpapi.io)

Real odds snapshots from [SharpAPI](https://sharpapi.io), the real-time sports betting odds API: 23 sources spanning sportsbooks, exchanges, and prediction markets. Free to use for research, coursework, backtesting experiments, and data journalism, with attribution (CC BY 4.0, see below).

## Files

| File | Rows | What it is |
|---|---|---|
| `data/worldcup_2026_odds_snapshot.csv` | 6,132 | Full odds board for the 2026 FIFA World Cup, captured 2026-07-13 (semifinals week: Argentina vs England, France vs Spain). 20 sources, 99 market types, from moneylines and Asian handicaps to correct score and outrights. |
| `data/mlb_odds_snapshot.csv` | 3,673 | MLB during All-Star week, captured the same night across 14 of those sources. Mostly a futures board (World Series, pennant, playoff and division odds; MVP/Cy Young/Rookie of the Year; All-Star selection contracts) plus a small set of regular-season game, inning, and player-prop markets. |

Both files share one schema: each row is one price on one selection at one source at capture time.

**Identifying a price / joining.** A single price is `event_id` + `sportsbook` + `market_type` + `selection` + `selection_type` + `line` (add `home_team` for Kalshi/Polymarket multi-entity contracts). For the World Cup head-to-head matches, `event_id` is a clean cross-book join key — every source (bovada included) shares one id per match, so you can line-shop the same real event directly on `event_id`. Outright/futures and prediction-market rows use source-specific ids, as does most of the MLB file (a futures board), so join those on teams + market rather than `event_id`. Prediction-market sources encode markets as yes/no questions, so `home_team` may hold the question or contract entity.

## Schema

| Column | Type | Description |
|---|---|---|
| `id` | string | Price record id from the API. Unique except 16 betrivers pairs (World Cup) where one id spans both teams of a team-total market. |
| `sportsbook` | string | Book slug (draftkings, fanduel, pinnacle, novig, kalshi, ...) |
| `event_id` | string | Event id. Clean cross-book join key for World Cup matches; source-specific for futures, prediction-market, and MLB rows — see the joining note above |
| `sport` / `league` | string | e.g. soccer / fifa_-_world_cup. `league` is mostly canonical but not uniform: 83 saba MLB rows `mlb_american_league`, 8 bovada MLB award rows `mlb_awards` |
| `home_team` / `away_team` | string | Event participants |
| `market_type` | string | One of 99 market slugs (moneyline, asian_handicap, total_goals, ...) |
| `selection` | string | The specific outcome priced; for player props this is the player/entity, with the Over/Under side in `selection_type` |
| `selection_type` | string | side, total, over, under, outright; needed together with `selection` to uniquely identify player-prop and total markets |
| `odds_american` | int | American odds (+150, -110) |
| `odds_decimal` | float | Decimal odds |
| `odds_probability` | float | Implied probability (vig included) |
| `line` | float or empty | Handicap/total line where applicable |
| `event_start_time` | ISO 8601 | Scheduled start. For prediction-market rows this is the market close/expiry time, not kickoff |
| `is_live` | bool | Whether the price was in-play at capture |
| `timestamp` | ISO 8601 | Capture time of this price |

## Quick start

Python:

```python
import pandas as pd
wc = pd.read_csv("data/worldcup_2026_odds_snapshot.csv")

# most-covered markets and sources
print(wc["market_type"].value_counts().head())
print(wc["sportsbook"].value_counts())

# a price is unique on this key (add home_team for prediction-market contracts)
key = ["event_id", "sportsbook", "market_type", "selection", "selection_type", "line"]
print("duplicate-key rows:", wc.duplicated(key).sum())   # 0 — the key is clean

# best price offered on each distinct outcome, within each source's own event ids
best = wc.loc[wc.groupby(key, dropna=False)["odds_decimal"].idxmax()]
print(best[["sportsbook", "market_type", "selection", "odds_american"]].head())
```

R:

```r
wc <- read.csv("data/worldcup_2026_odds_snapshot.csv")
head(sort(table(wc$market_type), decreasing = TRUE))   # market coverage
head(sort(table(wc$sportsbook), decreasing = TRUE))     # source coverage
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
