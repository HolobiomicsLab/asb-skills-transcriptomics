---
name: hmc-posterior-sampling
description: Use when a compositional scRNA-seq dataset (cell counts per sample) has
  been set up with sccoda's CompositionalAnalysis model and credible cell-type abundance
  effects must be estimated via posterior inference with sample_hmc().
when_to_use_negative:
- When the data are not compositional per-sample cell counts, e.g. already summarized
  feature tables without a sample-level covariate design
- When sample-level response variability or donor heterogeneity must be modeled —
  scCODA does not model response variability within a condition and cannot detect
  heterogeneities between samples
- When the log-linear relationship between covariates and cell abundance is likely
  mis-specified for the system under study
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: scCODA
  role: Provides the Bayesian Dirichlet-Multinomial model with spike-and-slab prior,
    the CompositionalAnalysis model class, sample_hmc() and sample_nuts() inference,
    and result evaluation via summary()/credible_effects()
  repo: https://github.com/theislab/scCODA
- name: TensorFlow
  role: 'Backend dependency (>=2.4; README: >=2.8) used by the HMC sampler in sample_hmc()'
- name: tensorflow-probability
  role: 'Probabilistic programming backend (>=0.12; README: >=0.16) implementing the
    Hamiltonian Monte Carlo sampler'
- name: scanpy
  role: Import cell-type annotated single-cell expression data that from_scanpy converts
    into the compositional analysis dataset
- name: pertpy
  role: Successor package where the maintained Jax version of the scCODA model now
    lives
  repo: https://github.com/scverse/pertpy
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-27150-6
    title: scCODA is a Bayesian model for compositional single-cell data analysis
  source_content_hash: sha256:e8d25b0f001ce1b108ac6c0cb03fb136fca2b1315e49863b2712679d4cd5a90c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scCODA
  tools:
  - Python
  - TensorFlow
  - pertpy
  - scCODA
  - scanpy
  - tensorflow-probability
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# hmc-posterior-sampling

## Summary

Hamiltonian Monte Carlo (HMC) posterior inference step of the scCODA Bayesian compositional model, which samples the posterior of the Dirichlet-Multinomial model with a Logit-normal spike-and-slab prior to estimate covariate effects on cell-type proportions from scRNA-seq cell counts.

## When to use

Use when a compositional scRNA-seq dataset (cell counts per sample) has been set up with sccoda's CompositionalAnalysis model and credible cell-type abundance effects must be estimated via posterior inference with sample_hmc().

## When NOT to use

- When the data are not compositional per-sample cell counts, e.g. already summarized feature tables without a sample-level covariate design
- When sample-level response variability or donor heterogeneity must be modeled — scCODA does not model response variability within a condition and cannot detect heterogeneities between samples
- When the log-linear relationship between covariates and cell abundance is likely mis-specified for the system under study

## Inputs

- Compositional anndata dataset of per-sample cell-type counts (from from_pandas or from_scanpy)
- CompositionalAnalysis model object configured with a patsy formula and reference cell type
- Cell-type annotated scRNA-seq data (e.g. from Single Cell Portal, e.g. SCP44 mouse intestinal epithelium)

## Outputs

- CAResult object containing posterior samples of covariate effects
- Summary data frame with Final Parameter significances per cell type and covariate
- credible_effects() output listing statistically credible compositional changes at the chosen FDR (e.g. 0.2)

## How to apply

After building the compositional anndata dataset (e.g. via sccoda.util.cell_composition_data.from_scanpy with infection condition as covariate) and instantiating the model with sccoda.util.comp_ana.CompositionalAnalysis using a patsy formula that encodes the condition and specifies a reference cell type (e.g. uninfected control), run posterior inference with sample_hmc(), the TensorFlow/tensorflow-probability-backed Hamiltonian Monte Carlo sampler; No-U-Turn sampling via sample_nuts() is an alternative. The model uses a hierarchical Dirichlet-Multinomial likelihood with a Logit-normal spike-and-slab prior and log-link, jointly modeling all cell-type proportions to handle compositionality and low replicate numbers. After sampling, call summary() on the CAResult object and extract credible_effects() after set_fdr (e.g. FDR 0.2), interpreting all effects relative to the reference cell type. Correct application requires the environment to have Python >=3.8, tensorflow >=2.4 (README: >=2.8) and tensorflow-probability >=0.12 (README: >=0.16); GPU features are untested and not recommended.

## Related tools

- **scCODA** (Provides the Bayesian Dirichlet-Multinomial model with spike-and-slab prior, the CompositionalAnalysis model class, sample_hmc() and sample_nuts() inference, and result evaluation via summary()/credible_effects()) — https://github.com/theislab/scCODA
- **TensorFlow** (Backend dependency (>=2.4; README: >=2.8) used by the HMC sampler in sample_hmc())
- **tensorflow-probability** (Probabilistic programming backend (>=0.12; README: >=0.16) implementing the Hamiltonian Monte Carlo sampler)
- **scanpy** (Import cell-type annotated single-cell expression data that from_scanpy converts into the compositional analysis dataset)
- **pertpy** (Successor package where the maintained Jax version of the scCODA model now lives) — https://github.com/scverse/pertpy

## Examples

```
import sccoda; model = sccoda.model.comp_ana.CompositionalAnalysis(data, formula="condition", reference_cell_type="Goblet"); results = model.sample_hmc(); print(results.summary()); print(results.credible_effects())
```

## Evaluation signals

- HMC sampling completes without TensorFlow/tensorflow-probability runtime errors on a Python >=3.8 environment
- summary() on the CAResult yields an effects data frame whose 'Final Parameter' column contains significances for every cell type and covariate
- credible_effects() at set_fdr 0.2 returns a sparse set of effects (e.g. in the SCP44 Salmonella/H. polygyrus case, exactly one credible increase — Enterocytes in Salmonella 2-day — and no credible effects in either H. polygyrus condition)
- All reported effects are interpreted relative to the specified reference cell type (e.g. uninfected control group)
- Reproduced effect calls match the published Supplementary Data 5 for the reference analysis

## Limitations

- The TensorFlow-based implementation is no longer maintained; the new Jax version is available in pertpy
- GPU computation features of tensorflow/tensorflow-probability have not been tested with scCODA and are not recommended
- scCODA does not model response variability within a condition and cannot detect heterogeneities between samples in response to treatment or donor variability
- scCODA assumes a log-linear relationship between covariates and cell abundance, which may be mis-specified in some cases
- scCODA requires pre-specified cell-type definitions that hinge on statistically sound and biologically meaningful clustering assignments
- Credible changes must be interpreted relative to the selected reference cell type, so conclusions depend on that choice

## Evidence

- [methods] Once the model is set up, inference via HMC sampling can be performed via ``sample_hmc()``.: "Once the model is set up, inference via HMC sampling can be performed via ``sample_hmc()``."
- [methods] Alternatively, No-U-Turn sampling is available via ``sample_nuts()``.: "Alternatively, No-U-Turn sampling is available via ``sample_nuts()``."
- [intro] The model uses a Logit-normal spike-and-slab prior10 with a log-link function to estimate effects of binary (or continuous) covariates on cell-type proportions: "The model uses a Logit-normal spike-and-slab prior10 with a log-link function to estimate effects of binary (or continuous) covariates on cell-type proportions"
- [methods] This package uses the tensorflow (>= 2.4) and tensorflow-probability (>=0.12) packages.: "This package uses the tensorflow (>= 2.4) and tensorflow-probability (>=0.12) packages."
- [methods] To see which effects were found to be significant, call ``summary()`` on the result object.: "To see which effects were found to be significant, call ``summary()`` on the result object."
- [readme] The GPU computation features of these packages have not been tested with scCODA and are thus not recommended.: [evidence span withheld — anchor status: external_doc]
- [readme] This implementation is no longer maintained. A new version in Jax is available in [pertpy](https://pertpy.readthedocs.io/en/latest).: "This implementation is no longer maintained. A new version in Jax is available in [pertpy](https://pertpy.readthedocs.io/en/latest)."
- [discussion] scCODA does not model the response variability within a condition and thus cannot detect heterogeneities between samples in response to treatment or donor variability: "scCODA does not model the response variability within a condition and thus cannot detect heterogeneities between samples in response to treatment or donor variability"
