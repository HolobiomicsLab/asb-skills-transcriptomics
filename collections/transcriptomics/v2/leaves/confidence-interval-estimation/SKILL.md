---
name: confidence-interval-estimation
description: Use when you have a continuous outcome (e.g., a signature score) and
  continuous predictors (e.g., reaction fluxes) per sample and need effect-size estimates
  with 95% CIs rather than only p-values.
when_to_use_negative:
- The input is already a coefficient/estimate table—no regression needs to be refit.
- Predictors or the outcome are not continuous per-sample quantities (e.g., single-cell
  cluster-level data requiring compute_sc_flux instead).
- The analysis goal is flux computation itself (MRAS/compute_flux) rather than statistical
  association of precomputed fluxes with a signature.
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Fit the OLS model with lm() and extract estimates and 95% CIs with summary()
    and confint(); the METAFlux R package is loaded in the same session
  repo: https://github.com/KChen-lab/METAFlux
- name: METAFlux
  role: Generates the reaction-level flux predictors (oxygen uptake, glucose uptake
    HMR_9034, lactate secretion) via compute_flux on the Human-GEM model that feed
    the regression
  repo: https://github.com/KChen-lab/METAFlux
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-40457-w
    title: Characterizing cancer metabolism from bulk and single-cell RNA-seq data
      using METAFlux
  source_content_hash: sha256:031b5341444d293bef8c73765e9ec4077551c33be3a4dd4744ce93ba08197039
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/METAFlux
  tools:
  - METAFlux
  - R
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# confidence-interval-estimation

## Summary

Fits an ordinary least squares regression in R and extracts coefficient estimates with 95% confidence intervals (summary() and confint()) to quantify associations—in this case between METAFlux-derived metabolic fluxes (oxygen uptake, glucose uptake, lactate secretion) and Winter hypoxia scores in 676 TCGA pan-cancer tumors.

## When to use

Use when you have a continuous outcome (e.g., a signature score) and continuous predictors (e.g., reaction fluxes) per sample and need effect-size estimates with 95% CIs rather than only p-values.

## When NOT to use

- The input is already a coefficient/estimate table—no regression needs to be refit.
- Predictors or the outcome are not continuous per-sample quantities (e.g., single-cell cluster-level data requiring compute_sc_flux instead).
- The analysis goal is flux computation itself (MRAS/compute_flux) rather than statistical association of precomputed fluxes with a signature.

## Inputs

- METAFlux-derived TCGA pan-cancer flux feature table (oxygen uptake, glucose uptake via reaction HMR_9034, lactate secretion exchange fluxes) for 676 tumors
- Winter hypoxia signature score table keyed by TCGA sample identifier

## Outputs

- regression_coefficients.csv with columns term, estimate, std.error, statistic, p.value, ci_lower, ci_upper
- Fitted OLS model object with per-coefficient 95% confidence intervals

## How to apply

Merge the flux feature table (oxygen uptake, glucose uptake from reaction HMR_9034, lactate secretion exchange fluxes produced by METAFlux's compute_flux on the Human-GEM model) with Winter hypoxia signature scores by TCGA sample identifier, keeping only samples with non-missing scores (676 tumors). Fit lm(Hypoxia_score ~ Oxygen_Uptake + Glucose_uptake + Lactate_secretion) with R's lm() on the merged data frame. Call summary() to obtain coefficient estimates, standard errors, statistics, and p-values, and confint() for 95% confidence intervals. Write the coefficient table (term, estimate, std.error, statistic, p.value, ci_lower, ci_upper) to regression_coefficients.csv. Rationale: the OLS coefficients quantify the direction and magnitude of hypoxia-associated metabolic reprogramming, and the CIs support the claim that the associations match known biology (oxygen uptake decreases, glucose uptake and lactate secretion increase with hypoxia).

## Related tools

- **R** (Fit the OLS model with lm() and extract estimates and 95% CIs with summary() and confint(); the METAFlux R package is loaded in the same session) — https://github.com/KChen-lab/METAFlux
- **METAFlux** (Generates the reaction-level flux predictors (oxygen uptake, glucose uptake HMR_9034, lactate secretion) via compute_flux on the Human-GEM model that feed the regression) — https://github.com/KChen-lab/METAFlux

## Examples

```
fit <- lm(Hypoxia_score ~ Oxygen_Uptake + Glucose_uptake + Lactate_secretion, data = merged); coef_tab <- cbind(as.data.frame(summary(fit)$coefficients), confint(fit)); write.csv(coef_tab, 'regression_coefficients.csv')
```

## Evaluation signals

- Coefficient estimates reproduce the reported values: oxygen uptake −0.19, glucose uptake +0.32, lactate secretion +0.09.
- 95% confidence intervals match the paper's TCGA pan-cancer hypoxia association analysis.
- Regression uses exactly the 676 TCGA samples with non-missing Winter hypoxia scores (sample-count check after the merge).
- Output CSV schema contains the columns term, estimate, std.error, statistic, p.value, ci_lower, ci_upper.
- Sign pattern is biologically consistent: oxygen uptake negatively associated with hypoxia, glucose uptake and lactate secretion positively associated.

## Limitations

- No software version numbers are stated for METAFlux, R, or Seurat in the provided text, so exact reproducibility of the flux inputs may vary across versions.
- METAFlux requires choosing one medium file (cell_medium or human_blood); the flux predictors entering the regression depend on that medium choice.
- Optional cubic-root normalization of flux scores is available; applying or omitting it changes the flux scale and hence regression coefficient magnitudes.

## Evidence

- [other] In the TCGA pan-cancer regression against hypoxia scores, oxygen uptake was negatively associated with hypoxia score (Estimate −0.19), while glucose uptake was positively associated (Estimate 0.32) and lactate secretion positively (Estimate 0.09): [evidence span withheld — anchor status: not_found]
- [other] Fit the ordinary least squares model with R's lm(): [evidence span withheld — anchor status: not_found]
- [other] Extract coefficient estimates and confidence intervals: [evidence span withheld — anchor status: not_found]
- [methods] We used Human-GEM (consisting of 13082 metabolic reactions and 8378 metabolites) as our underlying metabolic model: "We used Human-GEM[1] (consisting of 13082 metabolic reactions and 8378 metabolites)as our underlying metabolic model"
- [methods] Subsequently, we calculate the metabolic fluxes for the 13082 reactions: "Subsequently, we calculate the metabolic fluxes for the 13082 reactions"
