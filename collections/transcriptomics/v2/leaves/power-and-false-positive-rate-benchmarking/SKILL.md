---
name: power-and-false-positive-rate-benchmarking
description: Use when you need to quantify how the power and type-I error of variance-differentiation
  tests (LRT, Wald, jackknife Wald) for cell type-specific pseudobulk data depend
  on sample size and measurement-noise misestimation, e.g. reproducing Figures 1-2
  of the CTMM paper.
when_to_use_negative:
- When the goal is inference on real scRNA-seq data rather than evaluating the statistical
  operating characteristics of the tests via simulation
- When sample sizes are below ~50, where the article warns the Free test is slightly
  inflated and CTMM is not reliable
- 'When cell types are rare: CTMM requires >10 cells per individual-cell type pair,
  limiting it to common cell types'
edam_operation: http://edamontology.org/operation_3454
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_2269
tools:
- name: Snakemake
  role: Drives the simulation pipeline via the Snakefile for generating OP/CTP data
    across sample sizes and noise levels
  repo: https://github.com/Minhui-Chen/CTMM
- name: ctmm
  role: Python package providing op and ctp modules (e.g. ctp.free_REML, ctp.hom_REML)
    and util.lrt for fitting models and computing LRT p values
  repo: https://github.com/Minhui-Chen/ctmm_python
- name: rpy2
  role: Bridges Python ctmm calls to R routines; optimization via R optim with method='BFGS'
    and optim_by_R=True
  repo: https://rpy2.github.io/
- name: R
  role: Required runtime for ctmm's optimization and R packages (abind, mvtnorm, numDeriv,
    softImpute, tensor) used by the routines
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-49242-9
    title: A robust model for cell type-specific interindividual variation in single-cell
      RNA sequencing data
  source_content_hash: sha256:fa19d9cc3ccd6de321f92690a0929390ed75a40be9e7efb9438558f9b8e4a90e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Minhui-Chen/CTMM
  tools:
  - R
  - Snakemake
  - abind
  - ctmm
  - mvtnorm
  - numDeriv
  - rpy2
  edam_operation: http://edamontology.org/operation_3454
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# power-and-false-positive-rate-benchmarking

## Summary

Benchmark statistical power and false positive rate of CTMM's tests for cell type-specific variance by running simulations under the Hom (null) and Free (alternative) models with the Snakemake Snakefile and bin/sim scripts, fitting OP and CTP pseudobulk data with op/ctp module functions, and summarizing rejection rates at the 5% level.

## When to use

Use when you need to quantify how the power and type-I error of variance-differentiation tests (LRT, Wald, jackknife Wald) for cell type-specific pseudobulk data depend on sample size and measurement-noise misestimation, e.g. reproducing Figures 1-2 of the CTMM paper.

## When NOT to use

- When the goal is inference on real scRNA-seq data rather than evaluating the statistical operating characteristics of the tests via simulation
- When sample sizes are below ~50, where the article warns the Free test is slightly inflated and CTMM is not reliable
- When cell types are rare: CTMM requires >10 cells per individual-cell type pair, limiting it to common cell types

## Inputs

- Simulated OP (Overall Pseudobulk) gene expression files (one column per individual, no header)
- Simulated CTP (Cell Type-specific Pseudobulk) gene expression files (one column per cell type, no header)
- Cell type proportions P (one column per cell type, no header)
- Measurement noise variances nu (per individual) and ctnu (per individual-cell type pair)
- Snakemake Snakefile and simulation scripts from bin/sim in github.com/Minhui-Chen/CTMM

## Outputs

- Per-setting rejection rates at the 5% level for each model/method/test combination
- Power summary table (true positive rate vs. sample size under the Free model)
- False positive rate summary table (under the Hom null, across ctnu noise levels)
- Fitted CTMM dictionaries with beta, hom2, V, loglikelihood l, and Wald p values

## How to apply

Use the Snakemake Snakefile and scripts under bin/sim to simulate Overall Pseudobulk (OP) and Cell Type-specific Pseudobulk (CTP) gene expression data with cell type proportions (P) and measurement noise variances (nu, ctnu), sweeping sample sizes from 20 to 100 individuals and ctnu noise levels with coefficient of variation up to 0.45. Fit OP data with the ctmm op module (e.g. op.free_HE(), op.free_REML()) using R optim with method='BFGS' and optim_by_R=True, and fit CTP data with the ctp module (e.g. ctp.free_REML(), ctp.hom_REML()) under Hom, Free, and Full models with ML, REML, and HE. Extract parameter estimates (beta, hom2, V) and loglikelihood (l) from the first output dictionary, and Wald p values on ct_beta and V from the second. Compute LRT p values on variance differentiation with util.lrt(free['l'], hom['l'], C) using C=4 cell types, and run jackknife-based Wald tests by setting jack_knife=True. Aggregate per-setting rejection rates at the 5% level into power (under the Free alternative) and false positive rate (under the Hom null) for each model/method/test combination.

## Related tools

- **Snakemake** (Drives the simulation pipeline via the Snakefile for generating OP/CTP data across sample sizes and noise levels) — https://github.com/Minhui-Chen/CTMM
- **ctmm** (Python package providing op and ctp modules (e.g. ctp.free_REML, ctp.hom_REML) and util.lrt for fitting models and computing LRT p values) — https://github.com/Minhui-Chen/ctmm_python
- **rpy2** (Bridges Python ctmm calls to R routines; optimization via R optim with method='BFGS' and optim_by_R=True) — https://rpy2.github.io/
- **R** (Required runtime for ctmm's optimization and R packages (abind, mvtnorm, numDeriv, softImpute, tensor) used by the routines)

## Examples

```
from ctmm import ctp, util; free, p_wald = ctp.free_REML(y_f='CTP.gz', P_f='P.gz', ctnu_f='ctnu.gz', method='BFGS', optim_by_R=True, jack_knife=True); hom, _ = ctp.hom_REML(y_f='CTP.gz', P_f='P.gz', ctnu_f='ctnu.gz', method='BFGS', optim_by_R=True); p_lrt = util.lrt(free['l'], hom['l'], 4)
```

## Evaluation signals

- Under the Free model, REML (LRT) on CTP data should reach true positive rate above 80% at N=20 and ~100% at N=50
- False positive rate under the Hom null should be near ~5% at low noise levels
- With ctnu coefficient of variation at 0.45, REML (LRT) false positive rate should rise to ~80%, reproducing the misestimation pattern
- Each output dictionary must contain beta, hom2, V, and loglikelihood l, with Wald p values on ct_beta and V in the second dictionary
- util.lrt(free['l'], hom['l'], C) with C=4 should produce a valid LRT p value for variance differentiation (V=0)

## Limitations

- The Free test is slightly inflated and CTMM is not reliable for sample sizes below ~50
- Lowly-expressed genes can be severely non-Gaussian, increasing the number of cells needed for the Gaussian approximation
- Codes for the iid model are not thoroughly tested, so benchmarking should focus on Hom, Free, and Full models

## Evidence

- [methods] Reproduce CTMM simulation benchmark of power and false positive rate (Figs. 1-2): [evidence span withheld — anchor status: not_found]
- [results] Simulation design: [evidence span withheld — anchor status: not_found]
- [readme] LRT usage: "p_lrt = util.lrt(free['l'], hom['l'], C) # LRT on variance differentiation (V=0) # free['l'], hom['l']: loglikelihood"
- [readme] Optimization settings: "free, p_wald = ctp.free_REML(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f, method='BFGS', optim_by_R=True)"
- [discussion] Reliability caveat: "CTMM is not reliable for sample sizes below ~50"
