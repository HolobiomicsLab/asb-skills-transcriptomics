---
name: candidate-gene-prioritization
description: Use when scRNA-seq pseudobulk CTMM results are available for many genes
  and you need a ranked, replicated list of variance-differentiation hits, including
  variance-only genes without mean differentiation.
when_to_use_negative:
- Input is raw single-cell counts rather than preprocessed cell type-specific pseudobulk
  (CTP, ctnu, P) from ctmm.preprocess.pseudobulk
- Sample size is below ~50 individuals, where CTMM Free test inflation makes prioritized
  hits unreliable
- The goal is mean-expression differential analysis only — use standard pseudobulk
  DE tools instead of the variance-differentiation intersection
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: ctmm
  role: Fits the Free model on CTP inputs by REML with jack_knife=True and by Haseman-Elston
    regression to obtain per-gene Wald p-values for V and ct_beta
  repo: https://github.com/Minhui-Chen/ctmm_python
- name: rpy2
  role: Bridges the ctmm Python package to R routines used for optimization and jackknife
  repo: https://rpy2.github.io/
- name: R
  role: Runtime required by ctmm through rpy2 (R 3.6–4.4 with rpy2 3.5.x; R >= 4.5
    with rpy2 >= 3.6)
- name: Snakemake
  role: Orchestrates the simulation and iPSC analysis steps, including the hit-calling
    workflow
  repo: https://github.com/Minhui-Chen/CTMM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-49242-9
    title: A robust model for cell type-specific interindividual variation in single-cell
      RNA sequencing data
  source_content_hash: sha256:871ae8b9f1ee052120f991005b0e55926b323b05a3686ef9864fbc896a867079
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# candidate-gene-prioritization

## Summary

Prioritizes genes with significant cell type-specific interindividual variance differentiation by intersecting Bonferroni-corrected REML (jackknife) and Haseman-Elston Wald p-values from CTMM fits on cell type-specific pseudobulk (CTP) data, and flags hits that lack mean-expression differentiation.

## When to use

Use when scRNA-seq pseudobulk CTMM results are available for many genes and you need a ranked, replicated list of variance-differentiation hits, including variance-only genes without mean differentiation.

## When NOT to use

- Input is raw single-cell counts rather than preprocessed cell type-specific pseudobulk (CTP, ctnu, P) from ctmm.preprocess.pseudobulk
- Sample size is below ~50 individuals, where CTMM Free test inflation makes prioritized hits unreliable
- The goal is mean-expression differential analysis only — use standard pseudobulk DE tools instead of the variance-differentiation intersection

## Inputs

- CTP cell type-specific pseudobulk expression matrix (one column per cell type, headerless)
- ctnu measurement-noise variance matrix for individual-by-cell-type pairs (one column per cell type)
- P cell type proportion matrix (one column per cell type)
- Gene list for the 11,231 tested genes

## Outputs

- Ranked hit table (gene, REML (JK) p-values for V and ct_beta, HE p-values, variance-only flag), e.g. ipsc_ctp_hit_table.csv
- Count of variance-only genes (expected 85 in the iPSC-to-endoderm analysis)
- Top-ranked gene list (POU5F1, HSPA8, NDUFB4) with reported p-values

## How to apply

Fit the CTMM Free model on the CTP matrix (one column per cell type) with ctnu measurement-noise variances and P cell type proportions twice: once with REML and jack_knife=True (ctp.free_REML with method='BFGS', optim_by_R=True, jack_knife=True) and once with Haseman-Elston (ctp.free_HE()), collecting Wald p-values for variance differentiation (V) and mean differentiation (ct_beta) per gene. Apply Bonferroni correction across all 11,231 tested genes within each method and retain only genes significant in both REML (JK) and HE, so hits are supported by two independent statistical approaches. Rank the intersected candidates by p-value and flag genes whose REML ct_beta p-value exceeds 0.01 as variance-only hits; in the iPSC-to-endoderm analysis this flag identifies exactly 85 genes without significant stage-specific mean expression. Sanity-check the ranking by confirming known top hits (POU5F1, HSPA8, NDUFB4) appear at the top with their reported p-values before exporting the hit table.

## Related tools

- **ctmm** (Fits the Free model on CTP inputs by REML with jack_knife=True and by Haseman-Elston regression to obtain per-gene Wald p-values for V and ct_beta) — https://github.com/Minhui-Chen/ctmm_python
- **rpy2** (Bridges the ctmm Python package to R routines used for optimization and jackknife) — https://rpy2.github.io/
- **R** (Runtime required by ctmm through rpy2 (R 3.6–4.4 with rpy2 3.5.x; R >= 4.5 with rpy2 >= 3.6))
- **Snakemake** (Orchestrates the simulation and iPSC analysis steps, including the hit-calling workflow) — https://github.com/Minhui-Chen/CTMM

## Examples

```
from ctmm import ctp; free_jk, p_jk = ctp.free_REML(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f, method='BFGS', optim_by_R=True, jack_knife=True); free_he, p_he = ctp.free_HE(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f)
```

## Evaluation signals

- Bonferroni correction applied over exactly 11,231 tested genes within each method (REML JK and HE) before intersection
- The set of variance-significant genes with REML ct_beta p > 0.01 counts exactly 85
- POU5F1, HSPA8, and NDUFB4 appear at the top of the ranked hit table with their reported p-values
- Every selected candidate is significant after Bonferroni correction in BOTH REML (JK) and HE p-value sets
- Output CSV contains columns for gene, REML (JK) p-values, HE p-values, and the variance-only flag with one row per candidate gene

## Limitations

- The Free test is slightly inflated, so prioritized candidates require biological validation and replication
- CTMM requires >10 cells per individual-cell type pair, restricting hits to common cell types
- Lowly-expressed genes can be severely non-Gaussian, invalidating the Gaussian approximation underlying the Wald p-values
- Some circularity arises when cell types are inferred directly from the same scRNA-seq data, deflating cell type-specific variance estimates

## Evidence

- [abstract] CTMM identifies genes with significant stage-specific variability, including 85 genes without significant stage-specific mean expression: "including 85 genes that do not have significant stage-specific mean expression"
- [readme] Both REML jackknife and HE Wald p-values are produced by ctmm model fits: "The second one contains p values from Wald test on e.g. differentiation of mean expression between cell types (ct_beta) and differentiation of expression variance between cell types (V)."
- [readme] Jackknife-based REML fit is invoked with explicit ctmm arguments: "free_jk, p_jk = ctp.free_REML(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f, method='BFGS', optim_by_R=True, jack_knife=True)"
- [discussion] Jackknife-based REML is recommended as the most robust method: "propose a jackknife-based test using REML as the most powerful and robust method"
- [discussion] Results need biological validation and replication due to test inflation: "emphasizing the importance of biologically validating and replicating results"
