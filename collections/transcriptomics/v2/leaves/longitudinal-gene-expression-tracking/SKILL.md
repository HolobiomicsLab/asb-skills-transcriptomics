---
name: longitudinal-gene-expression-tracking
description: Use when pairwise differential-expression comparisons form a time course
  or experimental series (e.g., log2 fold-change and padj CSVs at t=3–90 min) and
  you need to confirm which scorecard-identified genes stay stable or shift over time.
when_to_use_negative:
- Only a single pairwise comparison exists — there is no time course or experiment
  series to track across
- count_frequencies/common_entries have not yet been run on the comparison sub-folders,
  so track_over_exper lacks its required pre-computed inputs
- Data are raw counts or lack pre-computed fold-change and adjusted p-value columns;
  the scorecard library expects log2 fold changes and padj as CSV columns
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scorecard_functions
  role: Python library whose track_over_exper function performs the longitudinal/experiment-series
    tracking of expression changes over the scorecard
  repo: github.com/m89p067/Scorecard
- name: Pandas
  role: Loads the per-comparison CSV of log2 fold changes and adjusted p-values into
    a DataFrame before scorecard runs
- name: Python
  role: Language in which the scorecard library is written and the pipeline is scripted
  repo: github.com/m89p067/Scorecard
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s00284-025-04435-3
    title: 'A Scorecard for Information Synthesis in Multiple Experimental Conditions:
      Application to Bacterial Biofilm Matrix Transcriptomics'
  source_content_hash: sha256:e345bbe2dbcb177f895229f7fa0ac836fb058c6b679fd0eb976b676744bd865d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pandas
  - Python
  - scorecard_functions
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# longitudinal-gene-expression-tracking

## Summary

A scorecard-based workflow that tracks how gene expression changes across time points or sequential experiments, using scorecard_functions.track_over_exper over sub-folders of pre-computed comparisons (e.g., Enterococcus faecalis on AGXX/Ag/V2A steel at t=3, 6, 12, 24, 60, 90 minutes).

## When to use

Use when pairwise differential-expression comparisons form a time course or experimental series (e.g., log2 fold-change and padj CSVs at t=3–90 min) and you need to confirm which scorecard-identified genes stay stable or shift over time.

## When NOT to use

- Only a single pairwise comparison exists — there is no time course or experiment series to track across
- count_frequencies/common_entries have not yet been run on the comparison sub-folders, so track_over_exper lacks its required pre-computed inputs
- Data are raw counts or lack pre-computed fold-change and adjusted p-value columns; the scorecard library expects log2 fold changes and padj as CSV columns

## Inputs

- Main folder containing sub-folders, each with scorecard outputs from one pairwise comparison (e.g., one per time point t=3, 6, 12, 24, 60, 90 min)
- Pre-computed log2 fold-change and adjusted p-value CSV columns per comparison (e.g., 'Treatment1vsCtrl_Log2FoldChange', 'Treatment1vsCtrl_padj')
- Outputs of scorecard_functions.count_frequencies / common_entries per sub-folder (required prerequisite)

## Outputs

- Longitudinal tracking plots/summaries of gene expression changes across experimental sub-folders
- Confirmation of time-stable vs time-varying behavior for scorecard-identified genes (e.g., EF.peg.3040, EF.peg.3041)
- JSON outcome files and scorecard figures stored in save_dir

## How to apply

First run the per-comparison scorecard pipeline (generate_parameters, data_loading, scorecard, reconstruct_scorecard) and then count_frequencies/common_entries in each comparison sub-folder of a main_folder, because track_over_exper re-uses information previously created by count_frequencies. Organize the directory as a main_folder containing one sub-folder per experimental comparison (e.g., the longitudinal time points t=3, 6, 12, 24, 60, 90 minutes of the Enterococcus faecalis AGXX vs Ag-coating vs V2A steel dataset). Call scorecard_functions.track_over_exper(main_folder) to track modifications of expression values across the scorecard over time, confirming behaviors such as time-stable extreme bioactivation (e.g., copper genes EF.peg.3040/EF.peg.3041). Validate by checking that genes known to be condition-specific or down-regulated (EF.peg.1868, EF.peg.426, EF.peg.481, EF.peg.2124, EF.peg.2384) appear in the expected quadrant tables and that the tracking outputs exist in the save_dir.

## Related tools

- **scorecard_functions** (Python library whose track_over_exper function performs the longitudinal/experiment-series tracking of expression changes over the scorecard) — github.com/m89p067/Scorecard
- **Pandas** (Loads the per-comparison CSV of log2 fold changes and adjusted p-values into a DataFrame before scorecard runs)
- **Python** (Language in which the scorecard library is written and the pipeline is scripted) — github.com/m89p067/Scorecard

## Examples

```
scorecard_functions.common_entries(main_folder); scorecard_functions.track_over_exper(main_folder)
```

## Evaluation signals

- Each comparison sub-folder contains count_frequencies/common_entries outputs before track_over_exper is invoked
- Known condition-specific genes appear in the expected quadrants across all time points: EF.peg.3040/EF.peg.3041 (AGXX-specific extreme activation vs silver), EF.peg.1868/EF.peg.426/EF.peg.481 (Ag/V2A-specific), EF.peg.2124/EF.peg.2384 (down-regulated)
- Tracking output confirms time-stable behavior of the extreme-bioactivation genes across t=3, 6, 12, 24, 60, 90 minutes
- Reconstructed scorecard figures and JSON outcome files exist in the output/save_dir for every comparison sub-folder
- Directory structure matches main_folder with one sub-folder per comparison (no missing time-point folders)

## Limitations

- track_over_exper must be called after common_entries because it re-uses information previously created by that function
- The scorecard library expects pre-computed log2 fold-change and adjusted p-value columns; it does not compute differential expression itself
- No version number is stated for the Python library or for Python itself, hindering reproducibility

## Evidence

- [results] the scorecard analyses were carried out longitudinally, to evaluate differential expression over time (t=3, 6, 12, 24, 60, 90 minutes) throughout the experimental cycle: "the scorecard analyses were carried out longitudinally, to evaluate differential expression over time (t=3, 6, 12, 24, 60, 90 minutes) throughout the experimental cycle"
- [readme] In case of experimental comparisons are over time or one wants to sum-up more experiments, this function can track modifications of the expression values over the scorecard.: "In case of experimental comparisons are over time or one wants to sum-up more experiments, this function can track modifications of the expression values over the scorecard."
- [readme] The input folder should contain sub-folders with pre-computed data regarding the comparisons (common_entries should be called first).: "The input folder should contain sub-folders with pre-computed data regarding the comparisons (common_entries should be called first)."
