---
name: fold-change-threshold-parameterization
description: Use when comparing two experimental conditions against a common control
  with pre-computed log2 fold-changes and adjusted p-values (CSV with gene ID column),
  and you need to tune standard vs. extreme fold-change cut-offs and the p-value significance
  level before scorecard classification.
when_to_use_negative:
- Your dataset does not contain pre-computed fold-change values and related adjusted
  p-values as columns in a CSV file — the library requires these as input, not raw
  count data
- You want a plain four-way plot rather than a scorecard — setting the multiplication
  factor to 1 collapses the scorecard into a four-way plot, which lacks the dual-threshold
  magnitude regions
- You have more than two treatment conditions to compare in a single plot — the scorecard
  partitions a Cartesian plane of two treatments' log fold-changes versus a common
  control, so multi-condition comparisons need separate pairwise runs per subfolder
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: scorecard_functions
  role: Python library whose generate_parameters() creates the parameter dictionary
    whose th_fold_change, multiplication factor, and th_significance keys define the
    fold-change and significance thresholds; data_loading, scorecard_legend, scorecard,
    and reconstruct_scorecard consume these parameters
  repo: https://github.com/m89p067/Scorecard
- name: Pandas
  role: data_loading() returns the CSV of pre-computed log2 fold-changes and adjusted
    p-values as a Pandas DataFrame for threshold-based classification
- name: Python
  role: scripting environment in which the scorecard parameter dictionary is generated
    and modified
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s00284-025-04435-3
    title: 'A Scorecard for Information Synthesis in Multiple Experimental Conditions:
      Application to Bacterial Biofilm Matrix Transcriptomics'
  source_content_hash: sha256:94388f0eaacb771006757d03062865f756f4f68b75b9bdcd144a8de0e5cc3c27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/m89p067/Scorecard
  tools:
  - Pandas
  - Python
  - scorecard_functions
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# fold-change-threshold-parameterization

## Summary

Configure the Scorecard library's dual fold-change thresholds and significance cut-off by setting th_fold_change, a multiplication factor that derives a second (extreme) threshold, and th_significance in the parameter dictionary returned by generate_parameters(), so that genes are partitioned into regions of interest by magnitude within each quadrant of two treatments' log2 fold-changes versus a common control.

## When to use

Use when comparing two experimental conditions against a common control with pre-computed log2 fold-changes and adjusted p-values (CSV with gene ID column), and you need to tune standard vs. extreme fold-change cut-offs and the p-value significance level before scorecard classification.

## When NOT to use

- Your dataset does not contain pre-computed fold-change values and related adjusted p-values as columns in a CSV file — the library requires these as input, not raw count data
- You want a plain four-way plot rather than a scorecard — setting the multiplication factor to 1 collapses the scorecard into a four-way plot, which lacks the dual-threshold magnitude regions
- You have more than two treatment conditions to compare in a single plot — the scorecard partitions a Cartesian plane of two treatments' log fold-changes versus a common control, so multi-condition comparisons need separate pairwise runs per subfolder

## Inputs

- CSV file of pre-computed log2 fold-changes and adjusted p-values (e.g., 'Treatment1vsCtrl_Log2FoldChange', 'Treatment2vsCtrl_Log2FoldChange' plus matching padj columns) with a header row and a gene ID column
- User-chosen thresholds: standard log2 fold-change cut-off (th_fold_change), multiplication factor for the extreme threshold, and th_significance (0.05 or lower)
- Parameter dictionary from scorecard_functions.generate_parameters() (base_dir, filename, Treatment/Control names, fig_size, save_dir, gene_name, CSV delimiter, etc.)

## Outputs

- Updated param_dict with user-defined fold-change and significance thresholds
- Downstream: color-code legend of regions of interest (scorecard_legend), per-quadrant/region assignments (A-E standard; M/R/S full scorecard) of statistically significant genes with individual quadrant plots, and a reconstructed full scorecard image with JSON outcome files per quadrant

## How to apply

Call scorecard_functions.generate_parameters() to obtain the parameter dictionary, then set th_fold_change to the standard log2 fold-change cut-off and a 'multiplication factor' that automatically adds a second, more extreme threshold (setting the factor to 1 collapses the scorecard into a plain four-way plot). Set th_significance to 0.05 or lower (e.g., the paper uses p<=0.05 with fold-change thresholds of 2-3 and factor 1.5 on dataset 1, and fold-change 2 and 4 with factor 2 and p<=0.001 on dataset 3 to detect fewer outliers). Also name the CSV file (base_dir, filename), the exact FC cond x/y and padj cond x/y column names, the gene_name column, and the save_dir where scorecards and JSON outcomes will be written. Rationale: extreme variations are selected during pairwise comparisons based on log2 fold-change thresholds and statistical significance, so these parameters define the regions of interest before running scorecard(), scorecard_legend(), and reconstruct_scorecard().

## Related tools

- **scorecard_functions** (Python library whose generate_parameters() creates the parameter dictionary whose th_fold_change, multiplication factor, and th_significance keys define the fold-change and significance thresholds; data_loading, scorecard_legend, scorecard, and reconstruct_scorecard consume these parameters) — https://github.com/m89p067/Scorecard
- **Pandas** (data_loading() returns the CSV of pre-computed log2 fold-changes and adjusted p-values as a Pandas DataFrame for threshold-based classification)
- **Python** (scripting environment in which the scorecard parameter dictionary is generated and modified)

## Examples

```
param_dict=scorecard_functions.generate_parameters(); param_dict['th_fold_change']=2; param_dict['multiplication factor']=1.5; param_dict['th_significance']=0.05
```

## Evaluation signals

- Setting the 'multiplication factor' to 1 must reproduce a four-way plot rather than the scorecard, confirming the second (extreme) threshold is derived by multiplication
- The reconstructed scorecard image and JSON outcome files recording significant genes per quadrant must exist in save_dir after running scorecard() and reconstruct_scorecard()
- Marker counts per region of the scorecard must be consistent with the count_frequencies summary
- With dataset 1 parameters (fold-change thresholds 2-3, multiplication factor 1.5, p<=0.05), six pairwise comparisons should yield markedly down-regulated genes in quadrant Q3 for VAN comparisons and markedly over-expressed genes for other antibiotics
- Loosening thresholds as on dataset 3 (fold-change 2 and 4, multiplication factor 2, p<=0.001) must reduce the number of detected outliers

## Limitations

- The multiplication factor must be > 1; setting it to 1 produces a four-way plot, not the scorecard
- No version number is stated for the Python library or for Python itself (from missing signals)
- Crowded scorecards may require inspecting individual quadrant plots, which the scorecard() function stores separately

## Evidence

- [readme] th_fold_change # log2 fold change standard threshold (another threshold will be automatically added based on 'multiplication factor'): "th_fold_change # log2 fold change standard threshold (another threshold will be automatically added based on 'multiplication factor')"
- [results] six comparisons were made applying the scorecard and setting fold-change thresholds between 2 and 3 (that is, a multiplication factor of 1.5) and a significance level of: "six comparisons were made applying the scorecard and setting fold-change thresholds between 2 and 3 (that is, a multiplication factor of 1.5) and a significance level of"
- [discussion] extreme variations selected during pairwise comparisons based on: "extreme variations selected during pairwise comparisons based on"
- [readme] multiplication factor" # Factor to multiply the log2 Fold change threshold to detect extreme values (setting it to 1 will produce a four-way plot, not the scorecard): "Factor to multiply the log2 Fold change threshold to detect extreme values (setting it to 1 will produce a four-way plot, not the scorecard)"
- [results] the thresholds were modified to detect fewer outliers (Table SM5) using fold-change thresholds of 2 and 4 (aka, multiplication factor of 2) and p: "the thresholds were modified to detect fewer outliers (Table SM5) using fold-change thresholds of 2 and 4 (aka, multiplication factor of 2) and p"
