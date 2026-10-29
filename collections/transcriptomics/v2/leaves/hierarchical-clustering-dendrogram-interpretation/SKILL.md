---
name: hierarchical-clustering-dendrogram-interpretation
description: 'Use when normalized microarray intensity samples need QC for systematic
  batch effects: the analysis goal is to see whether samples group by date, operator,
  or reagent lot rather than by response class before downstream testing.'
when_to_use_negative:
- Data are raw, unnormalized intensities — distances would reflect technical scale
  differences rather than sample biology; normalize first (quantile or LOESS).
- Data are RNA-seq counts rather than microarray intensities — this PIMENTo QC step
  targets microarray expression data.
- A batch effect is already confirmed and needs statistical correction rather than
  detection — use limma's batch-effect removal step instead of re-visualizing.
edam_operation: http://edamontology.org/operation_0315
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PIMENTo
  role: Generates the dendrogram plot of all chosen samples from Euclidean distances
    and Ward's clustering for batch-effect QC
  repo: https://www.github.com/MUSC-CGM/PIMENTo
- name: R
  role: Language in which PIMENTo's dendrogram generation and clustering are implemented
- name: RStudio
  role: Open-source IDE used to run PIMENTo commands with autocomplete capability
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/978-1-4939-9004-7_11
    title: Pipeline for Integrated Microarray Expression Normalization Tool Kit (PIMENTo)
      for Tumor Microarray Profiling Experiments
  source_content_hash: sha256:b815dfddeeed4dd4de2429a3d00161f449ecaf5d49e6875e75711d518d70fcd9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - PIMENTo
  - R
  - RStudio
  - affy
  - limma
  edam_operation: http://edamontology.org/operation_0315
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Hierarchical clustering dendrogram interpretation for microarray batch-effect QC

## Summary

Generate and interpret a sample-dendrogram (Euclidean distances + Ward's method) over normalized microarray samples in PIMENTo, judging whether samples cluster by biological response class or by technical covariates such as processing date, operator, or reagent lot.

## When to use

Use when normalized microarray intensity samples need QC for systematic batch effects: the analysis goal is to see whether samples group by date, operator, or reagent lot rather than by response class before downstream testing.

## When NOT to use

- Data are raw, unnormalized intensities — distances would reflect technical scale differences rather than sample biology; normalize first (quantile or LOESS).
- Data are RNA-seq counts rather than microarray intensities — this PIMENTo QC step targets microarray expression data.
- A batch effect is already confirmed and needs statistical correction rather than detection — use limma's batch-effect removal step instead of re-visualizing.

## Inputs

- Normalized microarray intensity matrix (tab-separated, e.g., output of PIMENTo Preprocess Data on example_unnorm.txt with ntext=5, data.col=6:11)
- Sample labels and batch covariates (processing date, operator, reagent lot, response class)

## Outputs

- Dendrogram plot of all chosen samples (hierarchical clustering via Euclidean distance and Ward's method)

## How to apply

After preprocessing the tab-separated intensity file (e.g., example_unnorm.txt loaded via PIMENTo's Preprocess Data with ntext=5, data.col=6:11, symbol.index=2, id.index=1) and applying normalization (quantile via 'limma' or LOESS via 'affy'), PIMENTo generates a dendrogram plot of all chosen samples by calculating Euclidean distances between samples and clustering with Ward's method. Inspect the dendrogram visually: if branches group samples by processing date, operator, or reagent lot, a batch effect is present and should be handled (e.g., limma's batch-effect removal 'if necessary'); if clusters align with the sample response classes, the data are acceptable. In the paper's example, inspection of Fig. 3 showed clusters aligning with sample response classes, so no batch effect was present and analysis proceeded with quantile normalization. Judge correct application by checking that the dendrogram contains all chosen samples and that its clustering structure is consistent with known sample metadata rather than with batch covariates.

## Related tools

- **PIMENTo** (Generates the dendrogram plot of all chosen samples from Euclidean distances and Ward's clustering for batch-effect QC) — https://www.github.com/MUSC-CGM/PIMENTo
- **R** (Language in which PIMENTo's dendrogram generation and clustering are implemented)
- **RStudio** (Open-source IDE used to run PIMENTo commands with autocomplete capability)

## Evaluation signals

- The dendrogram includes exactly all chosen samples (no samples silently dropped).
- Cluster branches align with known sample response classes rather than with processing date, operator, or reagent lot — the paper's Fig. 3 pattern.
- Clustering reproducibly uses Euclidean distances between samples combined with Ward's method, not alternative linkage choices.
- If batch covariates do dominate the clustering, the expected follow-up is limma-based batch-effect removal before downstream analysis.
- Batch-effect QC conclusion is consistent with the MA-plot check that the chosen normalization (quantile for this dataset) yields data linear on the MA scale.

## Limitations

- Batch-effect judgment is a visual inspection of the dendrogram; the paper gives no numeric threshold for declaring a batch effect.
- Batch-effect removal is only performed 'if necessary' via limma; detection and correction are separate decisions.
- No version numbers are stated for software dependencies (limma, affy, R, Bioconductor), so exact clustering behavior may vary across versions.
- The promised R Shiny user interface was not yet available; use requires running R commands (e.g., via RStudio).

## Evidence

- [methods] It is evident upon inspection of Fig. 3 that there is no batch effect present in the samples, as the clusters align with the sample response classes.: "It is evident upon inspection of Fig. 3 that there is no batch effect present in the samples, as the clusters align with the sample response classes."
- [other] PIMENTo generates a dendrogram of all chosen samples by calculating Euclidean distances between samples and clustering with Ward's method.: [evidence span withheld — anchor status: not_found]
- [methods] "limma" is used for quantile normalization as well as removing batch effects, if necessary: [evidence span withheld — anchor status: not_found]
- [readme] PIMENTo is an R package to normalize, analyze, and visualize microarray data: "PIMENTo is an R package to normalize, analyze, and visualize microarray data"
