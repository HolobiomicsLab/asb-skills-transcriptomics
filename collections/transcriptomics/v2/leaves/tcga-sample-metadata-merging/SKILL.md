---
name: tcga-sample-metadata-merging
description: Use when TCGA pan-cancer sample-level flux features must be joined to
  an external per-sample signature table (e.g., Winter hypoxia scores) by TCGA sample
  barcode before association testing.
when_to_use_negative:
- Input is already a single merged analysis-ready table with no need for sample-identifier
  joining
- Analysis targets single-cell community fluxes (compute_sc_flux with cell-type fractions),
  not bulk per-sample TCGA association testing
- Samples lack the external signature scores (e.g., missing Winter hypoxia scores),
  since the inner-join would drop them and the regression cannot be fit
tools:
- name: R
  role: Environment for the merge, lm() regression fitting, confint() CI extraction,
    and CSV export
  repo: https://github.com/KChen-lab/METAFlux
- name: METAFlux
  role: Source of the bulk-sample metabolic flux features (13,082 reaction flux scores
    per bulk sample) that are merged with hypoxia scores
  repo: https://github.com/KChen-lab/METAFlux
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-40457-w
    title: Characterizing cancer metabolism from bulk and single-cell RNA-seq data
      using METAFlux
  source_content_hash: sha256:7b7dfbbfe8118cccb0572d37b57c71fa7a331824e923dddd83c214b690aac278
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/METAFlux
  tools:
  - METAFlux
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# tcga-sample-metadata-merging

## Summary

Merges METAFlux-derived bulk flux features (oxygen uptake via reaction HMR_9034, glucose uptake, lactate secretion) with Winter hypoxia signature scores by TCGA sample identifier in R, retaining only the 676 tumors with non-missing hypoxia scores, to enable hypothesis-driven OLS regression of hypoxia-associated metabolic reprogramming.

## When to use

Use when TCGA pan-cancer sample-level flux features must be joined to an external per-sample signature table (e.g., Winter hypoxia scores) by TCGA sample barcode before association testing.

## When NOT to use

- Input is already a single merged analysis-ready table with no need for sample-identifier joining
- Analysis targets single-cell community fluxes (compute_sc_flux with cell-type fractions), not bulk per-sample TCGA association testing
- Samples lack the external signature scores (e.g., missing Winter hypoxia scores), since the inner-join would drop them and the regression cannot be fit

## Inputs

- METAFlux-derived TCGA pan-cancer flux feature table (oxygen uptake, glucose uptake via HMR_9034, lactate secretion exchange fluxes) keyed by TCGA sample identifier
- Winter hypoxia signature score table keyed by TCGA sample identifier (676 samples with non-missing scores)

## Outputs

- Merged per-sample table of flux features and hypoxia scores for 676 TCGA tumors
- regression_coefficients.csv with columns term, estimate, std.error, statistic, p.value, ci_lower, ci_upper

## How to apply

Load the METAFlux TCGA pan-cancer flux feature table for the 676 tumors — extracting oxygen uptake, glucose uptake (reaction HMR_9034), and lactate secretion exchange fluxes, looking up exchange Reaction IDs via the 'nutrient lookup file' (1648 exchange reactions) if needed. In R, merge these flux columns with the Winter hypoxia signature score table by TCGA sample identifier, inner-joining so only samples with non-missing hypoxia scores are kept. Fit the OLS model lm(Hypoxia_score ~ Oxygen_Uptake + Glucose_uptake + Lactate_secretion) with lm(), then extract coefficient estimates, standard errors, statistics, p-values, and 95% confidence intervals using summary() and confint(). Write the tidy coefficients table (term, estimate, std.error, statistic, p.value, ci_lower, ci_upper) to regression_coefficients.csv. The rationale is that hypoxia drives a shift from oxidative to glycolytic metabolism, so the merge directly tests whether METAFlux fluxes reproduce that reprogramming signature.

## Related tools

- **R** (Environment for the merge, lm() regression fitting, confint() CI extraction, and CSV export) — https://github.com/KChen-lab/METAFlux
- **METAFlux** (Source of the bulk-sample metabolic flux features (13,082 reaction flux scores per bulk sample) that are merged with hypoxia scores) — https://github.com/KChen-lab/METAFlux

## Examples

```
fit <- lm(Hypoxia_score ~ Oxygen_Uptake + Glucose_uptake + Lactate_secretion, data = merged_tcga); summary(fit); confint(fit)
```

## Evaluation signals

- Merged table contains exactly the 676 TCGA samples with non-missing Winter hypoxia scores
- Fitted OLS estimates reproduce the reported values: oxygen uptake −0.19, glucose uptake +0.32, lactate secretion +0.09
- confint() 95% confidence intervals match the paper's TCGA pan-cancer hypoxia association analysis
- regression_coefficients.csv has the required schema (term, estimate, std.error, statistic, p.value, ci_lower, ci_upper)
- Sign pattern is directionally correct: oxygen uptake negatively associated with hypoxia score, glucose uptake and lactate secretion positively associated

## Limitations

- Only 676 of the TCGA pan-cancer tumors have non-missing Winter hypoxia scores, so the regression is restricted to that subset after the merge
- No version numbers stated for METAFlux, R, or Seurat, so exact package versions for reproducibility are unverified
- Glucose uptake is tied specifically to reaction HMR_9034; using the wrong exchange Reaction ID would change the feature definition
- The fitted fluxes depend on the Human-GEM underlying model (13,082 metabolic reactions) and the chosen medium file (human_blood for human-derived samples)

## Evidence

- [abstract] Large-scale experiments using cell-lines, the cancer genome atlas (TCGA), and scRNA-seq data obtained from diverse cancer and immunotherapeutic contexts: "Large-scale experiments using cell-lines, the cancer genome atlas (TCGA), and scRNA-seq data obtained from diverse cancer and immunotherapeutic contexts"
- [intro] METAFlux can predict 13,082 reaction flux scores for each bulk sample: "METAFlux can predict 13,082 reaction flux scores for each bulk sample, and for each single cell data, it can predict (13,082 × number of cell-type/cluster + 1648) reaction flux scores."
- [methods] If one is strictly interested in metabolite uptake or release, one can search nutrient lookup file to obtain the metabolite exchange Reaction ID: "If one is strictly interested in metabolite uptake or release, one can search “nutrient lookup file” to obtain the metabolite exchange Reaction ID"
- [readme] METAFlux R package can be easily installed from Github using devtools: [evidence span withheld — anchor status: external_doc]
