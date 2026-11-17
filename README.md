# Butte County Precipitation Dashboard

Cumulative water-year precipitation for five CDEC stations in Butte County:
Cohasset (CST), Openshaw (OPS), Carpenter Ridge (CAR), Paradise (PDE), and
Chico University Farm (CES). Covers WY2016 through the current water year,
with a historical-average overlay for context.

**Live dashboard:** https://agubc-vina.github.io/butte-precip-dashboard/

## How it stays current

Completed water years are frozen in `cdec_cache/` and not re-fetched once
their readings reach the end of September, or once they were fetched at least
60 days after September 30 (a record still short by then is short at CDEC;
`CACHE_SETTLE_DAYS`). A monthly station's September total posts weeks after
September 30, so a year saved earlier is pulled again until then. The current
(in-progress) water year is pulled from CDEC on each refresh. The spike filter
cleans out single-point sensor glitches before computing cumulative totals.

The page's "Sources and methods" section, at the bottom, explains the data
and calculations in plain terms.

Through April 1, 2027 the dashboard refreshes daily at 10am Pacific: a
scheduled job on a Mac starts the GitHub Actions workflow
(`.github/workflows/refresh.yml`) at 10:00. The workflow also has its own
daily schedule at 18:00 UTC as a backup; GitHub starts scheduled runs late,
by 2.5 to 7.5 hours in October and November 2026. After April 1, 2027 the backup schedule lets runs
through on Sundays only. To trigger an on-demand refresh, go to the
[Actions tab](https://github.com/AGUBC-vina/butte-precip-dashboard/actions/workflows/refresh.yml)
and click "Run workflow."

The page works out the current water year from the data, so no year needs
changing when a new water year starts on October 1.

`refresh_dashboard.py` sets its clock to Pacific time before it does
anything, so "today" is the Pacific date wherever it runs. GitHub's servers
run on UTC; without this, a run after 4pm Pacific counted tomorrow as today
and compared the current year with tomorrow's average. CDEC's End date stops
at midnight at the start of that day, so the current water year is requested
through tomorrow to include today's readings so far.

## Saving and printing charts

Each station chart and the Water Year Totals chart has two buttons:

- **Save PNG** downloads the chart as an image, named for the station, water
  year and refresh date (for example `butte-precip-CST-WY2027-2026-11-16.png`).
- **Print / PDF** prints only that chart on one landscape letter page; choose
  "Save as PDF" in the print window for a PDF.

Both draw the chart as it is set on the page (water year, comparison period,
average line on or off, station filter for the totals chart), with a title
above it and a source footer below it: CDEC as the data source with the
refresh time, and AGUBC with the dashboard address for the averages and
chart. The export code is in `index.html` under "Chart export".

## Historical average

The dashed line is the average cumulative total for each day of the water
year across every completed water year in the dashboard (WY2016 through the
year before the current one). It is the dashboard's own average, not a DWR
published normal. For the current water year, the station tiles compare the
total to date with the average for the same day; for a completed year they
compare the year's total with the full-year average. A station that reported
no readings in a water year is shown as "No readings reported" rather than
as zero.

Days are counted on a 365-day water year: in leap years February 29 shares
February 28's slot (`wy_day_index`), so every year lines up by date when
averaged.

Only water years whose record runs the whole year go into an average.
Openshaw's CDEC record starts January 20, 2016, so its average starts with
WY2017 and its WY2016 bar is left off the totals chart.

## Comparison periods

The "Compare with" selector and the comparison table set the current year
against these averages, each built the same way as the main one:

- Last 5 years and last 10 years: the completed years before the current one.
- SGMA period: WY2015 through the last completed year.
- WY2007–2022: Cohasset and Carpenter Ridge only, the two stations with a
  complete record for those years. The label "driest 16 years since
  WY1929–1944" is an AGUBC calculation from DWR's Northern Sierra 8-Station
  Index (CDEC station 8SI, monthly, WY1920–2026, retrieved 2026-11-15): the
  only 16-year spans with a lower average start in WY1920 through WY1929.

Years before WY2016 are fetched once for these averages and frozen in
`cdec_cache/`; they are not shown as their own water years. The periods are
defined in `comparison_periods()` in `refresh_dashboard.py`.

## Gauge corrections

`GAUGE_READD` in `refresh_dashboard.py` lists readings where a gauge was reset
to zero mid-season and its earlier total was later added back in one step.
The add-back is removed so it is not counted as rain. The one entry so far is
Cohasset on 2009-05-07, which would otherwise put WY2009 at 43.14" instead of
33.58". Raw cache files are not changed.

## Files

- `index.html` — the dashboard (self-contained, single file)
- `refresh_dashboard.py` — fetches new data, applies spike filter, rebuilds the HTML
- `requirements.txt` — Python dependencies (`requests` only)
- `cdec_cache/` — frozen raw data for completed water years (one JSON per station per WY), including WY2007–2015 for the comparison periods
- `.github/workflows/refresh.yml` — scheduled + manual refresh job

## Running locally

```bash
pip install -r requirements.txt
python3 refresh_dashboard.py
open index.html
```

## Data source

[California Data Exchange Center (CDEC)](https://cdec.water.ca.gov/),
sensor 2 (precipitation, accumulated). Hourly data for CST and CAR; daily
for OPS and PDE; monthly manual entry for CES. CES has reported nothing to
CDEC for October or November 2026 (checked 2026-11-15); its last monthly value
is September 2026.

## Spike filter

Single-point readings that fall more than 1" below both their immediate
neighbors are dropped before cumulative totals are computed. This catches
sensor glitches like the 2019-06-02 CAR reading that briefly dipped from
72.32" to 15.00" and back, which would otherwise inflate the water-year
total by ~57".

## License

**Code.** The build and refresh scripts, the dashboard HTML/CSS/JS, and any
GitHub Actions workflows are released under the MIT License. See
[`LICENSE`](LICENSE).

**Content.** The written analysis, figures, tables, and derived values are
released by Agricultural Groundwater Users of Butte County (AGUBC) under
[Creative Commons Attribution 4.0 International](https://creativecommons.org/licenses/by/4.0/).
See [`LICENSE-CONTENT`](LICENSE-CONTENT). Attribute to AGUBC and link back to
this repository.

**Underlying data.** The third-party datasets named elsewhere in this README are
not AGUBC's to license. They remain subject to their own terms, and neither
license above extends to them.
