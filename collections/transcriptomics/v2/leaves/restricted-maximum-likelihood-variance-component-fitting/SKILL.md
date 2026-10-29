---
name: restricted-maximum-likelihood-variance-component-fitting
description: Use when partitioning interindividual variance of scRNA-seq gene expression
  across cell types from pseudobulk matrices (OP or CTP, ctnu, P) is needed, e.g.
  quantifying cell type-specific variance or stage-specific covariance along the iPSC-to-endoderm
  trajectory.
when_to_use_negative:
- 'Sample sizes below ~50 individuals: CTMM is not reliable there and the Free test
  is slightly inflated'
- 'Rare cell types with <= 10 cells per individual-cell type pair: CTMM requires >10
  cells, limiting it to common cell types'
- Continuous cell trajectories that cannot be reasonably discretized into cell types,
  since CTP pseudobulk expression requires discrete cell types
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: ctmm
  role: Python package providing op/ctp modules with [model]_[method] functions (e.g.
    ctp.free_REML) for fitting CTMM variance components
  repo: https://github.com/Minhui-Chen/ctmm_python
- name: rpy2
  role: Bridge through which CTMM calls R routines for BFGS optimization (optim_by_R=True)
  repo: https://rpy2.github.io/
- name: R
  role: Required runtime for the optimization and dependent packages (abind, mvtnorm,
    numDeriv, softImpute, tensor)
- name: Snakemake
  role: Snakefile drives the simulation and iPSCs analysis pipeline including REML
    fits
  repo: https://github.com/Minhui-Chen/CTMM
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-49242-9
    title: A robust model for cell type-specific interindividual variation in single-cell
      RNA sequencing data
  source_content_hash: sha256:8d902065d749eb0a6450c7a4d34683199467d59572f90fa48dea96dde9966974
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Minhui-Chen/ctmm_python
  tools:
  - R
  - Snakemake
  - ctmm
  - rpy2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# restricted-maximum-likelihood-variance-component-fitting

## Summary

Fit Cell Type-specific linear Mixed Models (CTMM) with REML (or ML/HE) via the ctmm Python package to estimate cell type-shared variance (hom2), cell type-specific variance (V), and mean effects (beta) from pseudobulk scRNA-seq data, as done in Chen & Dahl 2024 on Cuomo et al. 2020 iPSC differentiation data.

## When to use

Use when partitioning interindividual variance of scRNA-seq gene expression across cell types from pseudobulk matrices (OP or CTP, ctnu, P) is needed, e.g. quantifying cell type-specific variance or stage-specific covariance along the iPSC-to-endoderm trajectory.

## When NOT to use

- Sample sizes below ~50 individuals: CTMM is not reliable there and the Free test is slightly inflated
- Rare cell types with <= 10 cells per individual-cell type pair: CTMM requires >10 cells, limiting it to common cell types
- Continuous cell trajectories that cannot be reasonably discretized into cell types, since CTP pseudobulk expression requires discrete cell types

## Inputs

- CTP: cell type-specific pseudobulk gene expression matrix (individuals x cell types, no header)
- ctnu: variance of measurement noise per individual-cell type pair (individuals x cell types, no header)
- P: cell type proportions per individual (individuals x cell types, no header)
- OP and nu (overall pseudobulk and per-individual noise variance) when fitting the OP variant
- counts and meta tables from scRNA-seq (e.g. Cuomo et al. 2020 Nat Commun iPSC dataset) for preprocess.pseudobulk
- optional fixed_covars_d (PCA of pseudobulk expression) and random_covars_d (experiment batch) files

## Outputs

- Dictionary of fitted model parameters: beta (cell type-specific mean expression), hom2 (cell type-shared random-effect variance), V (cell type-specific random-effect variance/covariance matrix), and loglikelihood (l)
- Second dictionary of Wald p values on differentiation of mean expression (ct_beta) and of expression variance (V)
- Partitioned variance fractions (e.g. written to variance_partition_ipsc.csv) and 4x4 stage-effect correlation matrix (stage_effect_correlation_matrix.csv)

## How to apply

Compute cell type-specific pseudobulk (CTP), per individual-cell type noise variances (ctnu), and cell type proportions (P) with preprocess.pseudobulk, removing individuals with <= 100 cells (ind_cut=100) and individual-cell type pairs with <= 10 cells (ct_cut=10); optionally standardize with preprocess.std so the overall pseudobulk has mean 0 and variance 1. Then fit the model with REML using BFGS optimization delegated to R's optim (optim_by_R=True): ctp.free_REML for per-cell-type variance (used to show median cell type-specific variance of 32.8% in differentiating iPSCs) or ctp.full_REML for a full stage-specific covariance matrix (used to show adjacent-stage correlations CT1-CT2 = 0.318 and CT3-CT4 = 0.322 exceed the CT2-CT3 median of 0.051). Include additional fixed (PCA on pseudobulk expression) covariates via fixed_covars_d and random (experiment batch) covariates via random_covars_d. REML is preferred because the article's jackknife-based Wald test uses REML as the most powerful and robust approach; compare nested models via a likelihood-ratio test using the fitted loglikelihoods (util.lrt with C = number of cell types).

## Related tools

- **ctmm** (Python package providing op/ctp modules with [model]_[method] functions (e.g. ctp.free_REML) for fitting CTMM variance components) — https://github.com/Minhui-Chen/ctmm_python
- **rpy2** (Bridge through which CTMM calls R routines for BFGS optimization (optim_by_R=True)) — https://rpy2.github.io/
- **R** (Required runtime for the optimization and dependent packages (abind, mvtnorm, numDeriv, softImpute, tensor))
- **Snakemake** (Snakefile drives the simulation and iPSCs analysis pipeline including REML fits) — https://github.com/Minhui-Chen/CTMM

## Examples

```
from ctmm import op, ctp, util; free, p_wald = ctp.free_REML(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f, method='BFGS', optim_by_R=True)
```

## Evaluation signals

- Reproduced iPSC variance-partition fractions match reported values: median homogeneous variance 0.4%, median cell type-specific variance 32.8%, weighted cell type-specific variation 14%, proportion differences 12%, residual cell-level variation 10%, PCs of pseudobulk expression 39%, batch effects 21%, within REML numerical-convergence tolerance
- Full-model stage correlation matrix shows median adjacent-stage correlations CT1-CT2 = 0.318 and CT3-CT4 = 0.322 exceeding the median CT2-CT3 correlation of 0.051
- Output returns two dictionaries: parameter estimates (hom2, V, beta, l) and Wald p values (p_V, p_ct_beta); LRT on V=0 computed via util.lrt(free['l'], hom['l'], C) with C = number of cell types
- Input matrices satisfy preprocessing thresholds: individuals have >100 cells and every individual-cell type pair has >10 cells (missing ctp/ctnu otherwise)

## Limitations

- Codes for the iid model are not thoroughly tested
- Lowly-expressed genes can be severely non-Gaussian, increasing the number of cells needed for the Gaussian approximation
- Inferring cell types directly from scRNA-seq data induces circularity that can deflate estimates of cell type-specific variance
- Pseudobulk inherently discards cell-level information, sacrificing statistical power and resolution within cell types

## Evidence

- [methods] compute ctp (cell type-specific pseudobulk) and ctnu (cell type-specific noise variance) and P (cell type proportions): "compute ctp (cell type-specific pseudobulk) and ctnu (cell type-specific noise variance) and P (cell type proportions)"
- [discussion] REML most powerful and robust: "propose a jackknife-based test using REML as the most powerful and robust method"
- [readme] To fit CTP, import the ``ctp`` module and call the function in the same way. For example, ``ctp.full_ML()`` to fit Full model using ML.: "To fit CTP, import the ``ctp`` module and call the function in the same way. For example, ``ctp.full_ML()`` to fit Full model using ML."
- [methods] preprocessing cutoffs: "preprocess.pseudobulk(counts=counts, meta=meta, ind_cut=100, ct_cut=10) # remove individuals with <= 100 cells"
- [readme] output dictionaries: "The output of CTMM has two dictionaries."
