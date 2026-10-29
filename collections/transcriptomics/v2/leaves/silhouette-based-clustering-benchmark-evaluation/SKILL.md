---
name: silhouette-based-clustering-benchmark-evaluation
description: Use when comparing dimension-reduction or batch-integration methods for
  scRNA-seq (e.g., corralm vs PCA vs Seurat on multi-batch data) and you need a quantitative
  cell-type-preservation vs batch-mixing trade-off metric on cell embeddings.
when_to_use_negative:
- When the evaluation goal is label-transfer or ARI-based clustering accuracy against
  a reference — the article notes ARI may penalize methods that better distinguish
  sub-populations or rare cell types relative to reference labels
- When the input is raw counts rather than reduced-dimension embeddings and no dimension
  reduction has been run — silhouette must be computed on the method's embedding (e.g.,
  8 PCs/CA dimensions), not the expression matrix
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: corralm
  role: Produces the multi-table cell embeddings (e.g., with Freeman-Tukey residuals)
    whose silhouette scores are benchmarked; can be coupled with Harmony or MNN for
    the integration variants
  repo: https://github.com/laurenhsu1/corral
- name: corral
  role: R/Bioconductor package implementing CA for scRNA-seq dimensionality reduction
    that interfaces with SingleCellExperiment objects feeding the benchmark
  repo: https://github.com/laurenhsu1/corral
- name: Harmony
  role: Batch correction applied to corralm embeddings to generate the corralm+Harmony
    variant evaluated with ASW
- name: MNN
  role: Batch correction applied to corralm embeddings to generate the corralm+MNN
    variant evaluated with ASW
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:e6410785b612339e1630b32bc3e71463d4c2424425713b9c1805f0f2582d2d48
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/laurenhsu1/corral
  tools:
  - Bioconductor
  - Harmony
  - MNN
  - R
  - corral
  - corralm
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# silhouette-based-clustering-benchmark-evaluation

## Summary

Evaluates dimension-reduction and batch-integration embeddings for single-cell RNA-seq by computing average silhouette width (ASW) with cell-type labels and batch labels, trading off ASW_celltype against 1 - ASW_batch on the first 8 dimensions of each method's embedding. In the Hsu & Culhane corral paper this is how corralm with Freeman–Tukey residuals (alone and with Harmony/MNN) is compared against multibatch PCA and the Seurat routine on the Baron, Lawlor, and Muraro pancreas datasets.

## When to use

Use when comparing dimension-reduction or batch-integration methods for scRNA-seq (e.g., corralm vs PCA vs Seurat on multi-batch data) and you need a quantitative cell-type-preservation vs batch-mixing trade-off metric on cell embeddings.

## When NOT to use

- When the evaluation goal is label-transfer or ARI-based clustering accuracy against a reference — the article notes ARI may penalize methods that better distinguish sub-populations or rare cell types relative to reference labels
- When the input is raw counts rather than reduced-dimension embeddings and no dimension reduction has been run — silhouette must be computed on the method's embedding (e.g., 8 PCs/CA dimensions), not the expression matrix

## Inputs

- Cell embeddings (first 8 dimensions) from each method being benchmarked, e.g., corralm/CA-FT, corralm+Harmony, corralm+MNN, multibatch PCA
- Per-cell cell-type label vector (ground-truth annotations from original dataset publications, e.g., Baron/Lawlor/Muraro pancreas)
- Per-cell batch label vector (dataset/batch identity for each cell)

## Outputs

- ASW_celltype and 1 - ASWbatch per method on 8 dimensions
- Comparison table saved as asw_comparison.csv
- Scatter of ASW_celltype vs 1 - ASWbatch showing batch-integration vs cell-type-preservation trade-off (as in Fig. 4C-4D)

## How to apply

Take each method's embedding — here, the corralm Freeman–Tukey cell embeddings, optionally batch-corrected downstream with Harmony or MNN, plus multibatch PCA and the Seurat routine as baselines — and restrict to its first 8 dimensions. Compute average silhouette width twice per method: once using cell-type labels (ASW_celltype, higher is better) and once using batch labels, reported as 1 - ASW_batch (higher is better), so a good method sits toward the top-right of the ASW_celltype vs 1 - ASW_batch plane. Run this on the concatenated multi-batch SingleCellExperiment objects built from the Baron, Lawlor, and Muraro pancreas count matrices, which retain cell type and batch annotations. Tabulate ASW_celltype and 1 - ASWbatch per method and save the comparison table as asw_comparison.csv. A method (e.g., corralm-FT + Harmony) is judged successful when it matches the established Seurat routine in both batch integration and biological cluster separation.

## Related tools

- **corralm** (Produces the multi-table cell embeddings (e.g., with Freeman-Tukey residuals) whose silhouette scores are benchmarked; can be coupled with Harmony or MNN for the integration variants) — https://github.com/laurenhsu1/corral
- **corral** (R/Bioconductor package implementing CA for scRNA-seq dimensionality reduction that interfaces with SingleCellExperiment objects feeding the benchmark) — https://github.com/laurenhsu1/corral
- **Harmony** (Batch correction applied to corralm embeddings to generate the corralm+Harmony variant evaluated with ASW)
- **MNN** (Batch correction applied to corralm embeddings to generate the corralm+MNN variant evaluated with ASW)

## Examples

```
asw_ct <- summary(cluster::silhouette(celltype_labels, dist(embedding[, 1:8])))$avg.width; asw_batch <- summary(cluster::silhouette(batch_labels, dist(embedding[, 1:8])))$avg.width
```

## Evaluation signals

- ASW_celltype and 1 - ASWbatch are each computed on the first 8 dimensions of the embedding — using a different number of dimensions is a deviation from the protocol
- A batch-corrected method should push 1 - ASWbatch up (batch ASW down) without collapsing ASW_celltype; corralm-FT + Harmony achieving values comparable to Seurat on both axes is the paper's expected outcome
- All benchmarked methods (corralm-FT, corralm+Harmony, corralm+MNN, multibatch PCA, Seurat routine) appear as rows in asw_comparison.csv with both metrics populated
- 1 - ASWbatch must fall in [0, 1] and ASW_celltype in [-1, 1]; values outside indicate the embedding or labels were misaligned
- Cell type and batch label vectors must have the same length and ordering as the embedding rows (Baron + Lawlor + Muraro concatenated)

## Limitations

- Ground-truth cell-type labels come from original dataset publications and were not independently validated, bounding achievable ASW_celltype
- The pancreas-style benchmark datasets discussed (e.g., SCMixology, Zhengmix) comprise well-defined cell clusters, by design simpler than complex-tissue data, so silhouette results may not transfer to real tissues
- Cluster-recovery metrics like ASW/ARI relative to 'reference' labels may penalize methods that better distinguish sub-populations or rare cell types

## Evidence

- [discussion] CA-FT was most performant overall in a scRNAseq cluster recovery task.: "CA-FT was most performant overall in a scRNAseq cluster recovery task."
- [abstract] Switching from PCA to CA is achieved through a simple pipeline substitution: "Switching from PCA to CA is achieved through a simple pipeline substitution"
- [readme] README corralm description: "corral is a package to perform correspondence analysis on count data, particularly for single-cell (e.g., RNA-seq)."
- [readme] README integration scope: "It can be used on a single table for dimensionality reduction, or it can be used for integration across batches, samples, and modes."
- [discussion] Benchmark label caveat: "the datasets we analyzed did not have independently validated cell type annotations, so performance is limited by the original cell type assignments"
