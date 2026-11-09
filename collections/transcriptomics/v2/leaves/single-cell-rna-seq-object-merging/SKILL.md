---
name: single-cell-rna-seq-object-merging
description: Use when you have two or more independently pre-processed SingCellaR
  objects from biological replicates and need a single integrated object for downstream
  clustering, UMAP visualization, and cell-type annotation.
when_to_use_negative:
- You have only a single pre-processed object — use the standard per-sample workflow
  with get_variable_genes_by_fitting_GLM_model instead of the integrative variant
- Your objects are in Seurat or SingleCellExperiment format and cannot be converted
  — SingCellaR still lacks automatic object transformation to interact with packages
  such as Seurat
- You have not yet performed per-replicate preprocessing and QC — merging raw, unfiltered
  objects is outside this skill's scope
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SingCellaR
  role: R package providing the preprocess_integration step and the filter_cells_and_genes,
    normalize_UMIs, get_variable_genes_for_integrative_data_by_fitting_GLM_model,
    get_pca.result, get_umap.result, and identifyClusters functions used to merge
    and re-process replicates
  repo: https://github.com/supatt-lab/SingCellaR
- name: R
  role: Execution environment in which the SingCellaR integration workflow is run
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:4951bca0f030842a7008fecc50fc4f66812efcc6deacc979f2483e27bfb3f652
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/supatt-lab/SingCellaR
  tools:
  - R
  - SingCellaR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# single-cell-rna-seq-object-merging

## Summary

Merges pre-processed SingCellaR objects from biological replicates (e.g., early fetal liver samples eFL_1 and eFL_2) into a single combined object, then re-applies QC, normalization, regression, variable gene selection, PCA, UMAP, and Louvain clustering so cells cluster by major cell type rather than by sample of origin.

## When to use

Use when you have two or more independently pre-processed SingCellaR objects from biological replicates and need a single integrated object for downstream clustering, UMAP visualization, and cell-type annotation.

## When NOT to use

- You have only a single pre-processed object — use the standard per-sample workflow with get_variable_genes_by_fitting_GLM_model instead of the integrative variant
- Your objects are in Seurat or SingleCellExperiment format and cannot be converted — SingCellaR still lacks automatic object transformation to interact with packages such as Seurat
- You have not yet performed per-replicate preprocessing and QC — merging raw, unfiltered objects is outside this skill's scope

## Inputs

- Two or more pre-processed SingCellaR R objects from biological replicates (e.g., eFL_1.SingCellaR.rdata and eFL_2.SingCellaR.rdata)
- Optional multi-lineage signature gene sets (GMT file) for UMAP annotation of the integrated object

## Outputs

- A single integrated SingCellaR object saved as eFL_All.SingCellaR.rdata
- PCA and UMAP embeddings computed on the merged data
- Louvain cluster assignments on the kNN graph using integrative embeddings
- Figure 2 plots: UMAP colored by cluster and by sample of origin

## How to apply

Load the pre-processed replicate objects (e.g., eFL_1 and eFL_2) and merge them via the preprocess_integration step of the SingCellaR workflow. Re-apply quality control on the merged object with filter_cells_and_genes(object, min_UMIs, max_UMIs, min_detected_genes, max_detected_genes, min_percent_mito, max_percent_mito, genes_with_expressing_cells, isRemovedDoublets), then re-normalize with normalize_UMIs and regress out confounders via get_regressout_log_data. Re-identify variable genes on the integrated data using get_variable_genes_for_integrative_data_by_fitting_GLM_model (a GLM-fitting variant distinct from the single-object selection), run PCA (get_pca.result), generate a UMAP embedding (get_umap.result), and cluster with identifyClusters using Louvain clustering on the kNN graph with integrative embeddings. The rationale is that merging removes replicate-specific artifacts so cells group by lineage (erythroid, myeloid, lymphoid, megakaryocytic) rather than by sample. Save the result as eFL_All.SingCellaR.rdata and produce Figure 2-style plots (UMAP colored by cluster and sample of origin) from the merged object.

## Related tools

- **SingCellaR** (R package providing the preprocess_integration step and the filter_cells_and_genes, normalize_UMIs, get_variable_genes_for_integrative_data_by_fitting_GLM_model, get_pca.result, get_umap.result, and identifyClusters functions used to merge and re-process replicates) — https://github.com/supatt-lab/SingCellaR
- **R** (Execution environment in which the SingCellaR integration workflow is run)

## Examples

```
identifyClusters(eFL_All, dim_reduction_method="pca", useIntegrativeEmbeddings=TRUE, integrative_method="scanorama", n.dims.use=30, n.neighbors=10, knn.n_trees=50, knn.metric="euclidean", knn.n_threads=10)
```

## Evaluation signals

- After annotating the UMAP with multi-lineage gene sets, cells cluster together by major cell types (erythroid, myeloid, lymphoid, megakaryocytic), confirming successful integration
- UMAP colored by sample of origin shows mixing of eFL_1 and eFL_2 cells within clusters rather than replicate-specific segregation
- The integrated object can be saved and reloaded as eFL_All.SingCellaR.rdata without loss of embeddings or cluster assignments
- Cluster identification runs on integrative embeddings (useIntegrativeEmbeddings=TRUE) rather than the single-object embeddings
- QC metrics of the merged object satisfy the filter_cells_and_genes thresholds on UMIs, detected genes, and percent mitochondrial reads

## Limitations

- SingCellaR still lacks automatic object transformation to interact with other existing packages, such as Seurat
- UMAP/FDG plot rotations may differ from the protocol due to different software versions and seed number settings
- The fa2 and scanorama Python modules may not be installed or their R environmental path may not be found, breaking runFA2_ForceDirectedGraph or runScanorama integration steps

## Evidence

- [methods] QC on merged object: "`filter_cells_and_genes(object, min_UMIs, max_UMIs, min_detected_genes, max_detected_genes, min_percent_mito, max_percent_mito, genes_with_expressing_cells, isRemovedDoublets)`"
- [methods] Integrative variable gene selection: "`get_variable_genes_for_integrative_data_by_fitting_GLM_model(object, mean_expr_cutoff, disp_zscore_cutoff, quantile_genes_expr_for_fitting, quantile_genes_cv2_for_fitting)`"
- [methods] Clustering on kNN graph: "`identifyClusters(object, dim_reduction_method, useIntegrativeEmbeddings, integrative_method, n.dims.use, n.neighbors, knn.n_trees, knn.metric, knn.n_threads)`"
- [intro] SingCellaR as integrative tool: "SingCellaR an integrative analysis tool for analysing single-cell RNA sequencing data"
- [readme] Example datasets for tutorial: "The example datasets described in the tutorial can be downloaded from this link https://drive.google.com/drive/folders/1ko6XGu1OEoVdWDd2FdMlucx7IBK7Dpq0"
