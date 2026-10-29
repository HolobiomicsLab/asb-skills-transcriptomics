---
name: interindividual-variance-partition-analysis
description: Use when scRNA-seq data across multiple individuals and cell types (e.g.
  differentiating iPSCs) must be partitioned into cell type-specific vs shared interindividual
  variance components using cell type-specific pseudobulk linear mixed models.
when_to_use_negative:
- 'Sample sizes below ~50 individuals: CTMM''s Free test is slightly inflated and
  ''CTMM is not reliable for sample sizes below ~50'''
- 'Rare cell types: CTMM requires >10 cells per individual-cell type pair, limiting
  it to common cell types'
- Continuous cell type trajectories that cannot be reasonably discretized, since CTP
  pseudobulk requires discrete cell types
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: ctmm
  role: Python package providing preprocess.pseudobulk, preprocess.std, and ctp.free_REML
    used to build CTP/ctnu/P inputs and fit the Free model with REML
  repo: https://github.com/Minhui-Chen/ctmm_python
- name: rpy2
  role: Bridge through which CTMM calls R optimization routines (BFGS via R optim)
    during REML fitting
  repo: https://rpy2.github.io/
- name: R
  role: Runtime required by CTMM (via rpy2) for REML optimization; R packages abind,
    mvtnorm, numDeriv, softImpute, tensor must be installed
- name: Snakemake
  role: Snakefile pipeline steps for reproducing the simulations and iPSC variance-partitioning
    analyses
  repo: https://github.com/Minhui-Chen/CTMM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-49242-9
    title: A robust model for cell type-specific interindividual variation in single-cell
      RNA sequencing data
  source_content_hash: sha256:b487e9b987740dc7afe72d6e2db3dd311bf6325fee63a147d637fdb8fb159344
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Minhui-Chen/ctmm_python
  tools:
  - R
  - Snakemake
  - ctmm
  - rpy2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# interindividual-variance-partition-analysis

## Summary

Reproduce the CTMM-based transcriptome-wide variance partitioning of the Cuomo et al. 2020 iPSC-to-endoderm scRNA-seq data, quantifying how much interindividual expression variation is cell type-specific (median 32.8%) versus shared, proportion-driven, residual, PC, and batch fractions.

## When to use

Use when scRNA-seq data across multiple individuals and cell types (e.g. differentiating iPSCs) must be partitioned into cell type-specific vs shared interindividual variance components using cell type-specific pseudobulk linear mixed models.

## When NOT to use

- Sample sizes below ~50 individuals: CTMM's Free test is slightly inflated and 'CTMM is not reliable for sample sizes below ~50'
- Rare cell types: CTMM requires >10 cells per individual-cell type pair, limiting it to common cell types
- Continuous cell type trajectories that cannot be reasonably discretized, since CTP pseudobulk requires discrete cell types

## Inputs

- Cuomo et al. 2020 Nat Commun iPSC scRNA-seq counts file and meta table (QC'd and normalized single-cell expression)
- CTP: cell type-specific pseudobulk expression (one column per cell type, no header)
- ctnu: per individual-cell type measurement noise variance (one column per cell type, no header)
- P: cell type proportions per individual (one column per cell type, no header)
- Optional fixed covariates (PCA on OP) and random covariates (experiment batch) files

## Outputs

- variance_partition_ipsc.csv with fractions: homogeneous variance, cell type-specific variance, cell type proportion differences, residual cell-level variation, pseudobulk PCs, batch effects
- Fitted CTMM parameter dictionary (hom2, V, beta, loglikelihood) and Wald p-value dictionary from ctp.free_REML

## How to apply

Load the Cuomo et al. 2020 Nat Commun iPSC counts and meta tables and run ctmm's preprocess.pseudobulk with ind_cut=100 (drop individuals with <=100 cells) and ct_cut=10 (set CTP/ctnu to missing for individual-cell type pairs with <=10 cells) to obtain CTP, ctnu, and P. Optionally rescale expression and noise with preprocess.std so the overall pseudobulk has mean 0 and variance 1. Fit the Free model with REML by calling ctp.free_REML(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f, method='BFGS', optim_by_R=True), which returns model parameters (hom2, V, beta, loglikelihood) and Wald p values; include extra fixed (PCA) covariates via fixed_covars_d and random (experiment batch) covariates via random_covars_d. Partition interindividual variation into homogeneous variance, cell type-specific variance, cell type proportion differences, residual cell-level variation, pseudobulk-expression PCs, and batch-effect fractions, writing the fractions to variance_partition_ipsc.csv.

## Related tools

- **ctmm** (Python package providing preprocess.pseudobulk, preprocess.std, and ctp.free_REML used to build CTP/ctnu/P inputs and fit the Free model with REML) — https://github.com/Minhui-Chen/ctmm_python
- **rpy2** (Bridge through which CTMM calls R optimization routines (BFGS via R optim) during REML fitting) — https://rpy2.github.io/
- **R** (Runtime required by CTMM (via rpy2) for REML optimization; R packages abind, mvtnorm, numDeriv, softImpute, tensor must be installed)
- **Snakemake** (Snakefile pipeline steps for reproducing the simulations and iPSC variance-partitioning analyses) — https://github.com/Minhui-Chen/CTMM

## Examples

```
from ctmm import ctp; free, p_wald = ctp.free_REML(y_f='CTP.gz', P_f='P.gz', ctnu_f='ctnu.gz', method='BFGS', optim_by_R=True)
```

## Evaluation signals

- Reproduced median cell type-specific interindividual variance is 32.8% across cell types within tolerance consistent with REML numerical convergence
- Reproduced median homogeneous variance is 0.4%, proportion differences 12%, residual cell-level variation 10%, pseudobulk-expression PCs 39%, and batch effects 21%
- Weighted cell type-specific variation is 14%
- ctp.free_REML returns the expected two dictionaries: parameter estimates (hom2, V, beta, loglikelihood) and Wald p values (ct_beta, V)
- Filtering invariants: no individual with <=100 cells and no individual-cell type pair with <=10 cells contributes non-missing CTP/ctnu values

## Limitations

- CTMM requires >10 cells per individual-cell type pair, limiting it to common cell types
- Lowly-expressed genes can be severely non-Gaussian, increasing the number of cells needed for the Gaussian approximation
- Pseudobulk inherently discards cell-level information, sacrificing statistical power and resolution within cell types
- Circularity from inferring cell types directly from the scRNA-seq data can deflate estimates of cell type-specific variance by construction
- The Free test is slightly inflated; results require biological validation and replication

## Evidence

- [results] In differentiating iPSCs analyzed with CTMM (Free model, REML, CTP data), cell type-specific interindividual variance has a median of 32.8% across cell types.: [evidence span withheld — anchor status: not_found]
- [methods] Pseudobulk preprocessing cutoffs and inputs are defined in the methods/filter steps.: "compute ctp (cell type-specific pseudobulk) and ctnu (cell type-specific noise variance) and P (cell type proportions)"
- [methods] Individual and cell-count filtering thresholds are documented.: "preprocess.pseudobulk(counts=counts, meta=meta, ind_cut=100, ct_cut=10) # remove individuals with <= 100 cells"
- [readme] README documents the exact fitting call for CTP Free model with REML.: "free, p_wald = ctp.free_REML(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f, method='BFGS', optim_by_R=True)"
- [discussion] Discussion states the cell-count requirement limiting applicability.: "In practice, we require >10 cells per individual-cell type pair, which limits CTMM to common cell types."
