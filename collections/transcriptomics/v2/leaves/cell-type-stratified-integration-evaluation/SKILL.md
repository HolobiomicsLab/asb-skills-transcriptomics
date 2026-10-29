---
name: cell-type-stratified-integration-evaluation
description: Use when benchmarking scRNA-seq integration methods on datasets with
  imbalanced cell type proportions, where a cell type-aware batch-mixing metric (rather
  than plain iLISI) is needed to score batch mixing per cell type.
when_to_use_negative:
- Cell type annotations are unavailable — 'One obvious limitation of CiLISI and similar
  metrics is that it requires cell type annotations.'
- Evaluating mixing of technical replicates without meaningful cell type structure,
  where cell type-agnostic batch mixing metrics suffice
- A cell type is present in only one dataset — CiLISI cannot be calculated and that
  type must be dropped
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3272
tools:
- name: scIntegrationMetrics
  role: Reference R implementation of LISI, per-cell type LISI (CiLISI) and ASW used
    to compute and validate the batch-mixing scores
  repo: https://github.com/carmonalab/scIntegrationMetrics
- name: scib-pipeline
  role: Snakemake benchmarking pipeline that applies CiLISI (via scIntegrationMetrics)
    to score (semi-)supervised integration methods such as STACAS
  repo: https://github.com/carmonalab/scib-pipeline
- name: LISI
  role: Upstream algorithm (immunogenomics/LISI) on which scIntegrationMetrics builds
    to compute per-cell-type iLISI
  repo: https://github.com/immunogenomics/LISI
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-45240-z
    title: Semi-supervised integration of single-cell transcriptomics data
  source_content_hash: sha256:cf8beea1984e059c728a93576bd2b6cd992b77fd0faf1bfefe59a573ed781638
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/scIntegrationMetrics
  tools:
  - LISI
  - scIntegrationMetrics
  - scib-pipeline
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# cell-type-stratified-integration-evaluation

## Summary

Evaluates scRNA-seq batch integration quality using CiLISI, a per-cell-type normalized iLISI that measures how well cells of the same type mix across batches, computed with the scIntegrationMetrics R package. It corrects the inadequacy of cell type-agnostic iLISI in integration tasks with cell type imbalance.

## When to use

Use when benchmarking scRNA-seq integration methods on datasets with imbalanced cell type proportions, where a cell type-aware batch-mixing metric (rather than plain iLISI) is needed to score batch mixing per cell type.

## When NOT to use

- Cell type annotations are unavailable — 'One obvious limitation of CiLISI and similar metrics is that it requires cell type annotations.'
- Evaluating mixing of technical replicates without meaningful cell type structure, where cell type-agnostic batch mixing metrics suffice
- A cell type is present in only one dataset — CiLISI cannot be calculated and that type must be dropped

## Inputs

- Integrated single-cell dataset (e.g. Seurat object) with batch origin and cell type annotations in metadata
- Low-dimensional embedding produced by the integration method (e.g. STACAS-integrated space)
- Simulated single-cell integration datasets with known batch and cell type labels (for validation)

## Outputs

- Per-cell-type normalized iLISI (CiLISI) scores rescaled to [0,1]
- Table of per-cell-type batch-mixing scores (e.g. cilisi_scores.csv)
- Comparison against scIntegrationMetrics reference batch-mixing values

## How to apply

Run LISI independently on the subset of cells of each cell type present in both datasets, with perplexity set to 30, on the integrated embedding containing batch-origin and cell type annotations. Min-max rescale each per-cell-type iLISI score to the [0,1] interval and drop cell types occurring in only one dataset, as required by the CiLISI definition. Use the scIntegrationMetrics package (which extends immunogenomics/LISI with per-cell type metrics and normalization options) as the reference implementation, and compare a reimplementation against its output within a small numerical tolerance. Interpret scores near 1 as good mixing of same-type cells across batches and scores near 0 as failure to mix; write per-cell-type scores (e.g. cilisi_scores.csv) for downstream benchmark comparison as done in the scib-pipeline.

## Related tools

- **scIntegrationMetrics** (Reference R implementation of LISI, per-cell type LISI (CiLISI) and ASW used to compute and validate the batch-mixing scores) — https://github.com/carmonalab/scIntegrationMetrics
- **scib-pipeline** (Snakemake benchmarking pipeline that applies CiLISI (via scIntegrationMetrics) to score (semi-)supervised integration methods such as STACAS) — https://github.com/carmonalab/scib-pipeline
- **LISI** (Upstream algorithm (immunogenomics/LISI) on which scIntegrationMetrics builds to compute per-cell-type iLISI) — https://github.com/immunogenomics/LISI

## Evaluation signals

- Reimplemented CiLISI values match scIntegrationMetrics package output within a small numerical tolerance on the simulated datasets
- Each per-cell-type iLISI score is min-max rescaled to lie within [0,1]
- Cell types occurring in only one dataset are excluded from the output per the CiLISI definition
- Scores tend to 1 when same-type cells are well mixed across batches and 0 when cells from different batches do not mix
- Output table (cilisi_scores.csv) contains one score per retained cell type with perplexity 30 used in LISI

## Limitations

- CiLISI and similar metrics require cell type annotations
- CiLISI cannot be calculated for cell types that are only present in one dataset
- As LISI, CiLISI remains suboptimal when datasets have highly unequal numbers of cells
- Batch mixing metrics that are more robust to lopsided cell proportions are still lacking

## Evidence

- [other] CiLISI is a normalized batch LISI computed on a per-cell type basis: it tends to 1 when cells of the same type are well mixed across batches and becomes zero when cells from different batches do not mix.: [evidence span withheld — anchor status: not_found]
- [other] For each cell type present in both datasets, run the LISI algorithm independently on the subset of cells of that type with perplexity set to 30.: [evidence span withheld — anchor status: not_found]
- [discussion] One obvious limitation of CiLISI and similar metrics is that it requires cell type annotations.: "One obvious limitation of CiLISI and similar metrics is that it requires cell type annotations."
- [readme] A collection of metrics to evaluate scRNA-seq data integration quality, including LISI, per-cell type LISI (CiLISI), and silhouette average width (ASW).: [evidence span withheld — anchor status: external_doc]
- [readme] It includes our new integration metric CiLISI we computed with our scIngrationMetrics R package.: [evidence span withheld — anchor status: external_doc]
