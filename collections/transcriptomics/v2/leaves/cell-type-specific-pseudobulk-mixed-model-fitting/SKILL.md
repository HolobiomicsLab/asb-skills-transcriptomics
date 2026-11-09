---
name: cell-type-specific-pseudobulk-mixed-model-fitting
description: Use when per-individual, per-cell-type scRNA-seq data must be aggregated
  into CTP/ctnu/P pseudobulk inputs and fit with CTMM linear mixed models to quantify
  or test cell type-specific interindividual variance (and its differentiation between
  cell types/stages).
when_to_use_negative:
- 'Cohorts with fewer than ~50 individuals: the Free test is slightly inflated and
  CTMM is not reliable below ~50 sample size'
- Rare cell types or individual-cell type pairs with <=10 cells, or very lowly expressed
  genes that are severely non-Gaussian for the Gaussian pseudobulk approximation
- Continuous cell trajectories that cannot be meaningfully discretized, since CTMM
  assumes discrete cell types and pseudobulk discards cell-level information
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ctmm
  role: Python package providing preprocess, op, ctp, and util modules to fit hom/iid/free/full
    models on OP/CTP inputs with ML/REML/HE and Wald or LRT tests
  repo: https://github.com/Minhui-Chen/ctmm_python
- name: rpy2
  role: Bridges ctmm to R routines (e.g. BFGS optimization via R optim); version must
    match installed R (rpy2 3.5.x for R 3.6-4.4, rpy2>=3.6 for R>=4.5)
- name: R
  role: Required runtime for ctmm's optimization routines; needs abind, mvtnorm, numDeriv,
    softImpute, tensor packages
- name: Snakemake
  role: Snakefile pipeline orchestrating simulations and iPSC analyses (bin/cuomo
    scripts) in the full CTMM repository
  repo: https://github.com/Minhui-Chen/CTMM
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-49242-9
    title: A robust model for cell type-specific interindividual variation in single-cell
      RNA sequencing data
  source_content_hash: sha256:74c7d5242493b6f5c57d0c610141ce37d588a3cf7589762ad1dbf98a9245d6e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Minhui-Chen/ctmm_python
  tools:
  - R
  - Snakemake
  - ctmm
  - mvtnorm
  - numDeriv
  - rpy2
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# cell-type-specific-pseudobulk-mixed-model-fitting

## Summary

Fits CTMM (Cell Type-specific linear Mixed Model) to cell type-specific pseudobulk (CTP) scRNA-seq data with hom/iid/free/full models under ML, REML, or HE to estimate and test cell type-specific interindividual variance and inter-stage covariance.

## When to use

Use when per-individual, per-cell-type scRNA-seq data must be aggregated into CTP/ctnu/P pseudobulk inputs and fit with CTMM linear mixed models to quantify or test cell type-specific interindividual variance (and its differentiation between cell types/stages).

## When NOT to use

- Cohorts with fewer than ~50 individuals: the Free test is slightly inflated and CTMM is not reliable below ~50 sample size
- Rare cell types or individual-cell type pairs with <=10 cells, or very lowly expressed genes that are severely non-Gaussian for the Gaussian pseudobulk approximation
- Continuous cell trajectories that cannot be meaningfully discretized, since CTMM assumes discrete cell types and pseudobulk discards cell-level information

## Inputs

- CTP file: cell type-specific pseudobulk gene expression, one column per cell type, no header (e.g. test/CTP.gz)
- ctnu file: measurement-noise variance for each individual-cell type pair, one column per cell type, no header
- P file: cell type proportions per individual, one column per cell type, no header
- counts and meta tables (Cuomo et al. 2020 Nat Commun iPSC scRNA-seq or similar) for pseudobulk generation
- Optional: pca file (fixed_covars_d) and batch file (random_covars_d)

## Outputs

- Two dictionaries per fit: parameter estimates (hom2, V, beta/ct_beta, loglikelihood l) and Wald p-values (ct_beta, V)
- Jackknife-based Wald p-values per gene when jack_knife=True
- Stage-to-stage interindividual correlation matrix from the Full model (e.g. 4x4 for CT1-CT4 differentiation stages)
- Variance-partition fractions (e.g. variance_partition_ipsc.csv) and ranked hit tables (e.g. ipsc_ctp_hit_table.csv)

## How to apply

Aggregate QC'd single-cell counts with preprocess.pseudobulk(counts, meta, ind_cut=100, ct_cut=10), which removes individuals with <=100 cells and sets ctp/ctnu to missing for individual-cell type pairs with <=10 cells; optionally rescale with preprocess.std to mean 0 and variance 1. Import the ctmm.ctp module and fit, e.g. ctp.free_REML(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f, method='BFGS', optim_by_R=True, jack_knife=True), using jack_knife=True for the jackknife-based Wald test recommended as most powerful and robust; use ctp.full_REML to partition interindividual covariance between cell types/stages into a correlation matrix, and ctp.hom_REML plus util.lrt(free['l'], hom['l'], C) for a likelihood-ratio test on variance differentiation. Add pseudobulk-expression PCs as fixed covariates and experiment batch as random covariates via fixed_covars_d and random_covars_d. Each fit returns two dictionaries: the first with model parameters (hom2 shared variance, V cell type-specific variance, beta/ct_beta means, loglikelihood l) and the second with Wald p-values on differentiation of mean (ct_beta) and variance (V). Multiple testing is controlled by Bonferroni correction across all tested genes (11,231 in the iPSC application), and candidates are sought significant under both REML (JK) and HE.

## Related tools

- **ctmm** (Python package providing preprocess, op, ctp, and util modules to fit hom/iid/free/full models on OP/CTP inputs with ML/REML/HE and Wald or LRT tests) — https://github.com/Minhui-Chen/ctmm_python
- **rpy2** (Bridges ctmm to R routines (e.g. BFGS optimization via R optim); version must match installed R (rpy2 3.5.x for R 3.6-4.4, rpy2>=3.6 for R>=4.5))
- **R** (Required runtime for ctmm's optimization routines; needs abind, mvtnorm, numDeriv, softImpute, tensor packages)
- **Snakemake** (Snakefile pipeline orchestrating simulations and iPSC analyses (bin/cuomo scripts) in the full CTMM repository) — https://github.com/Minhui-Chen/CTMM

## Examples

```
from ctmm import ctp; free, p_wald = ctp.free_REML(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f, method='BFGS', optim_by_R=True, jack_knife=True)
```

## Evaluation signals

- Output schema: each fit returns two dictionaries — the first containing hom2, V, beta, and loglikelihood l; the second containing Wald p-values for ct_beta and V
- iPSC variance partition reproduces reported fractions within REML convergence tolerance: median homogeneous variance 0.4%, median cell type-specific variance 32.8%, proportion differences 12%, residual cell-level variation 10%, PCs of pseudobulk expression 39%, batch effects 21%
- Full-model inter-stage correlation matrix shows adjacent-stage medians (CT1-CT2 = 0.318, CT3-CT4 = 0.322) exceeding the non-adjacent CT2-CT3 median (0.051), recapitulating the differentiation trajectory
- Candidate-gene screen after Bonferroni correction over 11,231 genes yields hits significant in both REML (JK) and HE, with POU5F1, HSPA8, and NDUFB4 at the top and exactly 85 genes significant for variance but not mean differentiation (p>0.01 in REML)
- LRT sanity check: util.lrt(free['l'], hom['l'], C) with C = number of cell types (e.g. C = 4) gives a calibrated p-value on variance differentiation (V = 0)

## Limitations

- Requires >10 cells per individual-cell type pair, limiting CTMM to common cell types; lowly-expressed genes can be severely non-Gaussian, increasing the number of cells needed for the Gaussian approximation
- Inferring cell types directly from the same scRNA-seq data induces circularity that deflates cell type-specific variance estimates; pseudobulk discards cell-level information, sacrificing power and within-cell-type resolution
- The iid model code is not thoroughly tested, and results require biological validation and replication due to slight test inflation

## Evidence

- [discussion] The key idea is to summarize the scRNA-seq data into cell type-specific pseudobulk: "The key idea is to summarize the scRNA-seq data into cell type-specific pseudobulk"
- [discussion] jackknife-based REML test: "propose a jackknife-based test using REML as the most powerful and robust method"
- [methods] pseudobulk preprocessing: "compute ctp (cell type-specific pseudobulk) and ctnu (cell type-specific noise variance) and P (cell type proportions)"
- [methods] filter thresholds: "preprocess.pseudobulk(counts=counts, meta=meta, ind_cut=100, ct_cut=10) # remove individuals with <= 100 cells"
- [readme] rpy2 requirement: "CTMM calls R routines through [rpy2](https://rpy2.github.io/), so a working R installation is required."
- [readme] reference CTP fit call: "free_jk, p_jk = ctp.free_REML(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f, method='BFGS', optim_by_R=True, jack_knife=True)"
