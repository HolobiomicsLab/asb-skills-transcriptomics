---
name: glm-based-variable-gene-selection
description: Use when pre-processed SingCellaR objects (e.g. eFL_1, eFL_2) have been
  QC-filtered and normalized with normalize_UMIs, and a set of highly variable genes
  is needed before PCA, UMAP, and Louvain clustering.
when_to_use_negative:
- Data are not yet normalized (normalize_UMIs has not been run) or still contain unfiltered
  doublets and low-UMI cells, since GLM mean–CV fits would be distorted by low-quality
  cells.
- The goal is a basic variance screen without dispersion modeling — SingCellaR provides
  get_variable_genes_basic(object, mean_expr_cutoff, max_of_mean_expr_cutoff, cv.max,
  cv.min) for that simpler case.
- The object format is not a SingCellaR/SingleCellExperiment object (e.g. a Seurat
  object), since the function operates on SingCellaR's own object structure.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SingCellaR
  role: R package providing get_variable_genes_by_fitting_GLM_model and the integrative
    variant get_variable_genes_for_integrative_data_by_fitting_GLM_model used to select
    highly variable genes on normalized scRNA-seq objects.
  repo: github.com/supatt-lab/SingCellaR
- name: R
  role: Language in which SingCellaR and its SingleCellExperiment-based objects and
    GLM fitting routines are executed.
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:de75d2e072a79359713af43330d9c7cbf92101a8052a7d63271e2785fda9f82d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SingCellaR
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# GLM-based variable gene selection

## Summary

Identifies highly variable genes in SingCellaR scRNA-seq objects by fitting a generalized linear model of mean expression versus dispersion, providing data-driven cutoffs for both single-sample and integrative workflows.

## When to use

Use when pre-processed SingCellaR objects (e.g. eFL_1, eFL_2) have been QC-filtered and normalized with normalize_UMIs, and a set of highly variable genes is needed before PCA, UMAP, and Louvain clustering.

## When NOT to use

- Data are not yet normalized (normalize_UMIs has not been run) or still contain unfiltered doublets and low-UMI cells, since GLM mean–CV fits would be distorted by low-quality cells.
- The goal is a basic variance screen without dispersion modeling — SingCellaR provides get_variable_genes_basic(object, mean_expr_cutoff, max_of_mean_expr_cutoff, cv.max, cv.min) for that simpler case.
- The object format is not a SingCellaR/SingleCellExperiment object (e.g. a Seurat object), since the function operates on SingCellaR's own object structure.

## Inputs

- A QC-filtered, UMI-normalized SingCellaR object (single sample or merged/integrated object)
- GLM cutoff parameters: mean_expr_cutoff, disp_zscore_cutoff, quantile_genes_expr_for_fitting, quantile_genes_cv2_for_fitting

## Outputs

- A vector of highly variable genes annotated in the SingCellaR object
- Variable-gene statistics (mean expression, dispersion/CV2 z-scores) used for the GLM fit
- Genes suitable for PCA/UMAP dimensionality reduction and kNN-graph clustering

## How to apply

After loading the cellranger gene-expression matrix and running filter_cells_and_genes with UMI, detected-gene, and mitochondrial-percent thresholds, call get_variable_genes_by_fitting_GLM_model on the normalized object, specifying mean_expr_cutoff, disp_zscore_cutoff, quantile_genes_expr_for_fitting, and quantile_genes_cv2_for_fitting to define the expression window and dispersion z-score used to call variable genes. The GLM fit relates gene expression means to coefficients of variation, so genes deviating strongly from the expected mean–CV trend are flagged as variable rather than using fixed variance cutoffs. When merging biological replicates (e.g. eFL_1 and eFL_2 into eFL_All), use the integrative variant get_variable_genes_for_integrative_data_by_fitting_GLM_model with the same cutoff families so the selected genes support cross-sample integration. Downstream, feed the variable genes into get_pca.result, get_umap.result, and identifyClusters with Louvain clustering on the kNN graph.

## Related tools

- **SingCellaR** (R package providing get_variable_genes_by_fitting_GLM_model and the integrative variant get_variable_genes_for_integrative_data_by_fitting_GLM_model used to select highly variable genes on normalized scRNA-seq objects.) — github.com/supatt-lab/SingCellaR
- **R** (Language in which SingCellaR and its SingleCellExperiment-based objects and GLM fitting routines are executed.)

## Examples

```
get_variable_genes_by_fitting_GLM_model(eFL_1, mean_expr_cutoff=0.012, disp_zscore_cutoff=1.5, quantile_genes_expr_for_fitting=0.95, quantile_genes_cv2_for_fitting=0.95)
```

## Evaluation signals

- The GLM fitting step executes without error after filter_cells_and_genes and normalize_UMIs have completed on the eFL_1/eFL_2 objects.
- The saved eFL_1_SingCellaR_object.rds / eFL_All.SingCellaR.rdata contains a non-empty set of variable genes consistent with the chosen mean_expr_cutoff and disp_zscore_cutoff.
- Downstream PCA/UMAP on the selected variable genes yields cells clustered by major cell types (erythroid, myeloid, lymphoid, megakaryocytic) after multi-lineage gene-set annotation, indicating the gene set captures biological signal.
- For integration, the integrative variant (get_variable_genes_for_integrative_data_by_fitting_GLM_model) was used instead of the single-sample variant, and the merged UMAP shows replicate samples mixed within clusters.
- Mean–CV plots comparable to the protocol's Figure 1/Figure 2 variable-gene panels can be reproduced from the fitted model.

## Limitations

- No software versions are stated in the protocol, and plot rotations/UMAP embeddings can differ between runs due to software version and seed number differences.
- SingCellaR lacks automatic object transformation to interoperate with packages such as Seurat, so the selected gene set cannot be transferred directly into other pipelines without manual conversion.
- Only the expression/dispersion cutoffs supplied by the user control the gene set; the article gives no universal default, so cutoffs must be tuned per dataset.

## Evidence

- [methods] GLM model call with cutoffs: "`get_variable_genes_by_fitting_GLM_model(object, mean_expr_cutoff, disp_zscore_cutoff, quantile_genes_expr_for_fitting, quantile_genes_cv2_for_fitting)`"
- [methods] Integrative variant call: "`get_variable_genes_for_integrative_data_by_fitting_GLM_model(object, mean_expr_cutoff, disp_zscore_cutoff, quantile_genes_expr_for_fitting, quantile_genes_cv2_for_fitting)`"
- [methods] Variable gene step requires prior normalization and QC: "`normalize_UMIs(object, scale.factor, use.scaled.factor)`"
- [discussion] SingCellaR is an R package using SingleCellExperiment objects: "SingCellaR uses the SingleCellExperiment object, the standard object for storing single-cell experimental data in R."
- [readme] Tutorial and example datasets locations: "The example datasets described in the tutorial can be downloaded from this link https://drive.google.com/drive/folders/1ko6XGu1OEoVdWDd2FdMlucx7IBK7Dpq0"
