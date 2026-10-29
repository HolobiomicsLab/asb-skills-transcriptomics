---
name: single-cell-batch-integration-joint-clustering
description: Use when multi-donor or multi-batch scRNA/snRNA-seq datasets must be
  integrated for joint clustering, and interpretable gene factors/modules (not just
  a latent embedding) are needed for cell-type annotation or marker comparison.
when_to_use_negative:
- When a single, unbatched dataset is analyzed — joint batch integration is unnecessary
  and jointly() requires a batch.var grouping
- When the goal is purely batch correction without interpretable factors — simpler
  statistical (Harmony) or deep-learning (scVI) methods may suffice, since no method
  is always best and performance is task-dependent
- For complex multi-condition, multi-sample datasets when consensus PCA (cpca) would
  be used internally — the article advises regular PCA in JOINTLY for such cases
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: JOINTLY
  role: 'Core algorithm: kernel-based joint NMF integration and interpretable joint
    clustering across batches via jointly() or the preprocess/cpca/prepareData/JOINTLYsolve
    internals'
  repo: https://github.com/madsen-lab/rJOINTLY
- name: Seurat
  role: Object framework hosting the input data and the 'JOINTLY' DimReduc; FindNeighbors/FindClusters/RunUMAP
    clustering and embedding on the JOINTLY reduction
  repo: https://github.com/satijalab/seurat
- name: UCell
  role: AddModuleScore_UCell scores each learned JOINTLY module per cell for factor
    interpretation and FeaturePlot visualization
  repo: https://github.com/carmonalab/UCell
- name: aricode
  role: Computes ARI and NMI between transferred cell-type labels and seurat_clusters
    to evaluate clustering quality
- name: presto
  role: wilcoxauc differential expression on cell types, used in the marker-AUC comparison
    against JOINTLY module genes
  repo: https://github.com/immunogenomics/presto
- name: SingleCellExperiment
  role: Alternative input object class accepted by jointly() alongside Seurat and
    matrix-like objects
  repo: https://github.com/drisso/SingleCellExperiment
- name: pyJOINTLY
  role: Python/Scanpy port of JOINTLY; note the R version was used for all manuscript
    analyses and the two do not produce identical results
  repo: https://github.com/madsen-lab/pyJOINTLY
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:ceef763d2e4c54bb301053629a596ffc76062be38bb0e1ee2837402d15c47cda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madsen-lab/rJOINTLY
  tools:
  - JOINTLY
  - Seurat
  - SingleCellExperiment
  - UCell
  - aricode
  - presto
  - pyJOINTLY
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# single-cell-batch-integration-joint-clustering

## Summary

Run JOINTLY, a hybrid linear and non-linear NMF-based joint clustering algorithm, to integrate single-cell/single-nucleus RNA-seq datasets across batches while producing interpretable gene modules that support clustering evaluation and biological interpretation.

## When to use

Use when multi-donor or multi-batch scRNA/snRNA-seq datasets must be integrated for joint clustering, and interpretable gene factors/modules (not just a latent embedding) are needed for cell-type annotation or marker comparison.

## When NOT to use

- When a single, unbatched dataset is analyzed — joint batch integration is unnecessary and jointly() requires a batch.var grouping
- When the goal is purely batch correction without interpretable factors — simpler statistical (Harmony) or deep-learning (scVI) methods may suffice, since no method is always best and performance is task-dependent
- For complex multi-condition, multi-sample datasets when consensus PCA (cpca) would be used internally — the article advises regular PCA in JOINTLY for such cases

## Inputs

- List of scRNA-seq/snRNA-seq expression objects: Seurat objects, SingleCellExperiment objects, or raw expression matrices, split per batch
- Batch/label metadata column (e.g. batch_label) passed via batch.var
- Cell type label table (e.g. Human_Liver_labels.tsv) for ARI/NMI evaluation
- Test data downloadable from https://zenodo.org/record/8298157 (e.g. Human_Liver.rds)

## Outputs

- List with a Seurat object containing the joint H-matrix co-embedding as a 'JOINTLY' DimReduc (entry 1) and interpretable gene modules (entry 2)
- Cluster labels (seurat_clusters) from FindClusters on dims 1:15 at res = 0.05
- UMAP embedding computed on the JOINTLY reduction
- ARI and NMI values comparing cell-type labels to clusters (aricode)
- Per-cell UCell module scores (e.g. factor_1_UCell) for factor interpretation
- Optional batch-aware smoothed/imputed expression assay built from Fmat %*% Hmat (visualization only)

## How to apply

Install JOINTLY from GitHub (remotes::install_github("madsen-lab/rJOINTLY"), requires R >= 4.1.0) and run the jointly() wrapper on a Seurat, SingleCellExperiment, or matrix-list input with batch.var set to the batch label (e.g. jointly(liver, batch.var = "batch_label")); the returned list holds a Seurat object with the 'JOINTLY' reduction as its first entry and learned gene modules as its second entry. Because results vary with the seed, re-run JOINTLY (up to ~5 times, optionally with bpparam = BiocParallel::MulticoreParam()) and keep the best solution. Cluster cells on the JOINTLY reduction with FindNeighbors and FindClusters (dims 1:15, res = 0.05), embed with RunUMAP (dims 1:15), and quantify clustering quality against transferred cell-type labels using aricode::ARI and aricode::NMI (the liver test dataset yields ARI ≈ 0.74, NMI ≈ 0.63). Interpret the factors by scoring every learned module per cell with AddModuleScore_UCell, and, if per-batch smoothed expression is desired, run the internals (preprocess, cpca, prepareData, JOINTLYsolve) and reconstruct expression from Fmat %*% Hmat as a MAGIC-like imputation for visualization only.

## Related tools

- **JOINTLY** (Core algorithm: kernel-based joint NMF integration and interpretable joint clustering across batches via jointly() or the preprocess/cpca/prepareData/JOINTLYsolve internals) — https://github.com/madsen-lab/rJOINTLY
- **Seurat** (Object framework hosting the input data and the 'JOINTLY' DimReduc; FindNeighbors/FindClusters/RunUMAP clustering and embedding on the JOINTLY reduction) — https://github.com/satijalab/seurat
- **UCell** (AddModuleScore_UCell scores each learned JOINTLY module per cell for factor interpretation and FeaturePlot visualization) — https://github.com/carmonalab/UCell
- **aricode** (Computes ARI and NMI between transferred cell-type labels and seurat_clusters to evaluate clustering quality)
- **presto** (wilcoxauc differential expression on cell types, used in the marker-AUC comparison against JOINTLY module genes) — https://github.com/immunogenomics/presto
- **SingleCellExperiment** (Alternative input object class accepted by jointly() alongside Seurat and matrix-like objects) — https://github.com/drisso/SingleCellExperiment
- **pyJOINTLY** (Python/Scanpy port of JOINTLY; note the R version was used for all manuscript analyses and the two do not produce identical results) — https://github.com/madsen-lab/pyJOINTLY

## Examples

```
library(JOINTLY); liver <- readRDS("Human_Liver.rds"); liver <- jointly(liver, batch.var = "batch_label")
```

## Evaluation signals

- ARI between transferred cell-type labels and seurat_clusters should reach ~0.74 and NMI ~0.63 on the Zenodo Human_Liver test dataset (0.7415981 / 0.6325386 reported)
- The jointly() result must be a list whose first entry is a Seurat object with a 'JOINTLY' DimReduc (H matrix, dims usable as 1:15) and whose second entry is the gene-module list
- Re-running with different seeds changes results; run up to 5 times and confirm the chosen solution is the best-scoring one before clustering
- UCell module scores (e.g. factor_1_UCell) must be computable for all learned modules and visualizable on the UMAP via FeaturePlot
- Batch mixing can be inspected by DimPlot colored by batch_label alongside cluster and celltype plots

## Limitations

- Setting a seed and re-running produces a different result; it can be worth running JOINTLY up to 5 times and choosing the best solution
- No integration method is always the best across all tasks — performance must be evaluated on each new integration task
- Consensus PCA (cpca) is not advised for complex multi-condition, multi-sample datasets where samples do not share the same axes of variation; regular PCA in JOINTLY is advised instead
- The smoothed/imputed expression values built from the H and F matrices should be used with caution and only for visualization purposes
- Integration metrics (ARI/NMI) do not reveal the genes or features driving integration and clustering
- Software version numbers are not stated in the methods; only R >= 4.1.0 and the dependency list are documented

## Evidence

- [readme] JOINTLY is an R package for interpretable joint clustering of single-cell and single-nucleus RNA-seq, using kernel-based joint non-negative matrix factorization.: "JOINTLY is an R package for interpretable joint clustering of single-cell and single-nucleus RNA-seq, using kernel-based joint non-negative matrix factorization."
- [readme] The resulting object is a list that holds a Seurat object as the first entry and module as the second entry: "# The resulting object is a list that holds a Seurat object as the first entry and module as the second entry"
- [readme] Setting a seed and re-running produces a different result. Sometimes it is worth running JOINTLY up to 5 times and choosing the best solution.: "Setting a seed and re-running produces a different result. Sometimes it is worth running JOINTLY up to 5 times and choosing the best solution."
- [readme] aricode::ARI(liver$object$celltype, liver$object$seurat_clusters) # 0.7415981: "aricode::ARI(liver$object$celltype, liver$object$seurat_clusters) # 0.7415981"
- [discussion] For more complex datasets, where all samples do not necessarily share the same axes of variation, such as multi-condition, multi-sample datasets, we advise users to use regular PCA in JOINTLY: "For more complex datasets, where all samples do not necessarily share the same axes of variation, such as multi-condition, multi-sample datasets, we advise users to use regular PCA in JOINTLY"
- [abstract] JOINTLY performs on par or better than state-of-the-art batch integration methods in clustering tasks and outperforms other intrinsically interpretable methods.: "JOINTLY performs on par or better than state-of-the-art batch integration methods in clustering tasks and outperforms other intrinsically interpretable methods."
- [methods] liver <- jointly(liver, batch.var = "batch_label"): "liver <- jointly(liver, batch.var = "batch_label")"
