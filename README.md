# VORTEX — Tornado Forecast Dashboard

A single-file, browser-based severe weather dashboard built for real-time tornado risk monitoring. Designed for Jonesboro, AR (35.8423°N, 90.7043°W) but easily adapted to any location.

![Static Badge](https://img.shields.io/badge/single--file-HTML-blue) ![Static Badge](https://img.shields.io/badge/no_backend-required-green) ![Static Badge](https://img.shields.io/badge/data-live-orange)

---

## Features

- **Live NWS Alerts** — Fetches active tornado warnings, tornado watches, and severe thunderstorm alerts for your area with pulsing visual indicators based on severity
- **Tornado Probability Score** — Custom heuristic scoring model combining 12 atmospheric parameters (CAPE, SRH, bulk shear, CIN, LCL height, precipitable water, dewpoint, lapse rate, lifted index, CAPE×SRH synergy, SPC outlook, and NWS alert status)
- **SPC Convective Outlook** — Auto-fetches Day 1 tornado probability polygons from the Storm Prediction Center's ArcGIS MapServer; manual override buttons let you set the outlook level directly
- **Mesoscale Convective Discussion (MCD)** — Fetches and displays active MCDs from the SPC
- **NWS Hourly Forecast** — Scrollable forecast strip pulled from the NWS API with weather icons, temperature, precipitation probability, and wind
- **24-Hour CAPE Timeline** — Hourly risk color bar showing convective instability trends
- **Live Radar** — Animated radar loop via RainViewer (past 12 frames + nowcast), with play/pause controls and a Leaflet dark basemap
- **Hodograph & Indices** — Wind profile hodograph canvas with key indices (SRH, bulk shear, CAPE, CIN, LCL height, lifted index)
- **Obs Strip** — Surface observations including temperature, dewpoint, wind, pressure, visibility, and relative humidity from the nearest NWS station
- **Computed Alerts** — Threshold-based in-dashboard alerts derived from environmental parameters
- **Debug Panel** — Expandable factor breakdown showing individual scores, raw values, and the full probability equation with a live SRH multiplier display
- **Auto-refresh** — All data reloads every 15 minutes

---

## Data Sources

| Source | Data |
|---|---|
| [Open-Meteo](https://open-meteo.com/) | CAPE, CIN, lifted index, wind profiles, dewpoint, precipitable water, lapse rate, LCL height |
| [NWS API](https://www.weather.gov/documentation/services-web-api) | Active alerts, hourly forecast, surface observations |
| [SPC ArcGIS MapServer](https://www.spc.noaa.gov/gis/svrgis/) | Day 1 tornado probability and significant tornado polygons, active MCDs |
| [RainViewer](https://www.rainviewer.com/api.html) | Animated radar tiles |
| [CartoDB](https://carto.com/basemaps/) | Dark map tiles |

---

## Tornado Probability Model

The dashboard uses a custom heuristic scoring model — **not** a physically derived probability and **not** equivalent to SPC probabilistic outlooks. It is intended as a situational awareness tool.

### Scoring Factors

| Factor | Max Points | Notes |
|---|---|---|
| CAPE | 20 | 30% free, 70% gated on SRH multiplier |
| SRH 0–1km | 15 | Requires 0–6km shear ≥ 15 mph and CAPE > 100 |
| 0–6km Bulk Shear | 15 | 75% free, 25% gated on SRH multiplier |
| CAPE × SRH Synergy | 5 | EHI-based combined instability/rotation bonus |
| CIN (convective cap) | 8 | Handles both "loaded gun" and active outbreak scenarios |
| LCL Height | 8 | Fully gated on SRH multiplier |
| Precipitable Water | 8 | 50% free, 50% gated |
| Surface Dewpoint | 6 | — |
| Lapse Rate 850–500 hPa | 5 | 50% gated |
| Lifted Index | 6 | 50% gated |
| SPC Convective Outlook | 15 | Auto GeoJSON or manual override; SIG TOR adds +3 |
| NWS Alert | 15 | Tornado Warning triggers hard clamp to 85–95% |

**SRH Multiplier:** A smooth ramp function (0 at SRH < 25, 1.0 at SRH ≥ 200 m²/s²) that gates rotation-dependent factors. Shear and instability-only parameters are partially or fully ungated.

**Tornado Warning clamp:** When an active Tornado Warning is detected, the probability output is hard-clamped to 85–95% (scaled by CAPE), overriding the ingredient-based score.

**SPC-clear penalty:** If Day 1 polygon data was successfully loaded and the location falls outside all risk areas, a −8 point penalty is applied.

**Output curve:** `pct = min(99, round((raw / 130)^1.65 × 100))`

*Calibration targets (environmental factors only, no SPC/NWS):*
- Quiet/winter day: 0–3%
- MRGL environment: 5–13%
- SLGT environment: ~22–26%
- ENH environment: ~42–48%
- MDT outbreak: ~54–60%
- Extreme outbreak: ~65–70%

---

## Usage

1. Download `vortex.html`
2. Open it in any modern browser — no server, no install, no dependencies
3. An active internet connection is required for all API calls

> **Note:** The dashboard is hardcoded to Jonesboro, AR. To use a different location, update `LAT`, `LON`, `NWS_OFFICE`, `NWS_GRIDX`, and `NWS_GRIDY` near the top of the `<script>` block. Your NWS grid values can be found at `https://api.weather.gov/points/{LAT},{LON}`.

---

## Tech Stack

- Vanilla HTML/CSS/JavaScript — zero build step, zero frameworks
- [Chart.js 4.4](https://www.chartjs.org/) — hodograph canvas rendering
- [Leaflet 1.9](https://leafletjs.com/) — radar map
- [Barlow Condensed + Share Tech Mono](https://fonts.google.com/) — typography

---

## Limitations & Disclaimers

- **Not for life-safety decisions.** This dashboard is a personal hobby/learning tool. Always rely on official NWS warnings and guidance during severe weather.
- The probability model is a heuristic scoring system, not a statistical or ML-derived forecast.
- SPC GeoJSON polygon data may not be available until the Day 1 outlook is issued (typically by mid-morning).
- Radar data from RainViewer is typically 5–10 minutes delayed.
- Observation data accuracy depends on the nearest NWS ASOS station.

---

## License

MIT — do whatever you want with it, no warranty expressed or implied.
