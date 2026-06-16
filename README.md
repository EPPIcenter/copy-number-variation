# Copy number variation estimation

This repository estimates copy number variation (CNV) from [mad4hatter](https://github.com/eppicenter/mad4hatter) sequencing output. Use **`estCNV.qmd`** to run the analysis on your data.

## Prerequisites

1. **mad4hatter results** — Process your sequencing data with mad4hatter first. You need one or more run directories, each containing `allele_data.txt`.
2. **R and Quarto** — Install [R](https://cran.r-project.org/).
3. **R packages** — Install the packages loaded at the top of `estCNV.qmd`:
   - `tidyverse`
   - `mgcv`
   - `ggbeeswarm`
   - `gridExtra`
   - `readxl`

## Quick start

1. Clone or download this repository.
2. Open `estCNV.qmd` in RStudio, VS Code, or your preferred editor.
3. Set the user variables at the top of the document (see [Configuration](#configuration) below).
4. Render the document or run the chunks interactively in R.

5. Review the HTML report and output CSV files.

## Inputs

### 1. Allele data (`allele.data.path`)

Set `allele.data.path` to the parent directory that contains your mad4hatter run folders. The script searches recursively for `allele_data.txt` files.

Each run folder name becomes the `Run` identifier and must match the `Run` column in your manifest.

`allele_data.txt` must include these columns:

| Column        | Description                          |
|---------------|--------------------------------------|
| `sample_name` | Sample identifier                    |
| `target_name` | Amplicon target name                 |
| `reads`       | Read count per allele                |
| `pool`        | Primer pool used for the target      |

### 2. Manifest (`manifest.path`)

Provide a sample manifest as CSV, TSV, or Excel (`.xlsx`). An example is in `madh_utilities_estCNV/manifest_example.csv`.

Required columns:

| Column        | Description |
|---------------|-------------|
| `sample_name` | Sample ID. May be repeated across batches. Must match `allele_data.txt` after optional name standardisation. |
| `CNVControl`  | `TRUE` if the sample is a control known **not** to have a CNV; otherwise `FALSE`. |
| `Batch`       | Library preparation batch. Samples prepped together share a batch. |
| `SuperBatch`  | Group of batches where controls and samples are comparable. Correction factors are calculated per SuperBatch. |
| `Run`         | Sequencing run ID. Must match the mad4hatter output folder name containing that sample's `allele_data.txt`. |

### 3. Panel information (`panel_information/`)

The analysis uses the `panel_information/` directory bundled with this repository. Pool names in your `allele_data.txt` must match subdirectory names under `panel_information/`.

For each pool in your data, the script expects:

```
panel_information/
├── validated_target_filter.tsv          # Targets included in CNV analysis
├── D1.1/
│   ├── D1.1_amplicon_info.tsv           # Amplicon coordinates and target metadata
│   └── D1.1_target_of_interest.tsv      # CNV targets of interest, grouped by gene
├── R1.2/
│   ├── R1.2_amplicon_info.tsv
│   └── R1.2_target_of_interest.tsv
└── ...
```

- **`{pool}_amplicon_info.tsv`** — Defines amplicon coordinates and target names for each pool.
- **`{pool}_target_of_interest.tsv`** — Lists CNV targets of interest and their gene groups (e.g. `HRP2`, `HRP3`, `PM2`).
- **`validated_target_filter.tsv`** — Subset of targets used for filtering and normalisation. Only targets present in both this file and your pools' amplicon info are analysed.

All sequencing runs in a single analysis must use the same set of pools.

## Configuration

Edit the parameter chunks near the top of `estCNV.qmd`. You will likely only need to change allele.data.path and manifest.path:

```r
allele.data.path = "/path/to/mad4hatter/outputs/"
manifest.path = "path/to/manifest.csv"
panel.info.dir = "panel_information/"
validated.target.filter.path = "panel_information/validated_target_filter.tsv"
standardise_sample_name = TRUE   # Trim sequencing suffixes from sample names
use.controls.only = FALSE        # Use only controls (TRUE) or all samples (FALSE) for median correction
```

### Quality filters

Adjust these thresholds as needed:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `reaction.threshold.per.target` | 20 | Minimum mean reads per target within a reaction |
| `Pools.ratio` | 100 | Maximum fold difference allowed between the two reactions |
| `target.read.threshold` | 50 | Minimum reads per locus |
| `ntarget.proportion.threshold` | 0.8 | Minimum proportion of amplicons meeting `target.read.threshold` |

## Workflow overview

1. Read and validate the manifest.
2. Load `allele_data.txt` from all runs and detect primer pools.
3. Load amplicon info and targets of interest from `panel_information/` for each pool.
4. Filter samples and targets by read depth and validated target list.
5. Estimate CNV fold changes using a GAM-based normalisation (`estCNV` function).
6. Use CNV-negative controls within each SuperBatch to calculate correction factors.
7. Apply correction factors and produce final per-gene fold changes.

Review the control plots in the rendered report before interpreting sample results. You may need to adjust `Batch` or `SuperBatch` assignments in the manifest if controls cluster unexpectedly.

## Outputs

Results are written to `allele.data.path`:

| File | Description |
|------|-------------|
| `fold_changes_output.csv` | Fold changes per target group and sample |
| `fold_changes_final_output.csv` | Per-gene fold changes (maximum across target groups within each gene) |

The rendered HTML report includes QC plots, control diagnostics, and heatmaps of fold changes across samples.

## Supported primer pools

The `panel_information/` directory includes amplicon and target-of-interest files for mad4hatter primer pools:

| Pool | Description |
|------|-------------|
| **D1.1 (D1)** | 165 high-diversity targets plus ldh loci for species identification |
| **R1.1** | 82 resistance, species ID, and immune-related targets |
| **R1.2 (R1)** | Reduced R1 pool (47 targets) for improved sensitivity |
| **R2.1 (R2)** | 31 complementary resistance targets |
| **M1.1 (M1)** | PfPHAST minimal panel (41 targets) |
| **M2.1 (M2)** | PfPHAST complementary panel (15 targets) |
| **M1.addon** | Non-*P. falciparum* mitochondrial cytb targets for M1 |

For MADHatTeR, the recommended configuration is two mPCR reactions: one with D1 and R1.2 primers, and one with R2 primers. More protocol details are available from the [EPPIcenter](https://eppicenter.ucsf.edu/resources).

If your data uses different pool names (e.g. `D1` instead of `D1.1`), ensure the `pool` column in `allele_data.txt` matches the directory names under `panel_information/`.
