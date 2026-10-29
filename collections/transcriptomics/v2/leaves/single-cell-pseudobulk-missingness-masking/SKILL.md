---
name: single-cell-pseudobulk-missingness-masking
description: Use when individual-cell type pairs in scRNA-seq pseudobulk data have
  <=10 cells so ctp/ctnu entries were set to missing, and CTMM fitting requires a
  complete expression and noise-variance matrix.
when_to_use_negative:
- All individual-cell type pairs already have >10 cells so no ctp/ctnu entries are
  missing — imputation is unnecessary.
- Only a small subset of genes is of interest — the benchmark supporting transcriptome-wide
  softImpute was computed across all genes, so gene-subset imputation lacks this validation.
- Sample size is below ~50 individuals — CTMM itself is unreliable there regardless
  of imputation quality.
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: ctmm
  role: Python package providing preprocess.pseudobulk (masking with ind_cut/ct_cut)
    and preprocess.softimpute (imputation wrapper)
  repo: https://github.com/Minhui-Chen/ctmm_python
- name: softImpute
  role: R matrix-completion program invoked via preprocess.softimpute to fill missing
    ctp/ctnu entries
- name: rpy2
  role: Bridge through which CTMM calls R routines, including softImpute
  repo: https://rpy2.github.io/
- name: R
  role: Runtime required for the softImpute routine called from Python
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-49242-9
    title: A robust model for cell type-specific interindividual variation in single-cell
      RNA sequencing data
  source_content_hash: sha256:8e436d46ad27e510dd1d10098e3a6fbebb8ea9d82ee4a63155b821721f9a540a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Minhui-Chen/ctmm_python
  tools:
  - R
  - ctmm
  - rpy2
  - softImpute
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# single-cell-pseudobulk-missingness-masking

## Summary

Impute missing cell type-specific pseudobulk (ctp) and cell type-specific noise variance (ctnu) entries for individual-cell type pairs with too few cells using transcriptome-wide matrix completion (softImpute, called from Python via rpy2), which benchmarked best before fitting CTMM.

## When to use

Use when individual-cell type pairs in scRNA-seq pseudobulk data have <=10 cells so ctp/ctnu entries were set to missing, and CTMM fitting requires a complete expression and noise-variance matrix.

## When NOT to use

- All individual-cell type pairs already have >10 cells so no ctp/ctnu entries are missing — imputation is unnecessary.
- Only a small subset of genes is of interest — the benchmark supporting transcriptome-wide softImpute was computed across all genes, so gene-subset imputation lacks this validation.
- Sample size is below ~50 individuals — CTMM itself is unreliable there regardless of imputation quality.

## Inputs

- counts matrix from QC'd, normalized single-cell gene expression (genes x cells)
- meta table mapping cells to individuals and cell types
- ctp matrix (cell type-specific pseudobulk) with masked entries from preprocess.pseudobulk(ind_cut=100, ct_cut=10)
- ctnu matrix (cell type-specific noise variance per individual-cell type pair) with masked entries

## Outputs

- imputed ctp matrix (complete individual x cell type pseudobulk expression)
- imputed ctnu matrix (complete individual x cell type noise variance)
- imputation_metrics.csv with Pearson correlation and MSE of imputed vs held-out true masked entries

## How to apply

First generate ctp and ctnu matrices with preprocess.pseudobulk using ind_cut=100 and ct_cut=10, which sets entries to missing for individual-cell type pairs with <=10 cells. Hold out the masked entries as ground truth, and restrict entries to genes expressed in all retained cell types before imputation. Run matrix completion on ctp and ctnu with preprocess.softimpute, a wrapper around the R softImpute program invoked from Python through rpy2; the paper's benchmark (mirroring Supplementary Fig. 12) found transcriptome-wide softImpute performed best among compared imputation approaches, with MVN performing similarly. Validate by computing Pearson correlation and mean squared error between imputed values and held-out true values across all genes, then save the imputed ctp/ctnu matrices and write correlation/MSE summaries to imputation_metrics.csv before feeding ctp/ctnu to CTMM CTP model fitting.

## Related tools

- **ctmm** (Python package providing preprocess.pseudobulk (masking with ind_cut/ct_cut) and preprocess.softimpute (imputation wrapper)) — https://github.com/Minhui-Chen/ctmm_python
- **softImpute** (R matrix-completion program invoked via preprocess.softimpute to fill missing ctp/ctnu entries)
- **rpy2** (Bridge through which CTMM calls R routines, including softImpute) — https://rpy2.github.io/
- **R** (Runtime required for the softImpute routine called from Python)

## Examples

```
from ctmm import preprocess; ctp, ctnu, P = preprocess.pseudobulk(counts=counts, meta=meta, ind_cut=100, ct_cut=10); ctp = preprocess.softimpute(ctp); ctnu = preprocess.softimpute(ctnu)
```

## Evaluation signals

- Pearson correlation between imputed and held-out true masked entries across all genes is between 0 and 1
- MSE between imputed and true masked entries is non-negative and finite
- Masked entries correspond exactly to individual-cell type pairs with <=10 cells (ct_cut=10) produced by preprocess.pseudobulk(ind_cut=100, ct_cut=10)
- imputation_metrics.csv is written with both correlation and MSE summaries for ctp and ctnu
- Genes not expressed in all retained cell types were excluded before imputation, per the article's restriction step

## Limitations

- CTMM requires >10 cells per individual-cell type pair, limiting it to common cell types; rare cell types remain missing and drive reliance on imputation
- Lowly-expressed genes can be severely non-Gaussian, increasing the number of cells needed for the Gaussian approximation underlying ctnu estimates
- The iid/free model comparison context means imputation quality directly conditions downstream CTMM power; MVN imputation performed similarly, so gains over alternative matrix completion are marginal

## Evidence

- [methods] masking thresholds: "preprocess.pseudobulk(counts=counts, meta=meta, ind_cut=100, ct_cut=10) # remove individuals with <= 100 cells"
- [methods] missingness rule: "set ctp and ctnu to missing for individual-cell type pairs with <=10 cells"
- [methods] imputation call: "# ctp = preprocess.softimpute( ctp )"
- [methods] gene restriction: "# remove individuals or cell types to make sure that all genes are expressed in all cell types"
- [discussion] cell count requirement: "In practice, we require >10 cells per individual-cell type pair, which limits CTMM to common cell types."
- [readme] rpy2 dependency: "CTMM calls R routines through [rpy2](https://rpy2.github.io/), so a working R installation is required."
- [readme] R packages: "install.packages(c("abind", "mvtnorm", "numDeriv", "softImpute", "tensor"))"
