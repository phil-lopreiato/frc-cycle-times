# frc-cycle-times

A single-page web app for analyzing FRC match cycle times (qualifications and playoffs) across events, weeks, districts, and seasons, using data from [The Blue Alliance](https://www.thebluealliance.com/).

## What it does

**Cycle time** is the elapsed time between the *actual start* of one qualification match and the next. This tool lets you see how consistently an event is running on schedule, how individual events compare within a week or district, and how seasons trend over time.

## Views

| View | URL pattern | Description |
|---|---|---|
| Home | `#home`, `#home/2025` | All events for a season grouped by week. Shows live current CT for running events. |
| Event Detail | `#event/2026nyli2` | Per-match log (quals + playoffs) with cycle times, Δ schedule, qual stats/charts, playoff stats, and a live "Next Up" card. |
| Week Aggregate | `#week/2026/3`, `#week/2026/cmp` | Pooled stats and per-event comparison table for all events in a given week. |
| Season Overview | `#overall`, `#season/2025` | Pooled stats and per-event table for an entire season, plus a cycle time by week chart. |
| District Home | `#district/2026fim` | Season weeks for a single district. |
| District Week | `#district/2026fim/3` | Per-event table for one district week. |
| District Overall | `#district/2026fim/overall` | Pooled stats across all district events. |
| Regionals Home | `#regionals` | Regional events grouped by week. |
| Regionals Week | `#regionals/3` | Per-event table for one regionals week. |
| Regionals Overall | `#regionals/overall` | Pooled stats across all regional events. |

Seasons 2015–2026 are supported. Championship events are labeled **CMP**.

## Stats

### Per-event and pooled
- **In Scope Matches** — played matches after outlier exclusion (with outlier count subtitle)
- **Mean CT** / **Median CT** (with σ std dev subtitle) / **Min CT** / **Max CT**
- **Avg Scheduled CT** — average gap between scheduled start times (excluding break gaps)
- **Avg Δ Schedule** — how far ahead (positive) or behind (negative) the event ran on average
- **Matches / Team** — typical number of qual matches each team plays (mode across teams, excluding surrogates)

### Charts
- **Cycle time histogram** — distribution of actual cycle times per event
- **Δ schedule line chart** — per-match ahead/behind at the event level
- **Δ schedule bar chart** — per-event, on aggregate views
- **Cycle Time by Week** — on season/district/regionals overview pages; three lines:
  - Dashed blue — average *scheduled* CT by week
  - Solid blue — average *actual* CT by week
  - Solid purple — *median* actual CT by week

### Aggregate tables
All per-event aggregate tables are sortable by any column:
**Week · Event · Location · Matches · Teams · Matches/Team · Avg Sched CT · Mean CT · Median CT · Std Dev · Avg Δ Sched**
(and a **Live Δ** column when any event in the table is currently running).
Unplayed/future events are always sorted to the bottom.

### Per-block stats (event detail)
A collapsible card shows stats broken down by *schedule block* (stretches of qual matches between breaks), including match range, count, mean, median, std dev, and Δ schedule per block, plus a playoffs summary row.

## Outlier detection

Outlier cycle times are excluded from all stats and charts (but remain visible in the match log, greyed out and labelled).

A match's cycle time is flagged as an outlier if **any** of the following apply:

1. **IQR fence** — the cycle time falls outside Q1 − 1.5 × IQR or Q3 + 1.5 × IQR, computed separately for quals and playoffs. (Requires at least 4 data points in that phase; skipped otherwise.)
2. **Replay / out-of-order** — the match's actual start time is earlier than a lower-numbered match's actual start time, indicating it was replayed or recorded out of order. Replays are also excluded from the Δ schedule chart.

Break matches (first match after a scheduled gap > 15 minutes) are never assigned a cycle time at all, so they are neither included in stats nor counted as outliers.

## Live event features

When an event is currently running:

- **Event detail page** auto-refreshes match data every 60 seconds using HTTP conditional requests (`ETag` / `If-None-Match`). The page only re-renders when TBA reports new data (non-304 response).
- **"Next Up" card** — a live card shown when the next match is known:
  - **Next Match** — upcoming match number, with expected start time as subtitle
  - **Ahead / On Time / Behind** — schedule delta based on the last played match's actual vs. scheduled start, with previous match cycle time as subtitle
  - **Current CT** — live elapsed time since the last match started (ticks every second), with scheduled CT as subtitle
  - **Countdown** — time remaining to expected start; shows `—` once that time has passed
  - Expected start = `max(next match scheduled time, last match actual start + scheduled CT)`
- **Week aggregate pages** also auto-refresh and show a **Live Δ** column for running events.
- **Connection lost banner** — if any periodic refresh fails (e.g. network drop), a red banner appears at the top of the page showing the time of the last successful update. It dismisses automatically on the next successful refresh.

## Caching

- **Past-year events** — match data cached indefinitely in `localStorage`.
- **Current-year completed events** — cached indefinitely once `end_date` has passed.
- **Live / in-progress events** — never written to `localStorage`; memory-only so each page load fetches fresh data.
- **Auto-refresh** — uses `ETag` / `If-Modified-Since` conditional requests; a `304 Not Modified` response skips re-processing and re-rendering entirely.
- Cache is keyed with a version prefix (`ct_v3_…`); bumping `LS_VERSION` in the source invalidates all entries.

## Usage

Open `index.html` directly in a browser — no build step or server required. A [The Blue Alliance API key](https://www.thebluealliance.com/account) is embedded in the file.

## Data source

All match and event data comes from the [TBA API v3](https://www.thebluealliance.com/apidocs/v3). The API key is sent as a query parameter (`?X-TBA-Auth-Key=…`) to avoid CORS preflight issues.

## Dependencies

- [Chart.js 4.4](https://www.chartjs.org/) (loaded from CDN) — all charts
- No other runtime dependencies
