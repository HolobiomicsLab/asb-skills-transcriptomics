---
name: measurement-noise-variance-estimation
description: 'Use when preparing droplet scRNA-seq counts for CTMM: individual-level
  or individual-by-cell-type measurement-noise variances must be quantified before
  fitting op.free_REML or ctp.free_REML models.'
when_to_use_negative:
- Cells per individual-cell type pair are <= 10 or individuals have <= 100 cells —
  CTMM requires > 10 cells per pair and is not reliable for sample sizes below ~50
  individuals
- Genes are lowly expressed such that the Gaussian approximation of pseudobulk noise
  is invalid, requiring more cells per pair than the ctnu-based model assumes
- Data are not single-cell-derived pseudobulk (e.g., bulk RNA-seq without per-cell
  noise estimates) — no cell-level noise variance can be computed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ctmm
  role: Provides the preprocess module (preprocess.pseudobulk, preprocess.std) that
    computes ctnu/nu noise variances alongside pseudobulk expression from counts and
    metadata
  repo: https://github.com/Minhui-Chen/ctmm_python
- name: Python
  role: Language in which the ctmm preprocess functions are run to estimate noise
    variance from cell-level counts
  repo: https://github.com/Minhui-Chen/ctmm_python
- name: rpy2
  role: Bridges Python to R routines used downstream by CTMM models that consume the
    ctnu noise estimates
  repo: https://github.com/Minhui-Chen/CTMM
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-49242-9
    title: A robust model for cell type-specific interindividual variation in single-cell
      RNA sequencing data
  source_content_hash: sha256:ba90511ee53749a17e1544e3dad8d2130d095d2038130483eaac553ed7a41a8a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Minhui-Chen/ctmm_python
  tools:
  - Python
  - R
  - ctmm
  - rpy2
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# measurement-noise-variance-estimation

## Summary

Computes the variance of measurement noise for each individual (nu) or each individual-cell type pair (ctnu) from single-cell counts, a required input for fitting CTMM linear mixed models on Overall Pseudobulk (OP) and Cell Type-specific Pseudobulk (CTP) gene expression data.

## When to use

Use when preparing droplet scRNA-seq counts for CTMM: individual-level or individual-by-cell-type measurement-noise variances must be quantified before fitting op.free_REML or ctp.free_REML models.

## When NOT to use

- Cells per individual-cell type pair are <= 10 or individuals have <= 100 cells — CTMM requires > 10 cells per pair and is not reliable for sample sizes below ~50 individuals
- Genes are lowly expressed such that the Gaussian approximation of pseudobulk noise is invalid, requiring more cells per pair than the ctnu-based model assumes
- Data are not single-cell-derived pseudobulk (e.g., bulk RNA-seq without per-cell noise estimates) — no cell-level noise variance can be computed

## Inputs

- Cell-level gene counts matrix (genes x cells) from QC'd and normalized scRNA-seq data
- Metadata table with columns cell, ind, ct
- Filter parameters ind_cut=100 and ct_cut=10

## Outputs

- ctnu: variance of measurement noise for each pair of individual and cell type (one column per cell type, no header)
- nu: variance of measurement noise for each individual (one column, no header), via preprocess.std
- CTP: cell type-specific pseudobulk expression and P: cell type proportions, produced jointly with ctnu

## How to apply

Aggregate cell-level counts and metadata (cell, ind, ct) with ctmm.preprocess.pseudobulk using ind_cut=100 to remove individuals with <= 100 cells and ct_cut=10 to set ctp/ctnu to missing for individual-cell type pairs with <= 10 cells; this returns CTP, ctnu, and P (cell type proportions). The ctnu values capture the noise variance of each individual-cell type pseudobulk and are passed directly to model fits (e.g., ctp.free_REML(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f, method='BFGS', optim_by_R=True, jack_knife=True)). Before fitting, optionally standardize with preprocess.std(ctp, ctnu, P), which returns op, nu, ctp, ctnu, ensuring OP-derived noise (nu) is available if an Overall Pseudobulk model is also fit. Files may also be written without headers, one column per cell type, as required by the CTMM input format.

## Related tools

- **ctmm** (Provides the preprocess module (preprocess.pseudobulk, preprocess.std) that computes ctnu/nu noise variances alongside pseudobulk expression from counts and metadata) — https://github.com/Minhui-Chen/ctmm_python
- **Python** (Language in which the ctmm preprocess functions are run to estimate noise variance from cell-level counts) — https://github.com/Minhui-Chen/ctmm_python
- **rpy2** (Bridges Python to R routines used downstream by CTMM models that consume the ctnu noise estimates) — https://github.com/Minhui-Chen/CTMM

## Examples

```
from ctmm import preprocess; ctp, ctnu, P = preprocess.pseudobulk(counts=counts, meta=meta, ind_cut=100, ct_cut=10)
```

## Evaluation signals

- Output ctnu has one column per cell type with no header, matching the CTMM CTP/P file layout for the 7 primary PBMC cell types
- All individual-cell type pairs with <= 10 cells are set to missing (NaN) in ctnu and ctp, consistent with ct_cut=10
- Downstream ctp.free_REML with ctnu_f converges (BFGS optimization) and the second returned dictionary yields gene-level Wald p-values, e.g. reproducing 2310/11526 genes significant for variance differentiation at Bonferroni p < 0.05/11526 on OneK1K PBMCs
- preprocess.std returns op, nu, ctp, ctnu with standardized pseudobulk, and hom2/V estimates reproduce the expected ~10.9% shared vs ~21.5% cell type-specific variance split

## Limitations

- Requiring > 10 cells per individual-cell type pair limits CTMM to common cell types
- Lowly-expressed genes can be severely non-Gaussian, increasing the number of cells needed for the Gaussian noise approximation underlying ctnu
- The iid model (constant cell type-specific variance) is not thoroughly tested, so ctnu-based fits should use the free model with REML + jackknife

## Evidence

- [readme] ctnu: variance of measurement noise for each pair of individual and cell type.: "ctnu: variance of measurement noise for each pair of individual and cell type."
- [methods] filter pseudobulk call with cuts: "preprocess.pseudobulk(counts=counts, meta=meta, ind_cut=100, ct_cut=10) # remove individuals with <= 100 cells"
- [methods] missingness rule for small pairs: "set ctp and ctnu to missing for individual-cell type pairs with <=10 cells"
- [methods] standardization step: "op, nu, ctp, ctnu = preprocess.std( ctp, ctnu, P)"
