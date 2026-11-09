---
name: gene-significance-filtering
description: Use when a CSV of pre-computed log2 fold-changes and adjusted p-values
  for two treatments vs a common control must be partitioned into significance-filtered
  regions of interest (quadrants A–E, or M/R/S in the full scorecard) by fold-change
  magnitude.
when_to_use_negative:
- Setting the multiplication factor to 1 — this produces a four-way plot, not the
  scorecard, losing the extreme-vs-moderate partitioning
- 'Input lacks adjusted (FDR-corrected) p-values: typical four-way plots are not filtered
  by statistical significance, but this skill requires significance filtering to assign
  entries to regions'
- 'Fold-changes are not pre-computed: the workflow expects log2 fold-change and padj
  columns in the CSV, not raw count or TPM data'
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: scorecard_functions
  role: Python library whose generate_parameters, data_loading, scorecard_legend,
    scorecard and reconstruct_scorecard functions execute the threshold- and significance-based
    gene assignment into scorecard regions
  repo: github.com/m89p067/Scorecard
- name: Pandas
  role: Holds the loaded CSV of log2 fold-changes and adjusted p-values as the DataFrame
    passed to scorecard_functions.scorecard()
- name: Python
  role: Language in which the scorecard library is written and invoked (import scorecard_functions)
  repo: github.com/m89p067/Scorecard
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s00284-025-04435-3
    title: 'A Scorecard for Information Synthesis in Multiple Experimental Conditions:
      Application to Bacterial Biofilm Matrix Transcriptomics'
  source_content_hash: sha256:7c535fc7b23246dabb01b1ba3cf8d8ae7058ea6420900f56cf0225195059c49b
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# gene-significance-filtering

## Summary

The Scorecard library assigns statistically significant genes to regions of interest on the Cartesian plane of two treatments' log2 fold-changes versus a common control, using two fold-change thresholds (a standard threshold plus an extreme threshold set via a multiplication factor) and an adjusted p-value filter (e.g. 0.05 or lower).

## When to use

Use when a CSV of pre-computed log2 fold-changes and adjusted p-values for two treatments vs a common control must be partitioned into significance-filtered regions of interest (quadrants A–E, or M/R/S in the full scorecard) by fold-change magnitude.

## When NOT to use

- Setting the multiplication factor to 1 — this produces a four-way plot, not the scorecard, losing the extreme-vs-moderate partitioning
- Input lacks adjusted (FDR-corrected) p-values: typical four-way plots are not filtered by statistical significance, but this skill requires significance filtering to assign entries to regions
- Fold-changes are not pre-computed: the workflow expects log2 fold-change and padj columns in the CSV, not raw count or TPM data

## Inputs

- CSV of pre-computed log2 fold-changes and adjusted p-values (Benjamini-Hochberg corrected), first row as header
- Gene ID column in the CSV (one gene/protein name or ID per row)
- Column names matching 'FC cond x', 'FC cond y', 'padj cond x', 'padj cond y' in the parameter dictionary (e.g. 'Treatment1vsCtrl_Log2FoldChange')
- Parameter dictionary from scorecard_functions.generate_parameters() with th_fold_change, multiplication factor, th_significance, save_dir

## Outputs

- Reconstructed scorecard image saved in save_dir
- JSON outcome files recording significant genes per quadrant/region (A–E standard, M/R/S full scorecard)
- Individual quadrant plots for crowded comparisons
- Color-code legend of the regions of interest
- count_frequencies summary of marker counts per region

## How to apply

Create the parameter dictionary with scorecard_functions.generate_parameters() and set th_fold_change (the standard log2 fold-change cut-off), a multiplication factor that automatically derives the second (extreme) threshold — e.g. fold-change thresholds of 2 and 3 with a multiplication factor of 1.5, or 2 and 4 with a factor of 2 and p<=0.001 to detect fewer outliers — and th_significance set to 0.05 or lower. Load the CSV with scorecard_functions.data_loading(param_dict); the file needs gene IDs in a separate column and a header row matching the 'FC cond x/y' and 'padj cond x/y' parameter names. Generate the color-code legend of regions of interest with scorecard_functions.scorecard_legend(param_dict), then run scorecard_functions.scorecard(df,param_dict), which stores statistically significant entries per quadrant together with individual quadrant plots. Rebuild the full figure with scorecard_functions.reconstruct_scorecard(df,param_dict), tuning fig_size/use_figsize for image dimensions. Region A holds the extreme values of each quadrant, while genes expressed markedly in one condition and mildly in the other reside in regions C and B.

## Related tools

- **scorecard_functions** (Python library whose generate_parameters, data_loading, scorecard_legend, scorecard and reconstruct_scorecard functions execute the threshold- and significance-based gene assignment into scorecard regions) — github.com/m89p067/Scorecard
- **Pandas** (Holds the loaded CSV of log2 fold-changes and adjusted p-values as the DataFrame passed to scorecard_functions.scorecard())
- **Python** (Language in which the scorecard library is written and invoked (import scorecard_functions)) — github.com/m89p067/Scorecard

## Examples

```
import scorecard_functions; param_dict=scorecard_functions.generate_parameters(); df=scorecard_functions.data_loading(param_dict); scorecard_functions.scorecard(df,param_dict); scorecard_functions.reconstruct_scorecard(df,param_dict)
```

## Evaluation signals

- The reconstructed scorecard image and the JSON outcome files recording significant genes per quadrant exist in save_dir
- Marker counts per region are consistent with the count_frequencies summary
- Every gene assigned to a region passes the significance filter (padj <= th_significance, e.g. 0.05) in at least the relevant comparison
- Genes in region A exceed the extreme (multiplication-factor-scaled) fold-change threshold; region C/B genes exceed the standard threshold in one condition only and remain intermediate in the other
- Number of subfolders with per-comparison scorecards matches the number of requested pairwise comparisons (e.g. six comparisons for dataset 1)

## Limitations

- Setting the multiplication factor to 1 produces a four-way plot rather than the scorecard
- The CSV must have a header row matching the parameter dictionary column names, otherwise data_loading will fail or mis-map comparisons
- Individual quadrant plots exist because scorecards can get crowded; a single reconstructed figure may be unreadable for dense transcriptomes
- No version number is stated for the Python library or for Python itself
- p-value and fold-change thresholds must be chosen per dataset (e.g. p<=0.05 vs p<=0.001) and affect how many outliers are detected

## Evidence

- [results] The scorecard assigns genes by magnitude within each quadrant: region A holds the extreme values of each quadrant, while genes expressed markedly in one condition and mildly in the other reside in regions C and B.: [evidence span withheld — anchor status: not_found]
- [methods] generate_parameters call: "param_dict=scorecard_functions.generate_parameters()"
- [intro] the scorecard will identify extreme (or moderate) variations scanning transcriptome symbols, shrinking the massive records usually encountered in RNA-seq data by screening both in terms of: "the scorecard will identify extreme (or moderate) variations scanning transcriptome symbols, shrinking the massive records usually encountered in RNA-seq data by screening both in terms of"
- [intro] a bioinformatic workflow typically uses strict cut-offs for p-values (usually adjusted to control the false discovery rate and reduce the chance of false positives) and fold-changes: "a bioinformatic workflow typically uses strict cut-offs for p-values (usually adjusted to control the false discovery rate and reduce the chance of false positives) and fold-changes"
- [readme] README: scorecard function stores significant entries per quadrant: "This function will store statistically significant entries for each quadrant together with plots of individual quandrants."
- [readme] README: th_fold_change threshold and multiplication factor: "th_fold_change # log2 fold change standard threshold (another threshold will be automatically added based on 'multiplication factor')"
