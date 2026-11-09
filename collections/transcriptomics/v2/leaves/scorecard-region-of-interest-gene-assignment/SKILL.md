---
name: scorecard-region-of-interest-gene-assignment
description: Use when a CSV holds pre-computed log2 fold-change and padj columns for
  pairwise condition comparisons and you need to identify statistically significant
  extreme gene regulation (down- or up-regulated) per quadrant across multiple antibiotic
  or treatment contrasts.
when_to_use_negative:
- The input lacks pre-computed log2 fold-change and adjusted p-value columns — the
  scorecard requires these as CSV columns and does not compute differential expression
  itself
- Setting multiplication factor to 1 is requested — this produces a standard four-way
  plot rather than the scorecard region-of-interest structure
- Only a single condition pair is available with no interest in multi-comparison pattern
  synthesis — volcano plots or standard four-way plots are the simpler choices, and
  the scorecard's value is merging information across multiple comparisons
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scorecard_functions
  role: Python library whose generate_parameters, data_loading, scorecard_legend,
    scorecard, reconstruct_scorecard and count_frequencies functions execute the quadrant
    assignment workflow
  repo: github.com/m89p067/Scorecard
- name: Pandas
  role: Loads the CSV dataset into a DataFrame consumed by the scorecard functions
- name: Python
  role: Runtime environment for the scorecard library
  repo: github.com/m89p067/Scorecard
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s00284-025-04435-3
    title: 'A Scorecard for Information Synthesis in Multiple Experimental Conditions:
      Application to Bacterial Biofilm Matrix Transcriptomics'
  source_content_hash: sha256:324ffda3e5c9bee97a3238cc0021a1d1dab139da16483418391ad0df149e281b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pandas
  - Python
  - scorecard_functions
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# scorecard-region-of-interest-gene-assignment

## Summary

Assign genes from pairwise differential-expression comparisons (log2 fold change plus adjusted p-values) to quadrant/region-of-interest zones of a scorecard using the Python scorecard_functions library, exposing extreme or moderate regulation patterns across multiple experimental conditions (e.g., VAN/CPT/DEL/LZD vs untreated controls in S. aureus dataset 1).

## When to use

Use when a CSV holds pre-computed log2 fold-change and padj columns for pairwise condition comparisons and you need to identify statistically significant extreme gene regulation (down- or up-regulated) per quadrant across multiple antibiotic or treatment contrasts.

## When NOT to use

- The input lacks pre-computed log2 fold-change and adjusted p-value columns — the scorecard requires these as CSV columns and does not compute differential expression itself
- Setting multiplication factor to 1 is requested — this produces a standard four-way plot rather than the scorecard region-of-interest structure
- Only a single condition pair is available with no interest in multi-comparison pattern synthesis — volcano plots or standard four-way plots are the simpler choices, and the scorecard's value is merging information across multiple comparisons

## Inputs

- CSV file with a gene name/ID column plus pre-computed log2 fold-change and adjusted p-value columns per pairwise comparison (e.g., 'FC cond x', 'padj cond x')
- Parameter dictionary (param_dict) from scorecard_functions.generate_parameters() with base_dir, filename, gene_name, FC/padj column names, th_significance (0.05), th_fold_change and multiplication factor 1.5, save_dir
- Pandas DataFrame loaded via scorecard_functions.data_loading(param_dict)

## Outputs

- Per-quadrant statistically significant gene assignments stored with individual quadrant plots and JSON outcomes in save_dir
- Reconstructed full scorecard figures per comparison (via reconstruct_scorecard, default fig_size)
- quadrant_gene_counts.csv frequency table of entries/genes per quadrant per comparison (via count_frequencies)

## How to apply

Import scorecard_functions and build the parameter dictionary with scorecard_functions.generate_parameters(), setting base_dir/filename of the CSV, gene_name, the exact 'FC cond x/y' and 'padj cond x/y' column names per comparison, save_dir, and a significance threshold of padj <= 0.05. Set fold-change thresholds between 2 and 3 using multiplication factor 1.5 (as in the S. aureus USA300 dataset 1 analysis) so extreme values fall in outer regions; factor 1 would degenerate to a four-way plot. Load the CSV via scorecard_functions.data_loading(param_dict), generate the color-code/region-of-interest legend with scorecard_functions.scorecard_legend(param_dict), then run scorecard_functions.scorecard(df,param_dict) per comparison to compute single-quadrant assignments, storing significant entries per quadrant with individual quadrant plots and JSON outcomes. Rebuild full scorecards with scorecard_functions.reconstruct_scorecard(df,param_dict) and tabulate quadrant gene counts with scorecard_functions.count_frequencies(main_folder) into quadrant_gene_counts.csv to verify the region-of-interest assignments.

## Related tools

- **scorecard_functions** (Python library whose generate_parameters, data_loading, scorecard_legend, scorecard, reconstruct_scorecard and count_frequencies functions execute the quadrant assignment workflow) — github.com/m89p067/Scorecard
- **Pandas** (Loads the CSV dataset into a DataFrame consumed by the scorecard functions)
- **Python** (Runtime environment for the scorecard library) — github.com/m89p067/Scorecard

## Examples

```
import scorecard_functions; param_dict=scorecard_functions.generate_parameters(); df=scorecard_functions.data_loading(param_dict); scorecard_functions.scorecard(df,param_dict)
```

## Evaluation signals

- quadrant_gene_counts.csv from count_frequencies reproduces the reported region-of-interest counts (e.g., Q1 region A for CPT-vs-LZD, DEL-vs-CPT, DEL-vs-LZD; Q3 region D for VAN-vs-CPT/DEL/LZD comparisons)
- Only entries with padj <= 0.05 are retained inside regions of interest; non-significant points appear only as 'other' markers outside ROI colors
- Each comparison's folder contains four individual quadrant plots plus JSON outcome files, and reconstruct_scorecard merges them into one scorecard figure
- Fold-change thresholds land between 2 and 3 (multiplication factor 1.5); threshold values outside this range or factor=1 indicate misconfiguration
- Dataset 1 expectations: VAN-involving comparisons yield markedly down-regulated genes in Q3, other antibiotic comparisons yield over-expressed genes such as SAUSA300_RS03525, SAUSA300_RS04560, SAUSA300_RS08475

## Limitations

- Scorecard parameters (thresholds, multiplication factor) must be tuned per dataset; dataset 3 used stricter settings (fold-change 2 and 4, multiplication factor 2, p<=0.001) to avoid too many outliers
- Individual quadrant plots are needed for crowded scorecards since the full reconstructed scorecard can be overloaded with gene labels
- The library ships no stated version number for Python or the library itself (per the article's missing signals), which hinders exact reproducibility
- The scorecard itself does not compute fold changes; the user must supply pre-computed log2 FC and padj values (dataset 1 used TRANSIT resampling with beta-geometric normalization and Benjamini-Hochberg correction upstream)

## Evidence

- [results] six comparisons were made applying the scorecard and setting fold-change thresholds between 2 and 3 (that is, a multiplication factor of 1.5) and a significance level of: "six comparisons were made applying the scorecard and setting fold-change thresholds between 2 and 3 (that is, a multiplication factor of 1.5)"
- [readme] This function will store statistically significant entries for each quadrant together with plots of individual quandrants.: "This function will store statistically significant entries for each quadrant together with plots of individual quandrants."
- [methods] scorecard_functions.scorecard(df,param_dict): "scorecard_functions.scorecard(df,param_dict)"
- [discussion] the comparisons involving VAN mainly identified markedly down-regulated genes in Q3; in contrast, the other antibiotic-related genes were markedly over-expressed: "the comparisons involving VAN mainly identified markedly down-regulated genes in Q3; in contrast, the other antibiotic-related genes were markedly over-expressed"
- [readme] multiplication factor  # Factor to multiply the log2 Fold change threshold to detect extreme values (setting it to 1 will produce a four-way plot, not the scorecard): "setting it to 1 will produce a four-way plot, not the scorecard"
- [readme] After creating all four quadrants with previous function, one should execute scorecard reconstruction from individual quandrants: "After creating all four quadrants with previous function, one should execute scorecard reconstruction from individual quandrants"
