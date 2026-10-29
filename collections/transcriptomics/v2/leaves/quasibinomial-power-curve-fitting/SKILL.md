---
name: quasibinomial-power-curve-fitting
description: 'Use when planning scCODA case-control scRNA-seq experiments: compositional
  count data with known ground-truth changes have been simulated per (baseline abundance
  × fold-change × sample size), and per-setting detection rates are available.'
when_to_use_negative:
- The input is a real experimental dataset without replicated, ground-truth compositional
  changes — power estimation requires simulated data with known effects (scCODA reproducibility
  benchmarks supply such data via zenodo).
- The goal is differential abundance testing of an actual dataset rather than experimental
  design/sample-size planning — use CompositionalAnalysis with credible_effects directly
  instead.
- Sample-level response heterogeneity matters — scCODA does not model response variability
  within a condition, so power curves from it will not capture donor variability.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: scCODA
  role: Generates synthetic case-control count data and supplies the CompositionalAnalysis
    model, sample_hmc() HMC inference and credible_effects() calls whose true positive
    rates feed the quasibinomial power model
  repo: https://github.com/theislab/scCODA
- name: tensorflow-probability
  role: Backend for the HMC sampling used to fit scCODA models on each simulated dataset
- name: scCODA_reproducibility
  role: Contains generate_data.py and the benchmark scripts/notebooks that execute
    the simulated-data power and threshold-determination benchmarks
  repo: https://github.com/theislab/scCODA_reproducibility
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-27150-6
    title: scCODA is a Bayesian model for compositional single-cell data analysis
  source_content_hash: sha256:0ca274db5084521c9840e0003a7a564aeca48060f0c8ca989c27cae9699e46bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scCODA
  tools:
  - Python
  - TensorFlow
  - scCODA
  - scCODA_reproducibility
  - tensorflow-probability
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# quasibinomial-power-curve-fitting

## Summary

Fits a quasibinomial generalized linear model of true positive rate on sample size to scCODA detection results over synthetic case-control grids, then inverts the fit to estimate the sample size needed for power 0.8 at a given effect size and FDR level.

## When to use

Use when planning scCODA case-control scRNA-seq experiments: compositional count data with known ground-truth changes have been simulated per (baseline abundance × fold-change × sample size), and per-setting detection rates are available.

## When NOT to use

- The input is a real experimental dataset without replicated, ground-truth compositional changes — power estimation requires simulated data with known effects (scCODA reproducibility benchmarks supply such data via zenodo).
- The goal is differential abundance testing of an actual dataset rather than experimental design/sample-size planning — use CompositionalAnalysis with credible_effects directly instead.
- Sample-level response heterogeneity matters — scCODA does not model response variability within a condition, so power curves from it will not capture donor variability.

## Inputs

- Synthetic case-control compositional count datasets from sccoda.util.data_generation.generate_case_control (10 baseline abundance values × 15 effect fold-change values, K=5 cell types, ȳ=5000 mean counts, r=10 replicates per setting)
- Per-setting detection outcomes (credible effects calls) from scCODA CAResult objects at FDR levels 0.05, 0.1 and 0.2

## Outputs

- Fitted quasibinomial GLM of true positive rate on sample size (fit quality R² ≈ 0.937)
- Power curves per effect-size and FDR-level combination
- Sample-size estimates required for power 0.8 (e.g., 5 samples for log2 change 1 in abundant types; 20–30 samples for the same change in rare types; <10 samples for log-fold changes of 4 in rare types)
- power_analysis_results.csv containing sample-size estimates, power curves and R² check

## How to apply

Generate synthetic compositional count datasets with sccoda.util.data_generation.generate_case_control over the grid of 10 baseline abundance values and 15 effect fold-change values (K=5 cell types, ȳ=5000 mean counts, r=10 replicates). Fit each dataset with sccoda.util.comp_ana.CompositionalAnalysis and run Bayesian inference via sample_hmc() on the TensorFlow/tensorflow-probability backend, then call credible_effects() at FDR levels 0.05, 0.1 and 0.2 to compute true positive and false discovery rates per sample-size and effect-size setting. Aggregate the detection results and fit a quasibinomial GLM of true positive rate on sample size; the quasibinomial family is appropriate because detection indicators are binomial-like with overdispersion across simulation settings. Invert the fitted model to solve for the sample size achieving power 0.8 for each effect-size/FDR-level combination, and verify fit quality (expected R² ≈ 0.937) before writing estimates, power curves and the R² check to power_analysis_results.csv.

## Related tools

- **scCODA** (Generates synthetic case-control count data and supplies the CompositionalAnalysis model, sample_hmc() HMC inference and credible_effects() calls whose true positive rates feed the quasibinomial power model) — https://github.com/theislab/scCODA
- **tensorflow-probability** (Backend for the HMC sampling used to fit scCODA models on each simulated dataset)
- **scCODA_reproducibility** (Contains generate_data.py and the benchmark scripts/notebooks that execute the simulated-data power and threshold-determination benchmarks) — https://github.com/theislab/scCODA_reproducibility

## Examples

```
mod = sccoda.util.comp_ana.CompositionalAnalysis(counts, formula="condition", reference_cell_type="automatic"); res = mod.sample_hmc(); det = res.credible_effects(credible_interval_lev=0.8)
```

## Evaluation signals

- Quasibinomial GLM fit achieves R² ≈ 0.937 for true positive rate regressed on sample size
- Inverted estimates reproduce the reported benchmarks: 5 samples for log2 change 1 in abundant cell types (e.g., 1000 of 5000 cells) at FDR 0.2, and 20–30 samples for the same change in rare types (e.g., 125 of 5000 cells)
- Large relative changes (log-fold change 4) in rare cell types require fewer than ten samples
- All estimates, power curves and the R² check are present in power_analysis_results.csv
- FDR at the evaluated settings is controlled at the nominal levels (0.05, 0.1, 0.2) used in the credible_effects() calls

## Limitations

- Power estimates depend on the simulation grid (K=5 cell types, ȳ=5000 mean counts, r=10 replicates); other abundance regimes or cell-type counts may shift the required sample sizes.
- scCODA assumes a log-linear relationship between covariates and cell abundance, which may be mis-specified, biasing extrapolated power curves.
- scCODA requires pre-specified cell-type definitions, and does not model response variability within a condition, so heterogeneities between samples are outside the scope of the power model.
- The benchmarks are resource intensive; precomputed data and results are distributed via zenodo (https://doi.org/10.5281/zenodo.6473255) rather than rerun.

## Evidence

- [methods] Synthetic data generation grounds the power simulation: "or synthetically generating cell count data (``util.data_generation``)"
- [methods] Inference step producing per-setting detection results: "inference via HMC sampling can be performed via ``sample_hmc()``."
- [methods] Significance calls used to compute true positive and false discovery rates: "The ``Final Parameter`` column of the effects data frame shows the significances."
- [readme] Reproducibility benchmark structure for simulated-data power analysis: [evidence span withheld — anchor status: external_doc]
- [readme] Simulated benchmark resources are distributed rather than rerun: [evidence span withheld — anchor status: external_doc]
