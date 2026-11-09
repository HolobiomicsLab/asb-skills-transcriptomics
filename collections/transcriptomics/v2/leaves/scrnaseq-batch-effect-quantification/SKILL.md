---
name: scrnaseq-batch-effect-quantification
description: Use when comparing batch-integration/dimension-reduction methods (e.g.,
  corralm, corralm+Harmony, corralm+MNN, multibatch PCA, Seurat) on multi-batch scRNA-seq
  count matrices and you need paired metrics of cell-type separation (ASW_celltype)
  vs batch mixing (1-ASWbatch).
when_to_use_negative:
- Input is single-batch scRNA-seq data with no batch structure to integrate — ASWbatch
  is undefined/meaningless
- Cell type labels are not independently validated or absent, making ASW_celltype
  an unreliable benchmark axis (the source article notes labels came from original
  publications without independent validation)
- Goal is raw count QC rather than comparing dimension-reduction/integration embeddings
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: corralm
  role: Multi-table correspondence analysis producing the integrated cell embeddings
    (with Freeman-Tukey residuals) whose batch structure is quantified by ASW
  repo: github.com/laurenhsu1/corral
- name: corral
  role: Single-table CA dimension reduction in R/Bioconductor that feeds the corralm
    integration workflow
  repo: github.com/laurenhsu1/corral
- name: Harmony
  role: Batch correction applied to corralm embeddings to create the corralm+Harmony
    comparison variant
- name: MNN
  role: Batch correction applied to corralm embeddings to create the corralm+MNN comparison
    variant
- name: R/Bioconductor
  role: Runtime for SingleCellExperiment objects, silhouette computation, and the
    benchmark pipeline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:c6808a2e4e679e2e24d863a27393a283f870e7cc245482c3fa73c31220d43cf2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - Harmony
  - MNN
  - R
  - R/Bioconductor
  - corral
  - corralm
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# scrnaseq-batch-effect-quantification

## Summary

Quantifies batch effects and biological preservation in multi-batch scRNA-seq integration by computing average silhouette width (ASW) with cell-type labels and batch labels on the first 8 dimensions of each method's embedding, as in the corralm pancreas benchmark (Baron, Lawlor, Muraro).

## When to use

Use when comparing batch-integration/dimension-reduction methods (e.g., corralm, corralm+Harmony, corralm+MNN, multibatch PCA, Seurat) on multi-batch scRNA-seq count matrices and you need paired metrics of cell-type separation (ASW_celltype) vs batch mixing (1-ASWbatch).

## When NOT to use

- Input is single-batch scRNA-seq data with no batch structure to integrate — ASWbatch is undefined/meaningless
- Cell type labels are not independently validated or absent, making ASW_celltype an unreliable benchmark axis (the source article notes labels came from original publications without independent validation)
- Goal is raw count QC rather than comparing dimension-reduction/integration embeddings

## Inputs

- Multi-batch scRNA-seq count matrices as SingleCellExperiment objects (e.g., Baron, Lawlor, Muraro human pancreas)
- Per-cell cell type annotations
- Per-cell batch/dataset labels
- Reduced-dimension embeddings from each compared method (e.g., corralm FT-residual, corralm+Harmony, corralm+MNN, multibatch PCA)

## Outputs

- ASW_celltype per method (average silhouette width using cell type labels)
- 1-ASWbatch per method (batch silhouette reported as 1 - ASW on batch labels)
- Method comparison table (e.g., asw_comparison.csv)

## How to apply

Load the per-dataset count matrices (e.g., Baron, Lawlor, Muraro pancreas) into SingleCellExperiment objects retaining cell type and batch annotations, then run each integration method — for the CA route, corralm with Freeman-Tukey residuals on the concatenated multi-batch data, plus corralm+Harmony and corralm+MNN variants obtained by applying Harmony and MNN correction to the corralm embeddings, and multibatch PCA as the non-CA baseline. For each method's embedding, compute average silhouette width twice on the first 8 dimensions only: once using cell type labels (ASW_celltype) and once using batch labels reported as 1-ASWbatch, so higher is better on both axes. Tabulate the two metrics per method and save the comparison (e.g., asw_comparison.csv); judge success by whether a method scores well on both axes simultaneously — in the source benchmark, corralm with Freeman-Tukey residuals coupled with Harmony matched the Seurat routine in both batch integration and biological cluster separation.

## Related tools

- **corralm** (Multi-table correspondence analysis producing the integrated cell embeddings (with Freeman-Tukey residuals) whose batch structure is quantified by ASW) — github.com/laurenhsu1/corral
- **corral** (Single-table CA dimension reduction in R/Bioconductor that feeds the corralm integration workflow) — github.com/laurenhsu1/corral
- **Harmony** (Batch correction applied to corralm embeddings to create the corralm+Harmony comparison variant)
- **MNN** (Batch correction applied to corralm embeddings to create the corralm+MNN comparison variant)
- **R/Bioconductor** (Runtime for SingleCellExperiment objects, silhouette computation, and the benchmark pipeline)

## Examples

```
cluster::silhouette(as.integer(celltype), dist(emb[, 1:8]))  # ASW_celltype on first 8 dims; repeat with batch labels for ASWbatch
```

## Evaluation signals

- ASW_celltype and 1-ASWbatch are both computed on exactly the first 8 dimensions of each method's embedding, making methods directly comparable
- Higher ASW_celltype with higher 1-ASWbatch (i.e., lower batch ASW) indicates better joint biological preservation and batch integration
- Benchmark sanity check: corralm (Freeman-Tukey) + Harmony should approach parity with the Seurat routine on both metrics in the pancreas setup
- A method scoring high on batch mixing but low on ASW_celltype (or vice versa) signals a failed trade-off and should be flagged in the comparison table
- Output must be a per-method table (asw_comparison.csv) with two metric columns; row count equals number of methods compared

## Limitations

- Ground-truth cell type labels in the benchmark came from original dataset publications and were not independently validated, capping benchmark reliability
- ARI-style evaluations may penalize methods that better distinguish sub-populations or rare cell types relative to reference labels
- No tool or package versions for corral or its dependencies are reported, so exact reproducibility of the benchmark may vary across versions
- High-quality 'ground-truth' benchmark datasets are largely simple pseudo-cell mixtures or pools of distinct cell types, not complex real tissues

## Evidence

- [abstract] corralm for integrative multi-table dimension reduction of scRNAseq data: "we introduce corralm for integrative multi-table dimension reduction of scRNAseq data"
- [abstract] CA with Freeman-Tukey residuals performs especially well across diverse datasets: "CA with Freeman-Tukey residuals performs especially well across diverse datasets"
- [discussion] CA-FT most performant in cluster recovery: "CA-FT was most performant overall in a scRNAseq cluster recovery task."
- [readme] corral package description: "corral is a package to perform correspondence analysis on count data, particularly for single-cell (e.g., RNA-seq)."
- [readme] corral two main calls: "1. `corral` for dimensionality reduction on a single table 
2. `corralm` for alignment and batch integration of multiple tables"
