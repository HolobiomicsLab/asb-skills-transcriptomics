---
name: hypoxia-signature-association-analysis
description: Use when you have per-sample flux features from METAFlux and a hypoxia
  signature score table for the same TCGA samples, and need to quantify flux–hypoxia
  associations via OLS regression.
when_to_use_negative:
- The goal is to compute the fluxes themselves (MRAS/compute_flux) rather than to
  associate already-computed fluxes with a signature score
- The sample identifiers cannot be matched between the flux table and the hypoxia
  score table, leaving too few samples with non-missing Winter scores for a stable
  regression
- The analysis targets single-cell community-level flux interactions in the TME rather
  than bulk per-sample flux–signature associations
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3307
tools:
- name: R
  role: Environment for merging flux and hypoxia-score tables by TCGA sample identifier
    and fitting the OLS model with lm(), summary(), and confint()
  repo: https://github.com/KChen-lab/METAFlux
- name: METAFlux
  role: Produces the 13,082 reaction flux scores (including glucose uptake HMR_9034
    and lactate secretion exchange fluxes) that serve as regression predictors
  repo: https://github.com/KChen-lab/METAFlux
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-40457-w
    title: Characterizing cancer metabolism from bulk and single-cell RNA-seq data
      using METAFlux
  source_content_hash: sha256:2205132205d9fa0ffdacc3f840e642446d7ad9a2fcf67360511e0fa5e37d2c27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/METAFlux
  tools:
  - METAFlux
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# hypoxia-signature-association-analysis

## Summary

Regress METAFlux-derived metabolic flux features (oxygen uptake, glucose uptake via reaction HMR_9034, lactate secretion) against the Winter hypoxia signature score in TCGA pan-cancer tumors to test whether inferred fluxes capture hypoxia-mediated metabolic reprogramming.

## When to use

Use when you have per-sample flux features from METAFlux and a hypoxia signature score table for the same TCGA samples, and need to quantify flux–hypoxia associations via OLS regression.

## When NOT to use

- The goal is to compute the fluxes themselves (MRAS/compute_flux) rather than to associate already-computed fluxes with a signature score
- The sample identifiers cannot be matched between the flux table and the hypoxia score table, leaving too few samples with non-missing Winter scores for a stable regression
- The analysis targets single-cell community-level flux interactions in the TME rather than bulk per-sample flux–signature associations

## Inputs

- METAFlux-derived TCGA pan-cancer flux feature table (oxygen uptake, glucose uptake via HMR_9034, lactate secretion exchange fluxes) for 676 tumors, keyed by TCGA sample identifier
- Winter hypoxia signature score table keyed by TCGA sample identifier

## Outputs

- regression_coefficients.csv with columns term, estimate, std.error, statistic, p.value, ci_lower, ci_upper
- Fitted OLS model object (lm) with 95% confidence intervals per coefficient

## How to apply

Load the METAFlux-derived TCGA pan-cancer flux feature table (oxygen uptake, glucose uptake via reaction HMR_9034, and lactate secretion exchange fluxes) for the 676 tumors together with the Winter hypoxia signature score table in R. Merge the two tables by TCGA sample identifier, keeping only the samples with non-missing Winter hypoxia scores (676 samples). Fit the ordinary least squares model lm(Hypoxia_score ~ Oxygen_Uptake + Glucose_uptake + Lactate_secretion) with R's lm(), then extract coefficient estimates and 95% confidence intervals using summary() and confint(). The rationale is that hypoxic tumors downregulate oxygen uptake while increasing glucose uptake and lactate secretion, so the sign and magnitude of the coefficients should reflect this reprogramming if the fluxes are biologically meaningful.

## Related tools

- **R** (Environment for merging flux and hypoxia-score tables by TCGA sample identifier and fitting the OLS model with lm(), summary(), and confint()) — https://github.com/KChen-lab/METAFlux
- **METAFlux** (Produces the 13,082 reaction flux scores (including glucose uptake HMR_9034 and lactate secretion exchange fluxes) that serve as regression predictors) — https://github.com/KChen-lab/METAFlux

## Examples

```
model <- lm(Hypoxia_score ~ Oxygen_Uptake + Glucose_uptake + Lactate_secretion, data = tcga_merged); summary(model); confint(model)
```

## Evaluation signals

- Coefficient estimates reproduce the reported values: oxygen uptake −0.19, glucose uptake +0.32, lactate secretion +0.09
- 95% confidence intervals from confint() match the paper's TCGA pan-cancer hypoxia association analysis
- Merged dataset contains exactly the 676 samples with non-missing Winter hypoxia scores
- regression_coefficients.csv has the expected schema: term, estimate, std.error, statistic, p.value, ci_lower, ci_upper
- Sign pattern of coefficients matches hypoxia-mediated reprogramming: negative oxygen uptake, positive glucose uptake and lactate secretion

## Limitations

- Upstream flux predictions depend on the Human-GEM model (13,082 reactions, 8,378 metabolites) with a biomass-maximization objective, so regression coefficients inherit any biases in the flux inference
- No software version numbers for METAFlux, R, or Seurat are stated, so exact reproducibility of flux inputs is not guaranteed
- Association is cross-sectional OLS on TCGA bulk samples; it does not establish causality of hypoxia-driven reprogramming

## Evidence

- [methods] Subsequently, we calculate the metabolic fluxes for the 13082 reactions: "Subsequently, we calculate the metabolic fluxes for the 13082 reactions"
- [methods] We used Human-GEM[1] (consisting of 13082 metabolic reactions and 8378 metabolites)as our underlying metabolic model: "We used Human-GEM[1] (consisting of 13082 metabolic reactions and 8378 metabolites)as our underlying metabolic model"
- [readme] METAFlux R package can be easily installed from Github using devtools: "METAFlux R package can be easily installed from Github using devtools:

`devtools::install_github('KChen-lab/METAFlux')`"
- [readme] Use genome scale metabolic model and FBA to derive 13,082 metabolic fluxes: [evidence span withheld — anchor status: external_doc]
