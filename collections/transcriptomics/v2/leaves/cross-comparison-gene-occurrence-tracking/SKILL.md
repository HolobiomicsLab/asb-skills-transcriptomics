---
name: cross-comparison-gene-occurrence-tracking
description: Use when a CSV of pre-computed log2 fold-changes and adjusted p-values
  spans several pairwise comparisons and you must identify which genes recur across
  conditions or appear only in one (e.g. VAN-specific Q3 down-regulation).
when_to_use_negative:
- Input is raw sequencing data or raw RNA-seq counts without pre-computed log2 fold-change
  and adjusted p-value columns — the scorecard expects an already-processed CSV
- 'You want a standard four-way plot rather than the scorecard: setting ''multiplication
  factor'' to 1 collapses the scorecard into a four-way plot, losing the extreme-value
  regions'
- Your analysis strategy is experience-driven, targeting a preselected set of gene
  symbols, rather than the data-driven feature extraction the scorecard is designed
  for
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scorecard_functions
  role: Python library supplying generate_parameters, data_loading, scorecard, reconstruct_scorecard,
    count_frequencies and common_entries used to build per-comparison quadrants and
    cross-comparison occurrence reports
  repo: github.com/m89p067/Scorecard
- name: Pandas
  role: Loads the preprocessed Dataset 1 CSV of log2 fold-changes and adjusted p-values
    into a DataFrame consumed by the scorecard functions
- name: Python
  role: Language in which the scorecard library is written and in which the aggregation
    workflow is scripted
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s00284-025-04435-3
    title: 'A Scorecard for Information Synthesis in Multiple Experimental Conditions:
      Application to Bacterial Biofilm Matrix Transcriptomics'
  source_content_hash: sha256:7cefa9cf3fcc4492cb3b10c90d33d66cc3164b86f0ca6496ea09c461dc5bbdba
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cross-comparison-gene-occurrence-tracking

## Summary

A scorecard-based procedure that aggregates statistically significant quadrant entries across multiple pairwise experimental comparisons (e.g. VAN, LZD, CPT conditions of Staphylococcus aureus Dataset 1) and produces total-occurrence reports of common and uncommon genes, enabling tracking of pathway loci such as the Gra/Aps cell-envelope stress genes across antibiotic treatments.

## When to use

Use when a CSV of pre-computed log2 fold-changes and adjusted p-values spans several pairwise comparisons and you must identify which genes recur across conditions or appear only in one (e.g. VAN-specific Q3 down-regulation).

## When NOT to use

- Input is raw sequencing data or raw RNA-seq counts without pre-computed log2 fold-change and adjusted p-value columns — the scorecard expects an already-processed CSV
- You want a standard four-way plot rather than the scorecard: setting 'multiplication factor' to 1 collapses the scorecard into a four-way plot, losing the extreme-value regions
- Your analysis strategy is experience-driven, targeting a preselected set of gene symbols, rather than the data-driven feature extraction the scorecard is designed for

## Inputs

- Preprocessed CSV file with gene names/locus tags (e.g. SAUSA300 tags) as rows and pre-computed log2 fold-change and adjusted p-value columns per comparison (e.g. 'Treatment1vsCtrl_Log2FoldChange', 'Treatment1vsCtrl_padj')
- Parameter dictionary from scorecard_functions.generate_parameters() configured with 'gene_name', 'FC cond x/y', 'padj cond x/y', thresholds and 'save_dir'
- main_folder directory containing one sub-folder per reconstructed experimental comparison

## Outputs

- Per-quadrant statistically significant entry lists and individual quadrant plots (JSON/scorecard outputs in save_dir)
- Reconstructed combined scorecard images per comparison
- Total-occurrence frequency summary (count_frequencies) and common/uncommon entries report (CSV or Excel, via common_entries)
- Gene-level findings report, e.g. dataset1_gene_findings_report.csv

## How to apply

Create the parameter dictionary with scorecard_functions.generate_parameters(), set fold-change thresholds between 2 and 3 (multiplication factor 1.5) and a significance threshold of p<=0.05, and point 'gene_name' to the SAUSA300 locus-tag column while mapping the FC/padj column names of each comparison. Load the preprocessed CSV as a Pandas DataFrame with scorecard_functions.data_loading(param_dict). For each comparison run scorecard_functions.scorecard(df,param_dict) to store statistically significant entries per quadrant with individual quadrant plots, then scorecard_functions.reconstruct_scorecard(df,param_dict) to rebuild the combined scorecard. Organize outputs as a main_folder with one sub-folder per comparison, then call scorecard_functions.count_frequencies(main_folder) followed by scorecard_functions.common_entries(main_folder) — count_frequencies must run first because common_entries re-uses its outputs — to generate total-occurrence reports of common and uncommon entries. Finally, extract the loci of interest (e.g. SAUSA300_RS03455, SAUSA300_RS03465, SAUSA300_RS03475, SAUSA300_RS10935, SAUSA300_RS11125) from the quadrant and occurrence reports and compile a gene-level findings CSV such as dataset1_gene_findings_report.csv.

## Related tools

- **scorecard_functions** (Python library supplying generate_parameters, data_loading, scorecard, reconstruct_scorecard, count_frequencies and common_entries used to build per-comparison quadrants and cross-comparison occurrence reports) — github.com/m89p067/Scorecard
- **Pandas** (Loads the preprocessed Dataset 1 CSV of log2 fold-changes and adjusted p-values into a DataFrame consumed by the scorecard functions)
- **Python** (Language in which the scorecard library is written and in which the aggregation workflow is scripted)

## Examples

```
import scorecard_functions; df=scorecard_functions.data_loading(param_dict); scorecard_functions.scorecard(df,param_dict); scorecard_functions.reconstruct_scorecard(df,param_dict); scorecard_functions.count_frequencies(main_folder); scorecard_functions.common_entries(main_folder)
```

## Evaluation signals

- Each comparison sub-folder under main_folder contains four populated quadrant outputs plus a reconstructed scorecard image
- count_frequencies output shows non-zero entry counts for the expected quadrants (e.g. VAN comparisons dominated by Q3 down-regulated entries)
- common_entries report includes the target loci with their total occurrences, e.g. SAUSA300_RS03465 (GraS/ApsS) flagged as markedly downregulated in VAN vs CPT and LZA
- Frequency heatmap (quadrants_heatmap) renders without errors, confirming consistent folder structure across comparisons
- Final gene findings CSV (e.g. dataset1_gene_findings_report.csv) exists and rows match entries extracted from the quadrant/occurrence reports

## Limitations

- Requires pre-computed fold-change and adjusted p-value columns in the CSV with a matching header; the tool does not compute differential expression itself
- Setting the 'multiplication factor' to 1 produces a four-way plot instead of a scorecard, removing the extreme/moderate value stratification
- No version number is stated for the Python library or Python itself, and the discussion section omits the repository URL, which can hinder reproducibility
- Scorecards can become crowded with many entries, so individual quadrant plots should be kept for inspection; volcano plots similarly become crowded when minor differences reach significance with large samples

## Evidence

- [results] six comparisons were made applying the scorecard and setting fold-change thresholds between 2 and 3 (that is, a multiplication factor of 1.5): "six comparisons were made applying the scorecard and setting fold-change thresholds between 2 and 3 (that is, a multiplication factor of 1.5) and a significance level of"
- [results] the comparisons involving VAN mainly identified markedly down-regulated genes in Q3; in contrast, the other antibiotic-related genes were markedly over-expressed: "the comparisons involving VAN mainly identified markedly down-regulated genes in Q3; in contrast, the other antibiotic-related genes were markedly over-expressed"
- [readme] This function will store statistically significant entries for each quadrant together with plots of individual quandrants.: "This function will store statistically significant entries for each quadrant together with plots of individual quandrants."
- [readme] Remember to call it after *count_frequencies* because it re-uses information previously created by this function. Output file could be in CSV or Excel format showing total occurences.: "Remember to call it after *count_frequencies* because it re-uses information previously created by this function. Output file could be in CSV or Excel format showing total occurences."
- [methods] param_dict=scorecard_functions.generate_parameters(): "param_dict=scorecard_functions.generate_parameters()"
