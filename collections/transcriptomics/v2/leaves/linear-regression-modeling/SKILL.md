---
name: linear-regression-modeling
description: Use when you have a merged per-sample table of continuous predictors
  and a continuous response (e.g., 676 TCGA tumors with METAFlux flux features and
  Winter hypoxia scores) and need coefficient estimates, standard errors, p-values,
  and 95% CIs to test metabolic reprogramming associations.
when_to_use_negative:
- The input is already a fitted regression output or coefficient table rather than
  per-sample predictor/response data
- You need nonlinear or interaction effects between fluxes and the hypoxia score —
  this skill covers the paper's ordinary least squares formulation only
- Flux features have not yet been computed from RNA-seq (first run the METAFlux compute_flux
  pipeline on the Human-GEM model with the human_blood medium)
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Statistical environment used to load the merged flux/hypoxia table, fit lm(Hypoxia_score
    ~ Oxygen_Uptake + Glucose_uptake + Lactate_secretion), and export coefficients
    via summary() and confint()
- name: METAFlux
  role: Upstream package that produces the per-sample metabolic flux features (oxygen
    uptake, glucose uptake, lactate secretion) used as regression predictors
  repo: https://github.com/KChen-lab/METAFlux
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-40457-w
    title: Characterizing cancer metabolism from bulk and single-cell RNA-seq data
      using METAFlux
  source_content_hash: sha256:03c5b1971e32cfee9dac7b7bebf80b3f2ea46a984e42530e583d909da21485f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/METAFlux
  tools:
  - METAFlux
  - R
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# linear-regression-modeling

## Summary

Fit ordinary least squares multiple regression in R (lm()) to test whether METAFlux-derived metabolic flux features (oxygen uptake, glucose uptake via HMR_9034, lactate secretion) are linearly associated with Winter hypoxia signature scores across TCGA pan-cancer tumors, extracting coefficient estimates and 95% confidence intervals.

## When to use

Use when you have a merged per-sample table of continuous predictors and a continuous response (e.g., 676 TCGA tumors with METAFlux flux features and Winter hypoxia scores) and need coefficient estimates, standard errors, p-values, and 95% CIs to test metabolic reprogramming associations.

## When NOT to use

- The input is already a fitted regression output or coefficient table rather than per-sample predictor/response data
- You need nonlinear or interaction effects between fluxes and the hypoxia score — this skill covers the paper's ordinary least squares formulation only
- Flux features have not yet been computed from RNA-seq (first run the METAFlux compute_flux pipeline on the Human-GEM model with the human_blood medium)

## Inputs

- METAFlux-derived TCGA pan-cancer flux feature table (oxygen uptake, glucose uptake HMR_9034, lactate secretion exchange fluxes) for the 676 tumors
- Winter hypoxia signature score table keyed by TCGA sample identifier
- R environment with lm(), summary(), confint() (base stats)

## Outputs

- regression_coefficients.csv with columns term, estimate, std.error, statistic, p.value, ci_lower, ci_upper
- Fitted OLS model object for the hypoxia ~ fluxes regression

## How to apply

Load the METAFlux-derived TCGA pan-cancer flux feature table — oxygen uptake, glucose uptake (reaction HMR_9034), and lactate secretion exchange fluxes — together with the Winter hypoxia signature score table in R. Merge the two tables by TCGA sample identifier, keeping only the 676 samples with non-missing hypoxia scores. Fit the OLS model lm(Hypoxia_score ~ Oxygen_Uptake + Glucose_uptake + Lactate_secretion) with R's lm(), then extract coefficients and 95% confidence intervals using summary() and confint(). Write the coefficient table (term, estimate, std.error, statistic, p.value, ci_lower, ci_upper) to regression_coefficients.csv. The flux features themselves come from the METAFlux pipeline (MRAS from gene expression via GPR, then compute_flux against the human_blood medium on the Human-GEM model with 13,082 reactions), optionally with cubic-root normalization of flux scores.

## Related tools

- **R** (Statistical environment used to load the merged flux/hypoxia table, fit lm(Hypoxia_score ~ Oxygen_Uptake + Glucose_uptake + Lactate_secretion), and export coefficients via summary() and confint())
- **METAFlux** (Upstream package that produces the per-sample metabolic flux features (oxygen uptake, glucose uptake, lactate secretion) used as regression predictors) — https://github.com/KChen-lab/METAFlux

## Examples

```
fit <- lm(Hypoxia_score ~ Oxygen_Uptake + Glucose_uptake + Lactate_secretion, data = tcga_flux_hypoxia); summary(fit); confint(fit)
```

## Evaluation signals

- Coefficient estimates reproduce the reported values: oxygen uptake −0.19 (negative), glucose uptake +0.32, lactate secretion +0.09 (both positive)
- Output regression_coefficients.csv contains exactly the columns term, estimate, std.error, statistic, p.value, ci_lower, ci_upper
- The regression cohort equals the 676 TCGA tumors with non-missing Winter hypoxia scores after the sample-ID merge
- 95% confidence intervals from confint() match the paper's TCGA pan-cancer hypoxia association analysis
- Sign structure is consistent with hypoxia-mediated reprogramming: oxygen uptake negatively associated, glucose uptake and lactate secretion positively associated

## Limitations

- The association is an ordinary least squares fit; it captures only linear, additive relationships between fluxes and the hypoxia score
- Validation is anchored to the specific Winter hypoxia signature and the 676-sample TCGA pan-cancer cohort; other signatures or cohorts are not covered
- No version numbers were stated for METAFlux or R in the provided text, so exact package versions used for the regression are unrecorded

## Evidence

- [other] Fit the ordinary least squares model lm(Hypoxia_score ~ Oxygen_Uptake + Glucose_uptake + Lactate_secretion) with R's lm().: [evidence span withheld — anchor status: not_found]
- [other] Extract coefficient estimates and 95% confidence intervals with summary() and confint().: [evidence span withheld — anchor status: not_found]
- [intro] METAFlux can predict 13,082 reaction flux scores per bulk sample: "METAFlux can predict 13,082 reaction flux scores for each bulk sample, and for each single cell data, it can predict (13,082 × number of cell-type/cluster + 1648) reaction flux scores."
- [readme] Use genome scale metabolic model and FBA to derive 13,082 metabolic fluxes for bulk datasets: [evidence span withheld — anchor status: external_doc]
- [methods] Flux features derive from MRAS via GPR and compute_flux: "calculate a single sample normalized MRAS from gene expression using GPR(Gene-protein-reaction)"
