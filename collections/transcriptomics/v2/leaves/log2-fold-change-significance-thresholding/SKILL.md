---
name: log2-fold-change-significance-thresholding
description: Use when a CSV already contains pre-computed log2 fold-changes and adjusted
  p-values per pairwise comparison and you need to narrow thousands of RNA-seq/transposon-insertion
  entries to extreme or moderately changing genes without preselecting candidates.
when_to_use_negative:
- The input is raw sequencing data or a count matrix without pre-computed log2 fold-change
  and adjusted p-value columns — the CSV must already contain these as columns (e.g.
  'Treatment1vsCtrl_Log2FoldChange', 'Treatment1vsCtrl_padj')
- The analysis is experience-driven tracking of a preselected set of symbols rather
  than data-driven feature extraction — the article's goal is explicitly to bypass
  gene preselection
- '''multiplication factor'' is set to 1, which produces a plain four-way plot rather
  than the scorecard''s extreme-region regions of interest'
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: scorecard_functions
  role: Provides generate_parameters, data_loading, scorecard (per-quadrant significance/fold-change
    screening), reconstruct_scorecard, and count_frequencies/common_entries for the
    threshold-based selection
  repo: https://github.com/m89p067/Scorecard
- name: Pandas
  role: Loads the pre-computed log2 fold-change and adjusted p-value CSV into the
    DataFrame consumed by the scorecard functions
- name: Python
  role: Language of the scorecard library used to execute the thresholding workflow
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  - task_005
  source_papers:
  - doi: 10.1007/s00284-025-04435-3
    title: 'A Scorecard for Information Synthesis in Multiple Experimental Conditions:
      Application to Bacterial Biofilm Matrix Transcriptomics'
  source_content_hash: sha256:bfb0efbf22cd779bfb18cdc462e55edc16e859ae70b43286168ef0cd8a62a359
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/m89p067/Scorecard
  tools:
  - Pandas
  - Python
  - scorecard_functions
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# log2-fold-change-significance-thresholding

## Summary

A data-driven screening step that classifies genes as extreme or moderately changing by applying joint log2 fold-change thresholds (with a multiplication factor for extreme regions) and adjusted-p significance cut-offs, as implemented in the Scorecard Python library for bacterial biofilm transcriptomics.

## When to use

Use when a CSV already contains pre-computed log2 fold-changes and adjusted p-values per pairwise comparison and you need to narrow thousands of RNA-seq/transposon-insertion entries to extreme or moderately changing genes without preselecting candidates.

## When NOT to use

- The input is raw sequencing data or a count matrix without pre-computed log2 fold-change and adjusted p-value columns — the CSV must already contain these as columns (e.g. 'Treatment1vsCtrl_Log2FoldChange', 'Treatment1vsCtrl_padj')
- The analysis is experience-driven tracking of a preselected set of symbols rather than data-driven feature extraction — the article's goal is explicitly to bypass gene preselection
- 'multiplication factor' is set to 1, which produces a plain four-way plot rather than the scorecard's extreme-region regions of interest

## Inputs

- CSV file with one gene/gene-ID column (e.g. SAUSA300 locus tags, SA_RS gene IDs, SMU_ identifiers, EF.peg. identifiers) and pre-computed log2 fold-change and adjusted p-value columns per pairwise comparison, header row matching the param_dict column names
- Parameter dictionary from scorecard_functions.generate_parameters() configured with base_dir, filename, gene_name, FC cond x/y, padj cond x/y, th_fold_change, multiplication factor, th_significance, save_dir

## Outputs

- Per-quadrant tables/JSON outcome files of statistically significant gene entries stored in save_dir subfolders per comparison
- Individual quadrant plots and a reconstructed scorecard figure per comparison (scorecard_functions.reconstruct_scorecard)
- Entry-frequency summaries via count_frequencies and common_entries (total-occurrence reports of common and uncommon genes)

## How to apply

Build the parameter dictionary with scorecard_functions.generate_parameters(), setting th_fold_change (the standard log2 fold-change threshold), multiplication factor (which auto-generates the extreme-region threshold; set to 1 for a plain four-way plot), th_significance (e.g. 0.05), and gene_name plus the exact 'FC cond x/y' and 'padj cond x/y' column names. Load the pre-computed log2FC/padj CSV into a Pandas DataFrame via scorecard_functions.data_loading(param_dict) — the CSV must have a header row matching those column names. Choose dataset-appropriate cut-offs: Dataset 1 (SAUSA300 transposon insertion sequencing) used fold-change thresholds between 2 and 3 (multiplication factor 1.5) with p<=0.05, while Dataset 3 (S. mutans carbohydrates) tightened to fold-change thresholds of 2 and 4 (multiplication factor 2) and p<=0.001 to detect fewer outliers. Then run scorecard_functions.scorecard(df,param_dict) per quadrant so statistically significant entries are stored per quadrant with individual quadrant plots. Screening on both fold-change and significance is what distinguishes this from volcano plots, which 'are not filtered by statistical significance' in the four-way-plot tradition and crowd with large sample sizes.

## Related tools

- **scorecard_functions** (Provides generate_parameters, data_loading, scorecard (per-quadrant significance/fold-change screening), reconstruct_scorecard, and count_frequencies/common_entries for the threshold-based selection) — https://github.com/m89p067/Scorecard
- **Pandas** (Loads the pre-computed log2 fold-change and adjusted p-value CSV into the DataFrame consumed by the scorecard functions)
- **Python** (Language of the scorecard library used to execute the thresholding workflow)

## Examples

```
import scorecard_functions; param_dict=scorecard_functions.generate_parameters(); df=scorecard_functions.data_loading(param_dict); scorecard_functions.scorecard(df,param_dict)
```

## Evaluation signals

- The reconstructed scorecard figure and JSON outcome files exist in save_dir for each comparison subfolder
- Dataset 1 reproduction: SAUSA300_RS03465 (GraS/ApsS) is flagged as markedly downregulated in VAN comparisons (Q3 side) under fold-change thresholds 2–3 (multiplication factor 1.5) and p<=0.05
- Dataset 3 reproduction: SMU_494 is flagged as downregulated for sucrose under fold-change thresholds of 2 and 4 (multiplication factor 2) and p<=0.001
- Dataset 4 validation: extreme-region tables contain EF.peg.3040 and EF.peg.3041 (AGXX-specific copper genes) and Ag/V2A-specific EF.peg.1868, EF.peg.426, EF.peg.481 in expected regions
- count_frequencies/common_entries reports show the expected gene counts (e.g. 1126 outer-region genes for Dataset 2) and presence of the reported signatures in the quadrant records

## Limitations

- With multiplication factor set to 1 the output degenerates to a four-way plot rather than the scorecard's extreme regions
- The library does not state a version number, hindering reproducibility; the dataset CSV must already contain pre-computed fold changes and adjusted p-values
- Threshold tuning is dataset-dependent: default-significance datasets (p<=0.05) can flag many entries, so Dataset 3 needed stricter cut-offs (FC 2 and 4, p<=0.001) to detect fewer outliers

## Evidence

- [results] six comparisons were made applying the scorecard and setting fold-change thresholds between 2 and 3 (that is, a multiplication factor of 1.5) and a significance level of: "six comparisons were made applying the scorecard and setting fold-change thresholds between 2 and 3 (that is, a multiplication factor of 1.5)"
- [results] the thresholds were modified to detect fewer outliers (Table SM5) using fold-change thresholds of 2 and 4 (aka, multiplication factor of 2) and p: "using fold-change thresholds of 2 and 4 (aka, multiplication factor of 2) and p"
- [readme] th_fold_change # log2 fold change standard threshold (another threshold will be automatically added based on 'multiplication factor'): "th_fold_change # log2 fold change standard threshold (another threshold will be automatically added based on 'multiplication factor')"
- [discussion] extreme variations selected during pairwise comparisons based on log2 fold-change thresholds and statistical significance: [evidence span withheld — anchor status: not_found]
- [readme] Remember that the CSV file should have the first row as header, reporting the column names matching the information provided in the dictionary of parameters.: "the CSV file should have the first row as header, reporting the column names matching the information provided in the dictionary of parameters"
