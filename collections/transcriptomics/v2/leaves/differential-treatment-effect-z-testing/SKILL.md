---
name: differential-treatment-effect-z-testing
description: Use when you have patient-level biomarker values (e.g., binarized GEP
  weights or qPHATE-coordinate thresholds) from a randomized trial with PFS/OS outcomes
  and two treatment cohorts, and you need to test for a statistically significant
  differential treatment effect.
when_to_use_negative:
- The study design is not a randomized comparison of two treatments (e.g., single-arm
  or observational data), since contrasting Cox coefficients between cohorts presumes
  randomized arm assignment
- The biomarker values are continuous embeddings without a defined binary transfer/threshold
  step and no sparsity screen (5%-95% nonzero weights) has been applied
- The analysis goal is clustering or embedding quality (e.g., Tabula Sapiens ARI benchmarking)
  rather than patient-outcome differential effects
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: q-Diffusion.jl
  role: Upstream generator of the candidate latent-variable biomarkers (qNMF GEPs,
    qPHATE embeddings) whose transferred coefficients are contrasted by this z-testing
    procedure
  repo: github.com/marmarelis/QDiffusion.jl
- name: q-diffusion
  role: Method whose qNMF gene expression programs yield biomarkers expected to show
    FDR-significant differential treatment effects, unlike classical NMF baselines
  repo: github.com/marmarelis/QDiffusion.jl
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:9fcb2fadc0f2962b681c806e8f1b04ec3dae40c5ec2f59977333a1dda59d9956
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PHATE
  - q-Diffusion.jl
  - q-diffusion
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# differential-treatment-effect-z-testing

## Summary

Statistical procedure for testing whether a biomarker's Cox proportional-hazards coefficient differs between two treatment arms in a randomized trial (CALGB/SWOG 80405, bevacizumab vs cetuximab in mCRC), using an asymptotic normal (z) approximation on coefficient contrasts with Benjamini-Hochberg FDR correction.

## When to use

Use when you have patient-level biomarker values (e.g., binarized GEP weights or qPHATE-coordinate thresholds) from a randomized trial with PFS/OS outcomes and two treatment cohorts, and you need to test for a statistically significant differential treatment effect.

## When NOT to use

- The study design is not a randomized comparison of two treatments (e.g., single-arm or observational data), since contrasting Cox coefficients between cohorts presumes randomized arm assignment
- The biomarker values are continuous embeddings without a defined binary transfer/threshold step and no sparsity screen (5%-95% nonzero weights) has been applied
- The analysis goal is clustering or embedding quality (e.g., Tabula Sapiens ARI benchmarking) rather than patient-outcome differential effects

## Inputs

- Patient-level biomarker values transferred onto bulk RNA profiles (e.g., binary qNMF GEP-weight vector, 0 for zero and 1 for nonzero; or qPHATE coordinates thresholded along their principal component)
- Per-patient survival outcomes: progression-free survival (PFS) and overall survival (OS) times and event indicators
- Treatment-arm assignment from the CALGB/SWOG 80405 randomized phase III trial
- Clinical covariates: chemotherapy type, tumor side, sex, age, ECOG score, tumor mutations

## Outputs

- Cox regression coefficient estimates per treatment arm for each biomarker
- z-statistics and p-values for the nonzero-difference test on each coefficient pair (asymptotic normal approximation)
- Benjamini-Hochberg FDR-corrected q-values with significance calls at FDR < 0.05
- Direction-of-effect calls (opposing vs. concordant coefficient signs across arms)
- Bootstrap robustness assessment of the differential hazard

## How to apply

First exclude patients with high microsatellite instability (MSI-H), who require different protocols. Fit multivariate Cox proportional hazards separately for progression-free survival (PFS) and overall survival (OS) with covariates including chemotherapy type, tumor side, sex, age, ECOG score, and tumor mutations, in each treatment cohort. For each candidate biomarker coefficient, test the pair of coefficients for a nonzero difference using the asymptotic normal approximation, with variance estimates taken from the observed Fisher informations of the two fits. Apply Benjamini-Hochberg FDR correction across the tested coefficient pairs, and declare a differential effect only when the FDR-corrected difference is significant AND the coefficient signs are opposing between treatment cohorts. Confirm robustness by bootstrap resampling of patients.

## Related tools

- **q-Diffusion.jl** (Upstream generator of the candidate latent-variable biomarkers (qNMF GEPs, qPHATE embeddings) whose transferred coefficients are contrasted by this z-testing procedure) — github.com/marmarelis/QDiffusion.jl
- **q-diffusion** (Method whose qNMF gene expression programs yield biomarkers expected to show FDR-significant differential treatment effects, unlike classical NMF baselines) — github.com/marmarelis/QDiffusion.jl

## Evaluation signals

- At least one qNMF gene expression program yields an FDR-corrected significant differential treatment effect at FDR < 0.05, with opposing coefficient signs between the two treatment cohorts
- The classical (standard) NMF baseline produces no GEP with any statistically significant differential effect at the same FDR threshold
- Classical NMF and PHATE biomarkers likewise fail to produce significant differential effects, so the q-diffused variants should outperform them in significance
- Bootstrap resampling of patients reproduces the strong differential hazard on PFS
- Biomarker sparsity screen passes: patient GEP weights between 5% and 95% nonzero

## Limitations

- The difference test relies on an asymptotic normal approximation; bootstrap resampling of patients is needed to verify robustness of the differential hazard on PFS
- Results are reported as statistical significance for differential bevacizumab effects, but the article notes that no predictive marker for bevacizumab efficacy is currently available for clinical patient selection
- Patients with high microsatellite instability (MSI-H) must be excluded first, as they generally require different treatment protocols altogether

## Evidence

- [abstract] q-diffusion helps gain statistical significance for differential effects on patient outcomes in the CALGB/SWOG 80405 trial: "q-diffusion helps gain statistical significance for differential effects on patient outcomes when analyzing the CALGB/SWOG 80405 randomized phase III clinical trial"
- [results] Cox proportional hazards for PFS and OS with clinical covariates: "we estimated multivariate Cox proportional hazards for progression-free survival (PFS) and overall survival (OS)"
- [results] Binarization of patient GEP weights to form the qNMF biomarker: [evidence span withheld — anchor status: not_found]
- [results] Bootstrap robustness check of the differential hazard: "Bootstrap resampling of the patients revealed that the strong differential hazard on PFS"
- [results] MSI-H patients excluded before outcome modeling: "We excluded the minority of patients with high microsatellite instability (MSI-H)22, who generally require different protocols altogether."
