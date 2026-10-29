---
name: scaled-variance-batch-mixing-assessment
description: Use when you need to score or compare batch integration of multiple embedding
  methods (e.g., corralm vs. PCA+Harmony) on multi-batch scRNA-seq data, such as the
  SCMixology 10X/Celseq2/Dropseq CellBench dataset.
when_to_use_negative:
- The data are a single batch/table with no cross-batch structure to mix — corral
  (single-table CA) and no SV mixing assessment is needed
- The goal is cluster recovery or cell-type annotation accuracy rather than batch
  integration — the article evaluates that separately with ARI, and ARI may even penalize
  methods that better distinguish sub-populations
- Embeddings are not aligned across batches (e.g., raw per-batch PCA), since SV would
  measure meaningless mixing on incomparable coordinate systems
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: corralm
  role: Generates the multi-table correspondence-analysis embeddings whose batch integration
    is assessed with SV; also serves as the embedding input to Harmony for the corralm+Harmony
    arm of the comparison
  repo: github.com/laurenhsu1/corral
- name: corral
  role: R/Bioconductor package providing corral and corralm for dimensionality reduction
    and batch integration of single-cell count data
  repo: github.com/laurenhsu1/corral
- name: Harmony
  role: Batch-correction baseline applied to both corralm and PCA embeddings so that
    their SV integration scores can be compared
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:eeb50a26a5e940aa4e9c4706a862b5b86bc9b59a61a6dfbc7b517af2d2aa0984
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - Harmony
  - R
  - corral
  - corralm
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# scaled-variance-batch-mixing-assessment

## Summary

Quantifies batch integration quality of single-cell embeddings with the scaled variance (SV) batch-mixing metric, where SV close to 1 indicates successful cross-batch mixing. In Hsu & Culhane (2023) it was used to show that corralm and Harmony-on-corralm embeddings outperform Harmony on PCA embeddings for the SCMixology three-platform dataset.

## When to use

Use when you need to score or compare batch integration of multiple embedding methods (e.g., corralm vs. PCA+Harmony) on multi-batch scRNA-seq data, such as the SCMixology 10X/Celseq2/Dropseq CellBench dataset.

## When NOT to use

- The data are a single batch/table with no cross-batch structure to mix — corral (single-table CA) and no SV mixing assessment is needed
- The goal is cluster recovery or cell-type annotation accuracy rather than batch integration — the article evaluates that separately with ARI, and ARI may even penalize methods that better distinguish sub-populations
- Embeddings are not aligned across batches (e.g., raw per-batch PCA), since SV would measure meaningless mixing on incomparable coordinate systems

## Inputs

- Multi-batch scRNA-seq count matrices (e.g., SCMixology CellBench 10X, Celseq2, Dropseq) as SingleCellExperiment-based multi-table structures
- Feature-intersected batches sharing a common gene space
- Cell embeddings from each integration method (corralm, corralm+Harmony, PCA+Harmony) including batch labels per cell

## Outputs

- Scaled variance (SV) score per method per batch/condition
- sv_integration_results.csv containing the method-vs-SV comparison table

## How to apply

Load the multi-batch scRNA-seq count data (e.g., SCMixology CellBench 10X, Celseq2, and Dropseq batches) into SingleCellExperiment-based multi-table structures in R/Bioconductor, restricting all batches to the intersecting gene set so they share a common feature space. Generate the embeddings to be compared — here corralm (multi-table correspondence analysis via the corralm call), corralm followed by Harmony, and PCA followed by Harmony. Compute the scaled variance (SV) metric on each embedding: SV values near 1 indicate good batch mixing, while SV values far from 1 indicate unsuccessful cross-batch integration. Tabulate SV per method and batch/condition and write the comparison (method, SV score per batch/condition) to sv_integration_results.csv so the integration methods can be ranked.

## Related tools

- **corralm** (Generates the multi-table correspondence-analysis embeddings whose batch integration is assessed with SV; also serves as the embedding input to Harmony for the corralm+Harmony arm of the comparison) — github.com/laurenhsu1/corral
- **corral** (R/Bioconductor package providing corral and corralm for dimensionality reduction and batch integration of single-cell count data) — github.com/laurenhsu1/corral
- **Harmony** (Batch-correction baseline applied to both corralm and PCA embeddings so that their SV integration scores can be compared)

## Examples

```
devtools::install_github('laurenhsu1/corral')
```

## Evaluation signals

- SV values for well-integrated embeddings (corralm, corralm+Harmony) fall close to 1
- Poorly integrated baselines (Harmony on PCA embeddings) yield SV values far from 1 in this benchmark
- sv_integration_results.csv exists and contains one row per method with SV scores per batch/condition
- All compared embeddings were computed on the same gene-intersected feature space across the 10X, Celseq2, and Dropseq batches
- Each method's SV is computed on embeddings covering all three batches, so the metric reflects cross-platform mixing

## Limitations

- SCMixology and Zhengmix are by design simpler than data from complex tissues, comprising well-defined cell clusters, so SV results may not transfer to complex-tissue benchmarks
- Ground-truth labels in these benchmarks come from the original dataset publications and were not independently validated
- The article reports no tool or package versions for corral or dependencies, and no accession IDs or links for SCMixology/Zhengmix, complicating exact reproduction

## Evidence

- [abstract] we introduce corralm for integrative multi-table dimension reduction of scRNAseq data: "we introduce corralm for integrative multi-table dimension reduction of scRNAseq data"
- [abstract] Switching from PCA to CA is achieved through a simple pipeline substitution and improves dimension reduction of scRNAseq datasets.: "Switching from PCA to CA is achieved through a simple pipeline substitution and improves dimension reduction of scRNAseq datasets."
- [discussion] Except for SCMixology and Zhengmix (both comprising well-defined cell clusters and by design simpler than data from complex tissues): "Except for SCMixology and Zhengmix (both comprising well-defined cell clusters and by design simpler than data from complex tissues)"
- [readme] 2. corralm for alignment and batch integration of multiple tables: "2. `corralm` for alignment and batch integration of multiple tables"
