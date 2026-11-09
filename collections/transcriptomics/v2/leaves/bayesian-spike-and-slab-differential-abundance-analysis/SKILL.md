---
name: bayesian-spike-and-slab-differential-abundance-analysis
description: Use when testing cell-type compositional changes across few biological
  replicates (e.g. 7 supercentenarian vs 5 control PBMC samples) from an NxK cell-type
  count matrix with a binary covariate, where univariate tests would inflate false
  positives.
when_to_use_negative:
- Testing gene-level expression rather than cell-type proportions — standard univariate
  tests (Poisson regression, Beta-Binomial regression, t tests) are inadequate here,
  but this skill is also only defined for compositional count data, not expression
  values
- You need to detect heterogeneities between samples in response to treatment or donor
  variability — scCODA does not model response variability within a condition
- Cell-type cluster definitions are unsound — scCODA requires pre-specified cell-type
  definitions that hinge on statistically sound and biologically meaningful clustering
  assignments
tools:
- name: scCODA
  role: 'Core model and toolbox: CompositionalAnalysis, sample_hmc(), set_fdr(), summary()'
  repo: https://github.com/theislab/scCODA
- name: TensorFlow
  role: Backend (>=2.4/2.8 per paper/README) used together with tensorflow-probability
    for HMC inference
- name: Scanpy
  role: Import of annotated single-cell expression data and ecosystem integration
    (from_scanpy entry points)
  repo: https://scanpy.readthedocs.io
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-27150-6
    title: scCODA is a Bayesian model for compositional single-cell data analysis
  source_content_hash: sha256:f8ad18961aca27e4b645aca15c025ae911586f9bc85c8aa6f211a9c657beea8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scCODA
  tools:
  - Python (>=3.8)
  - Scanpy
  - TensorFlow
  - scCODA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# bayesian-spike-and-slab-differential-abundance-analysis

## Summary

Bayesian differential abundance testing of cell-type proportions using scCODA's hierarchical Dirichlet-Multinomial model with a Logit-normal spike-and-slab prior, which jointly models all measured cell-type proportions to control false discoveries under compositional bias and low replicates.

## When to use

Use when testing cell-type compositional changes across few biological replicates (e.g. 7 supercentenarian vs 5 control PBMC samples) from an NxK cell-type count matrix with a binary covariate, where univariate tests would inflate false positives.

## When NOT to use

- Testing gene-level expression rather than cell-type proportions — standard univariate tests (Poisson regression, Beta-Binomial regression, t tests) are inadequate here, but this skill is also only defined for compositional count data, not expression values
- You need to detect heterogeneities between samples in response to treatment or donor variability — scCODA does not model response variability within a condition
- Cell-type cluster definitions are unsound — scCODA requires pre-specified cell-type definitions that hinge on statistically sound and biologically meaningful clustering assignments

## Inputs

- NxK cell-type count matrix per sample (K cell types, N samples)
- sample-level covariates (e.g. age-group indicator: 7 supercentenarians vs 5 younger controls)
- cell-type cluster annotations (e.g. from Scanpy-annotated scRNA-seq data)

## Outputs

- CAResult object from sccoda.util.comp_ana.CompositionalAnalysis
- effects summary DataFrame whose 'Final Parameter' column marks credible cell types at the target FDR (0.2)
- summary table written to CSV (sccoda_pbmc_results.csv)

## How to apply

Build the compositional anndata object with sccoda.util.cell_composition_data.from_pandas, passing the NxK cell-type count matrix (data.X) and sample covariates such as the age-group indicator (data.obs). Create an instance of sccoda.util.comp_ana.CompositionalAnalysis with a patsy formula (e.g. 'age_group') and a pre-specified reference cell type — for the supercentenarian PBMC data, CD16+ monocytes — since credible changes must be interpreted relative to this reference. Run Bayesian inference via sample_hmc() (No-U-Turn sampling via sample_nuts() is an alternative), which fits the hierarchical Dirichlet-Multinomial model with a Logit-normal spike-and-slab prior and a log-link function. Apply set_fdr(target_fdr=0.2) and call summary() on the CAResult object, reading the 'Final Parameter' column to extract credible effects; in the supercentenarian case only B cells should be non-zero relative to CD16+ monocytes. Write the summary table to file (e.g. sccoda_pbmc_results.csv) and check it against the paper's Supplementary Data 1.

## Related tools

- **scCODA** (Core model and toolbox: CompositionalAnalysis, sample_hmc(), set_fdr(), summary()) — https://github.com/theislab/scCODA
- **TensorFlow** (Backend (>=2.4/2.8 per paper/README) used together with tensorflow-probability for HMC inference)
- **Scanpy** (Import of annotated single-cell expression data and ecosystem integration (from_scanpy entry points)) — https://scanpy.readthedocs.io

## Examples

```
from sccoda.util.comp_ana import CompositionalAnalysis; model = CompositionalAnalysis(data, formula="age_group", reference_cell_type="CD16+ monocytes"); res = model.sample_hmc(); res.set_fdr(target_fdr=0.2); res.summary()
```

## Evaluation signals

- The credible-effects table lists B cells as the only cell type with a non-zero 'Final Parameter' relative to CD16+ monocytes (FDR level 0.2), matching Supplementary Data 1
- All other cell types show zero Final Parameter values (spike-and-slab shrunken effects), confirming no spurious credible effects despite compositional negative correlations
- HMC sampling converges and the summary() table contains effect estimates for every cell type with the reference row (CD16+ monocytes) fixed
- The resulting sccoda_pbmc_results.csv round-trips the same credible set on re-run at target_fdr=0.2

## Limitations

- scCODA assumes a log-linear relationship between covariates and cell abundance, which may be mis-specified
- scCODA does not model or infer dependency structure among cell compositions beyond those induced by compositional effects
- scCODA does not model response variability within a condition and cannot detect heterogeneities between samples in response to treatment or donor variability
- Requires pre-specified cell-type definitions that hinge on statistically sound and biologically meaningful clustering assignments
- The TensorFlow-based implementation is no longer maintained; a Jax version is available in pertpy, and GPU computation of TensorFlow dependencies is untested/recommended against

## Evidence

- [intro] The model uses a Logit-normal spike-and-slab prior10 with a log-link function to estimate effects of binary (or continuous) covariates on cell-type proportions: "The model uses a Logit-normal spike-and-slab prior10 with a log-link function to estimate effects of binary (or continuous) covariates on cell-type proportions"
- [methods] HMC inference: "Once the model is set up, inference via HMC sampling can be performed via ``sample_hmc()``."
- [methods] Result interpretation: "To see which effects were found to be significant, call ``summary()`` on the result object."
- [readme] Maintenance status: "This implementation is no longer maintained. A new version in Jax is available in [pertpy]"
- [readme] Dependencies: [evidence span withheld — anchor status: external_doc]
