---
name: adjusted-pvalue-significance-quadrant-classification
description: Use when a CSV of pre-computed log2 fold changes and adjusted p-values
  (per gene) must be partitioned into significant extreme/moderate up- or down-regulated
  quadrants across pairwise treatment-vs-control comparisons.
when_to_use_negative:
- Input lacks pre-computed fold-change and adjusted p-value columns (the library expects
  these as CSV columns, not raw reads or count matrices)
- Setting 'multiplication factor' to 1 is desired — that produces a four-way plot
  rather than the scorecard quadrant classification
- The goal is a standard four-way plot not filtered by statistical significance, which
  the article identifies as a limitation the scorecard is designed to overcome
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scorecard_functions
  role: Python library providing generate_parameters, data_loading, scorecard_legend,
    scorecard (single-quadrant classification), reconstruct_scorecard, and count_frequencies
  repo: github.com/m89p067/Scorecard
- name: Pandas
  role: Loads the CSV into a DataFrame consumed by the scorecard quadrant computations
- name: Python
  role: Execution environment for the scorecard library pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s00284-025-04435-3
    title: 'A Scorecard for Information Synthesis in Multiple Experimental Conditions:
      Application to Bacterial Biofilm Matrix Transcriptomics'
  source_content_hash: sha256:44b2890bce272cdea6aae7c3d3831d61f6fb3fcaf5ede57535a29eb8bf86dfd5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pandas
  - Python
  - scorecard_functions
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# adjusted-pvalue-significance-quadrant-classification

## Summary

Classify transcriptomic entries into scorecard quadrants/regions-of-interest by jointly screening log2 fold-change thresholds (with a multiplication factor, e.g. 1.5 giving thresholds between 2 and 3) and adjusted p-value significance (padj <= 0.05), using the Python scorecard_functions library to separate extreme from moderate gene changes across pairwise experimental conditions.

## When to use

Use when a CSV of pre-computed log2 fold changes and adjusted p-values (per gene) must be partitioned into significant extreme/moderate up- or down-regulated quadrants across pairwise treatment-vs-control comparisons.

## When NOT to use

- Input lacks pre-computed fold-change and adjusted p-value columns (the library expects these as CSV columns, not raw reads or count matrices)
- Setting 'multiplication factor' to 1 is desired — that produces a four-way plot rather than the scorecard quadrant classification
- The goal is a standard four-way plot not filtered by statistical significance, which the article identifies as a limitation the scorecard is designed to overcome

## Inputs

- CSV file with gene names/IDs and pre-computed log2 fold-change columns ('FC cond x/y') and adjusted p-value columns ('padj cond x/y') per pairwise comparison, with a header row matching the parameter dictionary
- param_dict from scorecard_functions.generate_parameters() with th_significance (0.05), th_fold_change, 'multiplication factor' (e.g. 1.5), gene_name, save_dir, and CSV delimiter settings

## Outputs

- Per-quadrant plots of statistically significant entries (individual quadrants plus reconstructed full scorecards)
- JSON outcomes per quadrant stored in save_dir
- quadrant_gene_counts.csv from count_frequencies, tabulating number of entries/genes per quadrant per comparison

## How to apply

Build the parameter dictionary with scorecard_functions.generate_parameters(), setting base_dir/filename of the input CSV, the gene_name column, the exact 'FC cond x/y' and 'padj cond x/y' column names, th_significance to 0.05, th_fold_change to a log2 fold-change threshold, and 'multiplication factor' (e.g. 1.5, producing fold-change thresholds between 2 and 3; setting it to 1 degrades the scorecard into a four-way plot). Load the CSV as a Pandas DataFrame via scorecard_functions.data_loading(param_dict), generate the color-code/region-of-interest legend with scorecard_functions.scorecard_legend(param_dict), then run scorecard_functions.scorecard(df,param_dict) which stores statistically significant entries per quadrant together with individual quadrant plots and JSON outcomes in save_dir. Rebuild full scorecards with scorecard_functions.reconstruct_scorecard(df,param_dict) and tabulate genes per quadrant per comparison with scorecard_functions.count_frequencies(main_folder), saving quadrant_gene_counts.csv.

## Related tools

- **scorecard_functions** (Python library providing generate_parameters, data_loading, scorecard_legend, scorecard (single-quadrant classification), reconstruct_scorecard, and count_frequencies) — github.com/m89p067/Scorecard
- **Pandas** (Loads the CSV into a DataFrame consumed by the scorecard quadrant computations)
- **Python** (Execution environment for the scorecard library pipeline)

## Examples

```
import scorecard_functions; param_dict=scorecard_functions.generate_parameters(); df=scorecard_functions.data_loading(param_dict); scorecard_functions.scorecard(df,param_dict); scorecard_functions.reconstruct_scorecard(df,param_dict)
```

## Evaluation signals

- Quadrant assignments contain only entries with padj <= th_significance (0.05) and log2 fold-changes beyond the threshold and its multiplication-factor multiple
- count_frequencies output (quadrant_gene_counts.csv) reproduces reported region-of-interest counts, e.g. Q1 region A for CPT-vs-LZD, DEL-vs-CPT, DEL-vs-LZD and Q3 region D for VAN-vs-CPT/DEL/LZD comparisons
- Full reconstructed scorecards match the union of the four individually computed quadrants (JSON outcomes consistent with reconstruction)
- CSV column names match the 'FC cond x/y' and 'padj cond x/y' entries in param_dict (header-row check after data_loading)

## Limitations

- Multiplicity of isolated comparisons is limited: typical four-way plots are not filtered by statistical significance and cannot merge information across multiple plots, which motivated the scorecard
- Crowded scorecards may require consulting the individual quadrant plots rather than the reconstructed full scorecard
- The library requires the user to supply pre-computed fold changes and adjusted p-values; it does not compute differential expression itself (dataset 1 values came from TRANSIT resampling with beta-geometric normalization and Benjamini-Hochberg correction)

## Evidence

- [results] scorecard applied to dataset 1 with fold-change thresholds 2-3 (multiplication factor 1.5) and p<=0.05 across six comparisons: "six comparisons were made applying the scorecard and setting fold-change thresholds between 2 and 3 (that is, a multiplication factor of 1.5) and a significance level of"
- [readme] README on the scorecard function storing quadrant results: "This function will store statistically significant entries for each quadrant together with plots of individual quandrants."
- [readme] README on the multiplication factor parameter: "`multiplication factor` # Factor to multiply the log2 Fold change threshold to detect extreme values (setting it to 1 will produce a four-way plot, not the scorecard)"
- [results] Article findings on dataset 1 quadrant patterns: "the comparisons involving VAN mainly identified markedly down-regulated genes in Q3; in contrast, the other antibiotic-related genes were markedly over-expressed"
- [readme] README on the count_frequencies summary function: "To get a summary of the number of entries/genes indentified by the scorecard type:"
