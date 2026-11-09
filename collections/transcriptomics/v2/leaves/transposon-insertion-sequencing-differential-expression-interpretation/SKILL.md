---
name: transposon-insertion-sequencing-differential-expression-interpretation
description: Use when analyzing pre-computed log2 fold-change and adjusted p-value
  columns from bacterial transposon insertion sequencing or RNA-seq datasets with
  multiple antibiotic treatments, and you need a data-driven, quadrant-based summary
  of extreme gene changes.
when_to_use_negative:
- Input is raw sequencing data or differential analysis has not yet been performed
  — the scorecard requires pre-computed fold-change and adjusted p-value columns in
  the CSV
- You need standard four-way plots without statistical-significance filtering or cross-comparison
  merging (set 'multiplication factor' to 1 produces a four-way plot, defeating the
  scorecard's purpose)
- The dataset has fewer than two pairwise treatment-vs-control comparisons, since
  the scorecard's aggregation functions (multiple_view, count_frequencies, common_entries)
  assume a main folder of multiple comparison subfolders
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scorecard_functions
  role: Python library providing generate_parameters, data_loading, scorecard, reconstruct_scorecard,
    count_frequencies, and common_entries to build and aggregate quadrant scorecards
    from Dataset 1 comparisons
  repo: https://github.com/m89p067/Scorecard
- name: Pandas
  role: Loads the preprocessed CSV of log2 fold-changes and adjusted p-values into
    a DataFrame consumed by the scorecard functions
- name: TRANSIT
  role: Upstream tool whose resampling method with beta-geometric normalization produced
    the log2 fold-changes and Benjamini-Hochberg-corrected p-values for dataset 1
- name: Python
  role: Language in which the scorecard library is scripted and executed for the SAUSA300
    Tn-seq analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s00284-025-04435-3
    title: 'A Scorecard for Information Synthesis in Multiple Experimental Conditions:
      Application to Bacterial Biofilm Matrix Transcriptomics'
  source_content_hash: sha256:943e4a13ef1bf976e4410e86ee5c55d0c4b614c7eefaecc1f25197b443a4766f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/m89p067/Scorecard
  tools:
  - Pandas
  - Python
  - TRANSIT
  - scorecard_functions
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# transposon-insertion-sequencing-differential-expression-interpretation

## Summary

Interpret differential abundance/expression from Staphylococcus aureus USA300 (SAUSA300) transposon insertion sequencing data (Dataset 1, VAN/LZD/CPT/DEL) using the Python Scorecard library, which groups genes by fold-change magnitude and statistical significance across pairwise comparisons instead of preselecting genes.

## When to use

Use when analyzing pre-computed log2 fold-change and adjusted p-value columns from bacterial transposon insertion sequencing or RNA-seq datasets with multiple antibiotic treatments, and you need a data-driven, quadrant-based summary of extreme gene changes.

## When NOT to use

- Input is raw sequencing data or differential analysis has not yet been performed — the scorecard requires pre-computed fold-change and adjusted p-value columns in the CSV
- You need standard four-way plots without statistical-significance filtering or cross-comparison merging (set 'multiplication factor' to 1 produces a four-way plot, defeating the scorecard's purpose)
- The dataset has fewer than two pairwise treatment-vs-control comparisons, since the scorecard's aggregation functions (multiple_view, count_frequencies, common_entries) assume a main folder of multiple comparison subfolders

## Inputs

- Preprocessed Dataset 1 CSV with a SAUSA300 locus-tag/gene-name column and per-comparison log2 fold-change and adjusted p-value columns (header row matching 'FC cond x'/'padj cond x' parameter names)
- Parameter dictionary from scorecard_functions.generate_parameters() with fold-change thresholds 2-3 (multiplication factor 1.5) and th_significance 0.05
- Main folder containing subfolders per experimental comparison (e.g., VAN vs control comparisons)

## Outputs

- Scorecard quadrant plots and reconstructed combined scorecard images per comparison
- JSON/quadrant entries of statistically significant genes per region
- count_frequencies and common_entries occurrence reports (common and uncommon entries) as CSV in save_dir
- Gene-level findings report (e.g., dataset1_gene_findings_report.csv) covering GraX, VraG, AgrB, RsbW quadrant behavior under VAN

## How to apply

Create the parameter dictionary with scorecard_functions.generate_parameters() and set fold-change thresholds between 2 and 3 (multiplication factor 1.5) with significance level p<=0.05, pointing 'gene_name' to the SAUSA300 locus-tag column and mapping 'FC cond x'/'padj cond x' entries to each VAN comparison's column names. Load the preprocessed Dataset 1 CSV (pre-computed log2 fold-change and adjusted p-values per comparison, computed via TRANSIT resampling with beta-geometric normalization and Benjamini-Hochberg correction) into a Pandas DataFrame via data_loading(param_dict). Run scorecard(df, param_dict) per vancomycin-related comparison to store statistically significant quadrant entries and individual quadrant plots, then reconstruct_scorecard(df, param_dict) for combined plots. Aggregate across all comparison subfolders with count_frequencies(main_folder) and common_entries(main_folder) to produce total-occurrence reports of common and uncommon entries, then extract quadrant membership for the Gra/Aps loci (GraX/ApsX down-regulated on the Q3 side under VAN, AgrB differential, RsbW captured) and compile a gene-level findings report saved as CSV in save_dir. The rationale is that the scorecard, unlike four-way plots, filters by statistical significance and merges information across multiple comparisons.

## Related tools

- **scorecard_functions** (Python library providing generate_parameters, data_loading, scorecard, reconstruct_scorecard, count_frequencies, and common_entries to build and aggregate quadrant scorecards from Dataset 1 comparisons) — https://github.com/m89p067/Scorecard
- **Pandas** (Loads the preprocessed CSV of log2 fold-changes and adjusted p-values into a DataFrame consumed by the scorecard functions)
- **TRANSIT** (Upstream tool whose resampling method with beta-geometric normalization produced the log2 fold-changes and Benjamini-Hochberg-corrected p-values for dataset 1)
- **Python** (Language in which the scorecard library is scripted and executed for the SAUSA300 Tn-seq analysis)

## Examples

```
import scorecard_functions; param_dict=scorecard_functions.generate_parameters(); df=scorecard_functions.data_loading(param_dict); scorecard_functions.scorecard(df,param_dict); scorecard_functions.reconstruct_scorecard(df,param_dict)
```

## Evaluation signals

- VAN-involving comparisons predominantly place genes in the down-regulated Q3 quadrant, whereas non-VAN antibiotic comparisons show markedly over-expressed genes (e.g., SAUSA300_RS03525, SAUSA300_RS04560, SAUSA300_RS08475)
- SAUSA300_RS03465 (histidine kinase GraS/ApsS) appears markedly downregulated in VAN versus CPT and LZA with negative enhancement under vancomycin
- GraX/ApsX loci fall in extreme VAN quadrants (down-regulated Q3 side), AgrB is differential in all VAN quorum-sensing comparisons, and RsbW is captured in the occurrence reports
- Reconstructed scorecard images exist for all comparison subfolders and count_frequencies/common_entries reports are produced after (not before) all quadrant runs, with common_entries re-using count_frequencies output
- Input CSV header column names exactly match the 'FC cond x'/'padj cond x' strings in param_dict, and the gene_name key points to the SAUSA300 locus-tag column

## Limitations

- The comparison is limited to the conditions in Dataset 1 (VAN, LZD, CPT, DEL); thresholds were specifically set at fold-change 2-3 (multiplication factor 1.5) and p<=0.05 — other datasets used different thresholds (e.g., dataset 3: fold-change 2 and 4, factor 2, p<=0.001)
- The scorecard requires pre-computed fold-change and adjusted p-value columns; it does not perform the differential analysis itself
- Setting 'multiplication factor' to 1 collapses the scorecard into a standard four-way plot, losing the extreme-variation filtering; typical four-way plots are not filtered by statistical significance

## Evidence

- [results] six comparisons were made applying the scorecard and setting fold-change thresholds between 2 and 3 (that is, a multiplication factor of 1.5) and a significance level of: "six comparisons were made applying the scorecard and setting fold-change thresholds between 2 and 3 (that is, a multiplication factor of 1.5) and a significance level of"
- [results] the comparisons involving VAN mainly identified markedly down-regulated genes in Q3; in contrast, the other antibiotic-related genes were markedly over-expressed: "the comparisons involving VAN mainly identified markedly down-regulated genes in Q3; in contrast, the other antibiotic-related genes were markedly over-expressed"
- [methods] param_dict=scorecard_functions.generate_parameters(): "param_dict=scorecard_functions.generate_parameters()"
- [readme] This function will store statistically significant entries for each quadrant together with plots of individual quandrants.: "This function will store statistically significant entries for each quadrant together with plots of individual quandrants."
- [readme] To create a final report on single genes/entries run the following function. Remember to call it after *count_frequencies* because it re-uses information previously created by this function.: "To create a final report on single genes/entries run the following function. Remember to call it after *count_frequencies* because"
