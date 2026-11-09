---
name: differential-expression-quadrant-classification
description: Use when a CSV of pre-computed log2 fold-changes and adjusted p-values
  for two Treatment-vs-Control comparisons must be screened to identify extreme or
  moderate expression variations by fold-change thresholds and significance.
when_to_use_negative:
- Only a single treatment-vs-control comparison is available — the scorecard requires
  two fold-change columns versus a common control
- Fold-change and adjusted p-value columns are not pre-computed; the library expects
  them as CSV columns and does not perform differential expression testing itself
- A plain four-way plot suffices — setting 'multiplication factor' to 1 yields a four-way
  plot, not a scorecard, so the quadrant-region logic is unnecessary in that case
edam_operation: http://edamontology.org/operation_0571
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scorecard_functions
  role: Python library providing generate_parameters, data_loading, scorecard_legend,
    scorecard, and reconstruct_scorecard for quadrant classification and scorecard
    reconstruction
  repo: https://github.com/m89p067/Scorecard
- name: Pandas
  role: Holds the loaded CSV of log2 fold-changes and adjusted p-values as a DataFrame
    passed to the scorecard functions
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1007/s00284-025-04435-3
    title: 'A Scorecard for Information Synthesis in Multiple Experimental Conditions:
      Application to Bacterial Biofilm Matrix Transcriptomics'
  source_content_hash: sha256:1dca5f9726ddbfa1bb01e899b939a7b29915b4c40e12531e0aa5084044d88305
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/m89p067/Scorecard
  tools:
  - Pandas
  - Python
  - scorecard_functions
  edam_operation: http://edamontology.org/operation_0571
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# differential-expression-quadrant-classification

## Summary

A Python scorecard method that partitions the Cartesian plane of two treatments' log2 fold-changes versus a common control into regions of interest (A-E standard; M/R/S full scorecard), classifying statistically significant genes by quadrant and fold-change magnitude to overcome four-way-plot limitations.

## When to use

Use when a CSV of pre-computed log2 fold-changes and adjusted p-values for two Treatment-vs-Control comparisons must be screened to identify extreme or moderate expression variations by fold-change thresholds and significance.

## When NOT to use

- Only a single treatment-vs-control comparison is available — the scorecard requires two fold-change columns versus a common control
- Fold-change and adjusted p-value columns are not pre-computed; the library expects them as CSV columns and does not perform differential expression testing itself
- A plain four-way plot suffices — setting 'multiplication factor' to 1 yields a four-way plot, not a scorecard, so the quadrant-region logic is unnecessary in that case

## Inputs

- CSV file of pre-computed log2 fold-changes and adjusted p-values with a header row and gene IDs/protein names in a separate column (e.g. 'Treatment1vsCtrl_Log2FoldChange', 'Treatment1vsCtrl_padj')
- Parameter dictionary with base_dir, filename, gene_name, th_fold_change, multiplication factor, th_significance, save_dir, FC cond x/y and padj cond x/y column names

## Outputs

- Reconstructed scorecard image (PNG, dimensions set by fig_size or use_figsize/figsize_factor)
- JSON outcome files recording statistically significant genes per quadrant, stored in save_dir
- Individual quadrant plots for crowded scorecards
- Count-frequencies summary of marker counts per region

## How to apply

Build a parameter dictionary with scorecard_functions.generate_parameters(), setting th_fold_change (e.g. 2-3), a multiplication factor that automatically adds a second extreme threshold (e.g. 1.5), and th_significance of 0.05 or lower. Load the CSV (gene IDs in a separate column; header columns named 'FC cond x', 'FC cond y', 'padj cond x', 'padj cond y') into a Pandas DataFrame with scorecard_functions.data_loading(param_dict). Generate the color-code legend with scorecard_functions.scorecard_legend(param_dict), then run the per-quadrant classification with scorecard_functions.scorecard(df,param_dict), which stores statistically significant entries for each quadrant (Q1-Q4) together with individual quadrant plots. Rebuild the full scorecard with scorecard_functions.reconstruct_scorecard(df,param_dict) using fig_size/use_figsize for image dimensions. Region assignment is by magnitude within each quadrant: region A holds extreme values, while genes markedly expressed in one condition and mildly in the other fall in regions C and B; setting the multiplication factor to 1 degenerates to a four-way plot rather than a scorecard.

## Related tools

- **scorecard_functions** (Python library providing generate_parameters, data_loading, scorecard_legend, scorecard, and reconstruct_scorecard for quadrant classification and scorecard reconstruction) — https://github.com/m89p067/Scorecard
- **Pandas** (Holds the loaded CSV of log2 fold-changes and adjusted p-values as a DataFrame passed to the scorecard functions)

## Examples

```
import scorecard_functions; param_dict=scorecard_functions.generate_parameters(); df=scorecard_functions.data_loading(param_dict); scorecard_functions.scorecard(df,param_dict); scorecard_functions.reconstruct_scorecard(df,param_dict)
```

## Evaluation signals

- Reconstructed scorecard image and JSON outcome files recording significant genes per quadrant both exist in save_dir
- Marker counts per region are consistent with the count_frequencies summary
- Statistically significant entries (padj <= th_significance, e.g. 0.05) are stored for all four quadrants before reconstruction
- Region A contains the extreme fold-change values of each quadrant, consistent with the second threshold defined by the multiplication factor

## Limitations

- Crowded scorecards can be hard to read; individual quadrant plots are provided as a fallback
- Typical four-way plots are not filtered by statistical significance; setting the multiplication factor to 1 reproduces a four-way plot rather than the scorecard
- No version number is stated for the Python library or for Python itself

## Evidence

- [abstract] a scorecard that groups genes based on relative fold-change and statistical significance, providing additional functions that facilitate knowledge extraction: "a scorecard that groups genes based on relative fold-change and statistical significance, providing additional functions that facilitate knowledge extraction"
- [methods] generate_parameters in methods: "param_dict=scorecard_functions.generate_parameters()"
- [methods] scorecard per-quadrant function: "scorecard_functions.scorecard(df,param_dict)"
- [readme] reconstruct_scorecard from individual quadrants: "After creating all four quadrants with previous function, one should execute scorecard reconstruction from individual quandrants"
- [readme] multiplication factor threshold: "Factor to multiply the log2 Fold change threshold to detect extreme values (setting it to 1 will produce a four-way plot, not the scorecard)"
- [results] scorecard applied to dataset 1 with thresholds: "six comparisons were made applying the scorecard and setting fold-change thresholds between 2 and 3 (that is, a multiplication factor of 1.5) and a significance level of"
