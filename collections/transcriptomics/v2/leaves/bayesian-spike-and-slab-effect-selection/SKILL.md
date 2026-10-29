---
name: bayesian-spike-and-slab-effect-selection
description: Use when scRNA-seq samples (or other count compositional data) are aggregated
  per sample into cell-type counts and you need differential abundance testing that
  controls the FDR, especially with a low number of experimental replicates.
when_to_use_negative:
- When the data are not compositional cell-type counts but, e.g., already a feature
  table or expression matrix — scCODA models aggregated cell-type counts per sample.
- 'When you need to detect heterogeneities between samples in response to treatment:
  scCODA does not model response variability within a condition or donor variability.'
- 'When cell-type definitions are uncertain: scCODA requires pre-specified cell-type
  definitions that hinge on statistically sound clustering assignments.'
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: scCODA
  role: Bayesian Dirichlet-Multinomial model with Logit-normal spike-and-slab prior
    used to detect credible compositional changes; provides CompositionalAnalysis,
    sample_hmc, summary, credible_effects
  repo: https://github.com/theislab/scCODA
- name: scanpy
  role: Imports and holds the single-cell expression data that from_scanpy aggregates
    into the compositional analysis dataset
- name: TensorFlow
  role: Backend dependency (>=2.4) powering the HMC sampler used for posterior inference
- name: tensorflow-probability
  role: Backend dependency (>=0.12) for the Hamiltonian Monte Carlo inference in sample_hmc()
- name: pertpy
  role: Newer Jax-based implementation of scCODA functionality, as the TensorFlow
    version is no longer maintained
  repo: https://github.com/scverse/pertpy
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-27150-6
    title: scCODA is a Bayesian model for compositional single-cell data analysis
  source_content_hash: sha256:cd0e53339017b512f4399ebdc1d7912f6c3dd354de39e069f2864236432155d6
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
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# bayesian-spike-and-slab-effect-selection

## Summary

Identifies credible cell-type compositional changes by modeling cell-type counts with a hierarchical Dirichlet-Multinomial model and a Logit-normal spike-and-slab prior (scCODA), controlling false discoveries even with few replicates. Effects are inferred via Hamiltonian Monte Carlo and interpreted relative to a reference cell type.

## When to use

Use when scRNA-seq samples (or other count compositional data) are aggregated per sample into cell-type counts and you need differential abundance testing that controls the FDR, especially with a low number of experimental replicates.

## When NOT to use

- When the data are not compositional cell-type counts but, e.g., already a feature table or expression matrix — scCODA models aggregated cell-type counts per sample.
- When you need to detect heterogeneities between samples in response to treatment: scCODA does not model response variability within a condition or donor variability.
- When cell-type definitions are uncertain: scCODA requires pre-specified cell-type definitions that hinge on statistically sound clustering assignments.

## Inputs

- cell-type annotated single-cell data in Scanpy AnnData format (or a pandas DataFrame of per-sample cell counts)
- per-sample covariates (e.g., infection condition: Salmonella 2-day, H. polygyrus 3-day/10-day, uninfected control)
- reference cell type or covariate level specification

## Outputs

- CAResult object from CompositionalAnalysis
- summary table with Final Parameter column indicating significant/credible effects
- credible_effects() boolean output at a chosen FDR (e.g., 0.2)

## How to apply

Aggregate per-sample cell-type counts (e.g., via sccoda.util.cell_composition_data.from_scanpy on a Scanpy AnnData) into a compositional data object with covariates such as infection condition. Set up the model with sccoda.util.comp_ana.CompositionalAnalysis using a patsy formula encoding the covariate and specifying a reference cell type (e.g., the uninfected control condition). Perform inference via sample_hmc(), the TensorFlow/tensorflow-probability-backed Hamiltonian Monte Carlo sampler (No-U-Turn sampling via sample_nuts() is an alternative). Extract credible effects via summary()/credible_effects() after setting the false discovery rate (e.g., set_fdr to 0.2), interpreting all effects relative to the reference cell type. The spike-and-slab prior induces sparsity, so only effects surviving the FDR threshold are deemed credible, unlike univariate Poisson regression which can falsely deem population shifts real due to negative correlations of cell-type proportions.

## Related tools

- **scCODA** (Bayesian Dirichlet-Multinomial model with Logit-normal spike-and-slab prior used to detect credible compositional changes; provides CompositionalAnalysis, sample_hmc, summary, credible_effects) — https://github.com/theislab/scCODA
- **scanpy** (Imports and holds the single-cell expression data that from_scanpy aggregates into the compositional analysis dataset)
- **TensorFlow** (Backend dependency (>=2.4) powering the HMC sampler used for posterior inference)
- **tensorflow-probability** (Backend dependency (>=0.12) for the Hamiltonian Monte Carlo inference in sample_hmc())
- **pertpy** (Newer Jax-based implementation of scCODA functionality, as the TensorFlow version is no longer maintained) — https://github.com/scverse/pertpy

## Examples

```
import sccoda; mod = sccoda.model.other_models.CompositionalAnalysis?
```

## Evaluation signals

- The Final Parameter column of the summary effects data frame marks only a sparse set of credible effects due to the spike-and-slab prior; at FDR 0.2 the Salmonella SCP44 benchmark should yield exactly one credible increase (Enterocytes).
- Both H. polygyrus conditions should yield no credible effects, matching published Supplementary Data 5.
- All credible effects must be interpretable relative to the chosen reference cell type/condition (uninfected control in the SCP44 example).
- HMC diagnostics from the TensorFlow-probability sampler should show converged posterior inference (the advanced tutorial discusses model diagnostics).

## Limitations

- scCODA requires pre-specified cell-type definitions, which hinge on statistically sound and biologically meaningful clustering assignments.
- scCODA assumes a log-linear relationship between covariates and cell abundance, which may be mis-specified in some cases.
- scCODA does not model or infer dependency structure among cell compositions beyond those induced by compositional effects, and modeling additional dependencies via hyperpriors would considerably increase computational complexity.
- scCODA does not model response variability within a condition and cannot detect heterogeneities between samples in response to treatment or donor variability.
- The TensorFlow-based implementation is no longer maintained; a new Jax version is available in pertpy, and GPU computation features of TensorFlow have not been tested with scCODA.

## Evidence

- [intro] model uses Logit-normal spike-and-slab prior with log-link for covariate effects: "The model uses a Logit-normal spike-and-slab prior10 with a log-link function to estimate effects of binary (or continuous) covariates on cell-type proportions"
- [methods] inference via HMC sampling via sample_hmc(): "Once the model is set up, inference via HMC sampling can be performed via ``sample_hmc()``."
- [intro] credible effects must be interpreted relative to the reference: "scCODA can automatically select an appropriate cell type as the reference (Methods—“Automatic reference selection”) or uses a pre-specified reference cell type11"
- [readme] README: TensorFlow backend and reproducibility code pointer: [evidence span withheld — anchor status: external_doc]
- [intro] univariate models falsely deem shifts real: "standard univariate statistical models that test compositional changes of each cell type independently may falsely deem certain population shifts as real effects"
