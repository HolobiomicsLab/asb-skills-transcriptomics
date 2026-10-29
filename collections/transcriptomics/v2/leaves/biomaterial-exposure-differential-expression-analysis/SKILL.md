---
name: biomaterial-exposure-differential-expression-analysis
description: Use when transcriptomic CSVs contain pre-computed log2 fold-change and
  adjusted p-value columns for multiple treatment-vs-control biomaterial comparisons
  (e.g., AGXX® vs silver vs V2A steel), and you need quadrant classification plus
  time-course tracking of extreme genes.
when_to_use_negative:
- The CSV does not contain pre-computed log2 fold-change and adjusted p-value columns
  per comparison — raw count matrices must first be processed upstream (e.g., via
  TRANSIT resampling with beta-geometric normalization for Tn-seq data).
- You need a standard four-way plot without significance filtering — setting the multiplication
  factor to 1 produces a four-way plot, not the scorecard, so the scorecard quadrant
  logic should not be applied.
- The dataset was not preprocessed by removing low-expression, noise-prone genes and
  quality control, as the scorecard assumes cleaned differential-expression values.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scorecard_functions
  role: Python library providing generate_parameters, data_loading, scorecard_legend,
    scorecard, reconstruct_scorecard, make_volcano, count_frequencies, common_entries,
    and track_over_exper to classify and track extreme gene changes per comparison
  repo: github.com/m89p067/Scorecard
- name: Pandas
  role: Holds the loaded CSV of log2 fold changes and adjusted p-values as a DataFrame
    passed to scorecard
- name: Python
  role: Runtime environment in which the scorecard library executes the quadrant classification
    and plotting
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s00284-025-04435-3
    title: 'A Scorecard for Information Synthesis in Multiple Experimental Conditions:
      Application to Bacterial Biofilm Matrix Transcriptomics'
  source_content_hash: sha256:63a5ac3ae02db70eca27d3a917934d662f1a25e2bc92babde34fac192a17cc8a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pandas
  - Python
  - scorecard_functions
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# biomaterial-exposure-differential-expression-analysis

## Summary

Applies the Python scorecard library (scorecard_functions) to classify genes into significance/fold-change quadrants across pairwise biomaterial-exposure comparisons, and longitudinally tracks extreme expression changes over time points (e.g., E. faecalis on AGXX®, silver-coated, and uncoated V2A steel).

## When to use

Use when transcriptomic CSVs contain pre-computed log2 fold-change and adjusted p-value columns for multiple treatment-vs-control biomaterial comparisons (e.g., AGXX® vs silver vs V2A steel), and you need quadrant classification plus time-course tracking of extreme genes.

## When NOT to use

- The CSV does not contain pre-computed log2 fold-change and adjusted p-value columns per comparison — raw count matrices must first be processed upstream (e.g., via TRANSIT resampling with beta-geometric normalization for Tn-seq data).
- You need a standard four-way plot without significance filtering — setting the multiplication factor to 1 produces a four-way plot, not the scorecard, so the scorecard quadrant logic should not be applied.
- The dataset was not preprocessed by removing low-expression, noise-prone genes and quality control, as the scorecard assumes cleaned differential-expression values.

## Inputs

- CSV file of log2 fold changes and adjusted p-values with gene IDs as a separate column (e.g., Dataset 4 E. faecalis transcriptome on AGXX®, Ag-coated V2A steel, uncoated V2A steel)
- Parameter dictionary from generate_parameters(): base_dir, filename, FC cond x/y, padj cond x/y, th_fold_change, th_significance (p<=0.05), multiplication factor, gene_name, save_dir
- Folder structure with a main folder plus one subfolder per experimental comparison

## Outputs

- Per-quadrant JSON outcome files and individual quadrant plots storing statistically significant entries
- Reconstructed full scorecard figure per comparison
- Volcano plots (make_volcano) and entry-frequency summaries (count_frequencies, common_entries reports)
- Time-course tracking plots of scorecard entry modifications across subfolders (track_over_exper)

## How to apply

Configure the run with scorecard_functions.generate_parameters(), setting base_dir/filename to the Dataset 4 CSV, the exact 'FC cond x/y' and 'padj cond x/y' column names, significance threshold p<=0.05, and a log2 fold-change threshold with a multiplication factor (e.g., 1.5) defining extreme regions. Load the CSV into a Pandas DataFrame via scorecard_functions.data_loading(param_dict) and draw the region legend with scorecard_functions.scorecard_legend(param_dict). Run per-quadrant classification with scorecard_functions.scorecard(df,param_dict) for each pairwise folder (AGXX vs silver, Ag/V2A vs V2A, and longitudinal t=3, 6, 12, 24, 60, 90 minutes), which stores significant entries per quadrant with individual quadrant plots. Merge quadrants with scorecard_functions.reconstruct_scorecard(df,param_dict), then generate volcano plots (make_volcano) and entry-frequency summaries (count_frequencies, common_entries) per comparison folder. Track expression changes across subfolders with track_over_exper to confirm time-stable behavior of extreme genes (e.g., copper-related EF.peg.3040 and EF.peg.3041 activated under AGXX® vs silver).

## Related tools

- **scorecard_functions** (Python library providing generate_parameters, data_loading, scorecard_legend, scorecard, reconstruct_scorecard, make_volcano, count_frequencies, common_entries, and track_over_exper to classify and track extreme gene changes per comparison) — github.com/m89p067/Scorecard
- **Pandas** (Holds the loaded CSV of log2 fold changes and adjusted p-values as a DataFrame passed to scorecard)
- **Python** (Runtime environment in which the scorecard library executes the quadrant classification and plotting)

## Examples

```
import scorecard_functions; param_dict=scorecard_functions.generate_parameters(); df=scorecard_functions.data_loading(param_dict); scorecard_functions.scorecard(df,param_dict); scorecard_functions.reconstruct_scorecard(df,param_dict)
```

## Evaluation signals

- Extreme-bioactivation genes EF.peg.3040 (copper-translocating P-type ATPase) and EF.peg.3041 (negative transcriptional regulator-copper transport operon) appear in the expected scorecard regions/quadrant tables for the AGXX® vs silver comparison.
- Ag/V2A-specific genes EF.peg.1868, EF.peg.426, and EF.peg.481, and down-regulated EF.peg.2124 and EF.peg.2384, are captured in the expected quadrants.
- The reconstructed scorecard figure and JSON outcome files exist in the save_dir output directory for every comparison subfolder.
- Volcano plots and count_frequencies/common_entries summaries are generated for each comparison folder, and track_over_exper output confirms time-stable behavior over t=3, 6, 12, 24, 60, 90 minutes.
- Setting the multiplication factor to 1 should reproduce a four-way plot rather than a scorecard, confirming threshold configuration was applied correctly.

## Limitations

- Crowded scorecards can obscure genes; individual quadrant plots should be inspected in that case.
- No version number is stated for the Python library or Python itself.
- The multiplication factor must exceed 1; setting it to 1 produces a four-way plot rather than a scorecard.

## Evidence

- [results] the scorecard analyses were carried out longitudinally, to evaluate differential expression over time (t=3, 6, 12, 24, 60, 90 minutes) throughout the experimental cycle: "the scorecard analyses were carried out longitudinally, to evaluate differential expression over time (t=3, 6, 12, 24, 60, 90 minutes)"
- [discussion] In dataset 4, the work of [31] reported that exposure to the AgXX® coating significantly affected the gene expression of E. faecalis: "In dataset 4, the work of [31] reported that exposure to the AgXX® coating significantly affected the gene expression of E. faecalis"
- [readme] This function will store statistically significant entries for each quadrant together with plots of individual quandrants.: "This function will store statistically significant entries for each quadrant together with plots of individual quandrants."
- [readme] a scorecard to compare wet-lab experimental conditions, useful to identify extreme or moderately changing expression values based on fold change and statistical significance: "a scorecard to compare wet-lab experimental conditions, useful to identify extreme or moderately changing expression values based on fold change and statistical significance"
