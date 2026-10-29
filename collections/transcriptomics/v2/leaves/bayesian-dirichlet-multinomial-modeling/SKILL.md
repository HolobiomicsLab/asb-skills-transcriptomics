---
name: bayesian-dirichlet-multinomial-modeling
description: Use when per-sample cell-type counts from scRNA-seq (or similar compositional
  count data) must be tested for differential abundance across conditions, especially
  with few replicates.
when_to_use_negative:
- The goal is to detect heterogeneities between donors/samples in response to treatment
  — scCODA 'does not model the response variability within a condition'
- Cell-type clustering is unreliable — scCODA requires pre-specified cell-type definitions
  which hinge on statistically sound and biologically meaningful clustering assignments
- The relationship between covariates and abundance is expected to be strongly non-log-linear
  — scCODA assumes a log-linear relationship which may be mis-specified
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: scCODA
  role: Main toolbox providing CompositionalAnalysis, sample_hmc(), summary(), credible_effects(),
    set_fdr()
  repo: https://github.com/theislab/scCODA
- name: TensorFlow (>= 2.4)
  role: Backend for fitting the Dirichlet-Multinomial model via HMC sampling
- name: tensorflow-probability (>=0.12)
  role: Probabilistic primitives for the spike-and-slab prior and HMC inference
- name: scanpy
  role: Source of cell-annotated single-cell data imported via from_scanpy into the
    compositional dataset
  repo: https://scanpy.readthedocs.io
- name: patsy
  role: Formula syntax encoding disease covariates for model design matrix construction
provenance:
  source_task_ids:
  - task_007
  - task_001
  - task_002
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-27150-6
    title: scCODA is a Bayesian model for compositional single-cell data analysis
  source_content_hash: sha256:3067406051c068db45df9c65c8a1a49886c3dfb34c93001360b07f42195801be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scCODA
  tools:
  - ALDEx2
  - ANCOM
  - ANCOM-BC
  - Python
  - Python (>=3.8)
  - TensorFlow
  - TensorFlow (>= 2.4)
  - arviz
  - pandas
  - patsy
  - scCODA
  - scanpy
  - tensorflow-probability
  - tensorflow-probability (>=0.12)
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# bayesian-dirichlet-multinomial-modeling

## Summary

Build and fit scCODA's hierarchical Dirichlet-Multinomial model with a Logit-normal spike-and-slab prior on cell-type effects, inferred via Hamiltonian Monte Carlo, to detect credible compositional changes in single-cell data relative to a fixed reference cell type.

## When to use

Use when per-sample cell-type counts from scRNA-seq (or similar compositional count data) must be tested for differential abundance across conditions, especially with few replicates.

## When NOT to use

- The goal is to detect heterogeneities between donors/samples in response to treatment — scCODA 'does not model the response variability within a condition'
- Cell-type clustering is unreliable — scCODA requires pre-specified cell-type definitions which hinge on statistically sound and biologically meaningful clustering assignments
- The relationship between covariates and abundance is expected to be strongly non-log-linear — scCODA assumes a log-linear relationship which may be mis-specified

## Inputs

- Per-sample cell-type count matrix aggregated from cell-annotated scRNA-seq data (e.g., via sccoda.util.cell_composition_data.from_scanpy/from_pandas)
- Covariate metadata per sample (e.g., disease condition: healthy/moderate/severe; tissue region: cortex/cerebellum)
- Patsy formula string and a pre-specified reference cell type (e.g., Plasma cells, microglia 1)

## Outputs

- CAResult object with posterior samples of intercepts and effects from HMC
- Effects data frame with 'Final Parameter' column flagging credible effects at a target FDR (e.g., 0.2)
- credible_effects / summary tables (e.g., covid_balf_sccoda_credible_effects.csv, credible_effects_summary.csv)

## How to apply

Import aggregated per-sample cell-type counts and covariates into scCODA's compositional anndata structure via from_pandas or from_scanpy, then instantiate ana.CompositionalAnalysis with a patsy formula and either a pre-specified or automatically selected reference cell type whose effects are constrained to zero. Fit the hierarchical Dirichlet-Multinomial likelihood with normal-prior intercepts N(0,5) and a Logit-normal spike-and-slab prior on effects in TensorFlow/tensorflow-probability (e.g., tau via exp(t)/(1+exp(t)) with t/50 ~ N(0,1), slab variance sigma_m^2 ~ Half-Cauchy(0,1)). Run Bayesian inference via sample_hmc() (or No-U-Turn sampling via sample_nuts(), up to ~5 minutes per dataset). From the CAResult, compute posterior inclusion probabilities from spike-and-slab indicators and apply direct-posterior-probability FDR threshold determination, e.g., set_fdr(target_fdr_level=0.2), then call summary() and credible_effects(). All credible changes must be interpreted relative to the chosen reference cell type, and the 'Final Parameter' column of the effects data frame flags significance (0 = not credible, nonzero = credible).

## Related tools

- **scCODA** (Main toolbox providing CompositionalAnalysis, sample_hmc(), summary(), credible_effects(), set_fdr()) — https://github.com/theislab/scCODA
- **TensorFlow (>= 2.4)** (Backend for fitting the Dirichlet-Multinomial model via HMC sampling)
- **tensorflow-probability (>=0.12)** (Probabilistic primitives for the spike-and-slab prior and HMC inference)
- **scanpy** (Source of cell-annotated single-cell data imported via from_scanpy into the compositional dataset) — https://scanpy.readthedocs.io
- **patsy** (Formula syntax encoding disease covariates for model design matrix construction)

## Examples

```
import sccoda; mod = sccoda.model.other_models is not needed; use mod = sccoda.util.comp_ana.CompositionalAnalysis(data, formula="Health", reference_cell_type="Plasma cells"); res = mod.sample_hmc(); res.summary(); res.credible_effects()
```

## Evaluation signals

- The 'Final Parameter' column of the effects data frame is populated, with 0 flagging non-credible and nonzero values flagging credible effects consistent with the reference-relative interpretation
- Credible-effect calls at FDR 0.2 reproduce the paper's supplementary tables, e.g., neutrophils credibly increase in healthy and moderate vs severe COVID-19 BALF cases relative to Plasma (Supplementary Data 4), and DAM credibly increase in cortex but not cerebellum relative to microglia 1 (Supplementary Data 2)
- Effects for the reference cell type are constrained to zero in the fitted model
- In synthetic benchmarks, aggregated metrics show scCODA controlling false discoveries while retaining power (MCC 0.64, ROC AUC 0.99, average precision 0.94) with all competing model entries present in the output table
- The effects data frame can be exported as the expected csv (e.g., covid_balf_sccoda_credible_effects.csv) with all cell-type-by-covariate rows present

## Limitations

- scCODA does not model response variability within a condition and thus cannot detect heterogeneities between samples in response to treatment or donor variability
- scCODA assumes a log-linear relationship between covariates and cell abundance, which may be mis-specified in some cases
- scCODA does not model or infer dependency structure among cell compositions beyond those induced by compositional effects; modeling additional dependencies via hyperpriors would considerably increase computational complexity
- Requires pre-specified, biologically meaningful cell-type clustering assignments
- The theislab/scCODA implementation is no longer maintained; a Jax version is available in pertpy; GPU computation with TensorFlow has not been tested and is not recommended

## Evidence

- [intro] The model uses a Logit-normal spike-and-slab prior10 with a log-link function to estimate effects of binary (or continuous) covariates on cell-type proportions: "The model uses a Logit-normal spike-and-slab prior10 with a log-link function to estimate effects of binary (or continuous) covariates on cell-type proportions"
- [intro] scCODA models cell-type counts with a hierarchical Dirichlet-Multinomial distribution: "models cell-type counts with a hierarchical Dirichlet-Multinomial distribution that accounts for the uncertainty in cell-type proportions and the negative correlative bias"
- [methods] Once the model is set up, inference via HMC sampling can be performed via ``sample_hmc()``.: "Once the model is set up, inference via HMC sampling can be performed via ``sample_hmc()``."
- [methods] To see which effects were found to be significant, call ``summary()`` on the result object.: "To see which effects were found to be significant, call ``summary()`` on the result object."
- [methods] This package uses the tensorflow (>= 2.4) and tensorflow-probability (>=0.12) packages.: "This package uses the tensorflow (>= 2.4) and tensorflow-probability (>=0.12) packages."
- [discussion] scCODA does not model the response variability within a condition and thus cannot detect heterogeneities between samples in response to treatment or donor variability: "scCODA does not model the response variability within a condition and thus cannot detect heterogeneities between samples in response to treatment or donor variability"
- [readme] The GPU computation features of these packages have not been tested with scCODA and are thus not recommended.: [evidence span withheld — anchor status: external_doc]
