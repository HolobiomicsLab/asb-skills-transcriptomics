---
name: umi-normalization-and-confounder-regression
description: Use when a QC-filtered SingCellaR object (raw UMI matrix, e.g. after
  merge of replicates like eFL_1 and eFL_2) needs comparable per-cell expression values
  with library-size and nuisance-factor effects removed before PCA/UMAP/clustering.
when_to_use_negative:
- Input is raw count data that has not passed filter_cells_and_genes QC (low-UMI cells,
  high percent_mito cells, or doublets still present)
- Data are not UMI-based counts — use TargetSeq_filter_cells_and_genes/TargetSeq-specific
  processing for read-based target-sequence data
- The object has already been normalized and regressed for the same analysis; re-running
  without re-filtering after a merge can distort the integrated embedding
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SingCellaR
  role: R package providing normalize_UMIs and get_regressout_log_data for UMI scaling
    and confounder regression
  repo: github.com/supatt-lab/SingCellaR
- name: R
  role: Environment in which the SingCellaR normalization/regression functions are
    executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:87ede21d94b22e4b7df389aceb6e75a8552abc794cbf2439d4a5bef84fc22807
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SingCellaR
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# UMI normalization and confounder regression

## Summary

Scale UMI counts to a common factor and regress out technical confounders in SingCellaR to produce regressed log-transformed data suitable for downstream variable gene selection, PCA, and clustering of single-cell RNA-seq objects such as the integrated eFL fetal liver replicates.

## When to use

Use when a QC-filtered SingCellaR object (raw UMI matrix, e.g. after merge of replicates like eFL_1 and eFL_2) needs comparable per-cell expression values with library-size and nuisance-factor effects removed before PCA/UMAP/clustering.

## When NOT to use

- Input is raw count data that has not passed filter_cells_and_genes QC (low-UMI cells, high percent_mito cells, or doublets still present)
- Data are not UMI-based counts — use TargetSeq_filter_cells_and_genes/TargetSeq-specific processing for read-based target-sequence data
- The object has already been normalized and regressed for the same analysis; re-running without re-filtering after a merge can distort the integrated embedding

## Inputs

- SingCellaR object with raw UMI counts and sample metadata (e.g. eFL_All after merging eFL_1 and eFL_2)
- QC-filtered SingCellaR object produced by filter_cells_and_genes
- Optional confounder covariates: total UMIs, percent mitochondria, cell-cycle gene set scores (from add_cell_cycle_genes_score with a .gmt file)

## Outputs

- Normalized UMI matrix stored in the SingCellaR object
- Regressed log-transformed expression matrix via get_regressout_log_data(object)
- Downstream-ready object for variable gene fitting (GLM), PCA, UMAP, and Louvain clustering

## How to apply

First re-run quality control on the merged object with filter_cells_and_genes(object, min_UMIs, max_UMIs, min_detected_genes, max_detected_genes, min_percent_mito, max_percent_mito, genes_with_expressing_cells, isRemovedDoublets) so that only well-behaved cells and expressed genes enter normalization. Then normalize UMI counts with normalize_UMIs(object, scale.factor, use.scaled.factor), choosing a scale factor (e.g. 10,000) and whether to use a supplied fixed factor instead of a computed one. Regress out confounders (e.g. total UMIs, percent mitochondrial reads, cell-cycle score previously added via add_cell_cycle_genes_score) and retrieve the resulting matrix through get_regressout_log_data(object). The regressed log data is the required input for integrative variable gene selection via get_variable_genes_for_integrative_data_by_fitting_GLM_model and downstream PCA/UMAP/Louvain clustering; re-normalization and regression must be repeated on any merged/integrated object rather than reusing per-replicate values.

## Related tools

- **SingCellaR** (R package providing normalize_UMIs and get_regressout_log_data for UMI scaling and confounder regression) — github.com/supatt-lab/SingCellaR
- **R** (Environment in which the SingCellaR normalization/regression functions are executed)

## Examples

```
normalize_UMIs(object, scale.factor, use.scaled.factor); regressed <- get_regressout_log_data(object)
```

## Evaluation signals

- After normalization, per-cell UMI totals are comparable across cells (scaled to the chosen scale.factor), regardless of original library size
- The regressed log data matrix exists in the object and downstream PCA/UMAP show cells of the merged object clustering by cell type (erythroid, myeloid, lymphoid, megakaryocytic) rather than by sample of origin
- QC gates (min/max_UMIs, min/max_detected_genes, min/max_percent_mito, genes_with_expressing_cells threshold) were applied before normalization, since normalization on unfiltered data propagates outlier cells
- Variable genes fitted on the regressed data produce a sensible mean-expression/dispersion z-score distribution with no genes below mean_expr_cutoff driving clustering
- Saved integrated object (e.g. eFL_All.SingCellaR.rdata) reloads in R and reproduces the Figure 2-style UMAPs colored by cluster and sample

## Limitations

- SingCellaR lacks 'automatic object transformation' to interact with other packages such as Seurat, so regressed data cannot be directly piped into Seurat workflows
- UMAP/plot rotations can differ from the published protocol due to software versions and seed number settings
- Re-running normalization on a merged object requires re-applying QC, re-selecting variable genes, and re-clustering; per-replicate normalized values are not reusable as-is

## Evidence

- [methods] normalize_UMIs signature: "`normalize_UMIs(object, scale.factor, use.scaled.factor)`"
- [methods] QC filter before normalization: [evidence span withheld — anchor status: not_found]
- [readme] SingCellaR purpose: "an integrative analysis tool for analysing single-cell RNA sequencing data"
- [methods] clustering step downstream of normalization: "`identifyClusters(object, dim_reduction_method, useIntegrativeEmbeddings, integrative_method, n.dims.use, n.neighbors, knn.n_trees, knn.metric, knn.n_threads)`"
