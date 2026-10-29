---
name: stage-effect-correlation-matrix-analysis
description: Use when scRNA-seq pseudobulk data span an ordered trajectory (e.g. iPSC-to-endoderm
  stages CT1–CT4) and you need to test whether adjacent-stage interindividual correlations
  of cell type-specific expression effects exceed those between more distant stages.
when_to_use_negative:
- Sample sizes below ~50 individuals — the article states CTMM is not reliable for
  sample sizes below ~50
- Individual-cell type pairs with <=10 cells — ctp/ctnu must be set to missing for
  such pairs, so the skill is limited to common cell types
- Cell types expressed as continuous trajectories that cannot be meaningfully discretized
  into pseudobulk stages, since CTMM assumes discrete cell types
edam_operation: http://edamontology.org/operation_3657
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: ctmm
  role: Python package providing the ctp module and ctp.full_REML function used to
    fit the Full model and extract variance components (hom2, V)
  repo: https://github.com/Minhui-Chen/ctmm_python
- name: rpy2
  role: Bridge through which CTMM calls R routines during model fitting and optimization
  repo: https://rpy2.github.io/
- name: R
  role: Required runtime for the R routines (packages abind, mvtnorm, numDeriv, softImpute,
    tensor) invoked via rpy2
- name: Snakemake
  role: Workflow engine containing the steps for simulations and iPSCs analyses in
    the CTMM repository (Snakefile)
  repo: https://github.com/Minhui-Chen/CTMM
- name: Python
  role: Language in which the ctmm package is implemented and in which the correlation
    matrix is derived and written to CSV
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-49242-9
    title: A robust model for cell type-specific interindividual variation in single-cell
      RNA sequencing data
  source_content_hash: sha256:95f9865cff912b6df4beb8bdbf8bb515847e4956773a6854e54b5b5168e9f6d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Minhui-Chen/ctmm_python
  tools:
  - Python
  - R
  - Snakemake
  - ctmm
  - rpy2
  edam_operation: http://edamontology.org/operation_3657
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# stage-effect-correlation-matrix-analysis

## Summary

Fits the CTMM Full model by REML on cell type-specific pseudobulk (CTP) data to partition interindividual covariance between differentiation stages, then converts the partitioned covariance into a stage-by-stage correlation matrix that recapitulates the overall differentiation trajectory.

## When to use

Use when scRNA-seq pseudobulk data span an ordered trajectory (e.g. iPSC-to-endoderm stages CT1–CT4) and you need to test whether adjacent-stage interindividual correlations of cell type-specific expression effects exceed those between more distant stages.

## When NOT to use

- Sample sizes below ~50 individuals — the article states CTMM is not reliable for sample sizes below ~50
- Individual-cell type pairs with <=10 cells — ctp/ctnu must be set to missing for such pairs, so the skill is limited to common cell types
- Cell types expressed as continuous trajectories that cannot be meaningfully discretized into pseudobulk stages, since CTMM assumes discrete cell types

## Inputs

- CTP: cell type-specific pseudobulk gene expression file, one column per cell type/stage, no header (e.g. 'test/CTP.gz')
- ctnu: variance of measurement noise for each pair of individual and cell type, one column per cell type, no header (e.g. 'test/ctnu.gz')
- P: cell type proportions for each individual, one column per cell type, no header (e.g. 'test/P.gz')
- Cuomo et al. 2020 Nat Commun iPSC-to-endoderm differentiation scRNA-seq counts and metadata for pseudobulk generation

## Outputs

- First output dictionary with estimated model parameters: stage-shared random-effect variance (hom2), stage-specific covariance matrix (V), cell type-specific mean expression (beta), loglikelihood (l)
- Second output dictionary with p values from Wald tests (e.g. on ct_beta and V differentiation)
- 4x4 correlation matrix of stage-specific interindividual effects, written to stage_effect_correlation_matrix.csv

## How to apply

Load the CTP, ctnu, and P input files for the Cuomo et al. 2020 iPSC dataset, where the four 'cell types' correspond to four differentiation stages along the iPSC-to-endoderm trajectory. Import the ctmm ctp module and fit the Full model by restricted maximum likelihood via ctp.full_REML(y_f, P_f, ctnu_f, method='BFGS', optim_by_R=True). Extract the estimated variance components from the first output dictionary, including the stage-shared random-effect variance (hom2) and the stage-specific covariance matrix (V). Convert the partitioned interindividual covariance between stages into a 4x4 correlation matrix of stage-specific effects. Validate by checking that median adjacent-stage correlations (CT1-CT2 = 0.318, CT3-CT4 = 0.322) exceed the median CT2-CT3 correlation (0.051), consistent with rapid molecular changes at day 2, then write the fitted parameters and correlation matrix to stage_effect_correlation_matrix.csv.

## Related tools

- **ctmm** (Python package providing the ctp module and ctp.full_REML function used to fit the Full model and extract variance components (hom2, V)) — https://github.com/Minhui-Chen/ctmm_python
- **rpy2** (Bridge through which CTMM calls R routines during model fitting and optimization) — https://rpy2.github.io/
- **R** (Required runtime for the R routines (packages abind, mvtnorm, numDeriv, softImpute, tensor) invoked via rpy2)
- **Snakemake** (Workflow engine containing the steps for simulations and iPSCs analyses in the CTMM repository (Snakefile)) — https://github.com/Minhui-Chen/CTMM
- **Python** (Language in which the ctmm package is implemented and in which the correlation matrix is derived and written to CSV)

## Examples

```
from ctmm import ctp; full, p_wald = ctp.full_REML(y_f='CTP.gz', P_f='P.gz', ctnu_f='ctnu.gz', method='BFGS', optim_by_R=True)
```

## Evaluation signals

- Median adjacent-stage correlations CT1-CT2 = 0.318 and CT3-CT4 = 0.322 exceed the median CT2-CT3 correlation of 0.051, confirming the covariance partition recapitulates the differentiation trajectory
- The Full model converges under REML with BFGS optimization (optim_by_R=True) and returns a finite loglikelihood (l) in the first output dictionary
- The derived correlation matrix is 4x4, symmetric, with unit diagonal and off-diagonal entries in [-1, 1]
- Output CSV stage_effect_correlation_matrix.csv contains both the fitted parameters and the correlation matrix
- Variance components include a non-negative hom2 and a valid positive semi-definite stage-specific covariance matrix V

## Limitations

- The CT2-CT3 correlation (median 0.051) is much smaller than other adjacent stages, reflecting rapid molecular changes at day 2 — adjacent stages do not always show high correlation
- CTMM requires >10 cells per individual-cell type pair, limiting it to common cell types
- Lowly-expressed genes can be severely non-Gaussian, increasing the number of cells needed for the Gaussian approximation
- Pseudobulk inherently discards cell-level information, sacrificing statistical power and resolution within cell types

## Evidence

- [results] Full model with REML reproduces inter-stage correlation structure: [evidence span withheld — anchor status: not_found]
- [abstract] Covariance partition recapitulates differentiation trajectory: "we extend CTMM to partition interindividual covariance between stages, which recapitulates the overall differentiation trajectory."
- [readme] CTP fitting entry point: "To fit CTP, import the ``ctp`` module and call the function in the same way. For example, ``ctp.full_ML()`` to fit Full model using ML."
- [readme] Output dictionary contents: "The first one contains estimates of model parameters, including cell type-specific mean expression (beta), variance of cell type-shared random effect (hom2), variance of cell type-specific random"
- [discussion] Method reliability threshold: [evidence span withheld — anchor status: not_found]
