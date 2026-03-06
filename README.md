# 🌪️ VORTEX — Tornado Forecast Dashboard

A real-time tornado forecasting dashboard for **Jonesboro, AR** built with vanilla HTML/JS. No backend, no build step — just one file you can host anywhere.

**Live data. No API keys required.**

---

## What It Does

VORTEX pulls live atmospheric data from multiple free sources and synthesizes them into a single-page severe weather dashboard. It automatically refreshes every 15 minutes.

### Data Sources

| Source | What It Provides |
|--------|-----------------|
| [Open-Meteo HRRR](https://open-meteo.com/) | CAPE, CIN, wind shear, lifted index, PWAT, surface conditions — updated hourly |
| [NWS API](https://www.weather.gov/documentation/services-web-api) | Active alerts, official forecast, KJBR live surface observations |
| [SPC GeoJSON](https://www.spc.noaa.gov/) | Day 1 convective outlook (categorical + tornado probability + sig-tor hatching) |
| [SPC MCD Feed](https://www.spc.noaa.gov/products/md/) | Active mesoscale discussions covering the area |
| [Rain Viewer](https://www.rainviewer.com/api.html) | Animated NEXRAD composite radar (last ~1 hour + nowcast) |

---

## Dashboard Panels

- **Tornado Risk Banner** — categorical risk level (LOW / MARGINAL / ELEVATED / HIGH) with an estimated tornado probability % and the top contributing factors
- **NWS Active Alerts** — tornado warnings pulse red, watches pulse orange
- **KJBR Live Surface Observation** — temp, dewpoint, wind, gusts, humidity, pressure, visibility, conditions
- **Instability / CAPE** — arc gauge, LI, CIN, wet bulb temp
- **Wind Shear Profile** — 0–80m and 0–120m vector shear
- **Surface Conditions** — HRRR model temp, RH, pressure, visibility, precip probability, VPD
- **24-Hour Trend Chart** — CAPE, wind gusts, and CIN over the next 24 hours
- **Live Radar** — animated NEXRAD base reflectivity on a dark Leaflet map with play/pause controls
- **Hodograph & Wind Profile** — canvas-drawn hodograph, STP, SRH proxy, directional veering, LCL estimate
- **SPC Day 1 Convective Outlook** — point-in-polygon check against live SPC GeoJSON
- **SPC Mesoscale Discussions** — any active MCDs covering Jonesboro
- **Atmospheric Condition Alerts** — threshold-based computed alerts
- **NWS Official Forecast** — 14-period forecast strip (drag to scroll)
- **Hourly Outlook** — 24-hour scrollable timeline with CAPE-colored bars (drag to scroll)

---

## Tornado Probability Formula

The estimated probability is computed from a weighted scoring system across 10 atmospheric parameters, then passed through an `x^2.2` power curve to keep marginal days conservative.

| Factor | Max Points | Notes |
|--------|-----------|-------|
| CAPE | 20 | Primary instability |
| 0–80m Wind Shear | 22 | Thresholds scaled to 0–80m layer (not 0–1km) |
| CAPE × Shear Synergy | 15 | Requires both to be elevated simultaneously |
| CIN (cap) | 10 | Moderate cap (−50 to −150 J/kg) scores highest — "loaded gun" |
| Precipitable Water | 8 | Deep moisture |
| Relative Humidity | 5 | Surface moisture / LCL proxy |
| Lifted Index | 8 | Partial weight (redundant with CAPE) |
| Precipitation Probability | 2 | Capped low — elevated convection scores high PP with zero tornado risk |
| SPC Outlook | 20 | Human expert anchor — SPC tornado % mapped directly |
| NWS Active Alert | 15 | Tornado warning = near-ceiling signal |

**Max raw score: 125 → mapped via x^2.2 power curve → capped at 99%**

A SLGT-day moderate environment scores roughly 25%. An ENH-day well-stacked environment scores roughly 49%. An active tornado warning with stacked parameters approaches 80%+.

---

## Hosting on GitHub Pages

1. Rename `tornado-dashboard.html` to `index.html`
2. Create a new **public** GitHub repository
3. Upload `index.html` and `README.md`
4. Go to **Settings → Pages → Branch: main → Save**
5. Your dashboard will be live at `https://YOUR-USERNAME.github.io/REPO-NAME`

Updates go live within ~1 minute of pushing a new `index.html`.

---

## Running Locally

Just open `index.html` in any modern browser. All APIs are called client-side. No local server required, though some browsers block cross-origin requests from `file://` — if you hit issues, serve it with:

```bash
npx serve .
# or
python -m http.server 8080
```

Then visit `http://localhost:8080`.

---

## Tech Stack

- Vanilla HTML / CSS / JavaScript — no frameworks, no bundler
- [Chart.js](https://www.chartjs.org/) — 24-hour trend chart
- [Leaflet.js](https://leafletjs.com/) — radar map
- [CartoDB dark tiles](https://carto.com/basemaps/) — dark map base layer
- [Rain Viewer API](https://www.rainviewer.com/api.html) — radar tiles and animation frames
- Google Fonts (Share Tech Mono, Barlow Condensed) — UI typography

---

## Notes & Limitations

- **Location is hardcoded** to Jonesboro, AR (35.8423°N, 90.7043°W). To adapt for another location, update `LAT`, `LON`, and `STATION` at the top of the script.
- **Wind shear is 0–80m**, not the 0–1km layer SPC uses operationally. Thresholds are scaled accordingly, but it remains a proxy.
- **STP (Significant Tornado Parameter)** displayed in the hodograph panel uses 0–80m shear as a proxy for both the SRH and bulk shear terms — treat it as a relative indicator, not an operational STP value.
- All NWS data is specific to the **LZK (Little Rock)** forecast office and **KJBR** observation station.
- The dashboard requires an active internet connection. All data is fetched live — nothing is cached beyond the current session.

---

*Built for personal severe weather monitoring. Not a substitute for official NWS warnings or professional meteorological guidance.*
