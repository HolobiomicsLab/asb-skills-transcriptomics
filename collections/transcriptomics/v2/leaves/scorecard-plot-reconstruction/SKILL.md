---
name: scorecard-plot-reconstruction
description: Use when scorecard_functions.scorecard(df,param_dict) has already generated
  the four single-quadrant assignments and plots and you need the assembled full scorecard
  image per pairwise comparison, e.g. VAN/CPT/DEL/LZD vs control.
when_to_use_negative:
- The 'multiplication factor' is set to 1, which produces a standard four-way plot
  rather than the scorecard, so quadrant reconstruction of scorecard regions does
  not apply
- No individual quadrant outputs have been generated yet — reconstruct_scorecard must
  be called only after scorecard(df,param_dict) has processed all four quadrants
- The dataset lacks pre-computed log2 fold-change and adjusted p-value columns in
  the CSV; the scorecard does not compute differential expression itself (e.g. dataset
  1 fold-changes were pre-computed via TRANSIT resampling with beta-geometric normalization)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: scorecard_functions
  role: Python library whose scorecard() and reconstruct_scorecard() functions perform
    per-quadrant classification and full scorecard figure assembly
  repo: github.com/m89p067/Scorecard
- name: Pandas
  role: Holds the CSV of log2 fold-changes and adjusted p-values as a DataFrame passed
    to scorecard() and reconstruct_scorecard()
- name: Python
  role: Language in which the scorecard library is written and the reconstruction
    workflow is executed
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1007/s00284-025-04435-3
    title: 'A Scorecard for Information Synthesis in Multiple Experimental Conditions:
      Application to Bacterial Biofilm Matrix Transcriptomics'
  source_content_hash: sha256:8f9e5c053fb137082dc77b49105850566c547e46e7e2a04d1260bba37a5158f5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pandas
  - Python
  - scorecard_functions
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# scorecard-plot-reconstruction

## Summary

Rebuilds the complete scorecard figure from the four individually computed quadrant plots using scorecard_functions.reconstruct_scorecard, producing the full multi-region visualization plus JSON outcome files in save_dir after scorecard() has processed each quadrant.

## When to use

Use when scorecard_functions.scorecard(df,param_dict) has already generated the four single-quadrant assignments and plots and you need the assembled full scorecard image per pairwise comparison, e.g. VAN/CPT/DEL/LZD vs control.

## When NOT to use

- The 'multiplication factor' is set to 1, which produces a standard four-way plot rather than the scorecard, so quadrant reconstruction of scorecard regions does not apply
- No individual quadrant outputs have been generated yet — reconstruct_scorecard must be called only after scorecard(df,param_dict) has processed all four quadrants
- The dataset lacks pre-computed log2 fold-change and adjusted p-value columns in the CSV; the scorecard does not compute differential expression itself (e.g. dataset 1 fold-changes were pre-computed via TRANSIT resampling with beta-geometric normalization)

## Inputs

- Pandas DataFrame of pre-computed log2 fold-changes and adjusted p-values loaded by scorecard_functions.data_loading(param_dict) from a CSV with a header row and gene IDs in a separate column
- Parameter dictionary from scorecard_functions.generate_parameters() with th_fold_change, 'multiplication factor', th_significance (e.g. 0.05), fig_size, and exact 'FC cond x/y' and 'padj cond x/y' column names
- Output of scorecard_functions.scorecard(): individual quadrant plots and JSON outcomes per quadrant in save_dir

## Outputs

- Reconstructed full scorecard image saved in save_dir for each pairwise comparison
- JSON outcome files recording the statistically significant genes assigned to each quadrant/region (A-E standard, M/R/S full scorecard)
- Consistency-checkable per-quadrant counts via scorecard_functions.count_frequencies(main_folder) writing quadrant_gene_counts.csv

## How to apply

First run the per-quadrant classification with scorecard_functions.scorecard(df,param_dict), which stores statistically significant entries (th_significance, e.g. padj <= 0.05) assigned to each quadrant/region (A-E standard, M/R/S full scorecard) together with individual quadrant plots, since reconstruction requires all four quadrants to exist. Then call scorecard_functions.reconstruct_scorecard(df,param_dict) to merge the individual quadrants into one figure sized by the fig_size parameter; the parameters 'use_figsize' and 'figsize_factor' allow customizing image dimensions (set 'use_figsize' to False to fall back on fig_size, otherwise True for custom sizing). Fold-change thresholds come from th_fold_change plus a second extreme threshold obtained by multiplying it by the 'multiplication factor' (e.g. 1.5 giving fold-change thresholds between 2 and 3, or 2 giving thresholds 2 and 4). Repeat reconstruction for every requested comparison, keeping one subfolder per pairwise comparison under main_folder. Validate that the reconstructed scorecard image and the JSON outcome files recording significant genes per quadrant exist in save_dir and that per-region marker counts are consistent with the scorecard_functions.count_frequencies(main_folder) summary (quadrant_gene_counts.csv).

## Related tools

- **scorecard_functions** (Python library whose scorecard() and reconstruct_scorecard() functions perform per-quadrant classification and full scorecard figure assembly) — github.com/m89p067/Scorecard
- **Pandas** (Holds the CSV of log2 fold-changes and adjusted p-values as a DataFrame passed to scorecard() and reconstruct_scorecard())
- **Python** (Language in which the scorecard library is written and the reconstruction workflow is executed)

## Examples

```
import scorecard_functions; scorecard_functions.scorecard(df,param_dict); scorecard_functions.reconstruct_scorecard(df,param_dict)
```

## Evaluation signals

- The reconstructed scorecard image file exists in save_dir for each comparison
- JSON outcome files recording statistically significant genes per quadrant exist in save_dir
- Marker counts per region in the reconstructed figure are consistent with the count_frequencies summary (quadrant_gene_counts.csv)
- For dataset 1, counts reproduce the reported region-of-interest values, e.g. Q1 region A for CPT-vs-LZD, DEL-vs-CPT, DEL-vs-LZD and Q3 region D for VAN-vs-CPT/DEL/LZD
- Reconstruction is only executed after all four individual quadrant plots have been generated by scorecard(df,param_dict)

## Limitations

- Individual quadrant plots may be needed in case of crowded scorecards; the full reconstructed figure can become hard to read
- Setting 'multiplication factor' to 1 yields a four-way plot, not the scorecard, so region reconstruction is meaningless in that configuration
- No version number is stated for the Python library or Python itself; the README does not provide the library name or repository URL in the discussion-derived context, hindering reproducibility

## Evidence

- [readme] This function will store statistically significant entries for each quadrant together with plots of individual quandrants. Individual plots could be useful in case of crowded scorecards.: "This function will store statistically significant entries for each quadrant together with plots of individual quandrants."
- [readme] After creating all four quadrants with previous function, one should execute scorecard reconstruction from individual quandrants using this function call: scorecard_functions.reconstruct_scorecard(df,param_dict): "After creating all four quadrants with previous function, one should execute scorecard reconstruction from individual quandrants"
- [readme] The parameters "use_figsize" and "figsize_factor" allow for personalization of the image dimension. Default dimension is set by "fig_size" in the initial parameters, otherwise set "use_figsize" as False per customization.: "The parameters "use_figsize" and "figsize_factor" allow for personalization of the image dimension."
- [results] six comparisons were made applying the scorecard and setting fold-change thresholds between 2 and 3 (that is, a multiplication factor of 1.5) and a significance level of: "setting fold-change thresholds between 2 and 3 (that is, a multiplication factor of 1.5)"
- [other] Run the per-quadrant classification with scorecard_functions.scorecard(df,param_dict), which stores the statistically significant entries assigned to each of the four quadrants/regions (A-E standard, M/R/S full scorecard) together with individual quadrant plots.: [evidence span withheld — anchor status: not_found]
