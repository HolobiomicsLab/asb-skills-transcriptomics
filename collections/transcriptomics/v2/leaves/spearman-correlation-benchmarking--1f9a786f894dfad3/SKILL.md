---
name: spearman-correlation-benchmarking
description: Use when evaluating how well predicted reaction/metabolite fluxes (e.g.,
  from METAFlux compute_flux on NCI-60 RNA-seq) agree with matched experimental flux
  measurements, via rank-based Spearman correlation.
when_to_use_negative:
- Ground-truth flux measurements are not available or cannot be matched sample-wise
  to the predictions, making rank correlation undefined
- The task is flux prediction or model construction itself rather than benchmarking
  accuracy against experimental data
- Only a handful of paired values exist per stratum, since Spearman correlation on
  very few metabolites or cell lines is unstable
edam_operation: http://edamontology.org/operation_0496
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2259
tools:
- name: METAFlux
  role: Generates the predicted flux scores (compute_flux over 13,082 Human-GEM reactions)
    that are benchmarked via Spearman correlation against NCI-60 experimental fluxes
  repo: https://github.com/KChen-lab/METAFlux
- name: R
  role: Environment in which METAFlux is loaded (library(METAFlux)) and Spearman correlations
    are computed and tabulated
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-40457-w
    title: Characterizing cancer metabolism from bulk and single-cell RNA-seq data
      using METAFlux
  source_content_hash: sha256:a2d0155568e7902b4394fcc8c0bd65520c8ce4b35b2a70ae539df89f30ad1408
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/METAFlux
  tools:
  - METAFlux
  - R
  edam_operation: http://edamontology.org/operation_0496
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2259
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# spearman-correlation-benchmarking

## Summary

Benchmark predicted metabolic fluxes against experimentally measured fluxes by computing Spearman rank correlations, as used to validate METAFlux on the NCI-60 cell lines (ρ=0.74) and to quantify degradation when medium constraints are removed (ρ=0.15).

## When to use

Use when evaluating how well predicted reaction/metabolite fluxes (e.g., from METAFlux compute_flux on NCI-60 RNA-seq) agree with matched experimental flux measurements, via rank-based Spearman correlation.

## When NOT to use

- Ground-truth flux measurements are not available or cannot be matched sample-wise to the predictions, making rank correlation undefined
- The task is flux prediction or model construction itself rather than benchmarking accuracy against experimental data
- Only a handful of paired values exist per stratum, since Spearman correlation on very few metabolites or cell lines is unstable

## Inputs

- Predicted flux matrix (samples x 13,082 reactions) from METAFlux compute_flux
- Experimentally measured NCI-60 flux data matched to the 11 evaluated cell lines
- Nutrient lookup files (1,648 exchange reactions) and human_gem annotation for mapping reaction IDs (e.g., HMR_9034 for glucose uptake)
- Optional: ecGEMs baseline flux predictions or a no-medium-constraint control flux matrix

## Outputs

- Overall Spearman rho between predicted and experimental fluxes (e.g., 0.74 for METAFlux vs 0.45 for ecGEMs)
- Per-metabolite Spearman correlations across cell lines
- Per-cell-line Spearman correlations across metabolites
- Comparison table written to nci60_flux_correlations.csv

## How to apply

Predict fluxes for all 13,082 Human-GEM reactions per sample with compute_flux(mras=scores, medium=cell_medium), optionally applying cubic-root normalization via cbrt (sign(x)*abs(x)^(1/3)). Extract predicted exchange fluxes for the metabolites of interest plus the biomass reaction (e.g., glucose uptake via reaction HMR_9034) using the nutrient lookup files and human_gem annotation. Compute Spearman rank correlations between predicted and experimentally measured fluxes across the 11 NCI-60 cell lines, both overall and stratified per metabolite and per cell line, and compare against baselines such as ecGEMs (ρ=0.45) or a no-medium-constraint control (ρ=0.15). Write the correlation table to nci60_flux_correlations.csv and validate by checking that METAFlux correlations exceed the baseline (0.74 vs 0.45), consistent with the paper's reported substantial improvement.

## Related tools

- **METAFlux** (Generates the predicted flux scores (compute_flux over 13,082 Human-GEM reactions) that are benchmarked via Spearman correlation against NCI-60 experimental fluxes) — https://github.com/KChen-lab/METAFlux
- **R** (Environment in which METAFlux is loaded (library(METAFlux)) and Spearman correlations are computed and tabulated)

## Examples

```
library(METAFlux); scores <- calculate_reaction_score(bulk_test_example); flux <- compute_flux(mras=scores, medium=cell_medium); cor.test(as.numeric(flux["HMR_9034",]), measured_glucose_uptake, method="spearman")
```

## Evaluation signals

- Reproduced overall Spearman correlation on NCI-60 matches the reported ρ=0.74 for medium-constrained METAFlux
- No-medium-constraint control reproduces the reported overall ρ=0.15, confirming sensitivity to medium constraints
- METAFlux Spearman correlations exceed the ecGEMs baseline (0.74 vs 0.45) in every one of the 11 evaluated NCI-60 cell lines
- nci60_flux_correlations.csv is produced containing overall, per-metabolite, and per-cell-line correlations
- Predicted exchange fluxes for the 26 metabolites plus the biomass reaction are correctly mapped via nutrient lookup files and human_gem annotation

## Limitations

- Predicted glucose uptake does not correlate as consistently as other metabolites in the NCI-60 panel
- The no-medium-constraint condition degrades performance dramatically (overall ρ drops to 0.15), so medium choice (cell_medium, 44 metabolites) strongly drives accuracy
- No software version number for METAFlux is stated, complicating exact reproduction

## Evidence

- [intro] Evaluation plan: assess METAFlux prediction accuracy using matched flux data from NCI-60 cell lines: "We evaluate METAFlux prediction accuracy using the matched flux data generated on the NCI-60 cell lines and find a substantial improvement over existing approaches."
- [methods] Flux computation step grounded in methods: "Subsequently, we calculate the metabolic fluxes for the 13082 reactions"
- [discussion] Glucose uptake correlation caveat: "the predicted glucose uptake does not correlate as consistently as other metabolites in NCI-60 panel"
- [methods] Underlying metabolic model scale: "We used Human-GEM[1] (consisting of 13082 metabolic reactions and 8378 metabolites)as our underlying metabolic model"
- [readme] Installation path for reproduction: [evidence span withheld — anchor status: external_doc]
