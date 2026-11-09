---
name: cell-type-specific-variance-component-estimation
description: Use when scRNA-seq counts with cell and individual metadata need to be
  summarized into cell type-specific pseudobulk and tested for genes with significant
  cell type-specific interindividual expression variance.
when_to_use_negative:
- Individual-cell type pairs have <=10 cells or fewer than ~50 individuals, since
  CTMM is not reliable for sample sizes below ~50 and limited to common cell types
- Genes are very lowly expressed, where severe non-Gaussianity breaks the Gaussian
  approximation underlying the pseudobulk model
- Cell types are continuous rather than discrete, since pseudobulk CTP representation
  requires discretizing them
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: ctmm
  role: Python package providing preprocess.pseudobulk, preprocess.std, and ctp.free_REML
    with jack_knife for fitting the CTMM Free model
  repo: https://github.com/Minhui-Chen/ctmm_python
- name: rpy2
  role: Bridge through which CTMM calls R routines for optimization (e.g. BFGS via
    R optim) during REML fitting
  repo: https://rpy2.github.io/
- name: R
  role: Runtime required by rpy2 for the R routines used in model fitting; R packages
    abind, mvtnorm, numDeriv, softImpute, tensor must be installed
  repo: https://github.com/Minhui-Chen/CTMM
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-49242-9
    title: A robust model for cell type-specific interindividual variation in single-cell
      RNA sequencing data
  source_content_hash: sha256:35ee9730e1c95c4636b3e10124b9a04e9e09398bfbf1cbadf941924bb4326255
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Minhui-Chen/ctmm_python
  tools:
  - R
  - ctmm
  - rpy2
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cell-type-specific-variance-component-estimation

## Summary

Fit the CTMM (Cell Type-specific linear Mixed Model) Free model with REML and a jackknife-based Wald test to cell type-specific pseudobulk scRNA-seq data in order to detect genes whose interindividual expression variance differs across cell types and to partition variance into cell type-shared (hom2) and cell type-specific (V) components.

## When to use

Use when scRNA-seq counts with cell and individual metadata need to be summarized into cell type-specific pseudobulk and tested for genes with significant cell type-specific interindividual expression variance.

## When NOT to use

- Individual-cell type pairs have <=10 cells or fewer than ~50 individuals, since CTMM is not reliable for sample sizes below ~50 and limited to common cell types
- Genes are very lowly expressed, where severe non-Gaussianity breaks the Gaussian approximation underlying the pseudobulk model
- Cell types are continuous rather than discrete, since pseudobulk CTP representation requires discretizing them

## Inputs

- Cell-level scRNA-seq counts matrix (genes x cells) plus metadata with columns (cell, ind, ct), e.g. the OneK1K droplet PBMC data
- CTP: cell type-specific pseudobulk gene expression (one column per cell type, no header)
- ctnu: variance of measurement noise for each individual-cell type pair (one column per cell type)
- P: cell type proportions per individual (one column per cell type)

## Outputs

- First dictionary of model parameter estimates: hom2 (cell type-shared random effect variance), V (cell type-specific random effect variance), beta/ct_beta, loglikelihood
- Second dictionary of jackknife-based Wald test p-values: p_wald['V'] for variance differentiation and p_wald['ct_beta'] for mean expression differentiation
- Counts of genes significant for cell type-specific variance (e.g. 2310 of 11,526 genes in OneK1K at Bonferroni p < 0.05/11526)

## How to apply

Aggregate cell-level counts with preprocess.pseudobulk(counts=counts, meta=meta, ind_cut=100, ct_cut=10), which removes individuals with <=100 cells and sets ctp/ctnu to missing for individual-cell type pairs with <=10 cells, then standardize the pseudobulk with preprocess.std. Fit the Free model on CTP via ctp.free_REML(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f, method='BFGS', optim_by_R=True, jack_knife=True); REML with the jackknife-based Wald test is the most powerful and robust configuration per the paper. Read the second output dictionary for p_wald p-values on variance differentiation (V) and mean expression differentiation (ct_beta), applying the Bonferroni threshold p < 0.05/11526 for the 11,526 tested OneK1K genes. Extract hom2 and V from the first output dictionary to compute the transcriptome-wide split of interindividual variance (10.9% shared vs 21.5% cell type-specific in OneK1K PBMCs).

## Related tools

- **ctmm** (Python package providing preprocess.pseudobulk, preprocess.std, and ctp.free_REML with jack_knife for fitting the CTMM Free model) — https://github.com/Minhui-Chen/ctmm_python
- **rpy2** (Bridge through which CTMM calls R routines for optimization (e.g. BFGS via R optim) during REML fitting) — https://rpy2.github.io/
- **R** (Runtime required by rpy2 for the R routines used in model fitting; R packages abind, mvtnorm, numDeriv, softImpute, tensor must be installed) — https://github.com/Minhui-Chen/CTMM

## Examples

```
free_jk, p_jk = ctp.free_REML(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f, method='BFGS', optim_by_R=True, jack_knife=True)
```

## Evaluation signals

- OneK1K PBMC reference results: 2310 of 11,526 tested genes significant for cell type-specific variance at Bonferroni p < 0.05/11526, with RPS26 the top signal (p=1.78e-167)
- Transcriptome-wide variance split reproduces 10.9% shared (hom2) vs 21.5% cell type-specific (V)
- Output schema check: ctp.free_REML returns exactly two dictionaries, first with hom2/V/l and second with p_wald for ct_beta and V
- Filter check: pseudobulk output contains only the 7 primary PBMC cell types and only individual-cell type pairs with >10 cells
- Optional LRT cross-check via util.lrt(free['l'], hom['l'], C) on variance differentiation should be directionally consistent with the Wald p-values

## Limitations

- Requiring >10 cells per individual-cell type pair limits CTMM to common cell types; pseudobulk discards cell-level information, sacrificing power within cell types
- Lowly-expressed genes can be severely non-Gaussian, increasing the number of cells needed for the Gaussian approximation
- The Free test is slightly inflated and results are not reliable for sample sizes below ~50; cell types inferred from the same scRNA-seq data induce circularity that deflates cell type-specific variance estimates

## Evidence

- [methods] ind_cut filter: "preprocess.pseudobulk(counts=counts, meta=meta, ind_cut=100, ct_cut=10) # remove individuals with <= 100 cells"
- [discussion] jackknife REML recommendation: "propose a jackknife-based test using REML as the most powerful and robust method"
- [readme] output dictionaries: "The second one contains p values from Wald test on e.g. differentiation of mean expression between cell types (ct_beta) and differentiation of expression variance between cell types (V)."
- [discussion] sample size reliability: "CTMM is not reliable for sample sizes below ~50"
- [methods] ctmm on PyPI: "[ctmm](ctmm) contains the CTMM python package. It is also published on PyPI (``pip install ctmm``)"
