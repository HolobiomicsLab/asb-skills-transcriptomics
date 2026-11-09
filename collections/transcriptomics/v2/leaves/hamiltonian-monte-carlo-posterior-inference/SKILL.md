---
name: hamiltonian-monte-carlo-posterior-inference
description: Use when estimating covariate effects on cell-type proportions from compositional
  scRNA-seq count data with scCODA, where HMC (or NUTS) posterior sampling is needed
  to obtain credible effects relative to a reference cell type.
when_to_use_negative:
- When the analysis goal is detecting sample-to-sample response heterogeneity or donor
  variability — scCODA does not model response variability within a condition
- When the covariate-to-abundance relationship is suspected to be non-log-linear —
  scCODA assumes a log-linear relationship which may be mis-specified
- When modeling dependency structure among cell compositions beyond compositional
  effects is required
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3308
tools:
- name: scCODA
  role: Framework providing CompositionalAnalysis model setup, sample_hmc()/sample_nuts()
    inference, and result summary/credible_effects output
  repo: https://github.com/theislab/scCODA
- name: TensorFlow
  role: Backend (>=2.4, README says >=2.8) used to implement the probabilistic model
    and run HMC sampling
- name: tensorflow-probability
  role: Library (>=0.12, README says >=0.16) supplying the Dirichlet-Multinomial likelihood
    and spike-and-slab prior distributions used during sampling
- name: patsy
  role: Formula syntax used to build the covariate matrix for the CompositionalAnalysis
    model
- name: arviz
  role: Used for posterior diagnostics/summary of HMC samples
- name: scanpy
  role: Source of cell count data via from_scanpy / from_scanpy_list / from_scanpy_dir
    conversion into a compositional analysis dataset
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-27150-6
    title: scCODA is a Bayesian model for compositional single-cell data analysis
  source_content_hash: sha256:7611e51d219f2587e968f44bd35d1180b8feb2fad58fb6d5d72eaf10eda3bcbd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scCODA
  tools:
  - Python
  - TensorFlow
  - arviz
  - patsy
  - scCODA
  - scanpy
  - tensorflow-probability
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# hamiltonian-monte-carlo-posterior-inference

## Summary

Perform Bayesian posterior inference for the scCODA Dirichlet-Multinomial compositional model via Hamiltonian Monte Carlo sampling in TensorFlow/tensorflow-probability, drawing posterior samples of intercepts and spike-and-slab effects for cell-type-by-covariate differential abundance.

## When to use

Use when estimating covariate effects on cell-type proportions from compositional scRNA-seq count data with scCODA, where HMC (or NUTS) posterior sampling is needed to obtain credible effects relative to a reference cell type.

## When NOT to use

- When the analysis goal is detecting sample-to-sample response heterogeneity or donor variability — scCODA does not model response variability within a condition
- When the covariate-to-abundance relationship is suspected to be non-log-linear — scCODA assumes a log-linear relationship which may be mis-specified
- When modeling dependency structure among cell compositions beyond compositional effects is required

## Inputs

- cell count data from single-cell expression data (scanpy object or pandas DataFrame) as a compositional analysis dataset
- patsy formula string specifying covariates and a reference cell type
- pre-specified cell-type definitions / clustering assignments

## Outputs

- posterior samples of intercepts and covariate effects from HMC sampling
- effects data frame with Final Parameter column marking significant (credible) effects
- credible_effects.csv export of significant cell-type-by-covariate effects

## How to apply

After building the covariate matrix from the compositional anndata object via patsy formula syntax with a fixed reference cell type (effects constrained to zero), construct the hierarchical Dirichlet-Multinomial model in TensorFlow/tensorflow-probability: normal-prior intercepts N(0,5), a Logit-normal spike-and-slab prior on effects (tau via exp(t)/(1+exp(t)) with t/50 ~ N(0,1), slab variance sigma_m^2 ~ Half-Cauchy(0,1)). Run inference by calling sample_hmc() on the CompositionalAnalysis result object (sample_nuts() with No-U-Turn sampling is an alternative) to draw posterior samples of intercepts and effects. From the posterior draws, compute posterior inclusion probabilities for each cell-type-by-covariate effect from the spike-and-slab indicators and apply direct-posterior-probability FDR threshold determination to call credible effects. Interpret significant effects relative to the selected reference cell type, since credible changes must be interpreted relative to that reference.

## Related tools

- **scCODA** (Framework providing CompositionalAnalysis model setup, sample_hmc()/sample_nuts() inference, and result summary/credible_effects output) — https://github.com/theislab/scCODA
- **TensorFlow** (Backend (>=2.4, README says >=2.8) used to implement the probabilistic model and run HMC sampling)
- **tensorflow-probability** (Library (>=0.12, README says >=0.16) supplying the Dirichlet-Multinomial likelihood and spike-and-slab prior distributions used during sampling)
- **patsy** (Formula syntax used to build the covariate matrix for the CompositionalAnalysis model)
- **arviz** (Used for posterior diagnostics/summary of HMC samples)
- **scanpy** (Source of cell count data via from_scanpy / from_scanpy_list / from_scanpy_dir conversion into a compositional analysis dataset)

## Examples

```
import sccoda; res = sccoda.modeling.dirichlet_models.scCODA_model(sccoda_data, formula='x_cov', reference_cell_type='GABAergic'); res.sample_hmc(); res.summary(); res.credible_effects()
```

## Evaluation signals

- The Final Parameter column of the effects data frame must be populated, with 0 flagging non-credible effects and nonzero values flagging credible effects
- Effects for the reference cell type are constrained to zero; no credible effect should be reported for the reference
-  credible_effects.csv is written with significant cell-type-by-covariate entries
- Calling summary() on the result object lists which effects were found to be significant
- Posterior inclusion probabilities derived from spike-and-slab indicators must be consistent with the FDR-thresholded credible effect calls

## Limitations

- scCODA requires pre-specified cell-type definitions, which hinge on statistically sound and biologically meaningful clustering assignments
- scCODA assumes a log-linear relationship between covariates and cell abundance, which may be mis-specified in some cases
- scCODA does not model or infer any dependency structure among the cell compositions beyond those induced by compositional effects
- scCODA does not model response variability within a condition and cannot detect heterogeneities between samples in response to treatment or donor variability
- The TensorFlow-based scCODA implementation is no longer maintained; a Jax version is available in pertpy (per README)

## Evidence

- [methods] HMC inference method: "Once the model is set up, inference via HMC sampling can be performed via ``sample_hmc()``."
- [methods] NUTS alternative: "Alternatively, No-U-Turn sampling is available via ``sample_nuts()``."
- [intro] Model prior structure: "The model uses a Logit-normal spike-and-slab prior10 with a log-link function to estimate effects of binary (or continuous) covariates on cell-type proportions"
- [methods] Result significance output: "The ``Final Parameter`` column of the effects data frame shows the significances."
- [readme] Backend requirements: [evidence span withheld — anchor status: external_doc]
- [intro] Reference interpretation: "scCODA can automatically select an appropriate cell type as the reference (Methods—“Automatic reference selection”) or uses a pre-specified reference cell type11"
