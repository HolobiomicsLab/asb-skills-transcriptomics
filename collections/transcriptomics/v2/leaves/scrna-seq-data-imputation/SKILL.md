---
name: scrna-seq-data-imputation
description: Use when a normalized, log1p-transformed scRNA-seq count matrix (e.g.
  Klein dataset, top 5000 HVGs) contains dropouts and a denoised, complete cell-by-gene
  expression matrix is needed for downstream clustering.
when_to_use_negative:
- The goal is dimensionality reduction rather than imputation — use the 68K PBMC workflow
  (filter_genes_dispersion, n_top_genes=1000, then save obsm['X_ae_tpgg']) instead
- The input is raw unnormalized counts that have not been converted to semi-continuous
  form (normalize_per_cell + log1p) — the TPGG model expects that normalized representation
- The dataset's gene-expression distribution assumptions differ (e.g. not positive,
  right-skewed continuous positive part) and a different imputation model is preferred
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: Scanpy
  role: Loads and pre-processes the scRNA-seq data (read_csv, normalize_per_cell,
    log1p) before imputation
  repo: https://github.com/theislab/scanpy
- name: Keras
  role: Implements the AE-TPGG autoencoder used to fit the two-part-generalized-gamma
    parameters
  repo: https://github.com/keras-team/keras
- name: TensorFlow
  role: Backend for the Keras autoencoder
  repo: https://github.com/tensorflow/tensorflow
- name: Seurat
  role: Selects the top 5000 highly variable genes for the Klein dataset prior to
    imputation (klein_filterHVG.R)
- name: AE-TPGG
  role: The ae_tpgg() function that fits the TPGG autoencoder and returns alpha, beta,
    gamma, pi
  repo: https://github.com/PUGEA/AE-TPGG
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1007/s11704-022-2011-y
    title: 'AE-TPGG: a novel autoencoder-based approach for single-cell RNA-seq data
      imputation and dimensionality reduction'
  source_content_hash: sha256:c12392a67433bcc3d7c6088126452dccdc4e6330f9369ac4d181dc0b73e5fd03
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/theislab/scanpy
  tools:
  - AE-TPGG
  - Keras
  - Python 3
  - Scanpy
  - Seurat
  - TensorFlow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# scrna-seq-data-imputation

## Summary

Impute dropout events in semi-continuous normalized scRNA-seq data with AE-TPGG, an autoencoder based on a two-part-generalized-gamma (GG) distribution, by computing the TPGG mean from the fitted alpha, beta, gamma, and pi parameter matrices.

## When to use

Use when a normalized, log1p-transformed scRNA-seq count matrix (e.g. Klein dataset, top 5000 HVGs) contains dropouts and a denoised, complete cell-by-gene expression matrix is needed for downstream clustering.

## When NOT to use

- The goal is dimensionality reduction rather than imputation — use the 68K PBMC workflow (filter_genes_dispersion, n_top_genes=1000, then save obsm['X_ae_tpgg']) instead
- The input is raw unnormalized counts that have not been converted to semi-continuous form (normalize_per_cell + log1p) — the TPGG model expects that normalized representation
- The dataset's gene-expression distribution assumptions differ (e.g. not positive, right-skewed continuous positive part) and a different imputation model is preferred

## Inputs

- klein_filtered_data.csv — Klein scRNA-seq matrix restricted to top 5000 highly variable genes (selected with Seurat)
- Raw count matrix rendered semi-continuous via Scanpy normalize_per_cell + log1p

## Outputs

- klein_imputation_data.csv — complete (no-dropout) imputed cell-by-gene numeric table with row index and header
- TPGG parameter matrices: alpha (X), beta (obsm['X_tpgg_beta']), gamma (obsm['X_tpgg_gamma']), pi (obsm['X_tpgg_pi']) on the AnnData object

## How to apply

Preprocess the raw counts into semi-continuous form with Scanpy (sc.pp.normalize_per_cell then sc.pp.log1p); for the Klein example the top 5000 highly variable genes are preselected with Seurat (klein_filterHVG.R). Run the Keras/TensorFlow autoencoder via ae_tpgg(klein_data, threads=4, copy=True, log1p=False, return_info=True), then extract alpha (klein_data.X), beta (obsm['X_tpgg_beta']), gamma (obsm['X_tpgg_gamma']), and pi (obsm['X_tpgg_pi']). The two-part model captures mixed discrete-continuous random variables, and the GG distribution fits the positive, right-skewed continuous component; impute each cell–gene pair as the TPGG mean (1-pi)*alpha * gamma(1/gamma + beta) / gamma(beta) using math.gamma. Assign the tpgg_mean matrix back to klein_data.X and export with pandas to klein_imputation_data.csv (sep=',', index=True, header=True).

## Related tools

- **Scanpy** (Loads and pre-processes the scRNA-seq data (read_csv, normalize_per_cell, log1p) before imputation) — https://github.com/theislab/scanpy
- **Keras** (Implements the AE-TPGG autoencoder used to fit the two-part-generalized-gamma parameters) — https://github.com/keras-team/keras
- **TensorFlow** (Backend for the Keras autoencoder) — https://github.com/tensorflow/tensorflow
- **Seurat** (Selects the top 5000 highly variable genes for the Klein dataset prior to imputation (klein_filterHVG.R))
- **AE-TPGG** (The ae_tpgg() function that fits the TPGG autoencoder and returns alpha, beta, gamma, pi) — https://github.com/PUGEA/AE-TPGG

## Examples

```
klein_data = ae_tpgg(klein_data, threads=4, copy=True, log1p=False, return_info=True); klein_data.X = tpgg_mean; klein_data_df.to_csv('./klein_imputation_data.csv', sep=',', index=True, header=True)
```

## Evaluation signals

- The imputed matrix is complete (no-dropout) with shape equal to the input cell-by-gene matrix
- Each imputed entry equals (1-pi)*alpha * gamma(1/gamma + beta) / gamma(beta) recomputed from the four fitted parameter matrices
- k-means clustering (k-means++ seeding, k = number of known cell types, labels aligned by optimal permutation) on the imputed matrix yields higher ACC, ARI, NMI, and F1 than on the original klein_filtered_data.csv, matching the Figure 3 pattern
- klein_imputation_data.csv is written with sep=',', index=True, header=True and is fully numeric

## Limitations

- The README's Klein imputation loop computes the TPGG mean element-by-element in nested Python for-loops over row_num x col_num, which is slow for large matrices
- Imputation requires the semi-continuous normalized representation (log1p=False must be passed to ae_tpgg since preprocessing is done beforehand); applying it to other distributional regimes is untested here
- Validation of improvement relies on the clustering-metric comparison (Figure 3); no per-gene accuracy guarantee is provided

## Evidence
