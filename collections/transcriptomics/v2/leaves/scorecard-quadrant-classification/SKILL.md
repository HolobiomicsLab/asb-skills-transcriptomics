---
name: scorecard-quadrant-classification
description: Use when a CSV contains per-gene log2 fold-change and adjusted p-value
  columns for pairwise treatment-vs-control comparisons (e.g. VAN/LZD/CPT antibiotics,
  AGXX vs silver coatings) and you need data-driven identification of extreme or moderately
  changing genes without preselecting them.
when_to_use_negative:
- The dataset has raw counts or TPMs only — scorecard requires pre-computed log2 fold-change
  and adjusted p-value columns in the CSV, as stated in the README
- You only need a standard four-way plot without significance filtering or extreme-region
  detection (set 'multiplication factor' to 1; the added quadrant machinery is unnecessary)
- Input is a single-condition experiment with no pairwise treatment/control comparisons
  to contrast (the scorecard compares two treatments against a control per call)
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scorecard_functions
  role: Python library providing generate_parameters, data_loading, scorecard (per-quadrant
    classification), and reconstruct_scorecard used to build the quadrant scorecard
  repo: https://github.com/m89p067/Scorecard
- name: Pandas
  role: Loads the CSV of log2 fold changes and adjusted p-values into the DataFrame
    consumed by scorecard(df,param_dict)
- name: Python
  role: Language in which the scorecard library is written and invoked
  repo: https://github.com/m89p067/Scorecard
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_005
  source_papers:
  - doi: 10.1007/s00284-025-04435-3
    title: 'A Scorecard for Information Synthesis in Multiple Experimental Conditions:
      Application to Bacterial Biofilm Matrix Transcriptomics'
  source_content_hash: sha256:eeb5cd11a901c580181f1e319ebb5261a729589f76c97b6e79f46e5aad9bb741
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/m89p067/Scorecard
  tools:
  - Pandas
  - Python
  - scorecard library
  - scorecard_functions
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# scorecard-quadrant-classification

## Summary

Applies the Python scorecard_functions library to classify genes from pre-computed log2 fold-change and adjusted p-value columns into significance-filtered quadrants (regions of interest) of a four-way scorecard plot, per pairwise experimental comparison.

## When to use

Use when a CSV contains per-gene log2 fold-change and adjusted p-value columns for pairwise treatment-vs-control comparisons (e.g. VAN/LZD/CPT antibiotics, AGXX vs silver coatings) and you need data-driven identification of extreme or moderately changing genes without preselecting them.

## When NOT to use

- The dataset has raw counts or TPMs only — scorecard requires pre-computed log2 fold-change and adjusted p-value columns in the CSV, as stated in the README
- You only need a standard four-way plot without significance filtering or extreme-region detection (set 'multiplication factor' to 1; the added quadrant machinery is unnecessary)
- Input is a single-condition experiment with no pairwise treatment/control comparisons to contrast (the scorecard compares two treatments against a control per call)

## Inputs

- CSV file with one row per gene (gene name/locus tag column, e.g. SAUSA300, SMU, EF.peg IDs) and pre-computed log2 fold-change and adjusted p-value columns per comparison (header row matching param_dict column names)
- Parameter dictionary from scorecard_functions.generate_parameters() with fold-change threshold, multiplication factor, significance threshold, and column-name mappings
- Pandas DataFrame produced by scorecard_functions.data_loading(param_dict)

## Outputs

- Per-quadrant records of statistically significant gene entries (stored for each of the four regions of interest)
- Individual quadrant plots plus a reconstructed full scorecard figure
- JSON outcome files and scorecard figures written to the save_dir

## How to apply

Build the parameter dictionary with scorecard_functions.generate_parameters(), setting base_dir/filename to the CSV, the 'FC cond x'/'FC cond y' and 'padj cond x'/'padj cond y' column names, gene_name column, and thresholds — e.g. dataset 1 uses fold-change thresholds between 2 and 3 (multiplication factor 1.5) with significance p<=0.05, while dataset 3 uses fold-change 2 and 4 (multiplication factor 2) with p<=0.001. Load the CSV with data_loading(param_dict) into a Pandas DataFrame, optionally draw scorecard_legend(param_dict), then call scorecard_functions.scorecard(df,param_dict) to store statistically significant entries per quadrant together with individual quadrant plots (useful for crowded scorecards). Because scorecard() only computes the four single quadrants, complete the figure with scorecard_functions.reconstruct_scorecard(df,param_dict), tuning image size via fig_size/use_figsize and figsize_factor. The 'multiplication factor' defines the extreme regions beyond the standard fold-change threshold (setting it to 1 reduces the scorecard to a four-way plot).

## Related tools

- **scorecard_functions** (Python library providing generate_parameters, data_loading, scorecard (per-quadrant classification), and reconstruct_scorecard used to build the quadrant scorecard) — https://github.com/m89p067/Scorecard
- **Pandas** (Loads the CSV of log2 fold changes and adjusted p-values into the DataFrame consumed by scorecard(df,param_dict))
- **Python** (Language in which the scorecard library is written and invoked) — https://github.com/m89p067/Scorecard

## Examples

```
import scorecard_functions; param_dict=scorecard_functions.generate_parameters(); df=scorecard_functions.data_loading(param_dict); scorecard_functions.scorecard(df,param_dict); scorecard_functions.reconstruct_scorecard(df,param_dict)
```

## Evaluation signals

- The reconstructed scorecard figure and JSON outcome files exist in the save_dir output directory
- Genes reported for the dataset appear in the expected quadrant tables, e.g. dataset 1 VAN comparisons yield markedly down-regulated genes in Q3 (GraS/ApsS locus SAUSA300_RS03465 downregulated under VAN vs CPT and LZA)
- SMU_494 is flagged as downregulated for sucrose in the dataset 3 scorecard, matching the reported finding
- Dataset 4 extreme-activation genes EF.peg.3040 and EF.peg.3041 (copper-related) appear in the AGXX-vs-silver extreme regions
- All four quadrants were computed before reconstruction (scorecard() must be run for each quadrant; reconstruct_scorecard() merges them)

## Limitations

- Typical four-way plots are not filtered by statistical significance, which is essential in gene analysis — the scorecard adds this filtering, but 'multiplication factor' = 1 reverts to an unfiltered four-way plot
- Scorecards can be crowded with many significant entries; individual quadrant plots are provided as a fallback
- Adjusted p-values and log2 fold-changes must be pre-computed upstream (e.g. via TRANSIT resampling with beta-geometric normalization and Benjamini-Hochberg correction for dataset 1); the scorecard does not compute them

## Evidence

- [readme] Initially, run scorecard single quandrant calculations using the following code:: "Initially, run scorecard single quandrant calculations using the following code:"
- [readme] This function will store statistically significant entries for each quadrant together with plots of individual quandrants.: "This function will store statistically significant entries for each quadrant together with plots of individual quandrants."
- [results] six comparisons were made applying the scorecard and setting fold-change thresholds between 2 and 3 (that is, a multiplication factor of 1.5) and a significance level of: "six comparisons were made applying the scorecard and setting fold-change thresholds between 2 and 3 (that is, a multiplication factor of 1.5) and a significance level of"
- [results] the comparisons involving VAN mainly identified markedly down-regulated genes in Q3: "the comparisons involving VAN mainly identified markedly down-regulated genes in Q3"
- [methods] Now load the CSV data into the computer memory as Pandas Dataframe by typing: "Now load the CSV data into the computer memory as Pandas Dataframe by typing"
