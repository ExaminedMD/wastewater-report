# What's in the wastewater near you

A one-page web app that turns a US ZIP code into a COVID-19 and influenza A wastewater report. It follows a five-step method: national picture, state level, nearest treatment plants, WastewaterSCAN cross-check, then putting it together. It also adds the Pandemic Mitigation Collaborative's (PMC) latest COVID-19 estimates.

## What the report contains

| Section | Source | What it shows |
|---|---|---|
| At a glance | all | State, plants A and B, and your county for both viruses |
| 1. National picture | CDC NWSS | US and Census-region median level and trend |
| 2. Your state | CDC NWSS | Statewide median, level gauge, 16-week chart, number of sites reporting, limited-coverage warning |
| 3. Nearest plants | CDC NWSS | County map, two nearest plants in detail, next six in a table |
| 4. WastewaterSCAN | WastewaterSCAN | Two nearest WastewaterSCAN plants with recent samples |
| 5. Putting it together | all | Checklist of sustained rises, link to CDC clinical data, plain-language summary |
| PMC | PMC method and pmc19.com | County estimate using PMC's published method, PMC's latest dashboard images and report |
| CDC weekly US flu map | CDC FluView (ILINet) | CDC's own interactive map of influenza-like illness activity by state, opening on the latest week |

Every section states the dates its data covers. The newest CDC week is drawn hollow because CDC notes the latest one to two weeks can change.

## How a ZIP code becomes "the nearest plants"

CDC anonymizes treatment plants, so it publishes no coordinates. It does publish the county FIPS codes each site serves. The app uses those codes as follows:

1. **ZIP to point.** Zippopotam.us (GeoNames data), with OpenStreetMap Nominatim as a backup.
2. **Point to county.** A point-in-polygon test against U.S. Census county boundaries (us-atlas).
3. **Site to counties.** Each CDC site is linked to its counties using the `county_fips` field in CDC's SARS-CoV-2 (`j9g8-acpt`) and influenza A (`ymmh-divb`) sample datasets. The match is checked against the county names in the WVAL dataset, with name matching as a fallback.
4. **Ranking.** Plants serving your county come first. All others are ordered by distance from your ZIP code to the center of the nearest county they serve. Plants placed at the same county center tie on distance. Among ties, plants that report both COVID-19 and influenza A come first, then the larger population served.

The site-to-county mapping is rebuilt from CDC data automatically and cached in the visitor's browser for 7 days. New plants therefore appear without any manual maintenance.

WastewaterSCAN publishes exact plant coordinates, so its plants are ranked by true distance.

## Data sources

| Source | Dataset / endpoint | License |
|---|---|---|
| CDC National Wastewater Surveillance System | Wastewater Viral Activity Level, `data.cdc.gov` dataset `atcp-73re` (updated weekly, usually Fridays) | Public domain |
| CDC NWSS sample data | `j9g8-acpt` (SARS-CoV-2), `ymmh-divb` (influenza A), used only for site-to-county codes | Public domain |
| WastewaterSCAN | Public JSON feed behind data.wastewaterscan.org (`storage.googleapis.com/wastewater-dev-data/json/`) | CC BY-NC 4.0, attribution required |
| Pandemic Mitigation Collaborative | Weekly dashboard images and report PDF on pmc19.com | Re-use with attribution; PMC asks that its work not be monetized |
| CDC FluView weekly US map | ILINet State Activity Indicator Map widget (`gis.cdc.gov/grasp/fluview/main_Widget.html`), the same widget CDC's [Weekly US Map](https://www.cdc.gov/fluview/surveillance/usmap.html) page shows | Public domain |
| Zippopotam.us | `api.zippopotam.us/us/{zip}` | GeoNames, CC BY |
| OpenStreetMap Nominatim | Backup geocoder | ODbL |
| us-atlas 3.0.1 | Census county boundaries via jsDelivr | ISC; Census data public domain |
| d3 7.9.0 | Charts and map, via jsDelivr | ISC |

## About the COVID-19 level labels

On Aug 14, 2026, CDC revised the cut points it uses to label COVID-19 wastewater levels. As a result, the same reading now receives a lower label than before. PMC continues to use the earlier cut points. The report shows CDC's current label everywhere and adds the PMC-standard label next to COVID-19 values.

CDC also recalculates COVID-19 baselines around April 1 and October 1. Each time a report is built, the app compares its cut points with the categories CDC publishes for each site. If CDC has changed them, the app infers the new cut points and says so in the chart captions and in "How this report works". It's still worth updating `cdcThresholds` by hand once CDC announces new values.

## How levels and trends are calculated

- **National, regional and state levels** are the median wastewater viral activity level (WVAL) across reporting sites, as CDC describes for its own dashboard. Values can differ slightly from CDC's published figures.
- **"Rising for 2+ weeks"** means the level rose in each of the last two reporting weeks and by at least 10% overall. **"Up from the week before"** means a single-week rise of 15% or more.
- **WastewaterSCAN trend** compares the average of the last 10 days with the 11 days before. A ratio of 1.5 or more counts as rising, and 1/1.5 or less counts as falling. This is the app's own simple rule, not WastewaterSCAN's trend call.
- **County estimate, PMC method** follows PMC's Technical Appendix (v3.3, June 2026):
  - A county with two or more sites uses the median of sites with data in the latest 3 weeks.
  - A county with one site uses it if it has data in the latest 2 weeks.
  - Any other county is estimated by inverse-distance weighting of up to 8 nearby measured counties.
  
  This is labelled as not an official PMC number. PMC's own "1 in X infectious" figures appear in its dashboard images.

## Not medical advice

Wastewater data describe a community, not any individual. Combine them with symptoms, exposures, test positivity and healthcare data. This project is independent and not affiliated with CDC, WastewaterSCAN or PMC.
