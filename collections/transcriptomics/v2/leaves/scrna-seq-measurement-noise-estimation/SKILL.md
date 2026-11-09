---
name: scrna-seq-measurement-noise-estimation
description: Use when fitting CTMM to pseudobulk scRNA-seq (OP or CTP) and per-observation
  measurement noise variances (nu, ctnu) must be supplied as model inputs.
when_to_use_negative:
- 'Individual-cell type pairs have <=10 cells: CTMM requires >10 cells per pair and
  is limited to common cell types'
- 'Genes are very lowly expressed and severely non-Gaussian: the Gaussian approximation
  requires more cells than the ct_cut=10 threshold'
- 'Sample sizes are below ~50 individuals: CTMM is not reliable in that regime and
  the Free test is slightly inflated'
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: ctmm
  role: Python package whose preprocess module (preprocess.pseudobulk, preprocess.std)
    computes ctnu/nu, ctp, and P from counts and meta tables
  repo: https://github.com/Minhui-Chen/ctmm_python
- name: rpy2
  role: Bridge through which CTMM calls R routines during model fitting that consumes
    the noise variance inputs
  repo: https://rpy2.github.io/
- name: Python
  role: Implementation language for the ctmm package and preprocessing functions
  repo: https://github.com/Minhui-Chen/ctmm_python
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-49242-9
    title: A robust model for cell type-specific interindividual variation in single-cell
      RNA sequencing data
  source_content_hash: sha256:8e54c105fc01037a4d91ce07a77342211587895c32e9fd3f949088cf80000493
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Minhui-Chen/ctmm_python
  tools:
  - Python
  - R
  - ctmm
  - rpy2
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# scRNA-seq-measurement-noise-estimation

## Summary

Compute per-individual and per-individual-cell-type measurement noise variances (nu and ctnu) from single-cell RNA-seq data so that CTMM linear mixed models can account for pseudobulk sampling noise when partitioning interindividual variance.

## When to use

Use when fitting CTMM to pseudobulk scRNA-seq (OP or CTP) and per-observation measurement noise variances (nu, ctnu) must be supplied as model inputs.

## When NOT to use

- Individual-cell type pairs have <=10 cells: CTMM requires >10 cells per pair and is limited to common cell types
- Genes are very lowly expressed and severely non-Gaussian: the Gaussian approximation requires more cells than the ct_cut=10 threshold
- Sample sizes are below ~50 individuals: CTMM is not reliable in that regime and the Free test is slightly inflated

## Inputs

- counts matrix of gene expression from all cells (QC'd and normalized scRNA-seq data)
- metadata table (meta) mapping cells to individuals and cell types
- Preprocessed CTP file (one column per cell type, no header), ctnu file (one column per cell type, no header), and P file of cell type proportions when passed directly to fitting functions

## Outputs

- ctnu: variance of measurement noise for each pair of individual and cell type (one column per cell type, no header)
- nu: variance of measurement noise for each individual (overall pseudobulk case)
- ctp: cell type-specific pseudobulk expression matrix
- P: cell type proportions matrix

## How to apply

After quality-controlling and normalizing single-cell expression data, use the ctmm preprocess module: call preprocess.pseudobulk(counts=counts, meta=meta, ind_cut=100, ct_cut=10) to compute cell type-specific pseudobulk (CTP), the per individual-cell type noise variance (ctnu), and cell type proportions (P); individuals with <=100 cells are removed and individual-cell type pairs with <=10 cells are set to missing. Optionally rescale expression and noise files with preprocess.std so the overall pseudobulk has mean 0 and variance 1. The noise variances (nu for overall pseudobulk, ctnu for cell type-specific pseudobulk) are passed alongside the pseudobulk files to ctmm fitting functions such as ctp.free_REML(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f, method='BFGS', optim_by_R=True). Input files must be formatted without headers: CTP with one column per cell type, ctnu with one column per cell type, one row per individual. Rationale: the Gaussian LMM fits require observed measurement noise to be explicitly supplied so that interindividual variance components (hom2, V) are estimated unbiasedly.

## Related tools

- **ctmm** (Python package whose preprocess module (preprocess.pseudobulk, preprocess.std) computes ctnu/nu, ctp, and P from counts and meta tables) — https://github.com/Minhui-Chen/ctmm_python
- **rpy2** (Bridge through which CTMM calls R routines during model fitting that consumes the noise variance inputs) — https://rpy2.github.io/
- **Python** (Implementation language for the ctmm package and preprocessing functions) — https://github.com/Minhui-Chen/ctmm_python

## Examples

```
from ctmm import preprocess; op, nu, ctp, ctnu = preprocess.std(*preprocess.pseudobulk(counts=counts, meta=meta, ind_cut=100, ct_cut=10))
```

## Evaluation signals

- ctnu file has exactly one column per cell type and one row per individual-cell type pair, with no header, matching the CTP file dimensions
- Individual-cell type pairs with <=10 cells are set to missing in ctp and ctnu; individuals with <=100 cells are excluded
- After preprocess.std, the overall pseudobulk has mean 0 and variance 1
- Fitting ctp.free_REML with the resulting ctnu returns a dictionary containing hom2, V, beta, and loglikelihood without optimizer failure
- Reproduced variance partition for the Cuomo et al. 2020 iPSC data matches median cell type-specific variance of 32.8% within REML numerical convergence tolerance

## Limitations

- CTMM requires >10 cells per individual-cell type pair, limiting the method to common cell types
- Lowly-expressed genes can be severely non-Gaussian, increasing the number of cells needed for the Gaussian approximation
- CTMM is not reliable for sample sizes below ~50; the Free test is slightly inflated
- Low-depth sequencing data may require larger sample sizes, which is currently expensive

## Evidence

- [methods] compute ctp (cell type-specific pseudobulk) and ctnu (cell type-specific noise variance) and P (cell type proportions): [evidence span withheld — anchor status: not_found]
- [methods] preprocess.pseudobulk(counts=counts, meta=meta, ind_cut=100, ct_cut=10): "'preprocess.pseudobulk(counts=counts, meta=meta, ind_cut=100, ct_cut=10) # remove individuals with <= 100 cells'"
- [methods] set ctp and ctnu to missing for individual-cell type pairs with <=10 cells: "'set ctp and ctnu to missing for individual-cell type pairs with <=10 cells'"
- [readme] ctnu: variance of measurement noise for each pair of individual and cell type. This file should have one column for each cell type and without header.: "'ctnu: variance of measurement noise for each pair of individual and cell type. This file should have one column for each cell type and without header.'"
- [methods] op, nu, ctp, ctnu = preprocess.std( ctp, ctnu, P): [evidence span withheld — anchor status: not_found]
- [readme] free, p_wald = ctp.free_REML(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f, method='BFGS', optim_by_R=True): "'free, p_wald = ctp.free_REML(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f, method='BFGS', optim_by_R=True)'"
