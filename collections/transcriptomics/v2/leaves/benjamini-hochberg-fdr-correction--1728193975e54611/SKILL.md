---
name: benjamini-hochberg-fdr-correction
description: Use when many candidate biomarkers (e.g., 16 gene expression programs
  or transferred qPHATE coordinates) each yield a Cox regression treatment-effect
  contrast and the family of p-values must be FDR-corrected before declaring significance.
when_to_use_negative:
- When only a single pre-specified hypothesis is tested (no multiplicity), where a
  plain p-value threshold suffices
- When the goal is family-wise error rate control rather than FDR control (e.g., Bonferroni/Holm
  is required instead)
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: q-Diffusion.jl
  role: Produces the qNMF GEPs and qPHATE embeddings whose treatment-effect coefficient
    contrasts feed the BH-corrected test family
  repo: github.com/marmarelis/QDiffusion.jl
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:633d38af75038017f6981a47f2e6b68b237114745e2b59fd3533661f12be5a4b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PHATE
  - q-Diffusion.jl
  - q-diffusion
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# benjamini-hochberg-fdr-correction

## Summary

Apply Benjamini-Hochberg false discovery rate correction to the family of treatment-effect coefficient-difference tests in the CALGB/SWOG 80405 mCRC analysis, so that multiple Cox-model biomarker contrasts (e.g., qNMF GEPs and qPHATE-derived biomarkers) are assessed at FDR < 0.05 rather than raw p-values.

## When to use

Use when many candidate biomarkers (e.g., 16 gene expression programs or transferred qPHATE coordinates) each yield a Cox regression treatment-effect contrast and the family of p-values must be FDR-corrected before declaring significance.

## When NOT to use

- When only a single pre-specified hypothesis is tested (no multiplicity), where a plain p-value threshold suffices
- When the goal is family-wise error rate control rather than FDR control (e.g., Bonferroni/Holm is required instead)

## Inputs

- Cox proportional hazards regression coefficient pairs (one per treatment cohort) for each candidate biomarker (qNMF GEP weights, qPHATE coordinate threshold biomarker, classical NMF/PHATE baselines)
- Variance estimates for each coefficient difference from observed Fisher informations of the fitted Cox models
- Number of hypothesis tests m in the tested family (all latent variables screened)

## Outputs

- FDR-corrected q-values for each differential treatment-effect test
- Set of significant differential-effect biomarkers at FDR < 0.05 with opposing coefficient signs between treatment cohorts
- Comparison table showing qNMF/qPHATE significance versus classical NMF/PHATE baseline non-significance

## How to apply

First fit multivariate Cox proportional hazards models for PFS and OS with covariates including chemotherapy type, tumor side, sex, age, ECOG score, and tumor mutations, contrasting coefficients between the two treatment cohorts (bevacizumab vs. cetuximab arms). For each biomarker, test the coefficient pair for a nonzero difference using the asymptotic normal approximation, with variance estimates taken from the observed Fisher informations of the fitted Cox models. Collect the p-values across all tested latent variables (e.g., the 16 qNMF GEPs plus qPHATE-derived biomarkers) into one family and apply the Benjamini-Hochberg procedure to control FDR. Judge significance at FDR < 0.05, and additionally require opposing coefficient signs between treatment cohorts to call a genuine differential effect; validate that the standard-NMF baseline shows comparatively weaker or no significance.

## Related tools

- **q-Diffusion.jl** (Produces the qNMF GEPs and qPHATE embeddings whose treatment-effect coefficient contrasts feed the BH-corrected test family) — github.com/marmarelis/QDiffusion.jl

## Evaluation signals

- At least one qNMF gene expression program passes FDR < 0.05 with opposing coefficient signs between the bevacizumab and cetuximab cohorts, reproducing the reported differential bevacizumab effect on PFS
- Classical NMF GEPs and classical PHATE biomarkers yield no FDR-significant differential effects, confirming the baseline is comparatively weaker
- Bootstrap resampling of patients confirms the strong differential hazard on PFS is robust
- Corrected q-values are monotonically non-decreasing with rank-ordered raw p-values, as required by the BH procedure

## Limitations

- The coefficient-difference test relies on an asymptotic normal approximation with variances from observed Fisher informations, which may be inaccurate for small cohorts or weak biomarkers
- BH FDR control does not guarantee individual significant findings replicate; bootstrap resampling was needed to confirm robustness of the differential hazard

## Evidence

- [other] then apply Benjamini-Hochberg FDR correction: [evidence span withheld — anchor status: not_found]
- [results] one qNMF gene expression program with a significant differential effect at FDR < 0.05: [evidence span withheld — anchor status: not_found]
- [results] estimated multivariate Cox proportional hazards for progression-free survival (PFS) and overall survival (OS): "we estimated multivariate Cox proportional hazards for progression-free survival (PFS) and overall survival (OS)"
