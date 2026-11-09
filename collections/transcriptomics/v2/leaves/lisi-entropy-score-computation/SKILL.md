---
name: lisi-entropy-score-computation
description: Use when evaluating scRNA-seq integration quality on datasets with batch
  origin and cell type annotations, especially when cell type imbalance across batches
  makes standard cell type-agnostic iLISI mis.
when_to_use_negative:
- Datasets lack cell type annotations — CiLISI and similar metrics require cell type
  labels
- A cell type is present in only one dataset — CiLISI cannot be calculated for it
  and the type must be dropped
- Datasets have highly unequal numbers of cells — LISI/CiLISI remains suboptimal there
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scIntegrationMetrics
  role: Provides the per-cell type LISI (CiLISI), iLISI, and ASW metrics with normalization
    options, Seurat compatibility, and per-subset computation used for batch-mixing
    scoring
  repo: https://github.com/carmonalab/scIntegrationMetrics
- name: LISI
  role: Base immunogenomics LISI implementation upon which scIntegrationMetrics builds
    to add per-cell type metrics
  repo: https://github.com/immunogenomics/LISI
- name: scib-pipeline
  role: Snakemake benchmarking pipeline in which CiLISI, computed via scIntegrationMetrics,
    is used as an integration metric for (semi-)supervised integration methods
  repo: https://github.com/carmonalab/scib-pipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-45240-z
    title: Semi-supervised integration of single-cell transcriptomics data
  source_content_hash: sha256:e5d79983be17ea93cfa65f9fe18d3735e06415f9fbd06ed8c75a3d1f947177fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/scIntegrationMetrics
  tools:
  - LISI
  - scIntegrationMetrics
  - scib-pipeline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# lisi-entropy-score-computation

## Summary

Computes batch-mixing scores based on the LISI (Local Inverse Simpson Index) family, including the cell type-aware CiLISI implemented in the scIntegrationMetrics R package, to quantify how well batches are mixed in integrated scRNA-seq data while accounting for cell type composition.

## When to use

Use when evaluating scRNA-seq integration quality on datasets with batch origin and cell type annotations, especially when cell type imbalance across batches makes standard cell type-agnostic iLISI misleading.

## When NOT to use

- Datasets lack cell type annotations — CiLISI and similar metrics require cell type labels
- A cell type is present in only one dataset — CiLISI cannot be calculated for it and the type must be dropped
- Datasets have highly unequal numbers of cells — LISI/CiLISI remains suboptimal there

## Inputs

- Integrated/batch-corrected scRNA-seq datasets (e.g. Seurat objects or embeddings) with batch origin labels
- Cell type annotations for all cells (e.g. from scGate or semi-supervised integration priors)
- Perplexity parameter for LISI (default 30 per the source workflow)

## Outputs

- Per-cell-type CiLISI scores rescaled to [0,1] (e.g. cilisi_scores.csv)
- Batch-mixing quality assessment of the integrated embedding

## How to apply

Load the integrated (or batch-corrected) single-cell datasets with batch origin and cell type labels, e.g. as Seurat objects, which scIntegrationMetrics supports seamlessly. For each cell type present in both datasets, run the LISI algorithm independently on that cell type's cells with perplexity 30, then min-max rescale each per-cell-type iLISI score to [0,1]; CiLISI tends to 1 when cells of a type are well mixed across batches and 0 when they do not mix. Drop cell types that occur in only one dataset, as CiLISI cannot be calculated for them. Optionally compare against a reference batch-mixing score and write per-cell-type scores to a table such as cilisi_scores.csv; the reimplemented values must match scIntegrationMetrics output within a small numerical tolerance.

## Related tools

- **scIntegrationMetrics** (Provides the per-cell type LISI (CiLISI), iLISI, and ASW metrics with normalization options, Seurat compatibility, and per-subset computation used for batch-mixing scoring) — https://github.com/carmonalab/scIntegrationMetrics
- **LISI** (Base immunogenomics LISI implementation upon which scIntegrationMetrics builds to add per-cell type metrics) — https://github.com/immunogenomics/LISI
- **scib-pipeline** (Snakemake benchmarking pipeline in which CiLISI, computed via scIntegrationMetrics, is used as an integration metric for (semi-)supervised integration methods) — https://github.com/carmonalab/scib-pipeline

## Evaluation signals

- Reimplemented CiLISI values match the scIntegrationMetrics package output within a small numerical tolerance
- Each per-cell-type iLISI score lies in the [0,1] interval after min-max rescaling
- Cell types occurring in only one dataset are excluded from the output per the CiLISI definition
- Well-mixed cells of a given type across batches yield scores near 1; non-mixing batches yield scores near 0
- Per-cell-type scores are written to the expected output file (e.g. cilisi_scores.csv) with one entry per retained cell type

## Limitations

- CiLISI and similar metrics require cell type annotations
- CiLISI cannot be calculated for cell types that are only present in one dataset
- As with LISI, CiLISI remains suboptimal when datasets have highly unequal numbers of cells
- Batch mixing metrics more robust to lopsided cell proportions are still lacking

## Evidence

- [results] CiLISI is a normalized batch LISI computed on a per-cell type basis: it tends to 1 when cells of the same type are well mixed across batches and becomes zero when cells from different batches do not mix.: [evidence span withheld — anchor status: not_found]
- [readme] scIntegrationMetrics README description: [evidence span withheld — anchor status: external_doc]
- [readme] README on package capabilities: [evidence span withheld — anchor status: external_doc]
- [discussion] Discussion limitation on annotations: "One obvious limitation of CiLISI and similar metrics is that it requires cell type annotations."
- [discussion] Discussion limitation on single-dataset cell types: "Another limitation is that CiLISI cannot be calculated for cell types that are only present in one dataset."
