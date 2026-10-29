---
name: pandas-tabular-data-loading
description: Use when a CSV table of gene IDs with pairwise log2 fold-change and adjusted
  p-value columns (e.g. 'Treatment1vsCtrl_Log2FoldChange', 'Treatment1vsCtrl_padj')
  must be ingested before scorecard region-of-interest classification.
when_to_use_negative:
- Input is raw sequencing data (e.g. fastq or BAM) rather than a table of pre-computed
  fold changes and adjusted p-values — differential expression must be computed first
  (e.g. TRANSIT resampling for transposon insertion sequencing data).
- Data are not in delimited tabular format with a header row matching the parameter
  dictionary's column names (e.g. hierarchical formats like HDF5 or Excel without
  prior export to CSV).
- 'The goal is a four-way plot rather than the scorecard: the ''multiplication factor''
  must be set to 1 for a four-way plot, but data loading requirements remain the same.'
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scorecard_functions
  role: Provides data_loading(param_dict) which reads the configured CSV into a Pandas
    DataFrame as the first pipeline step before scorecard() classification
  repo: https://github.com/m89p067/Scorecard
- name: Pandas
  role: In-memory DataFrame representation of the gene-by-fold-change/p-value CSV
    consumed by all scorecard functions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s00284-025-04435-3
    title: 'A Scorecard for Information Synthesis in Multiple Experimental Conditions:
      Application to Bacterial Biofilm Matrix Transcriptomics'
  source_content_hash: sha256:64ca776b391c60c632a9f5fbb05ee87d4e94730a2fef99a3fb4d9d654d4c2c39
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/m89p067/Scorecard
  tools:
  - Pandas
  - Python
  - scorecard_functions
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# pandas-tabular-data-loading

## Summary

Load a CSV of pre-computed log2 fold-changes and adjusted p-values into a Pandas DataFrame as the entry point of the Scorecard pipeline, so that subsequent per-quadrant classification functions operate on a correctly keyed gene-by-comparison table.

## When to use

Use when a CSV table of gene IDs with pairwise log2 fold-change and adjusted p-value columns (e.g. 'Treatment1vsCtrl_Log2FoldChange', 'Treatment1vsCtrl_padj') must be ingested before scorecard region-of-interest classification.

## When NOT to use

- Input is raw sequencing data (e.g. fastq or BAM) rather than a table of pre-computed fold changes and adjusted p-values — differential expression must be computed first (e.g. TRANSIT resampling for transposon insertion sequencing data).
- Data are not in delimited tabular format with a header row matching the parameter dictionary's column names (e.g. hierarchical formats like HDF5 or Excel without prior export to CSV).
- The goal is a four-way plot rather than the scorecard: the 'multiplication factor' must be set to 1 for a four-way plot, but data loading requirements remain the same.

## Inputs

- CSV file with first-row header, one gene ID column, and pre-computed log2 fold-change and adjusted p-value columns per Treatment-vs-Control comparison
- param_dict from scorecard_functions.generate_parameters() configured with base_dir, filename, 'CSV delimiter', gene_name, and FC/padj column names

## Outputs

- Pandas DataFrame keyed by gene IDs with fold-change and adjusted p-value columns for each comparison

## How to apply

First create the parameter dictionary with scorecard_functions.generate_parameters(), then set base_dir, filename, 'CSV delimiter' (usually ','), gene_name (the gene ID column), and the exact column names 'FC cond x', 'FC cond y', 'padj cond x', 'padj cond y' matching the CSV header. The CSV must have its first row as a header and gene IDs in a separate column. Call scorecard_functions.data_loading(param_dict), which reads the CSV from disk and returns a Pandas DataFrame used by scorecard(), scorecard_legend(), and reconstruct_scorecard(). Correct column-name mapping is essential: misnamed 'FC cond x/y' or 'padj cond x/y' parameters will break downstream quadrant assignment based on the fold-change threshold (th_fold_change plus a multiplication-factor extreme threshold) and significance threshold (th_significance, e.g. 0.05).

## Related tools

- **scorecard_functions** (Provides data_loading(param_dict) which reads the configured CSV into a Pandas DataFrame as the first pipeline step before scorecard() classification) — https://github.com/m89p067/Scorecard
- **Pandas** (In-memory DataFrame representation of the gene-by-fold-change/p-value CSV consumed by all scorecard functions)

## Examples

```
param_dict=scorecard_functions.generate_parameters(); df=scorecard_functions.data_loading(param_dict)
```

## Evaluation signals

- data_loading returns a non-empty Pandas DataFrame whose column headers exactly match the 'FC cond x', 'FC cond y', 'padj cond x', 'padj cond y' strings in param_dict.
- The gene ID column specified via gene_name exists and contains one gene/protein ID per row with no header contamination.
- Numeric fold-change and padj columns parse as floats after applying the configured 'CSV delimiter'; p-values fall in [0,1].
- Downstream scorecard(df, param_dict) produces JSON outcome files of significant genes per quadrant in save_dir, confirming the DataFrame fed the classifier correctly.
- Marker counts per region are consistent with the count_frequencies summary computed from the same loaded DataFrame.

## Limitations

- The CSV must be pre-computed: the loader does not compute fold changes or p-values itself (dataset 1 values came from TRANSIT resampling with beta-geometric normalization and Benjamini-Hochberg correction).
- Header row must exactly match the parameter dictionary column names; notation like 'Treatment1vsCtrl_Log2FoldChange' must be mirrored in 'FC cond x' etc.
- No version number is stated for the Python library or for Python itself, hindering reproducibility of the loading step.

## Evidence

- [methods] Now load the CSV data into the computer memory as Pandas Dataframe by typing: "Now load the CSV data into the computer memory as Pandas Dataframe by typing"
- [readme] df=scorecard_functions.data_loading(param_dict): "df=scorecard_functions.data_loading(param_dict)"
- [readme] Remember that the CSV file should have the first row as header, reporting the column names matching the information provided in the dictionary of parameters.: "Remember that the CSV file should have the first row as header, reporting the column names matching the information provided in the dictionary of parameters."
- [readme] 'FC cond x' # replace with exact column name containing log2 fold change Treatment1 vs Control: "'FC cond x' # replace with exact column name containing log2 fold change Treatment1 vs Control"
