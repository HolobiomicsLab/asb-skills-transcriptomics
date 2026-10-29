---
name: scrna-seq-embedding-generation
description: Use when a sxRNA-seq dataset contains multiple batches/samples (batch.var
  present) and the goal is a batch-corrected embedding for joint clustering, cell-type
  annotation, or benchmarking against integration methods such as scVI or Harmony.
when_to_use_negative:
- The dataset is a single batch/sample with no batch.var column — joint NMF batch
  correction is unnecessary; ordinary Seurat clustering suffices.
- The goal is removing batch effects while the task requires no interpretability and
  other methods (scVI, Harmony, Scanorama) may be preferable for specific tasks, since
  no method is always best and per-task evaluation is required.
- Advanced usage output (imputed/smoothed expression via H and F matrices) is intended
  for visualization only, not for downstream quantitative differential expression
  analyses.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: JOINTLY (rJOINTLY)
  role: Primary R package performing kernel-based joint non-negative matrix factorization
    to generate the batch-corrected embedding and gene modules; installed via remotes::install_github("madsen-lab/rJOINTLY")
  repo: https://github.com/madsen-lab/rJOINTLY
- name: pyJOINTLY
  role: Python equivalent of JOINTLY working directly with Scanpy; produces very similar
    but not identical results — the R version was used for all manuscript analyses
  repo: https://github.com/madsen-lab/pyJOINTLY
- name: Seurat
  role: Handles input objects, hosts the JOINTLY DimReduc, and performs downstream
    clustering/visualization (FindNeighbors, FindClusters, RunUMAP, DimPlot, CreateDimReducObject)
  repo: https://github.com/satijalab/seurat
- name: SingleCellExperiment
  role: Supported input container for expression matrices passed to jointly()
  repo: https://github.com/drisso/SingleCellExperiment
- name: aricode
  role: Computes ARI and NMI between transferred cell type labels and seurat_clusters
    to select the best JOINTLY solution across repeated runs
- name: UCell
  role: Scores the JOINTLY-learned gene modules onto cells (AddModuleScore_UCell)
    for interpretation on the UMAP
  repo: https://github.com/carmonalab/UCell
- name: presto
  role: Dependency for the advanced (non-wrapper) smoothing/imputation workflow using
    H and F matrices
  repo: https://github.com/immunogenomics/presto
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:bb08e6bb3de7ca052014ec2f38b9280b51410b382b73529afa9e161d97c6c279
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
  - SingleCellExperiment
  - UCell
  - aricode
  - fastMNN
  - presto
  - pyJOINTLY
  - scVI
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7368421052631579
n_claims: 19
---

# scRNA-seq-embedding-generation

## Summary

Generates interpretable low-dimensional embeddings from single-cell/single-nucleus RNA-seq datasets across batches using JOINTLY's kernel-based joint non-negative matrix factorization, enabling batch-corrected joint clustering that feeds downstream Seurat clustering, ARI/NMI evaluation, and UMAP visualization.

## When to use

Use when a sxRNA-seq dataset contains multiple batches/samples (batch.var present) and the goal is a batch-corrected embedding for joint clustering, cell-type annotation, or benchmarking against integration methods such as scVI or Harmony.

## When NOT to use

- The dataset is a single batch/sample with no batch.var column — joint NMF batch correction is unnecessary; ordinary Seurat clustering suffices.
- The goal is removing batch effects while the task requires no interpretability and other methods (scVI, Harmony, Scanorama) may be preferable for specific tasks, since no method is always best and per-task evaluation is required.
- Advanced usage output (imputed/smoothed expression via H and F matrices) is intended for visualization only, not for downstream quantitative differential expression analyses.

## Inputs

- List of scRNA-seq/scRNA-seq expression objects per batch: Seurat, SingleCellExperiment, or raw Matrix-like expression matrices
- Batch/covariate metadata column (e.g., 'batch_label') present in the object metadata
- Cell type labels TSV for evaluation (e.g., Human_Liver_labels.tsv with Transferred_labels)
- Test/benchmark datasets from https://zenodo.org/record/8298157 (e.g., Human_Liver.rds)

## Outputs

- A list holding a Seurat object with a 'JOINTLY' dimensional reduction (first entry) and gene modules (second entry)
- Learned H matrix (Hmat.scaled) as a Seurat DimReduc object, and per-batch F matrices when running without the wrapper
- Interpretable gene modules scoreable via AddModuleScore_UCell
- Cluster assignments (seurat_clusters) and ARI/NMI scores (e.g., ARI 0.7415981, NMI 0.6325386 for the liver test dataset)
- UMAP embedding on the JOINTLY reduction (dims 1:15)

## How to apply

Load a list of scRNA-seq expression objects (Seurat, SingleCellExperiment, or Matrix-like) and run the jointly() wrapper with batch.var = "batch_label", which internally executes preprocess(), cpca(), prepareData(), and JOINTLYsolve() to yield a Seurat object (liver$object) with a 'JOINTLY' DimReduc and a list of interpretable gene modules. For complex multi-condition, multi-sample datasets where samples do not share the same axes of variation, use regular PCA instead of consensus PCA (cPCA). Because setting a seed and re-running produces different results, run JOINTLY up to 5 times with different seeds and keep the best solution (e.g., maximizing aricode::ARI against transferred cell type labels, reference liver ARI = 0.7415981). Downstream, cluster the embedding with Seurat's FindNeighbors/FindClusters on the 'JOINTLY' reduction using dims = 1:15 and res = 0.05, then embed with RunUMAP (dims 1:15) for visualization and interpretation via UCell module scores. To inspect the learned H and F matrices directly (e.g., for MAGIC-like batch-aware expression smoothing, visualization only), bypass the wrapper: preprocess -> cpca -> prepareData -> JOINTLYsolve, then insert solved$Hmat.scaled as a DimReduc via Seurat::CreateDimReducObject.

## Related tools

- **JOINTLY (rJOINTLY)** (Primary R package performing kernel-based joint non-negative matrix factorization to generate the batch-corrected embedding and gene modules; installed via remotes::install_github("madsen-lab/rJOINTLY")) — https://github.com/madsen-lab/rJOINTLY
- **pyJOINTLY** (Python equivalent of JOINTLY working directly with Scanpy; produces very similar but not identical results — the R version was used for all manuscript analyses) — https://github.com/madsen-lab/pyJOINTLY
- **Seurat** (Handles input objects, hosts the JOINTLY DimReduc, and performs downstream clustering/visualization (FindNeighbors, FindClusters, RunUMAP, DimPlot, CreateDimReducObject)) — https://github.com/satijalab/seurat
- **SingleCellExperiment** (Supported input container for expression matrices passed to jointly()) — https://github.com/drisso/SingleCellExperiment
- **aricode** (Computes ARI and NMI between transferred cell type labels and seurat_clusters to select the best JOINTLY solution across repeated runs)
- **UCell** (Scores the JOINTLY-learned gene modules onto cells (AddModuleScore_UCell) for interpretation on the UMAP) — https://github.com/carmonalab/UCell
- **presto** (Dependency for the advanced (non-wrapper) smoothing/imputation workflow using H and F matrices) — https://github.com/immunogenomics/presto

## Examples

```
liver <- readRDS("Human_Liver.rds"); liver <- jointly(liver, batch.var = "batch_label")
```

## Evaluation signals

- aricode::ARI between celltype labels and seurat_clusters on the Human Liver test dataset should reproduce the reference value 0.7415981 (NMI 0.6325386).
- The returned object must be a list whose first entry is a Seurat object containing a 'JOINTLY' DimReduc usable by FindNeighbors(reduction = "JOINTLY", dims = 1:15).
- Repeated runs with different seeds yield different results; running up to 5 times and selecting the best ARI should not degrade below the reference clustering accuracy.
- UMAP computed on the JOINTLY reduction (dims 1:15) should show clusters aligned with transferred cell types and mixing across batch_label.
- AddModuleScore_UCell on liver$modules should yield per-factor scores (e.g., factor_1_UCell) plottable via FeaturePlot on the UMAP.

## Limitations

- Setting a seed and re-running produces a different result; the wrapper's stochastic initialization requires up to 5 repeated runs and keeping the best solution.
- Consensus PCA (cPCA) is not advised for complex multi-condition, multi-sample datasets where samples do not share the same axes of variation; regular PCA is recommended instead.
- The advanced H/F-matrix-based smoothed (MAGIC-like) expression values should be used with caution and only for visualization purposes.
- The Python package (pyJOINTLY) does not produce identical results to the R version used for the manuscript analyses.
- Performance is task-dependent: no integration method is always the best across all five integration tasks, so the chosen method must be evaluated on each new task; performance could be further improved by supervised initialization with cell type labels (future work).

## Evidence

- [readme] JOINTLY is an R package for interpretable joint clustering of single-cell and single-nucleus RNA-seq, using kernel-based joint non-negative matrix factorization.: "JOINTLY is an R package for interpretable joint clustering of single-cell and single-nucleus RNA-seq, using kernel-based joint non-negative matrix factorization."
- [methods] Wrapper call with batch variable: "liver <- jointly(liver, batch.var = "batch_label")"
- [methods] Downstream clustering parameters: "liver$object <- FindNeighbors(liver$object, reduction = "JOINTLY", dims = 1:15)
liver$object <- FindClusters(liver$object, res = 0.05)"
- [readme] Reference ARI value: "aricode::ARI(liver$object$celltype, liver$object$seurat_clusters) # 0.7415981"
- [readme] Seed sensitivity advice: "Setting a seed and re-running produces a different result. Sometimes it is worth running JOINTLY up to 5 times and choosing the best solution."
- [discussion] cPCA caution for complex datasets: "For more complex datasets, where all samples do not necessarily share the same axes of variation, such as multi-condition, multi-sample datasets, we advise users to use regular PCA in JOINTLY"
- [abstract] Benchmark performance claim: "JOINTLY performs on par or better than state-of-the-art batch integration methods in clustering tasks and outperforms other intrinsically interpretable methods."
