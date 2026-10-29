---
name: reml-mixed-model-fitting
description: Use when scRNA-seq data from multiple donors has been aggregated into
  cell type-specific pseudobulk (CTP), ctnu, and cell type proportions P, and the
  goal is to partition interindividual expression variance into cell type-shared (hom2)
  vs cell type-specific (V) components and test genes for.
when_to_use_negative:
- Cohorts with fewer than ~50 individuals — 'CTMM is not reliable for sample sizes
  below ~50'
- Individual-cell type pairs with <=10 cells or rare cell types, since CTMM requires
  >10 cells per pair and is limited to common cell types
- Lowly-expressed genes that are severely non-Gaussian even after pseudobulking, where
  the Gaussian approximation fails
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ctmm
  role: Python package providing preprocess.pseudobulk, preprocess.std, and ctp.free_REML
    with jack_knife for REML fitting and jackknife-based Wald tests
  repo: https://github.com/Minhui-Chen/ctmm_python
- name: rpy2
  role: Bridge through which CTMM calls R routines; must be a version matching the
    installed R (3.5.x for R 3.6–4.4)
  repo: https://rpy2.github.io/
- name: R
  role: Runtime required for the optimization and jackknife routines CTMM calls; needs
    abind, mvtnorm, numDeriv, softImpute, tensor packages
- name: Python
  role: Host language for the ctmm package and the analysis scripts in the CTMM repository
  repo: https://github.com/Minhui-Chen/CTMM
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-49242-9
    title: A robust model for cell type-specific interindividual variation in single-cell
      RNA sequencing data
  source_content_hash: sha256:4e6c25cdb79c4c27745459edb4e657cbc1935488c2291b9ae5e4c403541f564f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Minhui-Chen/ctmm_python
  tools:
  - Python
  - R
  - ctmm
  - rpy2
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# reml-mixed-model-fitting

## Summary

Fit Cell Type-specific linear Mixed Models (CTMM) to cell type-specific pseudobulk scRNA-seq data using REML optimization, and run jackknife-based Wald tests to detect genes with cell type-specific interindividual variance (e.g. 2310/11,526 genes in the OneK1K PBMC cohort).

## When to use

Use when scRNA-seq data from multiple donors has been aggregated into cell type-specific pseudobulk (CTP), ctnu, and cell type proportions P, and the goal is to partition interindividual expression variance into cell type-shared (hom2) vs cell type-specific (V) components and test genes for variance differentiation.

## When NOT to use

- Cohorts with fewer than ~50 individuals — 'CTMM is not reliable for sample sizes below ~50'
- Individual-cell type pairs with <=10 cells or rare cell types, since CTMM requires >10 cells per pair and is limited to common cell types
- Lowly-expressed genes that are severely non-Gaussian even after pseudobulking, where the Gaussian approximation fails

## Inputs

- Cell-level gene counts matrix (genes x cells) from QC'd, normalized scRNA-seq
- Cell metadata with individual ID and cell type (meta: cell, ind, ct)
- CTP file: cell type-specific pseudobulk expression, one column per cell type, no header
- ctnu file: measurement noise variance per individual-cell type pair, one column per cell type
- P file: cell type proportions per individual, one column per cell type

## Outputs

- First output dictionary: model parameter estimates (beta, hom2 = cell type-shared random effect variance, V = cell type-specific random effect variance, loglikelihood l)
- Second output dictionary: Wald-test p-values for differentiation of expression variance (V) and mean expression (ct_beta) per gene
- Counts of significant genes at the Bonferroni threshold (e.g. 2310/11,526 in OneK1K PBMCs)

## How to apply

Aggregate cell-level counts with preprocess.pseudobulk(counts, meta, ind_cut=100, ct_cut=10), which removes individuals with <=100 cells and sets ctp/ctnu to missing for individual-cell type pairs with <=10 cells, then standardize with preprocess.std(ctp, ctnu, P). Fit the Free model with ctp.free_REML(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f, method='BFGS', optim_by_R=True, jack_knife=True); the first output dictionary holds parameter estimates (hom2, V, beta, loglikelihood l) and the second holds p_wald p-values for variance differentiation (V) and mean expression differentiation (ct_beta). The jackknife-based Wald test with REML is the most powerful and robust configuration per the authors; a likelihood-ratio test via util.lrt(free['l'], hom['l'], C) on ctp.hom_REML is an alternative for V=0. Additional fixed (e.g. PCA on OP) and random (e.g. experiment batch) covariates can be supplied via fixed_covars_d and random_covars_d. In the OneK1K application this workflow identified 2310 of 11,526 genes significant for cell type-specific variance (Bonferroni p < 0.05/11526), with RPS26 as the top signal (p=1.78e-167) and a transcriptome-wide variance split of 10.9% shared vs 21.5% cell type-specific.

## Related tools

- **ctmm** (Python package providing preprocess.pseudobulk, preprocess.std, and ctp.free_REML with jack_knife for REML fitting and jackknife-based Wald tests) — https://github.com/Minhui-Chen/ctmm_python
- **rpy2** (Bridge through which CTMM calls R routines; must be a version matching the installed R (3.5.x for R 3.6–4.4)) — https://rpy2.github.io/
- **R** (Runtime required for the optimization and jackknife routines CTMM calls; needs abind, mvtnorm, numDeriv, softImpute, tensor packages)
- **Python** (Host language for the ctmm package and the analysis scripts in the CTMM repository) — https://github.com/Minhui-Chen/CTMM

## Examples

```
from ctmm import ctp; free_jk, p_jk = ctp.free_REML(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f, method='BFGS', optim_by_R=True, jack_knife=True)
```

## Evaluation signals

- Replicate the OneK1K PBMC result: 2310 of 11,526 tested genes significant for cell type-specific variance at Bonferroni p < 0.05/11526
- Top signal should be RPS26 with p = 1.78e-167 when the workflow is run on OneK1K
- Transcriptome-wide variance split should recover ~10.9% cell type-shared (hom2) vs ~21.5% cell type-specific (V)
- Both output dictionaries are returned: first with hom2/V/beta/l estimates, second with p_wald entries for V and ct_beta
- Individual-cell type pairs failing the >10-cell filter must be set to missing before fitting

## Limitations

- CTMM requires >10 cells per individual-cell type pair, limiting it to common cell types
- Lowly-expressed genes can be severely non-Gaussian, increasing the number of cells needed for the Gaussian approximation
- CTMM is not reliable for sample sizes below ~50 individuals, and the Free test is slightly inflated
- Pseudobulk discards cell-level information, sacrificing statistical power and resolution within cell types; discretization of continuous cell types is required
- Circularity from inferring cell types directly from scRNA-seq data can deflate estimates of cell type-specific variance by construction

## Evidence

- [discussion] jackknife-based test using REML is the most powerful and robust method: "propose a jackknife-based test using REML as the most powerful and robust method"
- [methods] Remove individuals with <=100 cells and pairs with <=10 cells before pseudobulk aggregation: "preprocess.pseudobulk(counts=counts, meta=meta, ind_cut=100, ct_cut=10) # remove individuals with <= 100 cells"
- [readme] Free REML fit with jackknife flag on CTP data: "free_jk, p_jk = ctp.free_REML(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f, method='BFGS', optim_by_R=True, jack_knife=True)"
- [discussion] >10 cells per pair required, limiting applicability to common cell types: "In practice, we require >10 cells per individual-cell type pair, which limits CTMM to common cell types."
- [readme] Second output dictionary holds Wald p-values for V and ct_beta: "The second one contains p values from Wald test on e.g. differentiation of mean expression between cell types (ct_beta) and differentiation of expression variance between cell types (V)."
- [readme] Additional covariates can be included in the fit: "``fixed_covars_d``  and ``random_covars_d`` to include additional fixed and random effects"
