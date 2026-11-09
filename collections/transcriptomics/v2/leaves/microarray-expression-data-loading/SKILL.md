---
name: microarray-expression-data-loading
description: Use when a transcriptomic regression/classification task needs a ready-made
  high-dimensional microarray expression matrix (p >> n, e.g. 6283 genes x 116 AML
  samples) packaged in the suffpcr R package rather than raw array files.
when_to_use_negative:
- Input is raw microarray files (e.g. CEL/IDAT) or RNA-seq counts requiring preprocessing/normalization
  — suffpcr ships already-processed expression matrices, not raw arrays.
- The analysis needs a dataset other than the five packaged transcriptomic sets (aml,
  dlbcl, miller, nsclc, vantveer); no loader for external expression formats is provided.
- The goal is low-dimensional data where p is not much larger than n — the packaged
  datasets and downstream SuffPCR/screening logic target high-dimensional p >> n regimes.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: suffpcr
  role: Provides the packaged microarray expression datasets (aml, dlbcl, miller,
    nsclc, vantveer) loaded by this skill; also implements the downstream SuffPCR
    analysis
  repo: https://github.com/dajmcdon/suffpcr
- name: remotes
  role: Installs the suffpcr package from GitHub so its bundled datasets become available
- name: R
  role: Runtime environment in which the packaged datasets are loaded and the expression
    matrix and covariance Φ are computed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbac033
    title: Sufficient principal component regression for pattern discovery in transcriptomic
      data
  source_content_hash: sha256:b4c1f97d2b5089c68e0c2bbc3fc7fae005e21a72d18ace44a1b2831ce7618831
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dajmcdon/suffpcr
  tools:
  - R
  - remotes
  - stats
  - suffpcr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# microarray-expression-data-loading

## Summary

Load the pre-packaged transcriptomic microarray datasets (aml, dlbcl, miller, nsclc, vantveer) shipped with the suffpcr R package to obtain gene-expression matrices and phenotype vectors, e.g. the AML data with 116 observations across 6283 genes used in the screening-counterexample analysis.

## When to use

Use when a transcriptomic regression/classification task needs a ready-made high-dimensional microarray expression matrix (p >> n, e.g. 6283 genes x 116 AML samples) packaged in the suffpcr R package rather than raw array files.

## When NOT to use

- Input is raw microarray files (e.g. CEL/IDAT) or RNA-seq counts requiring preprocessing/normalization — suffpcr ships already-processed expression matrices, not raw arrays.
- The analysis needs a dataset other than the five packaged transcriptomic sets (aml, dlbcl, miller, nsclc, vantveer); no loader for external expression formats is provided.
- The goal is low-dimensional data where p is not much larger than n — the packaged datasets and downstream SuffPCR/screening logic target high-dimensional p >> n regimes.

## Inputs

- R session (>= installation via remotes from GitHub dajmcdon/suffpcr)
- suffpcr packaged microarray datasets: aml, dlbcl, miller, nsclc, vantveer
- aml dataset: 116 observations x 6283 genes expression matrix plus phenotype labels

## Outputs

- gene-expression matrix X (n x p, e.g. 116 x 6283 for aml)
- phenotype/class vector y for regression or classification
- empirical covariance matrix Φ (rank-deficient, top n=116 entries retained) for sparse precision estimation

## How to apply

Install the suffpcr package from GitHub with remotes::install_github("dajmcdon/suffpcr") and attach it with library(suffpcr). Load one of the packaged datasets — aml, dlbcl, miller, nsclc, or vantveer — via data(), extracting the expression matrix X (for aml: 116 observations x 6283 genes) and the phenotype vector y. Because p >> n, the empirical sample covariance matrix Φ computed from the loaded data is rank-deficient; retain only the top n (e.g. 116) entries of Φ before estimating the sparse precision matrix Σ^-1 with Graphical Lasso. The loaded expression data then feeds pseudo-true coefficient construction β* = Σ^-1 Φ for downstream screening or SuffPCR evaluation.

## Related tools

- **suffpcr** (Provides the packaged microarray expression datasets (aml, dlbcl, miller, nsclc, vantveer) loaded by this skill; also implements the downstream SuffPCR analysis) — https://github.com/dajmcdon/suffpcr
- **remotes** (Installs the suffpcr package from GitHub so its bundled datasets become available)
- **R** (Runtime environment in which the packaged datasets are loaded and the expression matrix and covariance Φ are computed)

## Examples

```
remotes::install_github("dajmcdon/suffpcr"); library(suffpcr); data(aml)
```

## Evaluation signals

- data(aml) loads without error and the expression matrix has the expected 116 observations x 6283 genes dimensions
- The suffpcr package installs from github.com/dajmcdon/suffpcr via remotes and library(suffpcr) attaches cleanly
- The empirical covariance Φ of the loaded aml data is rank-deficient (rank at most ~115 given n=116, p=6283), confirming the p >> n regime
- All five documented datasets (aml, dlbcl, miller, nsclc, vantveer) resolve via the documented `aml (suffpcr)`-style accessors
- The phenotype vector y is available for classification/regression, matching the suffPCR(X, y, d=1:3) calling convention

## Limitations

- Only the five packaged datasets are available through this skill; no mechanism for loading external expression matrices is documented.
- The aml sample covariance is rank-deficient (116 samples, 6283 genes), so only the top n=116 entries of Φ are retained before precision-matrix estimation — a caveat for any downstream covariance use.
- No changelog found for the repository, so dataset-version drift across package updates is untracked.

## Evidence

- [abstract] Code and raw data are freely available at https://github.com/dajmcdon/suffpcr.: "Code and raw data are freely available at https://github.com/dajmcdon/suffpcr."
- [methods] remotes::install_github("dajmcdon/suffpcr"): "remotes::install_github("dajmcdon/suffpcr")"
- [readme] The goal of suffpcr is to estimate sufficient principal component regression by implementing row-sparse PCA followed by ordinary least squares or logistic regression.: "The goal of suffpcr is to estimate sufficient principal component regression by implementing row-sparse PCA followed by ordinary least squares or logistic regression."
