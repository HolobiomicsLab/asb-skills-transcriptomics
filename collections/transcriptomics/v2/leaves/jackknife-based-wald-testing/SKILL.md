---
name: jackknife-based-wald-testing
description: Use when scRNA-seq data have been summarized to CTP/ctnu/P inputs and
  you need per-gene p-values for cell type-specific interindividual variance differentiation
  that are robust at modest sample sizes.
when_to_use_negative:
- Sample sizes below ~50 individuals, where the Free test is slightly inflated and
  CTMM is not reliable
- 'Rare cell types: CTMM requires >10 cells per individual-cell type pair, limiting
  it to common cell types'
- Lowly-expressed genes, which can be severely non-Gaussian and violate the Gaussian
  pseudobulk approximation
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: ctmm
  role: Python package providing ctp.free_REML with jack_knife=True for jackknife-based
    Wald testing on CTP data
  repo: https://github.com/Minhui-Chen/ctmm_python
- name: rpy2
  role: Bridges Python to R routines used for model optimization (R optim, BFGS)
  repo: https://rpy2.github.io/
- name: R
  role: Runtime required for ctmm optimization routines via rpy2
- name: numDeriv
  role: R dependency installed for the routines behind ctmm fits
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-49242-9
    title: A robust model for cell type-specific interindividual variation in single-cell
      RNA sequencing data
  source_content_hash: sha256:580a00bf7fb4d12740faaeb99a83fef7cdc27f94d35b4421940bff317b169c10
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Minhui-Chen/ctmm_python
  tools:
  - R
  - abind
  - ctmm
  - mvtnorm
  - numDeriv
  - rpy2
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# jackknife-based-wald-testing

## Summary

Fit the CTMM Free model on cell type-specific pseudobulk (CTP) data with REML and enable the jackknife-based Wald test to obtain calibrated gene-level p-values for differentiation of expression variance (V) and mean expression (ct_beta) across cell types.

## When to use

Use when scRNA-seq data have been summarized to CTP/ctnu/P inputs and you need per-gene p-values for cell type-specific interindividual variance differentiation that are robust at modest sample sizes.

## When NOT to use

- Sample sizes below ~50 individuals, where the Free test is slightly inflated and CTMM is not reliable
- Rare cell types: CTMM requires >10 cells per individual-cell type pair, limiting it to common cell types
- Lowly-expressed genes, which can be severely non-Gaussian and violate the Gaussian pseudobulk approximation

## Inputs

- CTP cell type-specific pseudobulk expression matrix (one column per cell type, no header)
- ctnu measurement-noise variance matrix per individual-cell type pair (one column per cell type)
- P cell type proportion matrix (one column per cell type, no header)

## Outputs

- First output dictionary with parameter estimates (beta, ct_beta, hom2, V, loglikelihood l)
- Second output dictionary with jackknife-based Wald test p-values on V and ct_beta per gene
- Ranked hit table (e.g. ipsc_ctp_hit_table.csv) with gene, REML (JK) p-values, HE p-values, and variance-only flags

## How to apply

Load the CTP expression matrix (one column per cell type), the ctnu measurement-noise variance matrix, and the P cell type proportion matrix, then call ctp.free_REML(y_f=CTP, P_f=P, ctnu_f=ctnu, method='BFGS', optim_by_R=True, jack_knife=True) so optimization runs through R optim via rpy2 and the jackknife-based Wald test is performed. Extract the second output dictionary (p_wald) for p-values on variance differentiation (V) and mean expression differentiation (ct_beta), and the first dictionary for parameter estimates (beta, hom2, V, loglikelihood l). Apply Bonferroni correction across all tested genes (e.g. 11,231 in the iPSC analysis, 11,526 in the OneK1K analysis) to call significant genes. REML with jackknife-based Wald testing is the paper's recommended combination as the most powerful and robust approach; optionally cross-validate hits with an independent HE fit (ctp.free_HE) and require significance in both. For variance-differentiation calls without jackknife, the LRT via util.lrt(free['l'], hom['l'], C) with C equal to the number of cell types is an alternative.

## Related tools

- **ctmm** (Python package providing ctp.free_REML with jack_knife=True for jackknife-based Wald testing on CTP data) — https://github.com/Minhui-Chen/ctmm_python
- **rpy2** (Bridges Python to R routines used for model optimization (R optim, BFGS)) — https://rpy2.github.io/
- **R** (Runtime required for ctmm optimization routines via rpy2)
- **numDeriv** (R dependency installed for the routines behind ctmm fits)

## Examples

```
free_jk, p_jk = ctp.free_REML(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f, method='BFGS', optim_by_R=True, jack_knife=True)
```

## Evaluation signals

- Genes called significant must pass Bonferroni correction across all tested genes (e.g. p < 0.05/11526 for 11,526 genes in OneK1K)
- Top-ranked hits should match reported signals (e.g. POU5F1, HSPA8, NDUFB4 at the top in iPSC data; RPS26 top in OneK1K with p=1.78e-167)
- Variance-only candidate count should match the reported 85 genes without significant stage-specific mean expression (p>0.01 in REML) in the iPSC analysis
- Fit outputs must contain both dictionaries: parameter estimates (hom2, V, l) and Wald p-values (p_wald on V and ct_beta)
- Simulated-data check: FPR near ~5% at the 5% level under the Hom null when ctnu noise is low

## Limitations

- The Free test is slightly inflated; CTMM is not reliable for sample sizes below ~50
- Results require biological validation and replication due to test inflation
- Requires >10 cells per individual-cell type pair, limiting application to common cell types
- Circularity from inferring cell types directly from scRNA-seq data can deflate cell type-specific variance estimates

## Evidence

- [discussion] propose a jackknife-based test using REML as the most powerful and robust method: "propose a jackknife-based test using REML as the most powerful and robust method"
- [readme] jack_knife argument: "* ``jack_knife`` to perform jackknife-based Wald test."
- [readme] jackknife example call: "free_jk, p_jk = ctp.free_REML(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f, method='BFGS', optim_by_R=True, jack_knife=True)"
- [methods] second dictionary contents: "The second one contains p values from Wald test on e.g. differentiation of mean expression between cell types (ct_beta) and differentiation of expression variance between cell types (V)."
- [discussion] reliability limit: "CTMM is not reliable for sample sizes below ~50"
