# frc-cycle-times

A single-page web app for analyzing FRC qualification match cycle times across events, weeks, and seasons, using data from [The Blue Alliance](https://www.thebluealliance.com/).

## What it does

**Cycle time** is the elapsed time between the start of one qualification match and the next. This tool lets you see how consistently an event is running on schedule, how individual events compare within a week, and how seasons compare over time.

### Views

| View | URL | Description |
|---|---|---|
| Home | `#home` / `#home/2025` | All events for a season, grouped by week. Includes a live filter when events are running. |
| Event Detail | `#event/2026nyli2` | Per-match log with cycle times, Δ schedule, stats, and charts for a single event. |
| Week Aggregate | `#week/2026/3` / `#week/2026/cmp` | Pooled stats and per-event comparison table for all events in a given week. |
| Season Overview | `#overall` / `#season/2025` | Pooled stats and per-event table for an entire season, plus an avg CT by week line chart. |

Seasons 2015–2026 are supported. Championship events are labeled **CMP**.

### Stats shown

- Mean, median, std dev, min, max actual cycle time
- Average scheduled cycle time (from the PDF/TBA schedule)
- Mean Δ schedule (how far ahead/behind the event ran on average)
- Cycle time distribution histogram
- Δ schedule line chart per match (event view) or bar chart per week/event (aggregate views)

### Data quality

- **Outlier detection**: IQR-based (1.5× fence). Outlier matches are shown greyed out in the match log and excluded from stats.
- **Break detection**: gaps > 15 minutes in the *scheduled* times are treated as breaks and excluded from cycle time calculation.
- **Cross-day matches**: if a match's actual start time falls on a different date than its scheduled time (e.g. pushed to the next day), it is excluded as an outlier.

### Live features

When an event is currently running:
- Event detail page auto-refreshes every 60 seconds
- A **Current CT** counter ticks up from the last played match's actual start time (shown when the next match is within 15 minutes)
- Week overview shows each live event's latest Δ schedule, sortable
- Week overview also auto-refreshes every 60 seconds

## Usage

Open `index.html` directly in a browser — no build step or server required. A [The Blue Alliance API key](https://www.thebluealliance.com/account) is embedded in the file.

## Data source

All match and event data comes from the [TBA API v3](https://www.thebluealliance.com/apidocs/v3). The API key is sent as a query parameter (`?X-TBA-Auth-Key=...`) to avoid CORS preflight issues.

## Dependencies

- [Chart.js 4.4](https://www.chartjs.org/) (loaded from CDN) — all charts
- No other runtime dependencies
