---
name: single-cell-multi-table-correspondence-analysis
description: Use when multiple scRNA-seq count tables (batches, samples, or platforms)
  sharing a common gene feature space must be integrated into a joint low-dimensional
  embedding, especially when PCA+Harmony shows poor batch mixing.
when_to_use_negative:
- When there is only a single count table with no batches to integrate — use corral
  (single-table CA for dimensionality reduction) instead of corralm
- When the batches do not share a comparable gene feature space and cannot be restricted
  to an intersecting gene set
- When the goal is independent cluster-accuracy benchmarking (ARI) on complex tissues
  — the article notes ground-truth datasets like SCMixology are simple pseudo-cell
  mixtures with unvalidated annotations
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: corralm
  role: Multi-table correspondence analysis function that computes aligned embeddings
    for batch integration across multiple SingleCellExperiment tables
  repo: github.com/laurenhsu1/corral
- name: corral
  role: Underlying single-table CA package; corralm is its multi-table counterpart
    for alignment and batch integration
  repo: github.com/laurenhsu1/corral
- name: Harmony
  role: Batch-correction baseline run on both corralm and PCA embeddings to compare
    batch-mixing performance
- name: SingleCellExperiment
  role: Bioconductor container class that corral interfaces with directly to hold
    the multi-table batch data
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:fb185801ae6a737befbf651893f82b8b6a016d4ad8785b192ded8fbf7b71a62d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - Harmony
  - R
  - SingleCellExperiment
  - corral
  - corralm
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# single-cell-multi-table-correspondence-analysis

## Summary

Apply corralm, the multi-table correspondence analysis routine from the corral R/Bioconductor package, to align and batch-integrate SingleCellExperiment scRNA-seq tables (e.g., the SCMixology 10X/Celseq2/Dropseq batches) as a CA-based substitute for PCA embeddings in batch-correction pipelines.

## When to use

Use when multiple scRNA-seq count tables (batches, samples, or platforms) sharing a common gene feature space must be integrated into a joint low-dimensional embedding, especially when PCA+Harmony shows poor batch mixing.

## When NOT to use

- When there is only a single count table with no batches to integrate — use corral (single-table CA for dimensionality reduction) instead of corralm
- When the batches do not share a comparable gene feature space and cannot be restricted to an intersecting gene set
- When the goal is independent cluster-accuracy benchmarking (ARI) on complex tissues — the article notes ground-truth datasets like SCMixology are simple pseudo-cell mixtures with unvalidated annotations

## Inputs

- Multiple single-cell RNA-seq count matrices as SingleCellExperiment objects (e.g., CellBench SCMixology batches from 10X, Celseq2, and Dropseq platforms)
- A common feature space: the intersecting gene set shared across all batches
- Batch labels for each cell, needed for downstream Harmony comparison and scaled-variance (SV) evaluation

## Outputs

- Integrated low-dimensional cell embeddings across batches from corralm
- Harmony-corrected embeddings computed on the corralm embeddings (and optionally on PCA embeddings as baseline)
- Scaled variance (SV) batch-mixing scores per method, e.g., written to sv_integration_results.csv with columns method and SV score per batch/condition

## How to apply

Load the per-batch scRNA-seq count matrices (e.g., CellBench SCMixology 10X, Celseq2, Dropseq) as SingleCellExperiment objects and restrict all batches to the intersecting gene set so the tables share a common feature space. Call corralm on the multi-table structure to compute integrated cell embeddings via multi-table correspondence analysis, which decomposes chi-squared residuals rather than requiring a log(x + 1) transformation followed by PCA. As a comparison arm, compute PCA embeddings of the same feature-intersected data and optionally run Harmony on both the corralm and PCA embeddings. Quantify batch mixing with the scaled variance (SV) metric; SV values near 1 indicate successful cross-batch integration, while SV values far from 1 (as seen for Harmony on PCA embeddings) indicate unsuccessful integration. Write the method-by-SV comparison (e.g., sv_integration_results.csv) to record which embedding achieved good batch integration.

## Related tools

- **corralm** (Multi-table correspondence analysis function that computes aligned embeddings for batch integration across multiple SingleCellExperiment tables) — github.com/laurenhsu1/corral
- **corral** (Underlying single-table CA package; corralm is its multi-table counterpart for alignment and batch integration) — github.com/laurenhsu1/corral
- **Harmony** (Batch-correction baseline run on both corralm and PCA embeddings to compare batch-mixing performance)
- **SingleCellExperiment** (Bioconductor container class that corral interfaces with directly to hold the multi-table batch data)

## Examples

```
devtools::install_github('laurenhsu1/corral'); sce_list <- list(tenx=sce_10x, celseq2=sce_celseq2, dropseq=sce_dropseq); corralm(sce_list)
```

## Evaluation signals

- Scaled variance (SV) for the corralm and corralm+Harmony embeddings should approach 1, indicating good batch integration
- SV values far from 1 (as observed for Harmony with PCA embeddings on SCMixology) indicate unsuccessful cross-batch integration
- All batches must share the same intersecting gene set before running corralm; verify feature-space equality across tables
- The output embeddings should be usable downstream (clustering/UMAP) via corral's SingleCellExperiment reducedDim interface
- The comparison table (method, SV per condition) should contain one row per embedding method tested: corralm, corralm+Harmony, and PCA+Harmony

## Limitations

- SV was assessed on SCMixology, a simple pseudo-cell mixture dataset with well-defined cell clusters that does not reflect real-world tissue complexity
- No tool or package versions for corral or its dependencies are reported
- No accession IDs or links for the SCMixology benchmarking dataset are provided in the discussion
- Whether corralm generalizes as well to complex-tissue datasets is untested in this evaluation

## Evidence

- [abstract] corralm for integrative multi-table dimension reduction of scRNAseq data: "we introduce corralm for integrative multi-table dimension reduction of scRNAseq data"
- [results] On SCMixology, corralm and Harmony with corralm achieved good batch integration, outperforming Harmony with PCA whose SV values were far from 1: [evidence span withheld — anchor status: not_found]
- [abstract] corral is a Bioconductor package that interfaces directly with single cell classes: "We implement CA for scRNAseq data in corral, an R/Bioconductor package which interfaces directly with single cell classes in Bioconductor."
- [readme] The package contains two main function calls: corral for single-table dimensionality reduction, corralm for multiple tables: "1. `corral` for dimensionality reduction on a single table 
2. `corralm` for alignment and batch integration of multiple tables"
- [readme] corral can be used on a single table or for integration across batches, samples, and modes: "It can be used on a single table for dimensionality reduction, or it can be used for integration across batches, samples, and modes."
- [abstract] CA is based on decomposition of a chi-squared residual matrix, avoiding distortive log-transformation: "CA is based on decomposition of a chi-squared residual matrix, avoiding distortive log-transformation"
