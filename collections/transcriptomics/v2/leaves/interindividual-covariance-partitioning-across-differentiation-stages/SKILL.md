---
name: interindividual-covariance-partitioning-across-differentiation-stages
description: Use when scRNA-seq pseudobulk data span ordered cell types or stages
  and the goal is to test whether interindividual expression correlations between
  adjacent stages exceed those between more distant stages.
when_to_use_negative:
- Sample size is below ~50 individuals — CTMM is not reliable at this scale and the
  Free test is slightly inflated
- Individual-cell type pairs have <=10 cells — CTMM requires >10 cells per pair and
  such pairs are filtered out (ct_cut=10), limiting it to common cell types
- Cell types are continuous rather than discrete — they can only be expressed in pseudobulk
  by discretizing them, which this workflow assumes has already happened
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: ctmm
  role: Python package whose ctp module provides ctp.full_REML for fitting the Full
    model and whose preprocess utilities build the CTP/ctnu/P inputs
  repo: https://github.com/Minhui-Chen/ctmm_python
- name: rpy2
  role: Bridge through which CTMM calls R routines for optimization and REML fitting
  repo: https://rpy2.github.io/
- name: R
  role: Runtime required by rpy2; BFGS optimization via R optim when optim_by_R=True
- name: Snakemake
  role: Snakefile in the analysis repository contains the steps for iPSCs analyses
    that this covariance partitioning reproduces
  repo: https://github.com/Minhui-Chen/CTMM
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-49242-9
    title: A robust model for cell type-specific interindividual variation in single-cell
      RNA sequencing data
  source_content_hash: sha256:1701618f12675bd05300ba1bdec2af94c1c3251d2060ef0d1a2df2a20c83b9ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Minhui-Chen/ctmm_python
  tools:
  - R
  - Snakemake
  - ctmm
  - rpy2
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# interindividual-covariance-partitioning-across-differentiation-stages

## Summary

Fits the CTMM Full model by REML on cell type-specific pseudobulk (CTP) data to partition interindividual covariance between ordered differentiation stages (here the 4 day-stages of the Cuomo et al. 2020 iPSC-to-endoderm trajectory), then converts the partitioned stage-specific covariance (V) into a stage-by-stage correlation matrix.

## When to use

Use when scRNA-seq pseudobulk data span ordered cell types or stages and the goal is to test whether interindividual expression correlations between adjacent stages exceed those between more distant stages.

## When NOT to use

- Sample size is below ~50 individuals — CTMM is not reliable at this scale and the Free test is slightly inflated
- Individual-cell type pairs have <=10 cells — CTMM requires >10 cells per pair and such pairs are filtered out (ct_cut=10), limiting it to common cell types
- Cell types are continuous rather than discrete — they can only be expressed in pseudobulk by discretizing them, which this workflow assumes has already happened

## Inputs

- CTP file: cell type-specific pseudobulk gene expression, one headerless column per cell type (differentiation stage) per individual
- ctnu file: variance of measurement noise for each pair of individual and cell type, one headerless column per cell type
- P file: cell type (stage) proportions for each individual, one headerless column per cell type

## Outputs

- Dictionary of fitted Full-model parameters: stage-shared random-effect variance (hom2), stage-specific covariance matrix (V), cell type-specific mean expression (beta), loglikelihood (l)
- Dictionary of Wald-test p values (e.g. on ct_beta and V differentiation)
- 4x4 correlation matrix of stage-specific interindividual effects (stage_effect_correlation_matrix.csv)

## How to apply

Load the CTP, ctnu, and P input files for the 4-stage iPSC-to-endoderm trajectory, where each 'cell type' is a differentiation stage. Import the ctmm ctp module and fit the Full model by restricted maximum likelihood via ctp.full_REML(y_f, P_f, ctnu_f, method='BFGS', optim_by_R=True), which optimizes with BFGS through R. Extract the stage-shared random-effect variance (hom2) and the stage-specific covariance matrix (V) from the first output dictionary; here almost 100% of transcriptome-wide between-donor variability is stage-specific. Convert the partitioned interindividual covariance between stages into a 4x4 correlation matrix of stage-specific effects, and validate that the median adjacent-stage correlations (CT1-CT2 = 0.318, CT3-CT4 = 0.322) exceed the CT2-CT3 correlation (0.051), confirming the covariance partition recapitulates the differentiation trajectory before writing stage_effect_correlation_matrix.csv.

## Related tools

- **ctmm** (Python package whose ctp module provides ctp.full_REML for fitting the Full model and whose preprocess utilities build the CTP/ctnu/P inputs) — https://github.com/Minhui-Chen/ctmm_python
- **rpy2** (Bridge through which CTMM calls R routines for optimization and REML fitting) — https://rpy2.github.io/
- **R** (Runtime required by rpy2; BFGS optimization via R optim when optim_by_R=True)
- **Snakemake** (Snakefile in the analysis repository contains the steps for iPSCs analyses that this covariance partitioning reproduces) — https://github.com/Minhui-Chen/CTMM

## Examples

```
from ctmm import ctp; full, p_wald = ctp.full_REML(y_f='test/CTP.gz', P_f='test/P.gz', ctnu_f='test/ctnu.gz', method='BFGS', optim_by_R=True)
```

## Evaluation signals

- Median adjacent-stage correlations CT1-CT2 = 0.318 and CT3-CT4 = 0.322 exceed the CT2-CT3 median correlation of 0.051 in the estimated 4x4 matrix
- The correlation matrix is 4x4, symmetric, with unit diagonal, derived from the fitted V and hom2 components
- The first output dictionary contains finite estimates of hom2 and V plus a loglikelihood (l) from the REML fit
- The stage-shared component hom2 contributes almost none of the between-donor variability, consistent with ~100% stage-specific variability reported for iPSC differentiation
- The BFGS optimization via R (optim_by_R=True) converges without error for the 4-stage, 50-individual-scale input

## Limitations

- Requires >10 cells per individual-cell type pair, limiting CTMM to common cell types
- CTMM is not reliable for sample sizes below ~50 and the Free test is slightly inflated, so results require biological validation and replication
- Lowly-expressed genes can be severely non-Gaussian, increasing the number of cells needed for the Gaussian approximation
- Circularity from inferring cell types directly from the scRNA-seq data can deflate estimates of cell type-specific variance by construction

## Evidence

- [abstract] we extend CTMM to partition interindividual covariance between stages, which recapitulates the overall differentiation trajectory: "we extend CTMM to partition interindividual covariance between stages, which recapitulates the overall differentiation trajectory."
- [readme] To fit CTP, import the ctp module and call the function in the same way. For example, ctp.full_ML() to fit the Full model using ML.: "To fit CTP, import the ``ctp`` module and call the function in the same way. For example, ``ctp.full_ML()`` to fit the Full model using ML."
- [readme] The first one contains estimates of model parameters, including cell type-specific mean expression (beta), variance of cell type-shared random effect (hom2), variance of cell type-specific random effect (V): "The first one contains estimates of model parameters, including cell type-specific mean expression (beta), variance of cell type-shared random effect (hom2), variance of cell type-specific random"
- [readme] CTP: Cell Type-specific Pseudobulk gene expression for each individual. This file should have one column for each cell type and without header.: "CTP: Cell Type-specific Pseudobulk gene expression for each individual. This file should have one column for each cell type and without header."
- [other] the correlation between CT2 and CT3 (median: 0.051) was smaller than the other adjacent stages (CT1-CT2 median: 0.318; CT3-CT4 median: 0.322): [evidence span withheld — anchor status: not_found]
- [abstract] Almost 100% of transcriptome-wide variability between donors is differentiation stage-specific in iPSC differentiation to endoderm: [evidence span withheld — anchor status: not_found]
