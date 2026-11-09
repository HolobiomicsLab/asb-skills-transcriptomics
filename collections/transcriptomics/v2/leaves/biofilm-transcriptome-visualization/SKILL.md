---
name: biofilm-transcriptome-visualization
description: Use when you have pre-computed log2 fold-change and adjusted p-value
  columns (CSV) for several pairwise biofilm experimental conditions (e.g. AGXX vs
  silver vs V2A steel, or sucrose/glucose/lactose/xylitol) and need data-driven, quadrant-grouped
  identification of extremely changing genes.
when_to_use_negative:
- You only have raw reads or unprocessed count matrices — the library expects pre-computed
  log2 fold-change and adjusted p-value columns in a CSV, not raw RNA-seq or TRANSIT
  output that still needs normalization/correction.
- You need a standard four-way plot without significance filtering or extreme-value
  regions — the README notes that setting the multiplication factor to 1 produces
  a four-way plot, not the scorecard, and the article's contribution is the significance-filtered,
  extensible scorecard.
- Data has not been preprocessed by removing low-expression, noise-prone genes and
  quality control, as the article's workflow assumes.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scorecard_functions
  role: Core Python module providing generate_parameters, data_loading, scorecard_legend,
    scorecard (per-quadrant), reconstruct_scorecard, multiple_view, make_volcano,
    count_frequencies, common_entries, and track_over_exper
  repo: github.com/m89p067/Scorecard
- name: Pandas
  role: Loads the CSV of log2 fold changes and adjusted p-values into memory as a
    DataFrame consumed by the scorecard functions
- name: Python
  role: Language of the free-to-use scorecard software library used to track fold
    changes and statistical significance
  repo: github.com/m89p067/Scorecard
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1007/s00284-025-04435-3
    title: 'A Scorecard for Information Synthesis in Multiple Experimental Conditions:
      Application to Bacterial Biofilm Matrix Transcriptomics'
  source_content_hash: sha256:9fab5fd65b2b7cbc31c0ee266c5934ed181d042af41e79a55a8b1abb10d4d258
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pandas
  - Python
  - scorecard_functions
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# biofilm-transcriptome-visualization

## Summary

A scorecard-based visualization skill from the Python 'scorecard' library (Nascimben & Rimondini, Curr Microbiol 2025) that groups bacterial biofilm transcriptome genes into quadrant regions of interest by log2 fold-change thresholds and statistical significance, replacing crowded four-way/volcano plots across multiple pairwise experimental comparisons.

## When to use

Use when you have pre-computed log2 fold-change and adjusted p-value columns (CSV) for several pairwise biofilm experimental conditions (e.g. AGXX vs silver vs V2A steel, or sucrose/glucose/lactose/xylitol) and need data-driven, quadrant-grouped identification of extremely changing genes.

## When NOT to use

- You only have raw reads or unprocessed count matrices — the library expects pre-computed log2 fold-change and adjusted p-value columns in a CSV, not raw RNA-seq or TRANSIT output that still needs normalization/correction.
- You need a standard four-way plot without significance filtering or extreme-value regions — the README notes that setting the multiplication factor to 1 produces a four-way plot, not the scorecard, and the article's contribution is the significance-filtered, extensible scorecard.
- Data has not been preprocessed by removing low-expression, noise-prone genes and quality control, as the article's workflow assumes.

## Inputs

- CSV file with pre-computed log2 fold-change columns and related adjusted p-value columns per comparison (first row header matching param_dict column names), with gene names/IDs in a separate column
- param_dict parameter dictionary from scorecard_functions.generate_parameters() (base_dir, filename, th_fold_change, multiplication factor, th_significance, FC/padj column names, save_dir, gene_name)
- Folder structure with main_folder containing one subfolder per pairwise experimental comparison (for multiple_view, make_volcano, count_frequencies, common_entries, track_over_exper)

## Outputs

- Individual quadrant plots plus a reconstructed full scorecard figure per comparison
- JSON outcome files of statistically significant entries per quadrant stored in save_dir
- Region-of-interest legend figure
- Volcano plots highlighting genes in scorecard regions of interest
- Entry-frequency summaries (count_frequencies heatmap, common_entries CSV/Excel report with total occurrences)
- Cross-experiment overviews (multiple_view) and longitudinal tracking of gene expression changes (track_over_exper)

## How to apply

First run scorecard_functions.generate_parameters() and configure param_dict: set base_dir/filename to the Dataset CSV, the exact 'FC cond x'/'FC cond y' and 'padj cond x'/'padj cond y' column names, th_fold_change, multiplication factor, th_significance (e.g. p<=0.05 for dataset 1 and 4; p<=0.001 with fold-change thresholds of 2 and 4, multiplication factor 2, for dataset 3), plus save_dir, gene_name, and custom treatment/control names. Load the CSV into a Pandas DataFrame with scorecard_functions.data_loading(param_dict), then draw the regions-of-interest legend via scorecard_functions.scorecard_legend(param_dict). Run per-quadrant classification with scorecard_functions.scorecard(df,param_dict), which stores statistically significant entries for each quadrant with individual quadrant plots (useful for crowded scorecards), then merge the four quadrants with scorecard_functions.reconstruct_scorecard(df,param_dict) controlling image size via use_figsize/figsize_factor or fig_size. Organize output as a main_folder with one subfolder per comparison and use multiple_view, make_volcano, count_frequencies, common_entries, and track_over_exper for cross-experiment overviews, volcano highlighting, frequency summaries, and longitudinal tracking (e.g. dataset 4 time points t=3, 6, 12, 24, 60, 90 minutes). Validate by checking that reported extreme-bioactivation genes appear in the expected quadrant tables and JSON outcome files, e.g. EF.peg.3040/EF.peg.3041 for AGXX vs silver in E. faecalis, SMU_494 downregulated under sucrose, and SAUSA300_RS03525/RS04560/RS08475 over-expressed in dataset 1.

## Related tools

- **scorecard_functions** (Core Python module providing generate_parameters, data_loading, scorecard_legend, scorecard (per-quadrant), reconstruct_scorecard, multiple_view, make_volcano, count_frequencies, common_entries, and track_over_exper) — github.com/m89p067/Scorecard
- **Pandas** (Loads the CSV of log2 fold changes and adjusted p-values into memory as a DataFrame consumed by the scorecard functions)
- **Python** (Language of the free-to-use scorecard software library used to track fold changes and statistical significance) — github.com/m89p067/Scorecard

## Examples

```
import scorecard_functions; param_dict=scorecard_functions.generate_parameters(); df=scorecard_functions.data_loading(param_dict); scorecard_functions.scorecard(df,param_dict); scorecard_functions.reconstruct_scorecard(df,param_dict)
```

## Evaluation signals

- Reconstructed scorecard figure exists as a rendered image in the output directory and quadrant JSON outcome files exist in save_dir.
- Reported genes land in the expected regions: EF.peg.3040 and EF.peg.3041 marked as AGXX-specific extreme activation vs silver coating (dataset 4); SMU_494 flagged as downregulated for sucrose (dataset 3); VAN comparisons concentrated in quadrant Q3 with SAUSA300_RS03525, RS04560, RS08475 over-expressed for other antibiotics (dataset 1).
- Quadrant output records contain the threshold-filtered entries: with dataset 3 settings (fold-change 2 and 4, multiplication factor 2, p<=0.001) fewer outliers are detected, and PEP-PTS/carbohydrate genes (ptcA/ptcB/ptcC, bglP, celR, pttB) appear in the records.
- count_frequencies/common_entries outputs exist before track_over_exper is run (the latter re-uses count_frequencies results), and the folder structure has one subfolder per comparison under main_folder.
- For longitudinal data, track_over_exper confirms time-stable behavior across t=3, 6, 12, 24, 60, 90 minutes for the target genes.

## Limitations

- Typical four-way plots are not filtered by statistical significance and cannot merge information across multiple plots — which is why the scorecard exists, but setting multiplication factor to 1 degenerates into a four-way plot.
- Volcano plots generated via make_volcano can be crowded because minor differences can be statistically significant if the sample size is large enough; individual quadrant plots help with crowded scorecards.
- Volcano plots show one condition per graph, limiting informativeness in multi-variable or time-course experiments; use multiple_view and track_over_exper instead.
- CSV files must have a first-row header with column names matching the FC/padj names in param_dict, and must contain pre-computed values — the library does not compute fold changes or p-values itself.
- No version number is stated for the Python library or Python itself in the discussion section.

## Evidence

- [readme] This function will store statistically significant entries for each quadrant together with plots of individual quandrants.: "This function will store statistically significant entries for each quadrant together with plots of individual quandrants."
- [results] Scorecard applied to dataset 3 with modified thresholds: "the thresholds were modified to detect fewer outliers (Table SM5) using fold-change thresholds of 2 and 4 (aka, multiplication factor of 2) and p"
- [intro] Preprocessing by removing low-expression genes: "After preprocessing by removing genes with low expression, which are prone to technical noise, and quality control,"
- [abstract] Four GEO datasets independently analyzed: "Four GEO datasets related to RNA-seq bacterial biofilm expression levels were independently analyzed"
- [discussion] Scorecard tracks fold changes and statistical significance: "a novel free software library written in Python that tracks fold changes and statistical significance"
