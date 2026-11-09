---
name: spike-and-slab-bayesian-inference
description: Use when testing covariate effects (e.g., health status) on cell-type
  compositions from scRNA-seq count data with few experimental replicates, and you
  need explicit control of false discoveries via spike-and-slab effect shrinkage rather
  than univariate tests.
when_to_use_negative:
- You need to detect heterogeneities between samples or donor variability, since scCODA
  does not model response variability within a condition.
- Cell-type definitions are not yet established — scCODA requires pre-specified cell-type
  definitions that hinge on statistically sound and biologically meaningful clustering
  assignments.
- You need to model dependency structure among cell compositions beyond those induced
  by compositional effects, or the true covariate-abundance relationship is not log-linear.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scCODA
  role: provides the CompositionalAnalysis model class, spike-and-slab prior specification,
    sample_hmc()/sample_nuts() inference, and credible_effects() result extraction
  repo: https://github.com/theislab/scCODA
- name: TensorFlow
  role: backend used to build and evaluate the probabilistic model graph for HMC inference
- name: tensorflow-probability
  role: distribution and sampler primitives (Dirichlet-Multinomial likelihood, Logit-normal
    spike-and-slab prior, HMC) backing sample_hmc()
- name: scanpy
  role: source of annotated single-cell data transformed into the compositional count
    dataset via from_scanpy
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-27150-6
    title: scCODA is a Bayesian model for compositional single-cell data analysis
  source_content_hash: sha256:b4b450429cd216dceda497ce4e46b4d1525f8a0f1ac7ddf33e70e1d27ea329cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scCODA
  tools:
  - Python
  - Python (>=3.8)
  - TensorFlow
  - arviz
  - scCODA
  - scanpy
  - tensorflow-probability
  - tensorflow-probability (>=0.12)
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# spike-and-slab-bayesian-inference

## Summary

A Bayesian spike-and-slab modeling approach (as implemented in scCODA) that fits a hierarchical Dirichlet-Multinomial likelihood with a Logit-normal spike-and-slab prior to detect credible covariate effects on cell-type proportions from compositional scRNA-seq count data, using HMC inference in TensorFlow/tensorflow-probability.

## When to use

Use when testing covariate effects (e.g., health status) on cell-type compositions from scRNA-seq count data with few experimental replicates, and you need explicit control of false discoveries via spike-and-slab effect shrinkage rather than univariate tests.

## When NOT to use

- You need to detect heterogeneities between samples or donor variability, since scCODA does not model response variability within a condition.
- Cell-type definitions are not yet established — scCODA requires pre-specified cell-type definitions that hinge on statistically sound and biologically meaningful clustering assignments.
- You need to model dependency structure among cell compositions beyond those induced by compositional effects, or the true covariate-abundance relationship is not log-linear.

## Inputs

- cell-type count matrix (compositional analysis dataset built from a pandas DataFrame via from_pandas or from scanpy data via from_scanpy/from_scanpy_list/from_scanpy_dir)
- patsy formula string specifying covariates (e.g., health status)
- reference cell type specification (pre-specified or 'automatic')

## Outputs

- CAResult object with posterior samples of intercepts and effects
- effects data frame with posterior inclusion probabilities and a 'Final Parameter' column flagging credible effects
- credible-effects table at a chosen FDR level (e.g., 0.2)

## How to apply

Build the covariate matrix from a compositional anndata count object using patsy formula syntax, fixing a reference cell type whose effects are constrained to zero (or use automatic reference selection). Construct the hierarchical Dirichlet-Multinomial model with normal-prior intercepts N(0,5) and a Logit-normal spike-and-slab prior on effects (tau = exp(t)/(1+exp(t)) with t/50 ~ N(0,1); slab variance sigma_m^2 ~ Half-Cauchy(0,1)) in TensorFlow/tensorflow-probability. Run Hamiltonian Monte Carlo via sample_hmc() (or No-U-Turn sampling via sample_nuts()) on the CompositionalAnalysis object, then compute posterior inclusion probabilities from the spike-and-slab indicators. Determine FDR thresholds by direct posterior probability and call credible effects, interpreting all effects relative to the selected reference cell type.

## Related tools

- **scCODA** (provides the CompositionalAnalysis model class, spike-and-slab prior specification, sample_hmc()/sample_nuts() inference, and credible_effects() result extraction) — https://github.com/theislab/scCODA
- **TensorFlow** (backend used to build and evaluate the probabilistic model graph for HMC inference)
- **tensorflow-probability** (distribution and sampler primitives (Dirichlet-Multinomial likelihood, Logit-normal spike-and-slab prior, HMC) backing sample_hmc())
- **scanpy** (source of annotated single-cell data transformed into the compositional count dataset via from_scanpy)

## Examples

```
model = mod.CompositionalAnalysis(data, formula="health_status", reference_cell_type="automatic"); results = model.sample_hmc(); results.credible_effects()
```

## Evaluation signals

- The 'Final Parameter' column of the effects data frame is populated, with 0 flagging effects not statistically credible and nonzero values flagging credible effects.
- Effects of the reference cell type are constrained to zero, and all credible effects are interpreted relative to that reference.
- Posterior inclusion probabilities are computed for each cell-type-by-covariate effect from the spike-and-slab indicators.
- On the UC colon reanalysis (SCP259), the credible-effects table at FDR 0.2 reproduces the significant Treg increase in lamina propria and the inflammatory fibroblast (IAF) increase, matching Supplementary Data 3.
- HMC inference completes without error in the tensorflow (>=2.4)/tensorflow-probability (>=0.12) environment with Python >=3.8.

## Limitations

- scCODA assumes a log-linear relationship between covariates and cell abundance, which may be mis-specified in some cases.
- scCODA does not model or infer any dependency structure among cell compositions beyond those induced by compositional effects; hyperpriors for additional dependencies would considerably increase computational complexity.
- scCODA does not model response variability within a condition and cannot detect heterogeneities between samples in response to treatment or donor variability.
- GPU computation features of tensorflow/tensorflow-probability have not been tested with scCODA and are not recommended (per README).
- The scCODA implementation is no longer maintained; a new Jax version is available in pertpy (per README).

## Evidence

- [intro] The model uses a Logit-normal spike-and-slab prior10 with a log-link function to estimate effects of binary (or continuous) covariates on cell-type proportions: "The model uses a Logit-normal spike-and-slab prior10 with a log-link function to estimate effects of binary (or continuous) covariates on cell-type proportions"
- [methods] Once the model is set up, inference via HMC sampling can be performed via ``sample_hmc()``.: "Once the model is set up, inference via HMC sampling can be performed via ``sample_hmc()``."
- [methods] Using the scCODA model is easiest by generating an instance of ``ana.CompositionalAnalysis``.: "Using the scCODA model is easiest by generating an instance of ``ana.CompositionalAnalysis``."
- [readme] This package uses the `tensorflow` (`>=2.8`) and `tensorflow-probability` (`>=0.16`) packages.: [evidence span withheld — anchor status: external_doc]
