---
name: gene-count-frequency-summarization
description: Use when scorecard single-quadrant analyses have been run for multiple
  comparisons (e.g., VAN/CPT/DEL/LZD vs untreated in S. aureus dataset 1) and you
  need a tabulated count of genes per quadrant per comparison.
when_to_use_negative:
- The individual scorecard quadrants and reconstructed scorecards have not yet been
  generated for the comparisons in main_folder, so no per-quadrant entries exist to
  count
- The goal is a radial multi-comparison visualization of fold-change magnitude rather
  than per-quadrant gene counts (use multiple_view instead)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0085
tools:
- name: scorecard_functions
  role: Python library whose count_frequencies(main_folder) function tabulates entries/genes
    per quadrant per comparison and writes quadrant_gene_counts.csv
  repo: github.com/m89p067/Scorecard
- name: Pandas
  role: DataFrame library used to load the source CSV (log2 fold-change and adjusted
    p-value columns) upstream of the scorecard quadrants that count_frequencies summarizes
- name: Python
  role: Language in which the scorecard library, including count_frequencies, is implemented
    and invoked
  repo: github.com/m89p067/Scorecard
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s00284-025-04435-3
    title: 'A Scorecard for Information Synthesis in Multiple Experimental Conditions:
      Application to Bacterial Biofilm Matrix Transcriptomics'
  source_content_hash: sha256:e9addef36dc5b805866836022e871dca7986c558e922267c3b050ae6a067971a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pandas
  - Python
  - scorecard_functions
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gene-count-frequency-summarization

## Summary

Summarize how many entries/genes the scorecard assigns to each quadrant and region of interest across all pairwise experimental comparisons, by running scorecard_functions.count_frequencies(main_folder) and saving quadrant_gene_counts.csv.

## When to use

Use when scorecard single-quadrant analyses have been run for multiple comparisons (e.g., VAN/CPT/DEL/LZD vs untreated in S. aureus dataset 1) and you need a tabulated count of genes per quadrant per comparison.

## When NOT to use

- The individual scorecard quadrants and reconstructed scorecards have not yet been generated for the comparisons in main_folder, so no per-quadrant entries exist to count
- The goal is a radial multi-comparison visualization of fold-change magnitude rather than per-quadrant gene counts (use multiple_view instead)

## Inputs

- main_folder: string containing the full path of the folder hosting all sub-directories with experimental comparisons, each populated by scorecard and reconstruct_scorecard outputs
- JSON outcomes and statistically significant quadrant entries stored per comparison by scorecard_functions.scorecard(df,param_dict)

## Outputs

- quadrant_gene_counts.csv: number of entries/genes identified per quadrant per comparison
- Per-quadrant/per-comparison frequency counts consumable by scorecard_functions.common_entries (CSV or Excel with total occurrences)

## How to apply

After scorecard_functions.scorecard(df,param_dict) and reconstruct_scorecard(df,param_dict) have been executed for every comparison, arrange the outputs so that each comparison has its own subfolder inside a main folder. Call scorecard_functions.count_frequencies(main_folder), where main_folder is a string with the full path to the directory hosting the comparison subfolders; the function tabulates the number of entries/genes identified per quadrant per comparison and saves quadrant_gene_counts.csv. For dataset 1 (Staphylococcus aureus USA300, fold-change thresholds between 2 and 3 with multiplication factor 1.5 and padj <= 0.05), check that the counts reproduce the reported region-of-interest values, e.g., Q3 region D for VAN-vs-CPT/DEL/LZD comparisons and Q1 region A for CPT-vs-LZD, DEL-vs-CPT, DEL-vs-LZD. This step must precede scorecard_functions.common_entries(main_folder), which re-uses the frequency information it creates. Optionally visualize the same summary as a heatmap with scorecard_functions.quadrants_heatmap(main_folder).

## Related tools

- **scorecard_functions** (Python library whose count_frequencies(main_folder) function tabulates entries/genes per quadrant per comparison and writes quadrant_gene_counts.csv) — github.com/m89p067/Scorecard
- **Pandas** (DataFrame library used to load the source CSV (log2 fold-change and adjusted p-value columns) upstream of the scorecard quadrants that count_frequencies summarizes)
- **Python** (Language in which the scorecard library, including count_frequencies, is implemented and invoked) — github.com/m89p067/Scorecard

## Examples

```
import scorecard_functions; scorecard_functions.count_frequencies(main_folder)
```

## Evaluation signals

- quadrant_gene_counts.csv exists in the output directory and contains one count per quadrant per comparison subfolder
- Counts reproduce the reported region-of-interest values for dataset 1: Q3 region D for VAN-vs-CPT/DEL/LZD and Q1 region A for CPT-vs-LZD, DEL-vs-CPT, DEL-vs-LZD
- Counts are consistent with the number of statistically significant (padj <= 0.05) entries stored in each quadrant's JSON outcomes
- common_entries(main_folder) runs successfully afterwards, confirming the frequency information it re-uses was created correctly

## Limitations

- Counts depend on user-tuned thresholds (th_fold_change, th_significance, multiplication factor); dataset 1 used fold-change thresholds 2-3 (multiplication factor 1.5) and padj <= 0.05, so changing thresholds changes the reported counts
- count_frequencies must be called before common_entries, which re-uses previously created frequency information
- Requires the expected main_folder/subfolder structure with one subdirectory per experimental comparison

## Evidence

- [readme] To get a summary of the number of entries/genes indentified by the scorecard type: scorecard_functions.count_frequencies(main_folder): "To get a summary of the number of entries/genes indentified by the scorecard type:"
- [results] Call scorecard_functions.count_frequencies(main_folder) to tabulate the number of entries/genes identified per quadrant per comparison and save quadrant_gene_counts.csv: "the comparisons involving VAN mainly identified markedly down-regulated genes in Q3; in contrast, the other antibiotic-related genes were markedly over-expressed"
- [results] six comparisons were made applying the scorecard and setting fold-change thresholds between 2 and 3 (that is, a multiplication factor of 1.5) and a significance level of: "six comparisons were made applying the scorecard and setting fold-change thresholds between 2 and 3 (that is, a multiplication factor of 1.5)"
- [readme] Remember to call it after *count_frequencies* because it re-uses information previously created by this function.: "Remember to call it after *count_frequencies* because"
- [methods] param_dict=scorecard_functions.generate_parameters(): "param_dict=scorecard_functions.generate_parameters()"
