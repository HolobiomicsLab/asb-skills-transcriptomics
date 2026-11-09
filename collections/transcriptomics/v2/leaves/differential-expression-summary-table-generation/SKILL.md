---
name: differential-expression-summary-table-generation
description: Use when a main folder hosts one sub-directory per pairwise comparison
  (each with pre-computed scorecard outputs) and you need per-gene totals across all
  comparisons to separate common gene-expression changes from condition-specific ones.
when_to_use_negative:
- You only have a single pairwise comparison (one sub-folder) — occurrence totals
  across multiple comparisons are meaningless and the multiple-view merge requires
  several sub-directories
- The sub-directories contain raw fold-change/p-value CSVs but no pre-computed scorecard
  outputs — run the scorecard and reconstruct_scorecard steps first
- You need a four-way plot summary — the merge (multiple_view/common_entries) works
  only for scorecards, not four-way plots
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: scorecard_functions
  role: Python library whose count_frequencies and common_entries functions compute
    per-comparison gene frequencies and merge them into a total-occurrence report
  repo: github.com/m89p067/Scorecard
- name: Pandas
  role: Handles the CSV expression data loaded as DataFrames that feed the scorecard
    pipeline and the tabular occurrence reports
- name: Python
  role: Language in which the scorecard library is written and the merge workflow
    is executed
  repo: github.com/m89p067/Scorecard
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1007/s00284-025-04435-3
    title: 'A Scorecard for Information Synthesis in Multiple Experimental Conditions:
      Application to Bacterial Biofilm Matrix Transcriptomics'
  source_content_hash: sha256:422aebb8f430514eb4d085f494900d09b1c4173d3ae8307d0997f21903c58a4e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pandas
  - Python
  - scorecard_functions
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# differential-expression-summary-table-generation

## Summary

Merges scorecard outputs from all pairwise experimental comparisons into a single per-gene occurrence report, distinguishing globally altered genes (e.g., SA_RS13430 active on 65 scorecards) from isolated, condition-specific modifications.

## When to use

Use when a main folder hosts one sub-directory per pairwise comparison (each with pre-computed scorecard outputs) and you need per-gene totals across all comparisons to separate common gene-expression changes from condition-specific ones.

## When NOT to use

- You only have a single pairwise comparison (one sub-folder) — occurrence totals across multiple comparisons are meaningless and the multiple-view merge requires several sub-directories
- The sub-directories contain raw fold-change/p-value CSVs but no pre-computed scorecard outputs — run the scorecard and reconstruct_scorecard steps first
- You need a four-way plot summary — the merge (multiple_view/common_entries) works only for scorecards, not four-way plots

## Inputs

- main_folder (string path) hosting one sub-directory per experimental comparison, each with pre-computed scorecard outputs
- Frequency summary previously created by scorecard_functions.count_frequencies

## Outputs

- CSV or Excel file listing per-gene occurrence totals across all scorecard comparisons
- Additional report of uncommon entries (genes specific to isolated conditions)
- Summary of the number of entries/genes identified by the scorecard in each comparison (from count_frequencies)

## How to apply

Organize the pipeline so a main folder contains one sub-directory per experimental comparison, each holding the scorecard outputs produced by scorecard_functions.scorecard and reconstruct_scorecard. First run scorecard_functions.count_frequencies(main_folder) to summarize the number of entries/genes identified by the scorecard in each comparison; this is required because the merge step re-uses the frequency information it creates. Then run scorecard_functions.common_entries(main_folder) to compute total occurrences per gene across all comparisons, and generate the additional report of uncommon entries highlighting genes specific to isolated conditions rather than globally altered ones. Save the occurrence-count report as CSV or Excel listing per-gene totals (e.g., gene X active in N of M comparisons); the rationale is that four-way plots cannot merge information across multiple plots, whereas the scorecard compresses all scorecards into a single graph of relative gene positioning across experiments.

## Related tools

- **scorecard_functions** (Python library whose count_frequencies and common_entries functions compute per-comparison gene frequencies and merge them into a total-occurrence report) — github.com/m89p067/Scorecard
- **Pandas** (Handles the CSV expression data loaded as DataFrames that feed the scorecard pipeline and the tabular occurrence reports)
- **Python** (Language in which the scorecard library is written and the merge workflow is executed) — github.com/m89p067/Scorecard

## Examples

```
scorecard_functions.count_frequencies(main_folder); scorecard_functions.common_entries(main_folder)
```

## Evaluation signals

- The output report exists in CSV or Excel format
- The report contains per-gene occurrence counts for every scorecard region of interest
- Occurrence counts are consistent with the number of sub-folder comparisons present in main_folder
- common_entries is called after count_frequencies, since it re-uses the frequency information previously created
- Globally changed genes (high occurrence counts across comparisons) are distinguishable from condition-specific ones (low/uncommon counts)

## Limitations

- common_entries must be called after count_frequencies because it re-uses information previously created by that function
- The merge/overview functionality applies only to the scorecard, not to four-way plots
- No version number is stated for the Python library or Python itself, hindering reproducibility

## Evidence

- [results] The library merges scorecards by compressing all scorecards from all combinations of the experimental conditions into a single graph reporting the relative positioning of a gene across experiments: "there is a specific function included in the software library that produces a general visualization of all comparisons"
- [readme] common_entries must run after count_frequencies and produces CSV or Excel occurrence reports: "To create a final report on single genes/entries run the following function. Remember to call it after *count_frequencies* because it re-uses information previously created by this function."
- [intro] Four-way plots cannot merge information across multiple plots, motivating the scorecard merge: "Finally, there is no ability to merge information from the four-way plots."
- [intro] The scorecard highlights common changes across multiple experiments versus isolated modifications: "Another feature introduced by the scorecard library is the ability to highlight common changes across multiple experiments or isolated modifications"
