# Individual-Project

**Preprocessing and comparison pipelines for HPLC-DAD/MS chromatography time-course data**

## Overview

This project processes chromatography data (HPLC-DAD and LC-MS traces) collected across a reaction time course, and compares several signal-preprocessing strategies to see how each one changes the resulting spectra. The guiding question is: *how do different preprocessing pipelines change the data?*

Raw traces are read in per sample, interpolated onto a common time axis, and then run through parallel pipelines (baseline correction, normalization + PCA, Savitzky–Golay smoothing) so the outputs can be compared side by side.

## Data

- `Data1/` and `Data2/` contain raw two-column CSVs (`time`, `intensity`) exported from the instrument, one file per sample/timepoint.
- Filenames encode the experimental metadata and are parsed automatically by the import scripts, e.g. `CYP_DAD_green_0_m4.CSV` → enzyme `CYP`, instrument `DAD`, species/group `green`, time point `0`, sample `m4`.
- `Data1` uses the filename order `enzyme_instrument_species_time_sample`; `Data2` uses `enzyme_instrument_species_sample_time` — the two import scripts (`01_import2` and `01_DataFrames.R`) parse each accordingly.
- Time points observed: 0, 2, 5, 15, 30 (minutes).
- Instruments: `DAD` (diode-array detector) and `MS` (mass spec).

## Pipeline

| Script | Purpose |
|---|---|
| `scripts/01_import2` | Reads `Data1` + `Data2`, parses filenames into metadata, interpolates all samples onto a shared time grid, combines both datasets, and runs an initial PCA. |
| `scripts/01_DataFrames.R` | Reads `Data2`, does the same import/interpolation, then branches into several parallel preprocessing pipelines (below) and writes each to `data/`. |
| `scripts/02_pca.R` | Runs PCA on the raw and baseline-corrected wide-format data; extracts scores, loadings, and variance explained. |
| `scripts/03_PlottingFunctions.R` | Scree-plot helper for visualizing variance explained by each principal component. |
| `scripts/plotDemo.R` | Interactive `plotly` versions of the long-format line plots, with per-sample tooltips. |

### Preprocessing pipelines (in `01_DataFrames.R`)

1. **Pipe One — Baseline correction**: `ptw::baseline.corr()` on the wide-format spectra.
2. **Pipe Two — Normalize + PCA**: `tidymodels` recipe (`step_normalize` + `step_pca`, 5 components).
3. **Pipe Three — Smoothing**: Savitzky–Golay filter (`prospectr::savitzkyGolay`) applied after baseline correction.
4. **Pipe Five — Smoothing (raw)**: Savitzky–Golay filter applied to the raw, non-baseline-corrected data, for comparison against Pipe Three.

Each pipeline writes its long- and wide-format outputs to `data/`, and comparison plots (e.g. `plots/basicLong.png`) are saved along the way.

## Requirements

R, plus:

```r
install.packages(c(
  "tidyverse", "patchwork", "here", "arrow", "ptw",
  "tidymodels", "GGally", "prospectr", "scales", "plotly"
))
```

## Usage

1. Place raw CSVs in `Data1/` and `Data2/` following the naming convention above.
2. Run `scripts/01_import2` and/or `scripts/01_DataFrames.R` to import, interpolate, and preprocess the data — outputs land in `data/`.
3. Run `scripts/02_pca.R` for PCA on the processed data.
4. Use `scripts/03_PlottingFunctions.R` and `scripts/plotDemo.R` for static and interactive visualizations.

## Project status

Actively in progress — this is a working comparison of preprocessing strategies rather than a finished analysis. Ideas for additional pipelines (different smoothing windows, alternative baseline methods, etc.) are welcome.
