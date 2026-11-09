---
name: autoencoder-training-on-gene-expression
description: Use when a normalized, log1p-transformed cell-by-gene scRNA-seq matrix
  (e.g. Klein dataset, top 5000 Seurat HVGs) still contains dropouts and you need
  a statistically grounded imputation via a two-part-generalized-gamma autoencoder.
when_to_use_negative:
- The task is dimensionality reduction rather than imputation — for that the README
  uses the 68K PBMC example and saves adata_filtered.obsm['X_ae_tpgg'] instead of
  computing the TPGG mean.
- The input matrix is raw counts that have not been converted to semi-continuous form
  (normalize_per_cell + log1p), since AE-TPGG is designed for semi-continuous normalized
  data.
- The dataset has more than the selected highly variable gene set (e.g. full transcriptome
  without the Seurat top-5000 HVG filter), which the imputation example does not cover.
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: AE-TPGG
  role: Autoencoder model (Keras/TensorFlow) whose ae_tpgg() call fits the two-part-generalized-gamma
    parameters used to compute the imputed matrix
  repo: https://github.com/PUGEA/AE-TPGG
- name: Scanpy
  role: Loads and preprocesses the scRNA-seq data (sc.read_csv, normalize_per_cell,
    log1p)
  repo: https://github.com/theislab/scanpy
- name: Keras
  role: Framework in which the AE-TPGG autoencoder is implemented
  repo: https://github.com/keras-team/keras
- name: TensorFlow
  role: Backend for the Keras implementation of AE-TPGG
  repo: https://github.com/tensorflow/tensorflow
- name: Seurat
  role: Used in R (klein_filterHVG.R) to select the top 5000 highly variable genes
    before imputation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11704-022-2011-y
    title: 'AE-TPGG: a novel autoencoder-based approach for single-cell RNA-seq data
      imputation and dimensionality reduction'
  source_content_hash: sha256:95b20b973e9bb3a809138ef9b721c80611ed25d33ec3d0a89e1e8d41f92fdb76
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PUGEA/AE-TPGG
  tools:
  - AE-TPGG
  - Keras
  - Python 3
  - Scanpy
  - Seurat
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# autoencoder-training-on-gene-expression

## Summary

Train the AE-TPGG autoencoder (Keras with TensorFlow backend) on semi-continuous normalized scRNA-seq data to obtain fitted two-part-generalized-gamma parameters, then compute the TPGG mean as a denoised, dropout-free imputed expression matrix.

## When to use

Use when a normalized, log1p-transformed cell-by-gene scRNA-seq matrix (e.g. Klein dataset, top 5000 Seurat HVGs) still contains dropouts and you need a statistically grounded imputation via a two-part-generalized-gamma autoencoder.

## When NOT to use

- The task is dimensionality reduction rather than imputation — for that the README uses the 68K PBMC example and saves adata_filtered.obsm['X_ae_tpgg'] instead of computing the TPGG mean.
- The input matrix is raw counts that have not been converted to semi-continuous form (normalize_per_cell + log1p), since AE-TPGG is designed for semi-continuous normalized data.
- The dataset has more than the selected highly variable gene set (e.g. full transcriptome without the Seurat top-5000 HVG filter), which the imputation example does not cover.

## Inputs

- Filtered scRNA-seq count matrix as CSV (e.g. klein_filtered_data.csv with top 5000 highly variable genes selected using the Seurat package)
- AnnData object after sc.pp.normalize_per_cell and sc.pp.log1p preprocessing

## Outputs

- klein_data.X overwritten with the TPGG mean matrix (complete, dropout-free cell-by-gene numeric table)
- klein_imputation_data.csv written with pandas (sep=',', index=True, header=True)
- Fitted TPGG parameter matrices: beta, gamma, pi stored in klein_data.obsm['X_tpgg_beta'/'X_tpgg_gamma'/'X_tpgg_pi']

## How to apply

First load the filtered dataset (e.g. klein_filtered_data.csv) with Scanpy and convert counts to semi-continuous form via sc.pp.normalize_per_cell followed by sc.pp.log1p. Run the autoencoder with ae_tpgg(klein_data, threads=4, copy=True, log1p=False, return_info=True), which returns the fitted TPGG parameters: alpha in klein_data.X, beta in klein_data.obsm['X_tpgg_beta'], gamma in klein_data.obsm['X_tpgg_gamma'], and pi in klein_data.obsm['X_tpgg_pi']. Compute the TPGG mean elementwise for every cell-gene pair as (1-pi)*alpha * gamma(1/gamma + beta) / gamma(beta), i.e. the generalized-gamma mean weighted by the non-zero probability from the two-part model. Assign this mean matrix back to klein_data.X and export it with pandas to klein_imputation_data.csv (sep=',', index=True, header=True). The rationale is that the two-part model captures the semi-continuous (zero-inflated) nature of the data while the generalized-gamma distribution fits the positive, right-skewed continuous component.

## Related tools

- **AE-TPGG** (Autoencoder model (Keras/TensorFlow) whose ae_tpgg() call fits the two-part-generalized-gamma parameters used to compute the imputed matrix) — https://github.com/PUGEA/AE-TPGG
- **Scanpy** (Loads and preprocesses the scRNA-seq data (sc.read_csv, normalize_per_cell, log1p)) — https://github.com/theislab/scanpy
- **Keras** (Framework in which the AE-TPGG autoencoder is implemented) — https://github.com/keras-team/keras
- **TensorFlow** (Backend for the Keras implementation of AE-TPGG) — https://github.com/tensorflow/tensorflow
- **Seurat** (Used in R (klein_filterHVG.R) to select the top 5000 highly variable genes before imputation)

## Examples

```
klein_data = ae_tpgg(klein_data, threads=4, copy=True, log1p=False, return_info=True); tpgg_mean[i][j] = (1-pi_one)*alpha_one * gamma(1/gamma_one + beta_one) / gamma(beta_one)
```

## Evaluation signals

- The imputed matrix should be a complete (no-dropout) cell-by-gene numeric table of the same shape as the input (row_num x col_num from tpgg_alpha.shape).
- Every imputed entry must equal (1-pi)*alpha * gamma(1/gamma + beta) / gamma(beta) recomputed from the four parameter matrices, and should be non-negative.
- klein_imputation_data.csv must exist with comma separators, an index column, and a header row.
- Downstream clustering on the imputed Klein data should yield ACC, ARI, NMI, and F1 metrics comparable to the repository's Klein evaluation figure, improving on the original data.
- The tSNE visualization of the imputed Klein dataset should match the dropout-imputed pattern shown in the repository's klein_tsne_2.png figure.

## Limitations

- Feature selection is a prerequisite and differs by example: the Klein imputation example requires top 5000 highly variable genes selected with the Seurat package, while the 68K PBMC dimensionality-reduction example uses top 1000 HVGs via Scanpy's filter_genes_dispersion.
- The mean computation is implemented as a nested Python loop over row_num x col_num, which can be slow for large matrices.
- The ae_tpgg call must use log1p=False because the data were already log1p-transformed during preprocessing; double-log transformation would corrupt the input.

## Evidence

- [readme] AE-TPGG uses a two-part-generalized-gamma model to capture the statistical characteristics of semi-continuous normalized data and adaptively explores the potential relationships between genes: "AE-TPGG uses a two-part-generalized-gamma model to capture the statistical characteristics of semi-continuous normalized data and adaptively explores the potential relationships between genes"
- [readme] Select the top 5000 high variable genes by using the Seurat package: "Step 1. Feature selection. Select the top 5000 high variable genes by using the Seurat package"
- [readme] Run AE-TPGG model and save the imputed data with the ae_tpgg call returning info: "klein_data = ae_tpgg(klein_data, threads=4, copy=True, log1p=False, return_info=True)"
- [readme] Compute the TPGG mean elementwise from the fitted parameters: "tpgg_mean[i][j] = (1-pi_one)*alpha_one * gamma(1 / gamma_one + beta_one) / gamma(beta_one)"
- [abstract] The AE-TPGG model also provides a denoised imputation according to statistical characteristic of gene expression: "In addition to the ability of achieving low-dimensional representation, the AE-TPGG model also provides a denoised imputation according to statistical characteristic of gene expression."
