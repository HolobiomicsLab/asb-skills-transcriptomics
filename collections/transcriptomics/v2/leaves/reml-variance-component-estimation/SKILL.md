---
name: reml-variance-component-estimation
description: Use when per-gene, per-cell-type variance components must be estimated
  from pseudobulk scRNA-seq and stage- or cell type-specific interindividual variation
  (with calibrated Wald p-values) is the target signal.
when_to_use_negative:
- Sample size is below ~50 individuals, where the Free test is slightly inflated and
  CTMM is not reliable
- Rare cell types with <=10 cells per individual-cell type pair, since CTMM requires
  >10 cells per pair and is limited to common cell types
- Genes with very low expression, which are severely non-Gaussian and violate the
  Gaussian pseudobulk approximation
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: ctmm
  role: Python package whose ctp module provides ctp.free_REML with jack_knife=True
    for REML fitting and jackknife-based Wald tests on CTP inputs
  repo: https://github.com/Minhui-Chen/ctmm_python
- name: rpy2
  role: Bridge through which ctmm calls the underlying R optimization and inference
    routines
  repo: https://rpy2.github.io/
- name: R
  role: Runtime required by ctmm via rpy2; supplies MASS and Matrix packages
- name: Snakemake
  role: Snakefile orchestrates the simulation and iPSC analysis steps that invoke
    the REML fits
  repo: https://github.com/Minhui-Chen/CTMM
- name: numDeriv
  role: R dependency of the CTMM routines (installed with abind, mvtnorm, softImpute,
    tensor)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-49242-9
    title: A robust model for cell type-specific interindividual variation in single-cell
      RNA sequencing data
  source_content_hash: sha256:2db4f6fe264ee399052a3f24a6c338ceea1745ec3a4d9c5d5fd89642be28f983
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Minhui-Chen/ctmm_python
  tools:
  - R
  - Snakemake
  - ctmm
  - mvtnorm
  - numDeriv
  - rpy2
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# reml-variance-component-estimation

## Summary

Fit the CTMM Free model to cell type-specific pseudobulk (CTP) scRNA-seq data with restricted maximum likelihood (REML), using jackknife-based Wald tests to detect cell type-specific interindividual variance differentiation (V) and mean expression differentiation (ct_beta) per gene.

## When to use

Use when per-gene, per-cell-type variance components must be estimated from pseudobulk scRNA-seq and stage- or cell type-specific interindividual variation (with calibrated Wald p-values) is the target signal.

## When NOT to use

- Sample size is below ~50 individuals, where the Free test is slightly inflated and CTMM is not reliable
- Rare cell types with <=10 cells per individual-cell type pair, since CTMM requires >10 cells per pair and is limited to common cell types
- Genes with very low expression, which are severely non-Gaussian and violate the Gaussian pseudobulk approximation

## Inputs

- CTP matrix: cell type-specific pseudobulk gene expression, one column per cell type (e.g., test/CTP.gz)
- ctnu matrix: variance of measurement noise per individual-cell type pair, one column per cell type
- P matrix: cell type proportions per individual, one column per cell type
- counts and meta files for preprocess.pseudobulk (QC'd, normalized single-cell expression)

## Outputs

- Dictionary of parameter estimates: beta/ct_beta (cell type-specific mean expression), hom2 (cell type-shared random effect variance), V (cell type-specific random effect variance), loglikelihood (l)
- Dictionary of Wald p-values for variance differentiation (V) and mean expression differentiation (ct_beta) per gene
- Ranked hit table (e.g., ipsc_ctp_hit_table.csv) with gene, REML (JK) p-values, HE p-values, and variance-only flag

## How to apply

Aggregate single-cell counts into CTP (one column per cell type), ctnu (measurement-noise variance per individual-cell type pair), and P (cell type proportions), after filtering with preprocess.pseudobulk(counts, meta, ind_cut=100, ct_cut=10). Fit the Free model on CTP with ctp.free_REML(y_f=CTP, P_f=P, ctnu_f=ctnu, method='BFGS', optim_by_R=True, jack_knife=True) so REML optimizes the Gaussian linear mixed model and the jackknife yields calibrated Wald p-values on V and ct_beta; optionally obtain an independent second set of p-values with ctp.free_HE(). Apply Bonferroni correction across all 11,231 tested genes within each method and require significance in both REML (JK) and HE before ranking hits. Verify calibration/power by checking that known top hits (e.g., POU5F1, HSPA8, NDUFB4) recover their reported p-values and that genes significant for V but with ct_beta p>0.01 in REML are enumerated separately (85 such genes in the iPSC-to-endoderm data).

## Related tools

- **ctmm** (Python package whose ctp module provides ctp.free_REML with jack_knife=True for REML fitting and jackknife-based Wald tests on CTP inputs) — https://github.com/Minhui-Chen/ctmm_python
- **rpy2** (Bridge through which ctmm calls the underlying R optimization and inference routines) — https://rpy2.github.io/
- **R** (Runtime required by ctmm via rpy2; supplies MASS and Matrix packages)
- **Snakemake** (Snakefile orchestrates the simulation and iPSC analysis steps that invoke the REML fits) — https://github.com/Minhui-Chen/CTMM
- **numDeriv** (R dependency of the CTMM routines (installed with abind, mvtnorm, softImpute, tensor))

## Examples

```
from ctmm import ctp; free_jk, p_jk = ctp.free_REML(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f, method='BFGS', optim_by_R=True, jack_knife=True)
```

## Evaluation signals

- Bonferroni correction across all 11,231 tested genes is applied within each method, and candidate hits are significant in both REML (JK) and HE
- Top-ranked hits reproduce the reported genes and p-values, with POU5F1, HSPA8, and NDUFB4 appearing at the top of the ranked table
- Count of genes significant for variance (V) whose mean test is not significant (p>0.01 in REML) equals 85 in the iPSC-to-endoderm analysis
- Output dictionaries contain both estimates (hom2, V, ct_beta, l) and Wald p-values for V and ct_beta for every gene
- Jackknife run flag: p-values are taken from ctp.free_REML(..., jack_knife=True), not the plain REML fit

## Limitations

- CTMM requires >10 cells per individual-cell type pair, which limits it to common cell types
- The Free test is slightly inflated; results are not reliable for sample sizes below ~50 and require biological validation and replication
- Lowly-expressed genes can be severely non-Gaussian, increasing the number of cells needed for the Gaussian approximation
- The iid model's code is not thoroughly tested; prefer Free, hom, or full models

## Evidence

- [discussion] Jackknife-based test using REML to fit Gaussian LMMs: "propose a jackknife-based test using REML as the most powerful and robust method"
- [readme] CTP REML call with jackknife: "free_jk, p_jk = ctp.free_REML(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f, method='BFGS', optim_by_R=True, jack_knife=True)"
- [readme] Wald test output structure: "The second one contains p values from Wald test on e.g. differentiation of mean expression between cell types (ct_beta) and differentiation of expression variance between cell types (V)."
- [methods] Pseudobulk filtering thresholds: "preprocess.pseudobulk(counts=counts, meta=meta, ind_cut=100, ct_cut=10) # remove individuals with <= 100 cells"
- [abstract] 85 variance-only genes finding: "including 85 genes that do not have significant stage-specific mean expression"
