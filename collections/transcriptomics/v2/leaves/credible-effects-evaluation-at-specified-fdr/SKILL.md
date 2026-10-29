---
name: credible-effects-evaluation-at-specified-fdr
description: Use when a fitted scCODA CompositionalAnalysis result (CAResult) is available
  and you must decide which cell-type compositional effects are credible at a specified
  FDR level.
when_to_use_negative:
- When the goal is to detect response heterogeneity between samples within a condition
  (donor variability) — scCODA does not model response variability within a condition
  and cannot detect such heterogeneities.
- When cell-type annotations are not based on statistically sound, biologically meaningful
  clustering — scCODA requires pre-specified cell-type definitions.
- When the quantity of interest is gene expression rather than cell-type composition
  — this skill only covers differential cell-type abundance.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scCODA
  role: Fits the Dirichlet-multinomial spike-and-slab model, performs HMC inference,
    and applies FDR-based credible-effect selection via set_fdr/summary
  repo: https://github.com/theislab/scCODA
- name: Scanpy
  role: Provides the annotated single-cell count data from which the compositional
    dataset is imported (from_scanpy) before effect evaluation
  repo: https://scanpy.readthedocs.io
- name: theislab/scCODA_reproducibility
  role: Contains the code used to produce the paper's results and plots, including
    FDR threshold determination benchmarks
  repo: https://github.com/theislab/scCODA_reproducibility
- name: pertpy
  role: Successor implementation of the scCODA model in Jax, for users needing a maintained
    version
  repo: https://github.com/scverse/pertpy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-27150-6
    title: scCODA is a Bayesian model for compositional single-cell data analysis
  source_content_hash: sha256:097a5db8570c492e97069f6f2ffe96413e39f7e1261a36129a8cd6dbb3adf592
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scCODA
  tools:
  - Python (>=3.8)
  - Scanpy
  - pertpy
  - scCODA
  - theislab/scCODA_reproducibility
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# credible-effects-evaluation-at-specified-fdr

## Summary

This skill evaluates scCODA's Bayesian Dirichlet-multinomial compositional analysis results by applying a false-discovery-rate threshold (set_fdr, e.g. target_fdr=0.2) and reading credible cell-type effects from the Final Parameter column, always interpreted relative to a reference cell type.

## When to use

Use when a fitted scCODA CompositionalAnalysis result (CAResult) is available and you must decide which cell-type compositional effects are credible at a specified FDR level.

## When NOT to use

- When the goal is to detect response heterogeneity between samples within a condition (donor variability) — scCODA does not model response variability within a condition and cannot detect such heterogeneities.
- When cell-type annotations are not based on statistically sound, biologically meaningful clustering — scCODA requires pre-specified cell-type definitions.
- When the quantity of interest is gene expression rather than cell-type composition — this skill only covers differential cell-type abundance.

## Inputs

- NxK per-sample cell-type count matrix with sample covariates (age-group indicator), as a scCODA compositional anndata object built via sccoda.util.cell_composition_data.from_pandas or from_scanpy
- Fitted scCODA CAResult object from CompositionalAnalysis(...).sample_hmc()
- Target FDR level (e.g. 0.2)
- Pre-specified reference cell type (e.g. CD16+ monocytes)

## Outputs

- Summary table with per-cell-type effects where the Final Parameter column flags credible (non-zero) effects relative to the reference
- CSV of credible effects (e.g. sccoda_pbmc_results.csv)
- List of credible cell-type compositional changes (e.g. B-cell decrease as sole credible effect)

## How to apply

Set up the model with sccoda.util.comp_ana.CompositionalAnalysis using a patsy formula (e.g. 'age_group') and a reference cell type (e.g. CD16+ monocytes, which by construction carries zero effect). Perform inference via sample_hmc() on the hierarchical Dirichlet-multinomial model with the Logit-normal spike-and-slab prior. Then call set_fdr(target_fdr=0.2) on the result object — scCODA's spike-and-slab design means credible effects are selected by controlling false discoveries rather than per-cell-type p-values, which is why univariate tests (Poisson regression, Beta-Binomial regression, t tests) are inadequate for compositional data. Call summary() and extract cell types whose Final Parameter is non-zero; in the supercentenarian PBMC benchmark (7 supercentenarians vs 5 younger controls), only the B-cell population is credible relative to CD16+ monocytes at FDR 0.2, matching the FACS-validated B-cell decrease. Write the summary table (e.g. sccoda_pbmc_results.csv) for reproducibility.

## Related tools

- **scCODA** (Fits the Dirichlet-multinomial spike-and-slab model, performs HMC inference, and applies FDR-based credible-effect selection via set_fdr/summary) — https://github.com/theislab/scCODA
- **Scanpy** (Provides the annotated single-cell count data from which the compositional dataset is imported (from_scanpy) before effect evaluation) — https://scanpy.readthedocs.io
- **theislab/scCODA_reproducibility** (Contains the code used to produce the paper's results and plots, including FDR threshold determination benchmarks) — https://github.com/theislab/scCODA_reproducibility
- **pertpy** (Successor implementation of the scCODA model in Jax, for users needing a maintained version) — https://github.com/scverse/pertpy

## Examples

```
res = sccoda.util.comp_ana.CompositionalAnalysis(data, 'age_group', reference_cell_type='CD16+ monocytes').sample_hmc(); res.set_fdr(target_fdr=0.2); res.summary()
```

## Evaluation signals

- In the supercentenarian PBMC analysis, the credible-effects table must list B cells as the only cell type with a non-zero Final Parameter relative to CD16+ monocytes, reproducing Supplementary Data 1.
- The reference cell type must show a zero effect by construction (CD16+ monocytes in the benchmark task).
- summary() output must contain a Final Parameter column whose significances correspond to the set_fdr(target_fdr=0.2) threshold.
- Benchmark-level check: scCODA should reliably control the false discovery rate at the chosen target while maintaining detection performance.
- Schema check: the result object supports set_fdr, summary, and credible-effect extraction; the environment must run Python >=3.8 with tensorflow >=2.4.

## Limitations

- scCODA requires pre-specified cell-type definitions, which hinge on statistically sound and biologically meaningful clustering assignments.
- scCODA assumes a log-linear relationship between covariates and cell abundance, which may be mis-specified in some cases.
- scCODA does not model response variability within a condition and cannot detect heterogeneities between samples in response to treatment or donor variability.
- The TensorFlow-based implementation in theislab/scCODA is no longer maintained; a new Jax version is available in pertpy.
- GPU computation features of TensorFlow have not been tested with scCODA and are not recommended.

## Evidence

- [intro] spike-and-slab prior with credible-effect selection: "The model uses a Logit-normal spike-and-slab prior10 with a log-link function to estimate effects of binary (or continuous) covariates on cell-type proportions"
- [methods] HMC inference step: "inference via HMC sampling can be performed via ``sample_hmc()``"
- [methods] Final Parameter column semantics: "The ``Final Parameter`` column of the effects data frame shows the significances."
- [abstract] FDR control motivation: "scCODA demonstrated excellent detection performance, while reliably controlling for false discoveries"
- [readme] runtime requirement for running the analysis: [evidence span withheld — anchor status: external_doc]
