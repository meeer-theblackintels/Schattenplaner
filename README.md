# Schattenplaner

Offline table-and-shade recommendation tool for Biergarten service. No server, no database, no build step, one HTML file plus one Excel file.

Given a party size and a reservation time, it recommends the best available table: prioritizing shade, respecting capacity, and accounting for combined tables. Available in German and English, toggled live in the app.

## Why this exists

Biergarten seating gets assigned by memory and habit, which breaks down under two conditions: staff turnover, and the fact that shade position changes hour by hour and month by month. This tool turns a shade observation log into something anyone on shift can query in two taps, without needing to know the garden by heart.

## Files

| File | Purpose |
|---|---|
| `biergarten-table-finder.html` | The app. Open it in any browser. |
| `shade-schedule-template.xlsx` | Starting template for your shade data. Fill it in, then import it into the app. |

## Getting started

1. [Download the template](shade-schedule-template.xlsx) and fill in your real table data (see **Data format** below). Replace the example rows, they're placeholders.
2. Open `biergarten-table-finder.html` in a browser.
3. Go to **Imported Data** (**Importierte Daten**) and upload your filled-in Excel file.
4. Switch to **Find a Table** (**Tisch finden**), enter party size and time, and read the recommendation.

The app remembers the last file you imported in that browser (`localStorage`), so step 3 only needs to happen once per device, until you re-import an updated sheet.

## Data format

The Excel file has two sheets. Column headers are bilingual (e.g. `Table (Tisch)`) and the parser matches on either language, so you can rename them to plain German or plain English if you prefer, as long as the row still starts with something recognizable as "Month" and "Table," or "Combo Name" and "Table 1."

### Sheet 1: Shade Schedule (`Schattenplan`)

One row per table, per month.

| Column | Meaning |
|---|---|
| `Month` | Arrangement name (e.g. `August`). Copy the full table block again for each new month. |
| `Table` | Table identifier (e.g. `G4`, `A10`). |
| `Capacity` | Real seat count. Required — rows without it are skipped on import. |
| `11`–`15` | Mittagdienst, hourly. Mark `X` if fully shaded that hour, leave blank if not. |
| `17:00`–`20:00` | Abenddienst, half-hourly. Same marking rule. |

### Sheet 2: Table Combinations (`Tischkombinationen`)

One row per pair (or trio) of tables that can be physically pushed together.

| Column | Meaning |
|---|---|
| `Combo Name` | Label, e.g. `A4+A7`. |
| `Table 1` / `Table 2` / `Table 3` | Member tables. Table 3 is optional. |
| `Combined Capacity` | Real seat count when combined. Not assumed to be the sum of the parts. |
| `Block Solo Use (X)` | Optional. Mark `X` to fully withhold the member tables from single-table recommendations, reserving them for the combo. Leave blank for default behavior (see below). |

Combo shade is never entered manually. It's computed at query time as the intersection of the member tables' shade data, a combo counts as shaded only if every member table is shaded at that moment.

## How recommendations are ranked

For a given party size and time, candidates are filtered to those with enough capacity, then sorted:

1. **Shaded before sunny.**
2. **Plain tables before combo-affected ones.** A table that's part of a combo is demoted, not hidden, unless that combo has `Block Solo Use` marked, in which case it's excluded outright.
3. **Smallest table that still fits**, to avoid handing a large table to a small party.

If a table isn't shaded yet but will be later in the same service block, the app tells you when ("Sun now — fully shaded from 19:30") instead of just marking it unshaded.

## Known limitations

- **No live booking or occupancy tracking.** This is a single-lookup recommendation tool, not a reservation system. It doesn't know that a table you looked up five minutes ago is now occupied. Two overlapping reservations can still both be told the same table is free.
- **Shade data is only as accurate as your observation log.** It's not computed from sun position or geometry, it reflects whatever hours you walked the garden and recorded.
- **Persistence is per browser, per device.** Clearing browser data or switching computers means re-importing the Excel file once.
- **No live Google Drive import.** Cross-origin restrictions on Drive's file endpoints make this unreliable without a backend and an exposed API key; import is manual, by design.
