# K. marxianus carbon source screen

Data and R code for "High-throughput Microbial Phenotyping across Diverse Conditions: A low cost, practical bench to data workflow" 

We screened 258 *Kluyveromyces marxianus* strains on seven carbon sources (glucose as the control, arabinose, xylose, galactose, cellobiose, lactose, pectin) using a manual pinning tool and a flatbed scanner. Plate images were processed with PhenoTypic, which gave 48,348 colony measurements. 38,778 of those passed quality control. This repo has the raw measurements, the filtered data, the growth rates, and the code behind the QC and statistics in the paper.

## Files

**input/**
- `phenotypic_data.csv`: raw PhenoTypic output, one row per colony per time point
- `filtered_data.csv`: the same data with a `Filter` column added by script 1
- `Modeled_Data.csv`: maximum growth rate for each strain on each carbon source (PhenoTypic, run on the filtered data)
- `Strain_Metadata.csv`: isolation source for each strain

**output/** has the summary tables used to make the figures (see below).

**Scripts** (R Markdown, run chunk by chunk or knit):
- `1_data_analysis`: filtering, then the QC checks (filtering patterns, edge effects, CV, replicate correlation)
- `2_data_modeling`: growth rate summaries, mixed model and ANOVA, lactose comparison

## Running it

1. Run script 1. It writes `input/filtered_data.csv`.
2. Run the filtered data through PhenoTypic's growth modeling to get `Modeled_Data.csv`. This step happens outside of R. Follow DOI 10.5281/zenodo.23026087 
3. Run script 2.

Packages: dplyr, ggplot2, tidyverse, broom, ggpointdensity, scales, afex, lme4, lmerTest

## Filter settings
Colonies are flagged if circularity ≤ 0.7, solidity ≤ 7.5, or compactness ≥ 2.5. After that, outliers in colony area are flagged by IQR (1.5 × IQR) within each strain × carbon source × set × time point. These are set at the top of script 1.

The `Filter` column says what happened to each colony: `Passed`, the name of the filter it failed (several are joined with " & "), or `Passed_IQR` if it passed the morphology filters but was an area outlier. When we count filtered colonies, one that fails several filters is counted once, under the first of circularity, solidity, IQR.

## Columns

| Column | Meaning |
|---|---|
| Strain | Strain ID |
| Condition | Carbon source |
| Set | Which of the three library sets (plates) the strain was on |
| Time | Imaging time point (h) |
| Biological_Replicate, Technical_Replicate | Replicate numbers |
| Grid_Number | Position on the 96-colony array (1 to 96, filled by row) |
| Area | Colony area (pixels) |
| Circularity, Solidity, Compactness | Colony shape metrics from PhenoTypic |
| Filter | QC result (see above) |
| Maximum.Growth.Rate | Maximum growth rate (px/h) |
| Second_Substrate | Isolation source (Dairy, Clinical, Environment, etc.)|

## Output files

| File | From | Used for |
|---|---|---|
| df_binned.csv | script 1 | Fig. 4a-b |
| filtering_by_condition.csv | script 1 | Fig. 4c |
| t_test_results.csv | script 1 | Fig. 4d |
| ECDF.csv | script 1 | Fig. 4e |
| paired_retained_plot.png | script 1 | Fig. 4f |
| growth_rates_wide.csv | script 2 | Fig. 5a |
| lactose_wide.csv | script 2 | Fig. 5c |

Most of the final figures were drawn in GraphPad Prism from these files.

## Things worth knowing

- For the edge-effect test, a colony counts as "edge" if it is on the outer ring of the array or next to a negative-control spot. Negative-control positions for each set are hard-coded in script 1.
- Growth rates are in pixels per hour, so they compare within this dataset but not across imaging setups.
- Not every strain was measured on every carbon source. The mixed model uses all available data. The ANOVA uses only strains measured on all seven.
- Strains from insect and lab sources (n = 5 each) and strains with no isolation source were left out of the isolation-source analysis.

Citation TBD 
