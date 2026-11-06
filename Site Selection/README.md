# Capstone Data, Part 2: Goudge Basins Screening

**Project:** *Identifying Sites on the Martian Surface with High Potential for Microfossil Preservation*
INDS 490 Capstone Project Seminar, Interdisciplinary Studies B.S., Astrobiology & Abiogenesis,
Individualized Study Program, University of Maryland, Baltimore County
**Author:** Sofia Dominesey
**Advisors:** Dr. Stephen Freeland, Dr. Dillon Mahmoudi
**Last updated:** November 2026

---

## Purpose

This folder holds the second stage of the capstone analysis. Here I screen the 425 ancient lake basins in the Goudge et al. (2016) catalog and narrow them to a shortlist of sites worth a closer look for microfossil preservation. Jezero Crater (`OBL_45`) is the reference site.

The workflow has four steps:

1. **Morphology score (AHP).** Each basin is scored 0–1 on four criteria from the Goudge catalog: preservation/degradation state, inlet type, Strahler stream order, and basin type (open vs. closed). The criteria are weighted with the Analytic Hierarchy Process (consistency ratio = 0.019).
2. **Basin size.** Each basin gets a search radius, taken from Fassett & Head (2008) lake areas where they exist and from the Robbins & Hynek crater database otherwise. Basins with neither use the median radius (~16.8 km).
3. **Hydrous minerals.** I count Carter et al. (2013) hydrous mineral detections within 1× and 1.5× each basin's search radius.
4. **Geologic age** *(in progress).* Basins are joined to the USGS global geologic map (Tanaka et al., 2014) to estimate unit age (Noachian / Hesperian / Amazonian).

The output so far is a top-20 shortlist: basins with at least one hydrous detection within 1.5× their radius, ranked by AHP score.

---

## How to run

1. Open `Goudge_basins.Rproj` in RStudio. Paths are built with `here::here()`, so the project has to be opened from this folder.
2. Install the required packages if you don't have them:
   ```r
   install.packages(c("tidyverse", "here", "readxl", "geosphere"))
   ```
3. Knit or run `Part_2.Rmd` from top to bottom. It reads from `R Data/` (plus `basins_base_table.xls` in the main folder) and writes results to `Outputs/`.

Nothing in `ArcGIS Data/` or `Archive/` is needed to run the R code.

---

## Folder contents

```
data_p2/
├── README.md                              ← this file
├── Goudge_basins.Rproj                    ← RStudio project (open this first)
├── Part_2.Rmd                             ← main analysis notebook
├── Part_2.html                            ← knitted version of Part_2.Rmd
├── basins_base_table.xls                  ← Goudge et al. (2016) attribute table (main input)
├── to-dos.Rmd                             ← personal task list (not part of the analysis)
├── R Data/                                ← data files read by Part_2.Rmd
├── ArcGIS Data/                           ← ArcGIS Pro inputs, layers, shapefiles, metadata
│   ├── Goudge Craters Locations Shapefile/
│   └── Goudge Crater Extents Shapefile/
├── Outputs/                               ← files produced by Part_2.Rmd
└── Archive/                               ← old files no longer used
```

### Main folder

| File | Description |
|---|---|
| `Part_2.Rmd` | Main analysis: loads the basins, runs the AHP weighting and scoring, sizes the basins, counts hydrous detections, and writes the shortlist. The geology section has started but isn't finished yet. |
| `Part_2.html` | Knitted output of `Part_2.Rmd`. Recreated every time the notebook is knit. |
| `basins_base_table.xls` | Attribute table from the Goudge et al. (2016) basin shapefile, exported from ArcGIS Pro. One row per basin (425 rows: 205 closed, 220 open). |
| `Goudge_basins.Rproj` | RStudio project file. |
| `to-dos.Rmd` | Running to-do list. |

### `R Data/`

Everything here is read by `Part_2.Rmd`, except the ArcGIS copy of the join table.

| File | Source | Description |
|---|---|---|
| `basins_w_size_craters.csv` | ArcGIS Pro spatial join | One-to-many spatial join of Goudge basins with buffered Robbins & Hynek craters: one row per basin–crater pair. Field names are cut to 10 characters (see key below). Used for basin size. |
| `basins_w_size_craters.dbf`, `.cpg` | ArcGIS Pro spatial join | ArcGIS copy of the same table. R reads the `.csv`. |
| `fassett_head_2008_table2.csv` | Fassett & Head (2008), Table 2 | Lake area, volume, and watershed area for 73 open-basin lakes. `lake_number` matches the Goudge OBL `Num`. Used for basin size. |
| `hydrous_sites_carter2013.csv` | Carter et al. (2013) | Point locations (`lon`, `lat`) of 1,855 orbital (CRISM/OMEGA) hydrous mineral detections. Used for the mineral counts. |
| `mars_global_geology.csv` | Tanaka et al. (2014), USGS SIM 3292, https://doi.org/10.3133/sim3292 | Attribute table of the global geologic map: unit code, description, and area. For the geology step (in progress). |

### `ArcGIS Data/`

Files used or produced in ArcGIS Pro. R doesn't read any of them.

| File | Source | Description |
|---|---|---|
| `robbins_craters_clean.csv` | Robbins & Hynek crater database (2020 release) | Cleaned crater database. Longitudes converted from 0–360°E to −180–180° to match the Goudge catalog. Input to the ArcGIS buffer and spatial join. |
| `craters_with_buff.lyrx` | ArcGIS Pro | Layer file for the crater buffers (each crater buffered by its radius, geodesic, GCS_Mars_2000). |
| `schema.ini` | ArcGIS Pro export | Column types for the exported CSVs. ArcGIS writes it automatically. |
| `*.xml` | ArcGIS Pro export | Metadata ArcGIS wrote alongside the exported tables (`basins_w_size_craters`, `mars_global_geology`). |
| `Goudge Craters Locations Shapefile/` | Goudge et al. (2016) | The original catalog of closed-basin (CBL) and open-basin (OBL) lakes, as point locations (`Goudge_et_al_2016_Geology_CBL_OBL_Catalogue.*`). |
| `Goudge Crater Extents Shapefile/` | ArcGIS Pro | Polygon shapefile of each Goudge basin's extent (crater/lake size), made for the geologic-unit step (`Goudge_crater_extents.*`). |

The `.shp`, `.shx`, `.dbf`, `.prj`, `.sbn`, `.sbx` and `.cpg` files in each shapefile folder make up one shapefile and need to stay together.

### `Outputs/`

| File | Description |
|---|---|
| `top_20_basins.csv` | The top 20 basins with a hydrous detection within 1.5× their search radius, ranked by AHP score. Columns are explained in the key below. |

### `Archive/`

| File | Description |
|---|---|
| `basin_crater_sizes_robbins.csv` | Earlier version of the basin-to-crater matching (one crater per basin, with offset and a `match_note`). Replaced by the ArcGIS spatial join. Kept for reference only. |

---

## Data dictionary

### Goudge et al. (2016) basin fields (`basins_base_table.xls`)

| Field | Meaning |
|---|---|
| `FID` | Row ID added by ArcGIS (not meaningful) |
| `Lat`, `Lon` | Basin center (degrees; Lon −180 to 180) |
| `Type` | `CBL` = closed-basin lake, `OBL` = open-basin lake |
| `Num` | Basin number. Numbering restarts for each type, so the script combines it with `Type` into `basin_id` (e.g. `OBL_45`). |
| `Inlet` | `VN` = valley-network fed, `II` = isolated inlet valley |
| `Degr` | Crater degradation state (Robbins & Hynek, 2012); 4 = freshest, 0 = heavily eroded / not catalogued |
| `DegrNote` | Notes on degradation state |
| `Strahler` | Strahler stream order of the inlet network |
| `StrahNote`, `StrahRef` | Notes on the stream order and where it came from (Hynek et al., 2010 where possible) |
| `Ejecta` | How eroded the crater ejecta is |

### Fields I added in `Part_2.Rmd`

| Field | Meaning |
|---|---|
| `basin_id` | Unique ID: `Type` + `_` + `Num` |
| `score_preserve`, `score_inlet`, `score_order`, `score_type` | Per-criterion scores (0–1) |
| `score` | Final weighted score (0–1). Weights: preservation 0.546, inlet 0.232, stream order 0.138, basin type 0.084 |
| `rank` | Rank by `score` (ties share a rank) |
| `crater_id`, `crater_diam_km`, `crater_offset_km` | Matched Robbins & Hynek host crater, its diameter, and its distance from the basin center |
| `lake_area_km2`, `lake_diam_km` | Fassett & Head lake area and equivalent circular diameter |
| `basin_diam_km` | Basin size used: lake diameter if available, otherwise crater diameter (`NA` if neither) |
| `size_source` | Where `basin_diam_km` came from (`Fassett & Head lake` / `Robbins & Hynek crater` / `none`) |
| `search_radius_km` | Half of `basin_diam_km`, or the median radius (~16.8 km) if no size |
| `hydrous_n` | Carter et al. detections within the search radius |
| `hydrous_n_1.5x` | Detections within 1.5× the search radius |
| `hydrous_nearest_km` | Distance to the nearest detection (km) |

### Truncated ArcGIS field names (`basins_w_size_craters.csv`)

| Field | Meaning |
|---|---|
| `CRATER_ID` | Robbins & Hynek crater ID |
| `DIAM_CIRC_` | Crater diameter, circle fit (km) |
| `LAT_CIRC_I` | Crater center latitude |
| `LON_CIRC_I` | Crater center longitude (original 0–360°E) |
| `correct_lo` | Crater center longitude converted to −180–180° |
| `DEG_RIM`, `DEG_EJC`, `DEG_FLR` | Rim, ejecta, and floor degradation |
| `BUFF_DIST` | Buffer distance used in ArcGIS |

---

## Key methods notes

- **Distances** are great-circle (Haversine) distances using Mars' mean radius of 3,389.5 km (`geosphere::distHaversine`).
- **Crater matching:** when a basin falls inside more than one crater, the script keeps the crater whose center is closest, as long as the basin is within half that crater's radius. This gives 343 matches. Jezero comes out at 47.5 km.
- **Size coverage:** 72 basins sized from lake area, 298 from crater diameter, and 55 with no size (median radius used).
- **The 1.5× buffer** catches detections on crater rims and just outside the lake. It also allows for uncertainty in the basin sizes and in the Carter point locations (one point per detection, not the full outcrop).
- **Jezero (`OBL_45`):** 0 detections within its radius, 1 within 1.5×, nearest about 21.8 km away.

---

## References

- Carter, J., Poulet, F., Bibring, J.-P., Mangold, N., & Murchie, S. (2013). Hydrous minerals on Mars as seen by the CRISM and OMEGA imaging spectrometers: Updated global view. *JGR: Planets, 118*, 831–858.
- Fassett, C. I., & Head, J. W. (2008). Valley network-fed, open-basin lakes on Mars: Distribution and implications for Noachian surface and subsurface hydrology. *Icarus, 198*, 37–56.
- Goudge, T. A., Fassett, C. I., Head, J. W., Mustard, J. F., & Aureli, K. L. (2016). Insights into surface runoff on early Mars from paleolake basin morphology and stratigraphy. *Geology, 44*(6), 419–422.
- Hynek, B. M., Beach, M., & Hoke, M. R. T. (2010). Updated global map of Martian valley networks and implications for climate and hydrologic processes. *JGR: Planets, 115*, E09008.
- Robbins, S. J., & Hynek, B. M. (2012). A new global database of Mars impact craters ≥1 km. *JGR: Planets, 117*, E05004.
- Tanaka, K. L., et al. (2014). *Geologic map of Mars* (USGS Scientific Investigations Map 3292). https://doi.org/10.3133/sim3292
