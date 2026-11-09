---
name: linear-mixed-model-variance-component-fitting
description: Use when scRNA-seq data have been summarized into OP or CTP pseudobulk
  per individual (with nu/ctnu noise variances and P cell type proportions) and the
  goal is to detect or quantify cell type-specific interindividual expression variance.
when_to_use_negative:
- 'Sample sizes below ~50: the Free test is slightly inflated and CTMM is not reliable
  for sample sizes below ~50'
- 'Rare cell types: CTMM requires >10 cells per individual-cell type pair, limiting
  it to common cell types'
- Lowly-expressed genes where the Gaussian approximation fails (severely non-Gaussian,
  needing many more cells), or continuous cell states that cannot be discretized without
  distortion
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: ctmm
  role: Python package implementing the Hom/Iid/Free/Full models with ML/REML/HE fitting
    for OP and CTP pseudobulk; installed via pip install ctmm
  repo: https://github.com/Minhui-Chen/ctmm_python
- name: rpy2
  role: bridge through which CTMM calls R routines (optim with method='BFGS'); rpy2
    3.5.x for R 3.6-4.4, rpy2 >=3.6 for R >=4.5
  repo: https://rpy2.github.io/
- name: R
  role: runtime for the optimization and statistical routines invoked by CTMM
- name: Snakemake
  role: Snakefile drives simulation and iPSCs analysis steps that generate and fit
    the pseudobulk inputs
  repo: https://github.com/Minhui-Chen/CTMM
- name: softImpute
  role: optional matrix completion (preprocess.softimpute) for missing ctp entries
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-49242-9
    title: A robust model for cell type-specific interindividual variation in single-cell
      RNA sequencing data
  source_content_hash: sha256:e89e80cb6d27b61a5115c2ec6611173fadc0c34cd754f53705d0ee0535ac1469
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Minhui-Chen/ctmm_python
  tools:
  - R
  - Snakemake
  - abind
  - ctmm
  - mvtnorm
  - numDeriv
  - rpy2
  - softImpute
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# linear-mixed-model-variance-component-fitting

## Summary

Fit Cell Type-specific linear Mixed Models (CTMM) on Overall Pseudobulk (OP) and Cell Type-specific Pseudobulk (CTP) scRNA-seq data to estimate cell type-shared (hom2) and cell type-specific (V) variance components, using ML, REML, or Haseman-Elston (HE) methods under Hom, Iid, Free, and Full models.

## When to use

Use when scRNA-seq data have been summarized into OP or CTP pseudobulk per individual (with nu/ctnu noise variances and P cell type proportions) and the goal is to detect or quantify cell type-specific interindividual expression variance.

## When NOT to use

- Sample sizes below ~50: the Free test is slightly inflated and CTMM is not reliable for sample sizes below ~50
- Rare cell types: CTMM requires >10 cells per individual-cell type pair, limiting it to common cell types
- Lowly-expressed genes where the Gaussian approximation fails (severely non-Gaussian, needing many more cells), or continuous cell states that cannot be discretized without distortion

## Inputs

- OP: Overall Pseudobulk gene expression per individual (one column per gene, no header; .gz)
- nu: variance of measurement noise per individual (one column, no header)
- P: cell type proportions per individual (one column per cell type, no header)
- CTP: Cell Type-specific Pseudobulk per individual (one column per cell type, no header)
- ctnu: variance of measurement noise per individual-cell type pair (one column per cell type)
- optional fixed_covars_d (e.g. PCA on OP) and random_covars_d (e.g. experiment batch) files

## Outputs

- First dictionary: parameter estimates (beta including ct_beta, hom2, V) and loglikelihood (l)
- Second dictionary: Wald test p values on ct_beta (mean differentiation) and V (variance differentiation)
- LRT p value on variance differentiation from util.lrt(free['l'], hom['l'], C)

## How to apply

Generate OP/CTP inputs via preprocess.pseudobulk from QC'd counts and meta, setting ind_cut=100 and ct_cut=10 so individuals with <=100 cells are removed and individual-cell type pairs with <=10 cells get missing ctp/ctnu. Fit OP by importing the ctmm 'op' module and calling [model]_[method] functions (e.g. op.free_REML with method='BFGS', optim_by_R=True), and CTP via the 'ctp' module (e.g. ctp.free_REML, ctp.hom_REML, ctp.full_ML). Use optim via R optim BFGS because CTMM calls R routines through rpy2 (R 3.6-4.4 with rpy2 3.5.x; R >=4.5 with rpy2 >=3.6). Read the two output dictionaries: the first holds beta, hom2, V, and loglikelihood (l); the second holds Wald p values on ct_beta and V. For the LRT on variance differentiation, fit both Free and Hom and compute util.lrt(free['l'], hom['l'], C) with C = number of cell types (e.g. C=4); for the jackknife-based Wald test, set jack_knife=True. REML with LRT on CTP data is the most powerful and robust combination per the paper's simulations (>80% power at N=20, ~100% at N=50 with ~5% FPR under Hom).

## Related tools

- **ctmm** (Python package implementing the Hom/Iid/Free/Full models with ML/REML/HE fitting for OP and CTP pseudobulk; installed via pip install ctmm) — https://github.com/Minhui-Chen/ctmm_python
- **rpy2** (bridge through which CTMM calls R routines (optim with method='BFGS'); rpy2 3.5.x for R 3.6-4.4, rpy2 >=3.6 for R >=4.5) — https://rpy2.github.io/
- **R** (runtime for the optimization and statistical routines invoked by CTMM)
- **Snakemake** (Snakefile drives simulation and iPSCs analysis steps that generate and fit the pseudobulk inputs) — https://github.com/Minhui-Chen/CTMM
- **softImpute** (optional matrix completion (preprocess.softimpute) for missing ctp entries)

## Examples

```
from ctmm import op, ctp, util; free, p_wald = ctp.free_REML(y_f='test/CTP.gz', P_f='test/P.gz', ctnu_f='test/ctnu.gz', method='BFGS', optim_by_R=True)
```

## Evaluation signals

- Reproduced benchmark: REML (LRT) on CTP achieves power >80% at N=20 and ~100% at N=50 under the Free model
- False positive rate is near ~5% under the Hom null for REML (LRT) at moderate noise on ctnu
- FPR of REML (LRT) rises to ~80% when the ctnu coefficient of variation reaches 0.45 — an expected failure signature of mis-specified noise variance
- Output structure check: each fit returns exactly two dictionaries, with hom2, V, and l present in the first and ct_beta / V p values in the second
- Optimization reproducibility: fits use R optim with method='BFGS' and optim_by_R=True

## Limitations

- Codes for the Iid model are not thoroughly tested
- CTMM requires larger sample sizes than bulk analyses (expensive), and is not reliable below ~50 individuals due to slight Free-test inflation
- Pseudobulk discards cell-level information, sacrificing power and within-cell-type resolution
- Circularity: inferring cell types from the same scRNA-seq data deflates estimates of cell type-specific variance
- Results require biological validation and replication due to test inflation; LRT mis-calibrates (~80% FPR) if ctnu CV is as high as 0.45

## Evidence

- [methods] Fit CTP models with ctmm: "To fit CTP, import the ``ctp`` module and call the function in the same way. For example, ``ctp.full_ML()`` to fit Full model using ML."
- [readme] LRT usage: "p_lrt = util.lrt(free['l'], hom['l'], C) # LRT on variance differentiation (V=0)"
- [readme] Jackknife Wald test: "free_jk, p_jk = ctp.free_REML(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f, method='BFGS', optim_by_R=True, jack_knife=True)"
- [methods] rpy2 requirement: "CTMM calls R routines through [rpy2](https://rpy2.github.io/), so a working R installation is required."
- [methods] Cell count thresholds: "preprocess.pseudobulk(counts=counts, meta=meta, ind_cut=100, ct_cut=10) # remove individuals with <= 100 cells"
- [discussion] Reliability limit: "CTMM is not reliable for sample sizes below ~50"
