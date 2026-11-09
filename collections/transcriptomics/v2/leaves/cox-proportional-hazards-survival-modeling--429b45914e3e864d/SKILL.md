---
name: cox-proportional-hazards-survival-modeling
description: Use when testing whether a candidate biomarker (e.g. binarized GEP weights
  or transferred embedding coordinates) has a statistically significant differential
  effect on patient outcomes between two treatment arms in a randomized trial with
  PFS/OS endpoints.
when_to_use_negative:
- The cohort lacks a randomized treatment contrast — a single-arm survival analysis
  cannot produce coefficient pairs to contrast for differential effects
- Patients requiring distinct treatment protocols (e.g. MSI-H colorectal cancer) are
  not excluded, since they confound the treatment-effect contrast
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2229
tools:
- name: q-diffusion
  role: Upstream method whose qNMF gene expression programs and qPHATE embedding coordinates
    supply the candidate biomarkers tested in the Cox models
  repo: github.com/marmarelis/QDiffusion.jl
- name: q-Diffusion.jl
  role: Julia package used to build the q-diffused kernel (q=1.2) and estimate rank=16
    qNMF GEPs whose transferred patient weights become Cox covariates
  repo: github.com/marmarelis/QDiffusion.jl
- name: PHATE
  role: Embedding framework whose q-diffused variant (qPHATE) yields patient coordinates
    thresholded along their principal component into the binary biomarker covariate
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:199d24216705188685cb499eaa837f509b28efffc9f09ac8d5ae04fdc0e85388
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PHATE
  - q-Diffusion.jl
  - q-diffusion
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cox-proportional-hazards-survival-modeling

## Summary

Fits multivariate Cox proportional hazards models for progression-free survival (PFS) and overall survival (OS) and contrasts coefficient pairs between treatment cohorts to test differential treatment effects, with Benjamini-Hochberg FDR correction.

## When to use

Use when testing whether a candidate biomarker (e.g. binarized GEP weights or transferred embedding coordinates) has a statistically significant differential effect on patient outcomes between two treatment arms in a randomized trial with PFS/OS endpoints.

## When NOT to use

- The cohort lacks a randomized treatment contrast — a single-arm survival analysis cannot produce coefficient pairs to contrast for differential effects
- Patients requiring distinct treatment protocols (e.g. MSI-H colorectal cancer) are not excluded, since they confound the treatment-effect contrast

## Inputs

- Patient-level bulk RNA-derived biomarker values (e.g. binarized qNMF GEP weights or thresholded qPHATE coordinates) merged with survival times and event indicators for PFS and OS
- Clinical covariate table per patient: chemotherapy type, tumor side (left/right), sex, age, ECOG score, tumor mutations (KRAS/NRAS/BRAF)
- Treatment arm assignment from the CALGB/SWOG 80405 randomized phase III trial

## Outputs

- Per-cohort multivariate Cox model coefficient estimates with standard errors for PFS and OS
- Coefficient-pair differences between treatment cohorts with asymptotic normal test statistics (variances from observed Fisher informations)
- Benjamini-Hochberg FDR-corrected p-values identifying biomarkers with significant differential treatment effects (FDR < 0.05, opposing signs)
- Bootstrap resampling estimates of differential hazard robustness

## How to apply

Fit multivariate Cox proportional hazards models for both progression-free survival (PFS) and overall survival (OS), including the biomarker as a covariate alongside clinical covariates: chemotherapy type, tumor side, sex, age, ECOG score, and tumor mutations. In the CALGB/SWOG 80405 mCRC application, patients with high microsatellite instability (MSI-H) are excluded first, and the model is fit separately for the two treatment cohorts (bevacizumab vs. cetuximab-containing regimens). The differential effect is obtained by contrasting each covariate coefficient pair between the two treatment cohorts and testing for a nonzero difference using the asymptotic normal approximation with variance estimates from the observed Fisher informations. Apply Benjamini-Hochberg FDR correction across all coefficient-pair tests; a significant differential effect requires FDR < 0.05 with opposing coefficient signs between the treatment cohorts. Robustness of a strong differential hazard can be checked by bootstrap resampling of patients.

## Related tools

- **q-diffusion** (Upstream method whose qNMF gene expression programs and qPHATE embedding coordinates supply the candidate biomarkers tested in the Cox models) — github.com/marmarelis/QDiffusion.jl
- **q-Diffusion.jl** (Julia package used to build the q-diffused kernel (q=1.2) and estimate rank=16 qNMF GEPs whose transferred patient weights become Cox covariates) — github.com/marmarelis/QDiffusion.jl
- **PHATE** (Embedding framework whose q-diffused variant (qPHATE) yields patient coordinates thresholded along their principal component into the binary biomarker covariate)

## Examples

```
coxph(Surv(time, event) ~ biomarker + chemo_type + tumor_side + sex + age + ecog + mutations, data = cohort_bev); coxph(..., data = cohort_cetuximab); z = (beta_bev - beta_cet) / sqrt(se_bev^2 + se_cet^2)
```

## Evaluation signals

- At least one qNMF gene expression program yields a differential treatment effect significant at Benjamini-Hochberg FDR < 0.05
- The significant coefficient pairs have opposing signs between the two treatment cohorts, reproducing the reported differential bevacizumab effect
- Classical NMF and PHATE baselines (same random seed, same deconvolution transfer) produce no statistically significant differential effects
- Bootstrap resampling of patients confirms robustness of the strong differential hazard on PFS
- Screened biomarkers satisfy the sparsity filter of nonzero GEP weights between 5% and 95% of patients before entering the model

## Limitations

- No predictive marker for bevacizumab efficacy is currently available for clinical patient selection, so the differential-hazard finding is not yet a clinical selection tool
- Variance estimation relies on the asymptotic normal approximation using observed Fisher informations, which may be fragile in smaller cohorts
- Differential-effect testing is specific to contrasting two treatment cohorts; extension to more arms requires additional contrast structure

## Evidence

- [results] Cox regression testing of differential treatment effects identified one qNMF gene expression program with a significant differential effect at FDR < 0.05: [evidence span withheld — anchor status: not_found]
- [results] we estimated multivariate Cox proportional hazards for progression-free survival (PFS) and overall survival (OS): "we estimated multivariate Cox proportional hazards for progression-free survival (PFS) and overall survival (OS)"
- [other] Test each coefficient pair for a nonzero difference using the asymptotic normal approximation with variance estimates from observed Fisher informations, then apply Benjamini-Hochberg FDR correction.: [evidence span withheld — anchor status: not_found]
- [results] We excluded the minority of patients with high microsatellite instability (MSI-H)22, who generally require different protocols altogether.: "We excluded the minority of patients with high microsatellite instability (MSI-H)22, who generally require different protocols altogether."
- [results] Bootstrap resampling of the patients revealed that the strong differential hazard on PFS: "Bootstrap resampling of the patients revealed that the strong differential hazard on PFS"
- [discussion] no predictive marker for bev efficacy is currently available for patient selection in the clinical setting.: "no predictive marker for bev efficacy is currently available for patient selection in the clinical setting."
