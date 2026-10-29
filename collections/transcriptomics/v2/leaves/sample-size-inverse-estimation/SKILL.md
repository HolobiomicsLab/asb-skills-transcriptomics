---
name: sample-size-inverse-estimation
description: Use when planning scCODA differential abundance experiments and you must
  know how many biological replicates are needed to detect a compositional shift of
  a given log-fold change, especially for rare vs abundant cell types.
when_to_use_negative:
- The experiment has many replicates per condition and no sample-size planning question
  — scCODA itself is already designed to handle the low-replicate regime, and no inversion
  is needed.
- The goal is detecting heterogeneity in response between individual samples or donors
  — scCODA does not model response variability within a condition, so sample-size
  curves for the joint model will not capture that.
- Effect of a covariate is expected to be non-log-linear with cell abundance — scCODA
  assumes a log-linear relationship, so the power estimates would be mis-specified.
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_2269
tools:
- name: scCODA
  role: Bayesian Dirichlet-Multinomial compositional model used to fit synthetic case-control
    count data and call credible effects at FDR 0.05/0.1/0.2
  repo: https://github.com/theislab/scCODA
- name: TensorFlow
  role: Backend for HMC sampling inference (sample_hmc()) on the scCODA model
- name: tensorflow-probability
  role: Probabilistic programming backend used by sample_hmc() for Bayesian inference
- name: Scanpy
  role: Ecosystem integration that scCODA is fully embedded in; optional source of
    real cell count data for extending the power analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-27150-6
    title: scCODA is a Bayesian model for compositional single-cell data analysis
  source_content_hash: sha256:9b044d939742ece93a5123a24344cb07c567463a32d954e8460743bbb8e401d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scCODA
  tools:
  - Python
  - Scanpy
  - TensorFlow
  - scCODA
  - tensorflow-probability
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# sample-size-inverse-estimation

## Summary

Estimates the number of case-control samples scCODA needs to detect a compositional change of a given log2 effect size, by fitting power curves on synthetic Dirichlet-multinomial data and inverting a quasibinomial GLM to solve for the sample size reaching a power of 0.8.

## When to use

Use when planning scCODA differential abundance experiments and you must know how many biological replicates are needed to detect a compositional shift of a given log-fold change, especially for rare vs abundant cell types.

## When NOT to use

- The experiment has many replicates per condition and no sample-size planning question — scCODA itself is already designed to handle the low-replicate regime, and no inversion is needed.
- The goal is detecting heterogeneity in response between individual samples or donors — scCODA does not model response variability within a condition, so sample-size curves for the joint model will not capture that.
- Effect of a covariate is expected to be non-log-linear with cell abundance — scCODA assumes a log-linear relationship, so the power estimates would be mis-specified.

## Inputs

- Synthetic case-control cell-type count datasets from sccoda.util.data_generation.generate_case_control (K=5 cell types, ȳ=5000 mean counts, r=10 replicates, grid over baseline abundance and log fold-change)
- Grid of FDR levels (0.05, 0.1, 0.2) for credible_effects() calls
- Aggregated detection results (true positive rate and FDR per sample-size and effect-size setting)

## Outputs

- power_analysis_results.csv with inverted sample-size estimates for power 0.8 per effect-size and FDR level
- Power curves (true positive rate vs sample size per effect-size setting)
- GLM fit-quality diagnostics (R² ≈ 0.937)

## How to apply

Generate synthetic case-control count data with sccoda.util.data_generation.generate_case_control over a grid of baseline abundances (e.g., 10 values) and effect fold-changes (e.g., 15 values), with K=5 cell types, mean counts ȳ=5000, and r=10 replicates per setting. Fit the scCODA model via sccoda.util.comp_ana.CompositionalAnalysis with HMC sampling (sample_hmc() on the TensorFlow/tensorflow-probability backend) for each synthetic dataset, then call credible_effects() at FDR levels 0.05, 0.1, and 0.2 to compute true positive rates and false discovery rates per sample-size and effect-size setting. Fit a quasibinomial GLM of true positive rate on sample size to the aggregated detection results, and invert it to find the sample size where power reaches 0.8 for each effect-size and FDR combination. Check fit quality (expect R² ≈ 0.937) and write sample-size estimates and power curves to a results table such as power_analysis_results.csv. Reference results: a log2 change of 1 in abundant types (e.g., 1000/5000 cells) is detectable with 5 samples, while the same change in rare types (125/5000 cells) needs 20–30 samples at FDR 0.2; large log2 fold-changes of 4 in rare types need fewer than 10 samples.

## Related tools

- **scCODA** (Bayesian Dirichlet-Multinomial compositional model used to fit synthetic case-control count data and call credible effects at FDR 0.05/0.1/0.2) — https://github.com/theislab/scCODA
- **TensorFlow** (Backend for HMC sampling inference (sample_hmc()) on the scCODA model)
- **tensorflow-probability** (Probabilistic programming backend used by sample_hmc() for Bayesian inference)
- **Scanpy** (Ecosystem integration that scCODA is fully embedded in; optional source of real cell count data for extending the power analysis)

## Examples

```
from sccoda.util.comp_ana import CompositionalAnalysis; res = CompositionalAnalysis(dat, formula="condition", reference_cell_type="automatic").sample_hmc(); sig = res.credible_effects(fdr_threshold=0.2)
```

## Evaluation signals

- The quasibinomial GLM of true positive rate on sample size fits with R² ≈ 0.937; substantially lower R² indicates the aggregated detection grid is too sparse or noisy for inversion.
- Inverted sample sizes reproduce reference points: power 0.8 at FDR 0.2 requires ~5 samples for a log2 change of 1 in an abundant type (1000/5000 cells), 20–30 samples for the same change in a rare type (125/5000 cells), and <10 samples for a log2 change of 4 in a rare type.
- Observed false discovery rate on synthetic datasets stays controlled at each FDR level (0.05, 0.1, 0.2) when credible_effects() is called on each CAResult.
- power_analysis_results.csv contains a sample-size estimate for every combination of effect size, baseline abundance, and FDR level in the generation grid (10 baseline values × 15 fold-change values).
- Each synthetic dataset converges under HMC sampling (sample_hmc()) without divergences or failure of the CompositionalAnalysis fit.

## Limitations

- scCODA requires pre-specified cell-type definitions that hinge on statistically sound and biologically meaningful clustering assignments; power estimates inherit any clustering error.
- scCODA assumes a log-linear relationship between covariates and cell abundance, which may be mis-specified, biasing power curves for non-log-linear effects.
- scCODA does not model response variability within a condition and cannot detect heterogeneities between samples in response to treatment or donor variability, so the power estimates reflect only the joint-composition effect.
- GPU computation features of TensorFlow/tensorflow-probability have not been tested with scCODA and are not recommended.
- The TensorFlow-based scCODA implementation is no longer maintained; a new Jax version is available in pertpy, which may require re-validating the power curves.

## Evidence

- [methods] Generating synthetic case-control datasets over grids of baseline abundance and effect fold-change, then fitting scCODA and inverting a quasibinomial GLM of true positive rate on sample size: "synthetically generating cell count data (``util.data_generation``)"
- [methods] Fitting the scCODA model and performing Bayesian inference: "Once the model is set up, inference via HMC sampling can be performed via ``sample_hmc()``."
- [intro] The scCODA model uses a Logit-normal spike-and-slab prior with a log-link function over a hierarchical Dirichlet-Multinomial likelihood: "The model uses a Logit-normal spike-and-slab prior10 with a log-link function to estimate effects of binary (or continuous) covariates on cell-type proportions"
- [methods] This package uses the tensorflow (>= 2.4) and tensorflow-probability (>=0.12) packages.: "This package uses the tensorflow (>= 2.4) and tensorflow-probability (>=0.12) packages."
- [methods] A functioning python environment (>=3.8) is necessary to run this package.: "A functioning python environment (>=3.8) is necessary to run this package."
- [discussion] scCODA compared favorably to commonly used models, particularly when only a low number of experimental replicates are available: "scCODA compared favorably to commonly used models for single-cell and microbiome compositional analysis, particularly when only a low number of experimental replicates are available."
- [readme] Note: The benchmarks are intended to be executed with scCODA>=0.1.2: [evidence span withheld — anchor status: external_doc]
- [discussion] scCODA does not model response variability within a condition and thus cannot detect heterogeneities between samples in response to treatment or donor variability: "scCODA does not model the response variability within a condition and thus cannot detect heterogeneities between samples in response to treatment or donor variability"
- [discussion] scCODA assumes a log-linear relationship between covariates and cell abundance, which may be mis-specified in some cases: "scCODA assumes a log-linear relationship between covariates and cell abundance, which may be mis-specified in some cases"
