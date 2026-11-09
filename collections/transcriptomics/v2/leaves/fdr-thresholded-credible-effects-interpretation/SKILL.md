---
name: fdr-thresholded-credible-effects-interpretation
description: Use when scRNA-seq cell-type count tables per sample (e.g. from Scanpy)
  have been modeled with scCODA and you must decide which cell-type abundance changes
  survive FDR control, e.g. FDR 0.2 in the Salmonella/H. polygyrus SCP44 dataset.
when_to_use_negative:
- When applying standard univariate tests (Poisson regression, Beta-Binomial regression,
  t tests) that ignore the compositional nature of the data — these can falsely deem
  population shifts real effects
- When detecting heterogeneities between samples in response to treatment or donor
  variability — scCODA does not model response variability within a condition
- When cell-type counts are not yet aggregated per sample (e.g. raw per-cell data
  lacking a covariate structure for the patsy formula)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scCODA
  role: Provides CompositionalAnalysis model, sample_hmc(), set_fdr, summary(), and
    credible_effects() for FDR-thresholded credible-effect extraction
  repo: https://github.com/theislab/scCODA
- name: TensorFlow
  role: Backend (>=2.4) powering the HMC sampler used for posterior inference whose
    results are then thresholded
  repo: https://github.com/theislab/scCODA
- name: tensorflow-probability
  role: Backend (>=0.12) supplying the Hamiltonian Monte Carlo sampler in sample_hmc()
  repo: https://github.com/theislab/scCODA
- name: scanpy
  role: Supplies single-cell expression data that from_scanpy aggregates into per-sample
    cell-count compositional datasets for the model
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-27150-6
    title: scCODA is a Bayesian model for compositional single-cell data analysis
  source_content_hash: sha256:6f83b3e30bc4248f9e87fcff5491b4f72b197ec930b1a2ee132fdb7d3a671a61
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scCODA
  tools:
  - Python
  - TensorFlow
  - scCODA
  - scanpy
  - tensorflow-probability
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# fdr-thresholded-credible-effects-interpretation

## Summary

Extract cell-type compositional effects from a scCODA Bayesian Dirichlet-Multinomial model with spike-and-slab priors, calling only effects that remain credible at a user-set false discovery rate (e.g. FDR 0.2) and interpreting them relative to a reference cell type.

## When to use

Use when scRNA-seq cell-type count tables per sample (e.g. from Scanpy) have been modeled with scCODA and you must decide which cell-type abundance changes survive FDR control, e.g. FDR 0.2 in the Salmonella/H. polygyrus SCP44 dataset.

## When NOT to use

- When applying standard univariate tests (Poisson regression, Beta-Binomial regression, t tests) that ignore the compositional nature of the data — these can falsely deem population shifts real effects
- When detecting heterogeneities between samples in response to treatment or donor variability — scCODA does not model response variability within a condition
- When cell-type counts are not yet aggregated per sample (e.g. raw per-cell data lacking a covariate structure for the patsy formula)

## Inputs

- CAResult object from scCODA (posterior output of CompositionalAnalysis + sample_hmc())
- Per-sample cell-type count compositional anndata dataset aggregated via sccoda.util.cell_composition_data.from_scanpy
- Chosen false discovery rate threshold (e.g. 0.2)
- Reference cell type specification (pre-specified, e.g. uninfected-control-relative, or automatic selection)

## Outputs

- Effects summary data frame with 'Final Parameter' significances (via summary())
- List of credible cell-type effects at the set FDR threshold (credible_effects()), e.g. Enterocyte increase for Salmonella 2-day

## How to apply

After HMC posterior inference via sample_hmc() on a CompositionalAnalysis model, call summary() on the resulting CAResult object to inspect the 'Final Parameter' effects data frame. Set the desired false discovery rate with set_fdr (the paper's mouse intestinal epithelium reproduction uses 0.2) and extract credible_effects() to list significant shifts. All credible effects must be interpreted relative to the reference cell type, which scCODA either selects automatically or uses from a pre-specified choice encoded in the patsy formula (uninfected control as reference for infection condition covariates). Validation compares the credible-effect list to published expectations: for SCP44, exactly one credible Enterocyte increase under Salmonella 2-day and no credible effects under either H. polygyrus condition, matching Supplementary Data 5.

## Related tools

- **scCODA** (Provides CompositionalAnalysis model, sample_hmc(), set_fdr, summary(), and credible_effects() for FDR-thresholded credible-effect extraction) — https://github.com/theislab/scCODA
- **TensorFlow** (Backend (>=2.4) powering the HMC sampler used for posterior inference whose results are then thresholded) — https://github.com/theislab/scCODA
- **tensorflow-probability** (Backend (>=0.12) supplying the Hamiltonian Monte Carlo sampler in sample_hmc()) — https://github.com/theislab/scCODA
- **scanpy** (Supplies single-cell expression data that from_scanpy aggregates into per-sample cell-count compositional datasets for the model)

## Examples

```
from sccoda.util.comp_ana import CompositionalAnalysis; res = CompositionalAnalysis(data, formula="condition", reference_cell_type="Goblet_cells").sample_hmc(); res.set_fdr(0.2); print(res.credible_effects())
```

## Evaluation signals

- Salmonella 2-day yields exactly one credible abundance increase (Enterocytes) and both H. polygyrus conditions yield none, matching published Supplementary Data 5
- summary() output shows 'Final Parameter' significances consistent with credible_effects() at the chosen FDR (0.2 in the reproduction)
- All reported effects are interpreted relative to the chosen reference cell type; changes conflicting with the original Poisson model (Tuft, Goblet, early transit-amplifying cells) are not confirmed at FDR 0.2
- HMC sampling converged and effects are significant only above the spike-and-slab inclusion threshold at the set FDR

## Limitations

- scCODA requires pre-specified cell-type definitions, which hinge on statistically sound and biologically meaningful clustering assignments
- scCODA assumes a log-linear relationship between covariates and cell abundance, which may be mis-specified in some cases
- scCODA does not model the response variability within a condition and cannot detect heterogeneities between samples in response to treatment or donor variability
- scCODA does not model or infer any dependency structure among cell compositions beyond those induced by compositional effects
- The TensorFlow-backed scCODA implementation is no longer maintained; the Jax version is available in pertpy

## Evidence

- [methods] HMC sampling inference: "Once the model is set up, inference via HMC sampling can be performed via ``sample_hmc()``."
- [methods] result evaluation via summary(): "To see which effects were found to be significant, call ``summary()`` on the result object."
- [intro] FDR control motivation: "standard univariate statistical models that test compositional changes of each cell type independently may falsely deem certain population shifts as real effects"
- [readme] README backend requirements: [evidence span withheld — anchor status: external_doc]
- [readme] README reproducibility code: "Code for reproducing the analysis from the paper is available [here](https://github.com/theislab/scCODA_reproducibility)."
