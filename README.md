# The Digital Border: Replication Files

Replication package for

> Makharadze, L. (2026). *The Digital Border: Data Localisation and the Margins of Services Trade.* ISET Working Paper Series, International School of Economics at Tbilisi State University (ISET).

The paper estimates the effect of cross-border data-flow restrictions (data localisation regulation, DLR) on bilateral services trade with a structural gravity model estimated by PPML and three-way fixed effects (importer-year, exporter-product-year, pair-product), following Heid, Larch & Yotov (2021). Headline result: an active DLR reduces bilateral services trade by about 9.8% (β = −0.1032, p = 0.024), operating mainly through the intensive margin.

## Repository layout

```
notebooks/                         Jupyter notebooks (R and Python kernels), in pipeline order below
data/
  mapping/dti_bpm6_mapping_final.xlsx   DTI × BPM6 regulation mapping used in the paper (authoritative input)
  raw/Digital Trade Indicators Measurements.xlsx   Raw DTI dataset (input to step 3)
  raw/                             Other raw source files, NOT committed (see data/raw/README.md)
  final_dataset/                   Generated: merged panel and CEPII subset
outputs/                           Generated: tables, figures, fitted models
reference_outputs/tables/          Tables as produced for the paper, for checking a replication run
```

## Replication workflow

Run the notebooks from the `notebooks/` folder (all paths are relative to it).

| Step | Notebook | Kernel | Input | Output |
|---|---|---|---|---|
| 1 | `dataset_creation.ipynb` | R | UN Comtrade API (needs key) | `data/raw/data_result.csv` |
| 2 | `cepii_preprocess_notebook.ipynb` | R | `data/raw/Gravity_csv_V202211/Gravity_V202211.csv` | `data/final_dataset/cepii_gravity_subset.csv` |
| 3 | `DTI_BPM6_Mapping_Final_Pipeline.ipynb` *(optional)* | Python | `data/raw/Digital Trade Indicators Measurements.xlsx` | `data/mapping/DTI_BPM6_Mapping.xlsx` |
| 4 | `dti_comtrade_final_merge.ipynb` | R | `data/mapping/dti_bpm6_mapping_final.xlsx`, `data/raw/data_result.csv` | `data/final_dataset/comtrade_dti_final.csv` |
| 5 | `gravity_thesis.ipynb` | R | merged panel + CEPII subset (+ WDI via API) | `outputs/gravity_results/` (all main-text models, event study, robustness, extensive margin) |
| 6 | `extensive_margin_country.ipynb` | R | merged panel (+ WDI via API) | `outputs/gravity_results/` (country-level Melitz-Chaney variety counts) |

### What each step does

1. **Comtrade pull.** Downloads bilateral services imports for all individual reporters and partners, all EBOPS codes, 2000-2025, in rate-limited batches, and binds them into one deduplicated CSV. The two subscription keys are read from the environment variables `COMTRADE_KEY_1` and `COMTRADE_KEY_2` (for example in `~/.Renviron`); never hard-code keys.
2. **CEPII pre-processing.** Slims the 1.2 GB CEPII Gravity V202211 file to the gravity controls used (distance, contiguity, common language, colonial ties, GATT/WTO/EU membership, FTA/RTA, Social Connectedness Index) for 2000-2019. Run once.
3. **DTI × BPM6 mapping.** Maps each of the 6,992 DTI regulations to the BPM6 Table 10.1 service categories, splits telecom/computer/information services into BPM6 9.1/9.2/9.3, flags partner-specific measures and EU-level measures, and writes the mapping and reasoning sheets. The published workbook `data/mapping/dti_bpm6_mapping_final.xlsx` additionally reflects the LLM-assisted classification and human review described in Appendix C of the paper, so **steps 4-6 read the committed workbook**, not the output of this notebook. Step 3 documents the rule-based construction and lets reviewers audit or rebuild it.
4. **Merge.** Builds Pillar-6 regulation activity timelines per importer, BPM6 category and year (with GDPR disaggregated to the EU-27 and the US Team Telecom entry excluded), harmonises country names, maps EBOPS codes to BPM6, and merges the flags into the Comtrade panel. Includes validation checks.
5. **Main estimation.** Constructs the treatment `dlr_any`, merges CEPII controls (with a Brexit override of joint EU membership), runs the headline PPML (`fixest::fepois`, clustered by pair), robustness and extension models, classic gravity benchmarks, the calendar-cohort event study, the forward-lead placebo, income heterogeneity and the extensive-margin models, and exports all tables and figures.
6. **Country-level extensive margin.** Counts traded service varieties per country pair and year and estimates the DLR effect on them.

## Software

* **R ≥ 4.3** with IRkernel. Packages are installed automatically by each notebook: `fixest`, `dplyr`, `tidyr`, `stringr`, `lubridate`, `purrr`, `readr`, `readxl`, `httr`, `jsonlite`, `ggplot2`, `scales`, `viridis`, `ggthemes`, `patchwork`, `modelsummary`, `kableExtra`, `gt`, `writexl`, `WDI`, `countrycode`, `broom`.
* **Python ≥ 3.10** for step 3: `pandas`, `numpy`, `openpyxl`.
* Memory: the merged panel is about 0.6 GB on disk; 16 GB RAM is recommended for step 5.

## Data sources and citation

* Ferracane, M. F., & van der Marel, E. (2021). Digital Trade Integration database.
* UN Comtrade Database, services trade (EBOPS 2010 / BPM6), United Nations Statistics Division.
* Conte, M., Cotterlaz, P., & Mayer, T. (2022). *The CEPII Gravity Database.* CEPII Working Paper 2022-05.
* World Bank, World Development Indicators.
* Bailey, M., et al. Social Connectedness Index (via CEPII Gravity).

Users must comply with each provider's terms of use. The raw DTI export is included for convenience and should be cited as above; raw UN Comtrade extracts and the full CEPII file are not redistributed here.

If you use this code or the mapping workbook, please cite the working paper above.

## Contact

Luka Makharadze, International School of Economics at Tbilisi State University (ISET).
