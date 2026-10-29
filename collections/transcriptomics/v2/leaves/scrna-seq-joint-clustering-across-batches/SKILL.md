---
name: scrna-seq-joint-clustering-across-batches
description: Use when multiple sxRNA-seq batches/samples must be jointly clustered
  with batch effects removed while retaining interpretable gene-module factors, e.g.
  to annotate cell types or compare integration performance against scVI, Harmony,
  or LIGER.
when_to_use_negative:
- When the goal requires standard, non-interpretable integration-only embeddings without
  gene-module interpretation, or when comparing against methods where task-specific
  performance matters — no method is always the best across all tasks, so verify on
  your specific task.
- When datasets are complex multi-condition, multi-sample samples that do not share
  the same axes of variation — the article advises regular PCA in JOINTLY rather than
  the consensus PCA default.
- For smoothed/imputed expression values beyond visualization — the README advises
  using MAGIC-like batch-aware smoothed values from the H/F matrices with caution
  and only for visualization purposes.
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: JOINTLY (rJOINTLY)
  role: Core joint NMF clustering and interpretable factor-learning method; invoked
    via jointly() wrapper with batch.var
  repo: https://github.com/madsen-lab/rJOINTLY
- name: Seurat
  role: Hosts the input/output object; FindNeighbors/FindClusters on the JOINTLY reduction
    and RunUMAP embedding
  repo: https://github.com/satijalab/seurat
- name: aricode
  role: Computes ARI and NMI between cell type labels and cluster labels for evaluation
- name: UCell
  role: AddModuleScore_UCell computes module scores on the learned JOINTLY modules
    for interpretation
  repo: https://github.com/carmonalab/UCell
- name: presto
  role: wilcoxauc differential expression used in the advanced batch-aware smoothing
    workflow
  repo: https://github.com/immunogenomics/presto
- name: pyJOINTLY
  role: Python/Scanpy alternative package; produces similar but not identical results
    to the R version
  repo: https://github.com/madsen-lab/pyJOINTLY
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:39c4a66406aa4f25abace6f26f54a7f729744a577994b9a6515189625be04944
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madsen-lab/rJOINTLY
  tools:
  - Harmony
  - JOINTLY (rJOINTLY)
  - LIGER
  - Scanorama
  - Seurat
  - UCell
  - aricode
  - fastMNN
  - presto
  - pyJOINTLY
  - scVI
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# scrna-seq-joint-clustering-across-batches

## Summary

Perform interpretable joint clustering of scRNA-seq/snRNA-seq datasets across batches using JOINTLY, a hybrid linear and non-linear kernel-based joint NMF method, followed by Seurat clustering and ARI/NMI evaluation against cell type labels.

## When to use

Use when multiple sxRNA-seq batches/samples must be jointly clustered with batch effects removed while retaining interpretable gene-module factors, e.g. to annotate cell types or compare integration performance against scVI, Harmony, or LIGER.

## When NOT to use

- When the goal requires standard, non-interpretable integration-only embeddings without gene-module interpretation, or when comparing against methods where task-specific performance matters — no method is always the best across all tasks, so verify on your specific task.
- When datasets are complex multi-condition, multi-sample samples that do not share the same axes of variation — the article advises regular PCA in JOINTLY rather than the consensus PCA default.
- For smoothed/imputed expression values beyond visualization — the README advises using MAGIC-like batch-aware smoothed values from the H/F matrices with caution and only for visualization purposes.

## Inputs

- List of scRNA-seq/snRNA-seq expression objects (Seurat, SingleCellExperiment, or Matrix-like)
- Batch/covariate metadata column (e.g. 'batch_label') in the object
- Cell type label table (e.g. Human_Liver_labels.tsv, read via read.delim) for ARI/NMI evaluation
- Test datasets from Zenodo record 8298157 (e.g. Human_Liver.rds)

## Outputs

- Seurat object with a 'JOINTLY' dimensional reduction (dims 1:15 usable)
- Learned gene modules (H and F matrices) interpretable as gene programs
- Cluster assignments (seurat_clusters) and cluster-level ARI/NMI scores
- UMAP embedding and module scores per cell (e.g. factor_1_UCell)

## How to apply

Install rJOINTLY via remotes::install_github("madsen-lab/rJOINTLY") (R >= 4.1.0) and load Seurat objects, SingleCellExperiment objects, or raw expression matrices. Run jointly(object, batch.var = "batch_label"), optionally passing bpparam = BiocParallel::MulticoreParam() for parallel execution; because setting a seed changes the result, re-run JOINTLY up to 5 times and keep the best solution. The returned list holds a Seurat object ($object) with a 'JOINTLY' reduction and gene modules ($modules). Cluster with FindNeighbors(liver$object, reduction = "JOINTLY", dims = 1:15) and FindClusters(res = 0.05), then score clustering accuracy with aricode::ARI and aricode::NMI against imported cell type labels (subset to named cell types with >= 10 cells). Interpret results by computing UCell module scores via AddModuleScore_UCell(features = liver$modules) and embedding with RunUMAP(reduction = "JOINTLY", dims = 1:15). For complex multi-condition, multi-sample datasets where samples do not share the same axes of variation, use regular PCA rather than the default consensus PCA (cPCA) path.

## Related tools

- **JOINTLY (rJOINTLY)** (Core joint NMF clustering and interpretable factor-learning method; invoked via jointly() wrapper with batch.var) — https://github.com/madsen-lab/rJOINTLY
- **Seurat** (Hosts the input/output object; FindNeighbors/FindClusters on the JOINTLY reduction and RunUMAP embedding) — https://github.com/satijalab/seurat
- **aricode** (Computes ARI and NMI between cell type labels and cluster labels for evaluation)
- **UCell** (AddModuleScore_UCell computes module scores on the learned JOINTLY modules for interpretation) — https://github.com/carmonalab/UCell
- **presto** (wilcoxauc differential expression used in the advanced batch-aware smoothing workflow) — https://github.com/immunogenomics/presto
- **pyJOINTLY** (Python/Scanpy alternative package; produces similar but not identical results to the R version) — https://github.com/madsen-lab/pyJOINTLY

## Examples

```
liver <- readRDS("Human_Liver.rds"); liver <- jointly(liver, batch.var = "batch_label"); liver$object <- FindNeighbors(liver$object, reduction = "JOINTLY", dims = 1:15); liver$object <- FindClusters(liver$object, res = 0.05); aricode::ARI(liver$object$celltype, liver$object$seurat_clusters)
```

## Evaluation signals

- aricode::ARI(liver$object$celltype, liver$object$seurat_clusters) reproduces the deposited reference value 0.7415981 on the Human Liver dataset
- aricode::NMI(...) matches the reference value 0.6325386 on the same dataset
- JOINTLY ranks among the top methods (second overall behind scVI) in the five-task integration benchmark (Lung, Pancreas, Kidney, Liver, PBMC) with metrics ARI, iLISI, cLISI, bASW, cASW
- Cells are subset to named cell types with >= 10 cells before evaluation, and empty labels ('') are excluded
- Module scores (e.g. factor_1_UCell) localize to expected cell populations on the UMAP, e.g. IGF1 in hepatocytes

## Limitations

- Setting a seed and re-running produces a different result; it may be worth running JOINTLY up to 5 times and choosing the best solution
- No method is always the best across all tasks — evaluate JOINTLY on your specific integration task (clustering tasks favor it; other tasks may favor scVI or Harmony)
- Integration metrics do not reveal the genes or features driving integration and clustering
- Consensus PCA (cPCA) is not advised for complex multi-condition, multi-sample datasets where samples do not share the same axes of variation; regular PCA is recommended
- The Python (pyJOINTLY) and R versions do not produce identical results; all manuscript analyses were performed in R
- Batch-aware smoothed/imputed expression (MAGIC-like from H and F matrices) should be used with caution and only for visualization

## Evidence

- [readme] JOINTLY is an R package for interpretable joint clustering of single-cell and single-nucleus RNA-seq, using kernel-based joint non-negative matrix factorization.: "JOINTLY is an R package for interpretable joint clustering of single-cell and single-nucleus RNA-seq, using kernel-based joint non-negative matrix factorization."
- [readme] run up to 5 times: "Setting a seed and re-running produces a different result. Sometimes it is worth running JOINTLY up to 5 times and choosing the best solution."
- [readme] clustering parameters: "liver$object <- FindNeighbors(liver$object, reduction = "JOINTLY", dims = 1:15)
liver$object <- FindClusters(liver$object, res = 0.05)"
- [readme] reference ARI: "aricode::ARI(liver$object$celltype, liver$object$seurat_clusters) # 0.7415981"
- [abstract] benchmark outcome: "JOINTLY performs on par or better than state-of-the-art batch integration methods in clustering tasks and outperforms other intrinsically interpretable methods."
- [discussion] cPCA caveat: "For more complex datasets, where all samples do not necessarily share the same axes of variation, such as multi-condition, multi-sample datasets, we advise users to use regular PCA in JOINTLY"
