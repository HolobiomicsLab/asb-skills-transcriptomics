---
name: elementwise-statistical-mean-computation
description: Use when AE-TPGG has been run with return_info=True on semi-continuous
  normalized scRNA-seq data (e.g. per-cell normalized and log1p-transformed Klein
  data) and the alpha/beta/gamma/pi parameter matrices must be converted into a dropout-free
  imputed expression matrix.
when_to_use_negative:
- When the goal is dimensionality reduction (e.g. on the 68K PBMC dataset), where
  the ae_tpgg latent embedding in obsm['X_ae_tpgg'] is saved directly instead of computing
  the TPGG mean.
- When raw count data has not been converted to semi-continuous form (per-cell normalization
  + log1p); the TPGG parameters are fitted to semi-continuous normalized data, not
  raw counts.
- When only the top 1000 highly variable genes selected via Scanpy filter_genes_dispersion
  are needed (the 68K dimensionality-reduction workflow), rather than the Klein imputation
  workflow's Seurat-selected top 5000 genes.
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: AE-TPGG
  role: Provides the fitted two-part-generalized-gamma parameter matrices (alpha,
    beta, gamma, pi) whose elementwise mean is computed for imputation
  repo: https://github.com/PUGEA/AE-TPGG
- name: Scanpy
  role: Loads and pre-processes the scRNA-seq data (sc.read_csv, sc.pp.normalize_per_cell,
    sc.pp.log1p) and hosts the AnnData object carrying the TPGG parameters in .X and
    .obsm
  repo: https://github.com/theislab/scanpy
- name: Keras
  role: Implements the AE-TPGG autoencoder that produces the TPGG parameters (with
    TensorFlow backend)
  repo: https://github.com/keras-team/keras
- name: TensorFlow
  role: Backend for the Keras autoencoder used to fit the TPGG parameters
  repo: https://github.com/tensorflow/tensorflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11704-022-2011-y
    title: 'AE-TPGG: a novel autoencoder-based approach for single-cell RNA-seq data
      imputation and dimensionality reduction'
  source_content_hash: sha256:8b319c4ab0f5a27970a1476b3f0fa6f96e2c7bf897985ea5709f416f6b862007
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/PUGEA/AE-TPGG
  tools:
  - AE-TPGG
  - Keras
  - Python 3
  - Scanpy
  - TensorFlow
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# Elementwise statistical mean computation

## Summary

Computes a complete, denoised expression matrix by evaluating the two-part-generalized-gamma (TPGG) mean elementwise from the fitted alpha, beta, gamma, and pi parameter matrices output by AE-TPGG. This converts the model's distributional parameters into the imputed cell-by-gene values used for downstream clustering and evaluation.

## When to use

Use when AE-TPGG has been run with return_info=True on semi-continuous normalized scRNA-seq data (e.g. per-cell normalized and log1p-transformed Klein data) and the alpha/beta/gamma/pi parameter matrices must be converted into a dropout-free imputed expression matrix.

## When NOT to use

- When the goal is dimensionality reduction (e.g. on the 68K PBMC dataset), where the ae_tpgg latent embedding in obsm['X_ae_tpgg'] is saved directly instead of computing the TPGG mean.
- When raw count data has not been converted to semi-continuous form (per-cell normalization + log1p); the TPGG parameters are fitted to semi-continuous normalized data, not raw counts.
- When only the top 1000 highly variable genes selected via Scanpy filter_genes_dispersion are needed (the 68K dimensionality-reduction workflow), rather than the Klein imputation workflow's Seurat-selected top 5000 genes.

## Inputs

- Fitted TPGG alpha matrix (klein_data.X after ae_tpgg call)
- Fitted TPGG beta matrix (klein_data.obsm['X_tpgg_beta'])
- Fitted TPGG gamma matrix (klein_data.obsm['X_tpgg_gamma'])
- Fitted TPGG pi zero-inflation matrix (klein_data.obsm['X_tpgg_pi'])

## Outputs

- Complete cell-by-gene imputed expression matrix (assigned to klein_data.X)
- CSV file klein_imputation_data.csv (sep=',', index=True, header=True)

## How to apply

After calling ae_tpgg(klein_data, threads=4, copy=True, log1p=False, return_info=True), extract the four fitted TPGG parameter matrices: alpha from klein_data.X, beta from klein_data.obsm['X_tpgg_beta'], gamma from klein_data.obsm['X_tpgg_gamma'], and pi from klein_data.obsm['X_tpgg_pi']. For every cell–gene pair (i, j), compute the TPGG mean as (1-pi[i][j]) * alpha[i][j] * gamma(1/gamma[i][j] + beta[i][j]) / gamma(beta[i][j]), where gamma() is the mathematical gamma function imported from Python's math module (note the name clash with the TPGG gamma parameter matrix). The two-part weighting by (1-pi) accounts for the zero-inflation component while the generalized-gamma mean models the positive, right-skewed continuous expression part. Assign the resulting mean matrix to klein_data.X and export it with pandas to_csv (sep=',', index=True, header=True), e.g. as klein_imputation_data.csv.

## Related tools

- **AE-TPGG** (Provides the fitted two-part-generalized-gamma parameter matrices (alpha, beta, gamma, pi) whose elementwise mean is computed for imputation) — https://github.com/PUGEA/AE-TPGG
- **Scanpy** (Loads and pre-processes the scRNA-seq data (sc.read_csv, sc.pp.normalize_per_cell, sc.pp.log1p) and hosts the AnnData object carrying the TPGG parameters in .X and .obsm) — https://github.com/theislab/scanpy
- **Keras** (Implements the AE-TPGG autoencoder that produces the TPGG parameters (with TensorFlow backend)) — https://github.com/keras-team/keras
- **TensorFlow** (Backend for the Keras autoencoder used to fit the TPGG parameters) — https://github.com/tensorflow/tensorflow

## Examples

```
tpgg_mean = (1 - tpgg_pi) * tpgg_alpha * gamma(1 / tpgg_gamma + tpgg_beta) / gamma(tpgg_beta); klein_data.X = tpgg_mean; pd.DataFrame(klein_data.X).to_csv('./klein_imputation_data.csv', sep=',', index=True, header=True)
```

## Evaluation signals

- The imputed matrix must be a complete (no-dropout) cell-by-gene numeric table with the same shape (row_num, col_num) as the input Klein data.
- Every imputed value must be finite and non-negative, since the TPGG mean (1-pi)*alpha * gamma(1/gamma + beta) / gamma(beta) is a non-negative expectation.
- The output CSV klein_imputation_data.csv must open as a proper pandas DataFrame with row indices and column headers (index=True, header=True).
- Downstream clustering on the imputed Klein data should yield ACC, ARI, NMI, and F1 metrics reported against the original data, as in the repository's Klein evaluation figure.
- The computed matrix should differ from the raw normalized/log1p input (values correspond to distribution means, not observed expression), confirming the imputation actually replaced dropout entries.

## Limitations

- The math.gamma import shadows the TPGG gamma parameter name; care is needed to distinguish the scalar gamma function from the tpgg_gamma matrix.
- The README's reference implementation uses nested Python loops over row_num x col_num, which is slow for large cell-by-gene matrices (Klein: 5000 highly variable genes).
- The imputation is fitted to semi-continuous normalized (per-cell normalized, log1p) data with log1p=False passed to ae_tpgg to avoid double transformation; deviating from this preprocessing invalidates the parameter interpretation.

## Evidence

- [readme] TPGG mean formula from README imputation example: "tpgg_mean[i][j] = (1-pi_one)*alpha_one * gamma(1 / gamma_one + beta_one) / gamma(beta_one)"
- [readme] ae_tpgg invocation with return_info to expose parameters: "klein_data = ae_tpgg(klein_data, threads=4, copy=True, log1p=False, return_info=True)"
- [readme] Saving the imputed matrix: "klein_data.X = tpgg_mean"
- [intro] Article rationale for the two-part-gamma model: "AE-TPGG uses a two-part-generalized-gamma model to capture the statistical characteristics of semi-continuous normalized data and adaptively explores the potential relationships between genes for"
- [abstract] Abstract statement of denoised imputation: "In addition to the ability of achieving low-dimensional representation, the AE-TPGG model also provides a denoised imputation according to statistical characteristic of gene expression."
