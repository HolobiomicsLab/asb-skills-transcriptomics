---
name: multi-condition-transcriptome-scorecard-visualization
description: Use when a CSV of gene IDs with log2 fold-change and adjusted p-value
  columns for multiple treatment-vs-control comparisons must be screened for extreme
  or moderate expression variations across several experimental conditions (e.g.,
  antibiotic, adjuvant, and co-treatment RNA-seq).
when_to_use_negative:
- Input lacks pre-computed fold-change and adjusted p-value columns — the CSV must
  already contain these; the library does not compute differential expression from
  raw counts
- Only a single pairwise comparison is needed and a standard volcano plot suffices
  — note that volcano plots get crowded with large samples and show one condition
  per graph
- Setting 'multiplication factor' to 1 degrades the scorecard into a plain four-way
  plot, which the library is designed to improve upon
edam_operation: http://edamontology.org/operation_0571
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scorecard_functions
  role: Core Python library providing generate_parameters, data_loading, scorecard_legend,
    scorecard, reconstruct_scorecard, multiple_view, count_frequencies, common_entries
    and related functions
  repo: github.com/m89p067/Scorecard
- name: Pandas
  role: Loads the CSV of log2 fold-changes and adjusted p-values into a DataFrame
    consumed by the scorecard functions
- name: Python
  role: Language in which the scorecard library is written and in which the analysis
    pipeline is scripted
- name: TRANSIT
  role: Used upstream (dataset 1) to compute log2 fold-changes via resampling with
    beta-geometric normalization and Benjamini-Hochberg correction before scorecard
    analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s00284-025-04435-3
    title: 'A Scorecard for Information Synthesis in Multiple Experimental Conditions:
      Application to Bacterial Biofilm Matrix Transcriptomics'
  source_content_hash: sha256:6dad31be147a477b50c8efee1282d3b2921cf3b72625589e54c26b03e3596c30
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pandas
  - Python
  - TRANSIT
  - scorecard library
  - scorecard_functions
  edam_operation: http://edamontology.org/operation_0571
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# multi-condition-transcriptome-scorecard-visualization

## Summary

A Python scorecard library that groups genes from pre-computed log2 fold-change and adjusted p-value tables into significance-filtered quadrants across multiple pairwise wet-lab comparisons, overcoming the limits of four-way plots and volcano plots in multi-condition differential-expression analyses.

## When to use

Use when a CSV of gene IDs with log2 fold-change and adjusted p-value columns for multiple treatment-vs-control comparisons must be screened for extreme or moderate expression variations across several experimental conditions (e.g., antibiotic, adjuvant, and co-treatment RNA-seq).

## When NOT to use

- Input lacks pre-computed fold-change and adjusted p-value columns — the CSV must already contain these; the library does not compute differential expression from raw counts
- Only a single pairwise comparison is needed and a standard volcano plot suffices — note that volcano plots get crowded with large samples and show one condition per graph
- Setting 'multiplication factor' to 1 degrades the scorecard into a plain four-way plot, which the library is designed to improve upon

## Inputs

- CSV file of pre-computed log2 fold-change values and adjusted p-values per gene, with a gene ID/name column and header row matching the parameter column names (e.g., 'Treatment1vsCtrl_Log2FoldChange', 'Treatment1vsCtrl_padj')
- Parameter dictionary from scorecard_functions.generate_parameters() (base_dir, filename, gene_name, FC/padj column names, th_fold_change, multiplication factor, th_significance, save_dir)
- Main folder containing sub-folders, one per experimental comparison

## Outputs

- Scorecard plots per comparison with statistically significant genes assigned to quadrants/regions of interest
- Individual quadrant plots for crowded scorecards
- Radial multiple_view overview of all comparisons
- Volcano plots highlighting scorecard regions of interest (make_volcano)
- Frequency summaries and heatmaps of quadrant counts (count_frequencies, quadrants_heatmap)
- CSV/Excel reports of common and uncommon gene entries (common_entries)
- Bar plots and rankings of single-gene expression levels (multiple_bars, ranking_bars, largest_diff); longitudinal tracking via track_over_exper

## How to apply

Import scorecard_functions and build the parameter dictionary with scorecard_functions.generate_parameters(), setting base_dir/filename to the preprocessed CSV (which must contain pre-computed log2 fold-change and adjusted p-values with matching column names), gene_name to the gene-ID column, th_significance (e.g., 0.05), th_fold_change plus multiplication factor, and save_dir. Load the CSV as a Pandas DataFrame via data_loading(param_dict), generate the scorecard_legend, then run scorecard(df,param_dict) to compute per-quadrant statistically significant entries and individual quadrant plots. Reconstruct each full comparison with reconstruct_scorecard(df,param_dict) for all requested comparisons (e.g., Ox, Cmpd8, Lor, and co-treatments vs Control), then aggregate sub-folders under a main_folder with multiple_view. Summarize with count_frequencies and common_entries, and optionally use make_volcano, multiple_bars, ranking_bars, quadrants_heatmap, track_over_exper, and largest_diff for downstream extraction.

## Related tools

- **scorecard_functions** (Core Python library providing generate_parameters, data_loading, scorecard_legend, scorecard, reconstruct_scorecard, multiple_view, count_frequencies, common_entries and related functions) — github.com/m89p067/Scorecard
- **Pandas** (Loads the CSV of log2 fold-changes and adjusted p-values into a DataFrame consumed by the scorecard functions)
- **Python** (Language in which the scorecard library is written and in which the analysis pipeline is scripted)
- **TRANSIT** (Used upstream (dataset 1) to compute log2 fold-changes via resampling with beta-geometric normalization and Benjamini-Hochberg correction before scorecard analysis)

## Examples

```
import scorecard_functions; param_dict=scorecard_functions.generate_parameters(); df=scorecard_functions.data_loading(param_dict); scorecard_functions.scorecard_legend(param_dict); scorecard_functions.scorecard(df,param_dict); scorecard_functions.reconstruct_scorecard(df,param_dict)
```

## Evaluation signals

- Reconstructed scorecard for each comparison matches the union of its four per-quadrant significant-entry sets from scorecard(df,param_dict)
- count_frequencies on the main_folder reproduces expected region counts (e.g., the 1126 outer-region genes reported for Dataset 2)
- common_entries reports contain the expected signature genes in expected regions, e.g., SA_RS03110/CwrA/DUF4930 (oxacillin-specific), SA_RS05690/SA_RS05695/SA_RS01280/AdhP (co-treatment-specific), and SA_RS09375/LiaF in the Lor_Ox-vs-Cmpd8_Ox comparison
- All quadrant markers are restricted to genes with padj below th_significance (e.g., 0.05) and fold-change beyond th_fold_change and its multiplied threshold
- Folder structure matches main_folder with one sub-folder per experimental comparison before calling multiple_view

## Limitations

- The CSV must contain pre-computed fold-change values and related adjusted p-values as columns; raw count matrices are not supported
- No version number is stated for the Python library or for Python itself
- Scorecards can be crowded; individual quadrant plots may be needed when many genes are significant
- Multiple_view works only for the scorecard, not for four-way plots

## Evidence

- [methods] generate_parameters: "param_dict=scorecard_functions.generate_parameters()"
- [methods] data_loading: "df=scorecard_functions.data_loading(param_dict)"
- [readme] This function will store statistically significant entries for each quadrant together with plots of individual quandrants.: "This function will store statistically significant entries for each quadrant together with plots of individual quandrants."
- [readme] Indeed, your dataset should contain pre-computed fold change values (and related adjusted p-values) as columns in the CSV file.: "Indeed, your dataset should contain pre-computed fold change values (and related adjusted p-values) as columns in the CSV file."
- [intro] To overcome the limitations of four-way plots in DE analyses, a scorecard has been released as a free-to-use Python software library.: "To overcome the limitations of four-way plots in DE analyses, a scorecard has been released as a free-to-use Python software library."
- [results] there is a specific function included in the software library that produces a general visualization of all comparisons: "there is a specific function included in the software library that produces a general visualization of all comparisons"
- [results] the expression level visualization of each gene identified by the scorecard was plotted as bar plot through a specific function included in the scorecard library: "the expression level visualization of each gene identified by the scorecard was plotted as bar plot through a specific function included in the scorecard library"
