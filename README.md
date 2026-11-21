# WATER — Water Quality Explorer

Built for the **AISO Water Hackathon**.

WATER explores surface-water quality in the area of the Dutch water board
Amstel, Gooi en Vecht (AGV). It has three parts:

- an **interactive map** showing where pollutants exceed a reference level, how
  that changes over the years, and which pollutants dominate;
- a **nitrogen analysis pipeline** that cleans the AGV data and fits exploratory
  models of where nitrate may be coming from;
- a small **read-only API** meant to serve the map from a local database.

> **This repository contains dummy data only.** No real AGV measurements, model
> outputs or access keys are included. See [Data](#data).

---

## Repository layout

```
WATER-main/
├── map_mock_final.html     Interactive map (self-contained, dummy data embedded)
├── dummy.csv               Synthetic measurements in the AGV 35-column export format
├── api/
│   └── server.py           Read-only JSON API + static file server (stdlib only)
├── scripts/                Data cleaning, nitrogen analysis, report builders, tests
│   └── README.md           Detailed method notes for the nitrogen pipeline
└── weight.txt              Side experiment: 12×12 weight grids (not used by any script)
```

## Quick start

### 1. Open the map

Open `map_mock_final.html` in a browser. No build step or server is needed.

An internet connection is needed for the Leaflet library, the PDOK base map and
the province/municipality borders. Without it the map still loads the sites, just
without a background.

What you can do on the map:

- colour areas by the share of samples above the reference level, per pollutant
  or for all pollutants combined;
- zoom into an area to see individual sites and their statistics;
- filter by year and pollutant;
- open **Trends** for a per-year line chart and the **pie** view for the pollutant
  mix;
- switch between English and Dutch.

### 2. Run the tests

```bash
python -m unittest discover -s scripts -p "test_*.py"
node scripts/test_nitrogen_map.js
node scripts/test_daily_report.js
```

### 3. Run the analysis pipeline (needs the real data)

The pipeline expects the raw AGV exports in `datasets/AGV/`, which are not in this
repository. Full instructions, options and output descriptions are in
[`scripts/README.md`](scripts/README.md). In short:

```bash
python scripts/prepare_nitrogen.py          # nitrogen evidence tables + report
python scripts/build_nitrogen_map.py        # local Leaflet nitrogen map
python scripts/fit_nitrate_distance.py      # v1: distance-only source baseline
python scripts/fit_nitrate_network_distance.py   # v2: water-network distance
python scripts/fit_nitrate_daily.py         # v3: independent daily fits
python scripts/build_report_bundle.py       # collect everything into report/
```

## Requirements

- Python 3.10+ with `pandas`, `numpy`, `scipy`, `matplotlib` and `pyproj`
- For the network-distance and GIS steps: `pip install -r scripts/requirements-gis.txt`
  (`shapely`, `networkx`)
- Node.js, only to run the JavaScript tests
- A modern browser for the map

## Data

The hackathon dataset (AGV water-quality, pumping-station and water-level exports)
was provided by the AISO Water Hackathon organisers behind an access key. It is
**not** redistributed here, and neither is the key.

`dummy.csv` replaces it for demos and tests:

- 56 fictional monitoring sites (codes `DUM-…`), 2,432 rows, 2022–2024;
- 10 pollutants: total phosphorus, orthophosphate, ammonium, nitrate, total
  nitrogen, chloride, cadmium, copper, zinc and *E. coli*;
- values are synthetic and say nothing about real water quality;
- site names marked "(fictional)" and the `DUM-FX` locations are invented.

The map's reference level for each pollutant is the 75th percentile of the
values in the dataset. It is a relative benchmark, **not a legal standard**.

If you have hackathon access, put the raw files in `datasets/AGV/`. The
`datasets/` and `report/` folders are git-ignored, so real data and generated
outputs stay out of the repository.

## API (work in progress)

`api/server.py` is a standard-library HTTP server that is meant to replace the
map's embedded data with live queries:

| Endpoint | Returns |
|---|---|
| `/api/meta` | pollutants, categories, years, reference levels |
| `/api/sites` | every site with per-year aggregates |
| `/api/sites/{code}` | exact statistics for one site |
| `/api/areas` | share above reference per area |
| `/api/trends` | per-pollutant, per-year series |
| `/api/composition` | pollutant or category mix of exceedances |
| `/api/geo/areas` | area polygons |
| `/api/export` | one offline snapshot |
| `/api/health` | health check |

It is **not runnable yet**. It reads `datasets/water.sqlite`, which is built by
`scripts/load_db.py`, and serves pages from `frontend/`. Neither of those is in the
repository yet.

## Limitations

- The source-finding models are exploratory. They rank candidate locations; they do
  not prove where pollution comes from. See `scripts/README.md` for the assumptions.
- Map areas are approximations: each site covers the land closest to it, up to
  1.5 km.
- With dummy data, every number on the map is synthetic.

## Credits

Made for the AISO Water Hackathon. Original data: Waterschap Amstel, Gooi en Vecht.
Base map: PDOK / Kadaster. Map library: Leaflet.
