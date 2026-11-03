# Code for observed trends and analyses related to Greening, Browning, and Freshening at the Pacific Arctic Inflow Shelves

Jupyter notebooks that produce every figure in the manuscript and its supplement. This is a
snapshot of a working research project, not a package. Each notebook has been cut down to the
cells that the manuscript's figures actually need; outputs are left in place so a reader can see each
figure next to the code that made it. Note that much of this code has been written with help from 
claude code.

If you use this code, cite the Zenodo archive: https://doi.org/10.5281/zenodo.23921319 


## Setup

```bash
conda env create -f environment.yaml
conda activate pace_chl_v2      # name as given in environment.yaml
```

**Earthdata login.** All NASA inputs (MODIS-Aqua ocean colour and SST, MERRA-2, MODIS cloud
properties, PACE, NSIDC sea ice) are streamed with the `earthaccess` library. You
need a free NASA Earthdata account (https://urs.earthdata.nasa.gov). The first
`earthaccess.login()` call prompts for it, or put the credentials in `~/.netrc`. Sea-surface
salinity is read from the OISSS archive at oisss.esr.org, and Yukon discharge from USGS NWIS;
both are open.

Every input product, its version, DOI and the month it was accessed is listed in Table 1 of the
paper. 

## How the notebooks fit together

The repository has two numbered folders, run in order.

**`1_pull_data/` — pull notebooks (run first).** These stream the satellite products, reduce
them to the DBO boxes or to an EASE-2 grid, and write checkpoint files (`*_ckpt.pkl`,
`*_products.pkl`, NetCDF). They are slow (hours for the daily products) and resumable. None of
them draws a paper figure.

**`2_figures/` — figure notebooks.** One notebook per figure (or group of figures), named by
the figure number it produces. They read the checkpoints and will fail with a file-not-found
until the matching pull notebook has been run. A few of them also stream their own inputs
(noted in the table).

Checkpoints are written to the home directory, so paths are expressed as
`Path.home() / "<file>"`. Every such line is marked `# add your path` so you can redirect it; a
handful of machine-specific absolute paths have been replaced with the literal string
`ADD_YOUR_PATH`. Several notebooks also honour an environment variable `MODIS_CKPT_DIR`.

### `1_pull_data/`

| Notebook | Product | Writes | Needed by |
|---|---|---|---|
| `01_modis-aoemp-chl-daily` | Lewis & Arrigo AO.emp chl-a from daily MODIS Rrs | `modis_LA_dbo_DAY_v2_ckpt.pkl` | Fig. 2, 8, S5, S12, S13 |
| `02_modis-chlor-a-daily-ease2` | standard chlor_a, daily, bucket-binned to EASE-2 | `modis_ease2_products.pkl` | Fig. 3 |
| `03_modis-bbp443-monthly` | bbp(443) monthly, plus a chlor_a helper pass | `modis_bbp_dbo_ckpt.pkl`, `modis_chl_dbo_ckpt.pkl` | Fig. 2, 8, S2, S3, S12, S13 |
| `04_modis-adg443-8day` | adg(443), 8-day | `modis_adg_dbo_8d_ckpt.pkl` | Fig. 2, 8, S3, S12, S13 |
| `05_modis-kd490-8day` | Kd(490), 8-day | `modis_kd490_dbo_8d_ckpt.pkl` | Fig. 8, S3, S12, S13, S15 |
| `06_modis-sst-daily` | nighttime SST, daily | `outputs_MODIS_SST/` zarrs, `box_ts_portable.pkl`, `monthly_sst_cache.nc`, `annual_sst_cache.nc` | Fig. 6, 8, S3 |
| `07_sea-ice-ssmi-and-nsidc0747` | Part A: NSIDC-0079 sea-ice concentration + 15 % edge composites; Part B: NSIDC-0747 day of retreat | `ssmi_seaice_ckpt.pkl`, `dbo_edge_maps.pkl`, `nsidc0747_retreat_products.pkl` | Fig. 4, 8, S6 |
| `08_sea-surface-density` | Part A: monthly SST stack + TEOS-10 surface density; Part B: backfill to 2011 and per-pixel density trend | `monthly_sst_ease2.nc`, `sea_surface_density_sigma0_ease2_monthly.nc`, `monthly_sst_ease2_2011on.nc`, `sigma0_trend_map_ease2.nc` | Fig. 6 |
| `09_bering-and-anadyr-caches` | Part A: full-Bering 8-day adg(443) and Kd(490) subsets; Part B: OISSS stack on the western (Anadyr) grid; Part C: region means for boxes A/B/C and DBO 2 | `bering_full_cache/`, `oisss_west_cache/monthly_sss_ease2west_gauss.nc`, `ease2_output/ease2_region_means_ckpt.pkl` | Fig. 7, S8, S9, S14 |

### `2_figures/`

Where a
notebook holds several figures, each figure's section starts from its own imports, so you can run
only the part you need.

| Figure | Notebook (cell) | Reads / notes |
|---|---|---|
| Fig. 1 map of DBO sites | made outside this repository (GIS); not a notebook | — |
| Fig. 2 chl / bbp / adg trend maps | `fig02_trend-maps-chl-bbp-adg` (cell 24) | streams the three products itself, saves `*_trends.npz`, then draws from the saved trends |
| Fig. 3 high-chl event days | `fig03_4a_6_S3_S4_re-do-figs` (cell 6) | `modis_ease2_products.pkl` |
| Fig. 4.1 sea-ice edge, decadal | `fig03_4a_6_S3_S4_re-do-figs` (cell 9) | `dbo_edge_maps.pkl` |
| Fig. 4.2 day of retreat per box | `fig04b_sea-ice-day-of-retreat` (cell 14) | `ssmi_seaice_ckpt.pkl`, `nsidc0747_retreat_products.pkl` |
| Fig. 5.1 wind-speed trend map | `fig05_wind-speed-and-direction` (cell 14) | builds the MERRA-2 u/v stack itself if absent |
| Fig. 5.2 wind-direction panels | `fig05_wind-speed-and-direction` (cell 16) | same stack |
| Fig. 6a surface-density trend map | `fig03_4a_6_S3_S4_re-do-figs` (cell 25) | `sigma0_trend_map_ease2.nc` |
| Fig. 6b density per box | `fig03_4a_6_S3_S4_re-do-figs` (cell 16) | SST + SSS monthly stacks |
| Fig. 7 browning / freshening 2×2 | `fig07_S14_browning-freshening` (cell 12) | `oisss_west_cache`, `bering_full_cache/adg443` |
| Fig. 8 scorecard | assembled outside this repository; the trend values come from the per-box tables printed by `1_pull_data/03`–`07`, `figS05_S06_S12_fig8rows_chl-record` Part C (chl rows), `fig04b_sea-ice-day-of-retreat` (ice rows), `fig05_wind-speed-and-direction` (wind rows) and `figS04_oisss-sss-box-trends` (salinity row) | checkpoints |
| Fig. 9 Fichot S275–295 map + east/west boxplot | `fig09_fichot-s275-map-and-boxplot` (cells 19, 20) | streams the cruise-window Rrs itself; `fichot_*.pkl` |
| Fig. S1 match-up window + AO.emp validation | `figS01_modis-chl-matchups` (cells 19, 25); `figS01_pace-chl-matchups` (cell 13) | both build their own L2 match-up caches; need the cruise chemistry logs |
| Fig. S2 chl vs bbp | `figS02_S03_S13_product-record-checks` (cell 10) | bbp, chl checkpoints |
| Fig. S3 SST trends + pre/post-2024 split | `fig03_4a_6_S3_S4_re-do-figs` (cell 14); `figS02_S03_S13_product-record-checks` (cells 23, 25) | `monthly_sst_cache.nc`, `box_ts_portable.pkl`, adg/Kd/bbp checkpoints |
| Fig. S4 salinity trend map + per-box | `fig03_4a_6_S3_S4_re-do-figs` (cell 19); `figS04_oisss-sss-box-trends` (cell 16) | `figS04` builds the OISSS monthly stack itself if absent |
| Fig. S5 fall event days | `figS05_S06_S12_fig8rows_chl-record` (cell 13) | `modis_ease2_products.pkl` |
| Fig. S6 sea-ice concentration per box | `figS05_S06_S12_fig8rows_chl-record` (cell 14) | `ssmi_seaice_ckpt.pkl` |
| Fig. S7 Yukon discharge | `figS07_S08_S09_yukon-and-attribution` (cell 1) | USGS NWIS (downloaded in-notebook) |
| Fig. S8 Yukon vs turbidity (4 panels) | `figS07_S08_S09_yukon-and-attribution` (cells 10, 12, 14, 16) | `ease2_region_means_ckpt.pkl` |
| Fig. S9 DBO2 adg teleconnection map | `figS07_S08_S09_yukon-and-attribution` (cell 28) | `bering_full_cache/adg443`, `ease2_region_means_ckpt.pkl` |
| Fig. S10 cloud-free / valid days | `figS10_S11_clouds-and-valid-days` (cell 18 PACE panel; cell 40 MODIS panel) | PACE L3M dailies (downloaded in-notebook); cloud products pickle |
| Fig. S11 cloud-fraction trends | `figS10_S11_clouds-and-valid-days` (cell 38) | streams MODIS cloud fraction itself, writes `modis_cloudfraction_products.pkl` |
| Fig. S12 nested-record panels, five products | `figS05_S06_S12_fig8rows_chl-record` (cell 9) | all checkpoints |
| Fig. S13 tail-end sensitivity | `figS02_S03_S13_product-record-checks` (cell 39) | all checkpoints |
| Fig. S14 SSS east vs west DBO2 | `fig07_S14_browning-freshening` (cell 21) | `oisss_west_cache` |
| Fig. S15 Kd(490) per-pixel trend map | `figS15_kd490-pixel-trend-map` (cell 25) | `modis_kd490_dbo_8d_ckpt.pkl` (pre-refresh copy; see the note in the notebook) |

Trend statistics throughout are Theil–Sen slopes on deseasonalised anomalies with a
Mann–Kendall test corrected for lag-1 autocorrelation.

## In situ data

In situ chlorophyll-a and CDOM come from the Distributed Biological Observatory cruise datasets
at the Arctic Data Center (DOIs in Table 1); the validation notebooks expect the cruise
chemistry-log CSVs in a local folder (`# add your path`). 

## Station coordinates

`DBO1_8_StationAndBoxCoordinates_DBO4nOnly_072522.xlsx` defines the DBO station and box
polygons used by every notebook.

## License

MIT (see `LICENSE`).
