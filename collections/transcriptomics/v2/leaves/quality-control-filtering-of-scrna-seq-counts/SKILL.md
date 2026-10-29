---
name: quality-control-filtering-of-scrna-seq-counts
description: Use when a raw gene-by-cell count matrix is about to be fed into scATOMIC
  (or similar hierarchical classifiers), to drop cells whose mitochondrial read fraction
  exceeds 25% or which express fewer than 500 unique features.
when_to_use_negative:
- Input is already a QC-filtered or processed expression matrix (e.g. normalised/log-transformed
  data), where re-filtering with raw-count thresholds is inappropriate
- Running scATOMIC with the normal_tissue parameter on normal tissue samples where
  the default cancer-cell assumptions — and the standard QC targets tuned for tumour
  biopsies — may need re-examination
- Datasets where mitochondrial-gene annotation is missing, making the 25% mitochondrial-read
  filter uncomputable
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scATOMIC
  role: Downstream classifier whose methods specify the recommended QC thresholds
    (25% mitochondrial reads, 500 unique features); operates on sparse gene-by-cell
    count matrices
  repo: https://github.com/abelson-lab/scATOMIC
- name: Seurat
  role: Optional source of raw count matrices via seurat_object@assays$RNA@counts
    for filtering
- name: R
  role: Environment in which the sparse-matrix conversion and per-cell QC filtering
    are performed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-37353-8
    title: Pan-cancer classification of single cells in the tumour microenvironment
  source_content_hash: sha256:850baf38fde15f139bb99b2675c9f5042737c79605ba4023da8cf7481e5d0e56
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abelson-lab/scATOMIC
  tools:
  - R
  - Rmagic
  - Seurat
  - scATOMIC
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Quality-control filtering of scRNA-seq count matrices

## Summary

Removes low-quality cells from gene-by-cell scRNA-seq count matrices prior to scATOMIC classification using the paper's recommended thresholds: exclude cells with more than 25% mitochondrial reads or fewer than 500 unique features expressed.

## When to use

Use when a raw gene-by-cell count matrix is about to be fed into scATOMIC (or similar hierarchical classifiers), to drop cells whose mitochondrial read fraction exceeds 25% or which express fewer than 500 unique features.

## When NOT to use

- Input is already a QC-filtered or processed expression matrix (e.g. normalised/log-transformed data), where re-filtering with raw-count thresholds is inappropriate
- Running scATOMIC with the normal_tissue parameter on normal tissue samples where the default cancer-cell assumptions — and the standard QC targets tuned for tumour biopsies — may need re-examination
- Datasets where mitochondrial-gene annotation is missing, making the 25% mitochondrial-read filter uncomputable

## Inputs

- Gene-by-cell raw scRNA-seq count matrix (dense or sparse) per sample
- Seurat object with RNA assay counts (optional source format)
- List of mitochondrial gene identifiers for the genome build used

## Outputs

- Filtered sparse count matrix containing only cells passing pct_mt <= 25% and nFeatures >= 500
- Per-sample QC summary of retained vs removed cells (implicit)

## How to apply

Load each sample's raw count matrix (ideally as a sparse matrix via as(as.matrix(input_matrix), "sparseMatrix") or from a Seurat object's @assays$RNA@counts slot). Compute the fraction of reads mapping to mitochondrial genes per cell and the number of unique features expressed per cell. Remove any cell with more than 25% mitochondrial reads and any cell with fewer than 500 expressed unique features — these are the strongly recommended defaults from the scATOMIC methods. In the breast cancer subtype validation task (task_005), this filtering was applied to the 38-tumour validation cohort and 2 double-positive samples before running run_scATOMIC(). Rationale: low-quality cells carry noisy profiles that degrade random-forest-based hierarchical annotation and downstream subtype calls.

## Related tools

- **scATOMIC** (Downstream classifier whose methods specify the recommended QC thresholds (25% mitochondrial reads, 500 unique features); operates on sparse gene-by-cell count matrices) — https://github.com/abelson-lab/scATOMIC
- **Seurat** (Optional source of raw count matrices via seurat_object@assays$RNA@counts for filtering)
- **R** (Environment in which the sparse-matrix conversion and per-cell QC filtering are performed)

## Examples

```
sparse_matrix <- as(as.matrix(input_matrix), "sparseMatrix"); filtered <- sparse_matrix[, pct_mt <= 25 & colSums(sparse_matrix > 0) >= 500]
```

## Evaluation signals

- No retained cell has >25% of reads mapping to mitochondrial genes
- No retained cell expresses fewer than 500 unique features
- Filtered matrices remain gene-by-cell with identical gene identifiers to the input, only cell columns removed
- Downstream run_scATOMIC() produces non-trivial classifications (cells assigned across the pan-cancer hierarchy) rather than being dominated by low-quality dropouts
- In the benchmark setting, per-sample subtype aggregation over filtered cells reproduces 37/38 (97.4%) correct ER+/HER2+/TN calls versus immunostaining ground truth

## Limitations

- Thresholds (25% mitochondrial, 500 features) are recommendations tuned for tumour-biopsy scRNA-seq; other tissue types or chemistries may require different cutoffs
- QC filtering alone does not address doublets or ambient RNA; scATOMIC assumes samples are individual patient/biopsy samples and the default model assumes at least some cancer cells are present
- Sparse-matrix conversion is recommended for performance; running on dense matrices of >15,000 cells may exceed standard 16GB RAM

## Evidence

- [methods] Mitochondrial and feature-count filter thresholds: "We recommend to remove any cells with more than 25% of reads mapping to mitochondrial genes or those with fewer than 500"
- [methods] Sparse matrix conversion rationale: "We recommend running scATOMIC on sparse matrices to speed up performance and save memory."
- [readme] Per-sample application recommendation: "We strongly recommend running scATOMIC only on individual patient/biopsy samples."
- [supplementary] Filtering precedes run_scATOMIC in the validation workflow: [evidence span withheld — anchor status: not_found]
