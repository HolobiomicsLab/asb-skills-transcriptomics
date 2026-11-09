---
name: adjusted-pvalue-significance-filtering
description: Use when a pre-computed log2 fold-change and adjusted p-value table (CSV
  with gene-name column, per-comparison padj columns) must be filtered by significance
  and fold-change cut-offs to identify extreme gene entries across pairwise conditions.
when_to_use_negative:
- Input contains raw counts or unadjusted p-values without multiple-comparison correction
  — the workflow expects pre-computed adjusted p-values in the CSV
- Multiplication factor is set to 1, which produces a standard four-way plot without
  the extreme/moderate significance-filtered region structure
- The goal is gene preselection or experience-driven tracking of a predefined symbol
  set rather than data-driven feature extraction
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scorecard_functions
  role: Python library whose generate_parameters/data_loading/scorecard functions
    enforce the significance and fold-change cut-offs that filter genes into scorecard
    quadrants
  repo: github.com/m89p067/Scorecard
- name: Pandas
  role: Holds the CSV of log2 fold changes and adjusted p-values as the DataFrame
    that is filtered by the scorecard
- name: Python
  role: Execution environment for the scorecard library and the thresholding/filtering
    workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s00284-025-04435-3
    title: 'A Scorecard for Information Synthesis in Multiple Experimental Conditions:
      Application to Bacterial Biofilm Matrix Transcriptomics'
  source_content_hash: sha256:988b07fa969f30aedc18969b2e0d7cb873cfe23125001a63ea2ba3b3a79e6806
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pandas
  - Python
  - scorecard_functions
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# Adjusted-pvalue significance filtering

## Summary

Filters differentially expressed gene entries by strict adjusted p-value cut-offs (false-discovery-rate controlled) combined with log2 fold-change thresholds before/while constructing scorecard quadrants, so that only statistically significant, extreme or moderate expression variations (e.g. Dataset 3 S. mutans comparisons with fold-change thresholds of 2 and 4 and p<=0.001) are retained in regions of interest.

## When to use

Use when a pre-computed log2 fold-change and adjusted p-value table (CSV with gene-name column, per-comparison padj columns) must be filtered by significance and fold-change cut-offs to identify extreme gene entries across pairwise conditions.

## When NOT to use

- Input contains raw counts or unadjusted p-values without multiple-comparison correction — the workflow expects pre-computed adjusted p-values in the CSV
- Multiplication factor is set to 1, which produces a standard four-way plot without the extreme/moderate significance-filtered region structure
- The goal is gene preselection or experience-driven tracking of a predefined symbol set rather than data-driven feature extraction

## Inputs

- CSV file with log2 fold-change and adjusted p-value columns per pairwise comparison (first row as header, one gene name/ID per row)
- Parameter dictionary (param_dict) with th_fold_change, multiplication factor, th_significance, gene_name column, FC cond x/y and padj cond x/y column names
- Preprocessed differential expression table, e.g. Dataset 3 S. mutans RNA-seq under glucose, sucrose, lactose, xylitol

## Outputs

- Filtered per-quadrant records of statistically significant gene entries (stored by scorecard_functions.scorecard)
- Scorecard quadrant plots and reconstructed merged figure highlighting regions of interest (e.g. SMU_494, pdhC, treA downregulated for sucrose; ptcA/ptcB/ptcC, bglP, celR in PEP-PTS-related regions)
- Quadrant frequency/heatmap summaries when combined with count_frequencies or quadrants_heatmap

## How to apply

Configure the parameter dictionary with scorecard_functions.generate_parameters(), setting th_significance (e.g. 0.001 for the Dataset 3 S. mutans carbohydrate analysis) and th_fold_change together with a multiplication factor (Dataset 3 used fold-change thresholds of 2 and 4, i.e. multiplication factor 2) to define extreme-value regions of interest. The adjusted p-value column names ('padj cond x'/'padj cond y') must exactly match the CSV headers, since the CSV is expected to carry pre-computed log2 fold changes and Benjamini-Hochberg-style adjusted p-values. Load the CSV as a Pandas DataFrame via scorecard_functions.data_loading(param_dict), then run scorecard_functions.scorecard(df,param_dict), which stores only statistically significant entries per quadrant; setting multiplication factor to 1 degrades the scorecard into an unfiltered four-way plot. Rationale: strict significance cut-offs prevent minor fold differences that reach significance only with large sample sizes from crowding the result, and the two-tier fold-change threshold separates extreme from moderately changing genes.

## Related tools

- **scorecard_functions** (Python library whose generate_parameters/data_loading/scorecard functions enforce the significance and fold-change cut-offs that filter genes into scorecard quadrants) — github.com/m89p067/Scorecard
- **Pandas** (Holds the CSV of log2 fold changes and adjusted p-values as the DataFrame that is filtered by the scorecard)
- **Python** (Execution environment for the scorecard library and the thresholding/filtering workflow)

## Examples

```
param_dict=scorecard_functions.generate_parameters(); param_dict['th_significance']=0.001; param_dict['multiplication factor']=2; df=scorecard_functions.data_loading(param_dict)
```

## Evaluation signals

- The adjusted p-value columns declared in param_dict ('padj cond x', 'padj cond y') match the CSV header exactly and all retained entries have padj <= th_significance (e.g. <=0.001 for Dataset 3)
- Reconstructed scorecard figure exists on disk (rendered via reconstruct_scorecard with fig_size/use_figsize and figsize_factor)
- Known-signature check: SMU_494 is flagged as downregulated for sucrose, and PEP-PTS-related genes (ptcA, ptcB, ptcC, bglP, lacE, lacF, celR) appear in quadrant output records
- Setting 'multiplication factor' to 1 yields a four-way plot (not a scorecard), confirming the factor differentiates extreme from moderate regions
- Per-quadrant plots produced by scorecard_functions.scorecard(df,param_dict) contain only statistically significant entries

## Limitations

- Stricter thresholds (fold-change 2 and 4, p<=0.001) were needed on Dataset 3 to detect fewer outliers; default looser cut-offs can crowd the plot when sample sizes make minor differences statistically significant
- The workflow requires pre-computed fold-change and adjusted p-value columns; it does not itself compute differential expression
- No version number is stated for the Python library or Python itself, and no name/repository URL is repeated in all article sections, which can hinder reproducibility

## Evidence

- [intro] a bioinformatic workflow typically uses strict cut-offs for p-values (usually adjusted to control the false discovery rate and reduce the chance of false positives) and fold-changes: "a bioinformatic workflow typically uses strict cut-offs for p-values (usually adjusted to control the false discovery rate and reduce the chance of false positives)"
- [results] Scorecard applied to dataset 3 with modified thresholds: "the thresholds were modified to detect fewer outliers (Table SM5) using fold-change thresholds of 2 and 4 (aka, multiplication factor of 2) and p"
- [methods] param_dict generation call: "param_dict=scorecard_functions.generate_parameters()"
- [readme] data loading call: "df=scorecard_functions.data_loading(param_dict)"
- [readme] This function will store statistically significant entries for each quadrant together with plots of individual quandrants.: "This function will store statistically significant entries for each quadrant together with plots of individual quandrants."
