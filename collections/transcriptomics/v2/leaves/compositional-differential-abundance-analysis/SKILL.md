---
name: compositional-differential-abundance-analysis
description: Use when per-sample cell-type count tables from scRNA-seq (e.
when_to_use_negative:
- The question is whether response variability within a condition or donor-to-donor
  heterogeneity in treatment response exists — scCODA does not model response variability
  within a condition and cannot detect heterogeneities between samples
- Cell-type annotations come from unreliable clustering — scCODA requires pre-specified
  cell-type definitions that hinge on statistically sound and biologically meaningful
  clustering assignments
- The expected covariate-to-abundance relationship is not log-linear — scCODA assumes
  a log-linear relationship which may be mis-specified
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scCODA
  role: Core Bayesian Dirichlet-Multinomial model with logit-normal spike-and-slab
    prior; provides CompositionalAnalysis, sample_hmc(), sample_nuts(), set_fdr(),
    summary(), credible_effects()
  repo: https://github.com/theislab/scCODA
- name: scanpy
  role: Reads single-cell expression data and supplies the annotated adata objects
    that from_scanpy/from_scanpy_list/from_scanpy_dir transform into the compositional
    analysis dataset
  repo: https://scanpy.readthedocs.io
- name: TensorFlow
  role: Backend (>=2.4) for the Hamiltonian Monte Carlo sampler used by sample_hmc()
- name: tensorflow-probability
  role: Probability distributions and HMC machinery (>=0.12) backing the scCODA posterior
    inference
provenance:
  source_task_ids:
  - task_006
  - task_008
  - task_005
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-27150-6
    title: scCODA is a Bayesian model for compositional single-cell data analysis
  source_content_hash: sha256:26a744090b252850baa9e12cf80c0c47a7764aa10fbb62b9ac86f929dc7e9eda
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# compositional-differential-abundance-analysis

## Summary

Bayesian differential abundance testing for cell-type composition changes in scRNA-seq using scCODA: a hierarchical Dirichlet-Multinomial model with a logit-normal spike-and-slab prior that jointly models all cell-type proportions to control false discoveries, even with few replicates.

## When to use

Use when per-sample cell-type count tables from scRNA-seq (e.g., disease vs. control conditions, sometimes with as few as 2 replicates per group) must be tested for credible compositional shifts, and univariate tests like Poisson regression or Wilcoxon rank-sum risk false positives from compositional effects.

## When NOT to use

- The question is whether response variability within a condition or donor-to-donor heterogeneity in treatment response exists — scCODA does not model response variability within a condition and cannot detect heterogeneities between samples
- Cell-type annotations come from unreliable clustering — scCODA requires pre-specified cell-type definitions that hinge on statistically sound and biologically meaningful clustering assignments
- The expected covariate-to-abundance relationship is not log-linear — scCODA assumes a log-linear relationship which may be mis-specified

## Inputs

- Cell-type annotated scRNA-seq data readable via scanpy (in-memory or on disk) or a pandas DataFrame of per-sample cell counts
- Per-sample covariates (e.g., health status, infection condition: Salmonella 2-day, H. polygyrus 3-day/10-day, uninfected control)
- A patsy formula string encoding the condition covariate
- Choice of reference cell type (pre-specified, e.g., microglia 1, or 'automatic')

## Outputs

- Compositional anndata dataset (data.X count matrix, data.obs covariates)
- CAResult object from HMC posterior samples of intercepts and effects
- summary()/credible_effects() table with 'Final Parameter' column marking credible effects at FDR 0.2 (e.g., sccoda_uc_colon_results.csv, credible_effects_summary.csv)

## How to apply

Aggregate annotated single-cell data per sample into a compositional anndata object (data.X = cell counts, data.obs = covariates) using sccoda.util.cell_composition_data.from_pandas or from_scanpy. Generate a sccoda.util.comp_ana.CompositionalAnalysis instance with a patsy formula encoding the condition covariate and either a pre-specified reference cell type (e.g., microglia 1 in the Alzheimer's model) or scCODA's automatic reference selection; the reference cell type's effects are constrained to zero, so all credible changes must be interpreted relative to it. Run posterior inference via sample_hmc() (TensorFlow >=2.4 / tensorflow-probability >=0.12 backed HMC; sample_nuts() is the No-U-Turn alternative, ~5 minutes per small dataset). Set the false discovery rate via set_fdr(target_fdr_level=0.2) and extract credible effects with summary()/credible_effects(); the 'Final Parameter' column flags significance (0 = not credible). Validate against published supplementary tables, e.g., the Treg and IAF increases in UC colon (SCP259, Supplementary Data 3) or the single Enterocyte increase in Salmonella-infected epithelium (SCP44, Supplementary Data 5).

## Related tools

- **scCODA** (Core Bayesian Dirichlet-Multinomial model with logit-normal spike-and-slab prior; provides CompositionalAnalysis, sample_hmc(), sample_nuts(), set_fdr(), summary(), credible_effects()) — https://github.com/theislab/scCODA
- **scanpy** (Reads single-cell expression data and supplies the annotated adata objects that from_scanpy/from_scanpy_list/from_scanpy_dir transform into the compositional analysis dataset) — https://scanpy.readthedocs.io
- **TensorFlow** (Backend (>=2.4) for the Hamiltonian Monte Carlo sampler used by sample_hmc())
- **tensorflow-probability** (Probability distributions and HMC machinery (>=0.12) backing the scCODA posterior inference)

## Examples

```
from sccoda.util import comp_ana as ca; model = ca.CompositionalAnalysis(data, formula="condition", reference_cell_type="automatic"); res = model.sample_hmc(); res.set_fdr(0.2); res.credible_effects()
```

## Evaluation signals

- The 'Final Parameter' column of the effects data frame is populated, with 0 flagging effects not statistically credible and nonzero values flagging credible effects
- Effects of the chosen (or automatically selected) reference cell type are exactly zero, and all reported effects are interpreted relative to that reference
- Credible-effects output at FDR 0.2 reproduces published supplementary tables, e.g., Treg and IAF increases in UC colon (Supplementary Data 3), single Enterocyte increase in Salmonella with no credible H. polygyrus effects (Supplementary Data 5), and DAM increase in cortex but not cerebellum (Supplementary Data 2)
- Posterior inclusion probabilities from spike-and-slab indicators are computed for each cell-type-by-covariate effect before FDR threshold determination
- Reproducibility benchmarks (threshold determination, model comparison, overall benchmark) from theislab/scCODA_reproducibility can be re-run; data and results are provided via zenodo (doi:10.5281/zenodo.6473255)

## Limitations

- scCODA assumes a log-linear relationship between covariates and cell abundance, which may be mis-specified in some cases
- scCODA does not model or infer any dependency structure among the cell compositions beyond those induced by compositional effects; adding hyperpriors would considerably increase computational complexity
- scCODA does not model response variability within a condition and thus cannot detect heterogeneities between samples in response to treatment or donor variability
- Results depend on pre-specified cell-type definitions from statistically sound, biologically meaningful clustering
- The TensorFlow-based theislab/scCODA implementation is no longer maintained; a Jax version is available in pertpy

## Evidence

- [intro] propose a Bayesian approach for cell-type composition differential abundance analysis: "propose a Bayesian approach for cell-type composition differential abundance analysis to further address the low replicate issue"
- [intro] Logit-normal spike-and-slab prior with log-link: "The model uses a Logit-normal spike-and-slab prior10 with a log-link function to estimate effects of binary (or continuous) covariates on cell-type proportions"
- [methods] HMC inference entry point: "Once the model is set up, inference via HMC sampling can be performed via ``sample_hmc()``."
- [methods] Final Parameter significance column: "The ``Final Parameter`` column of the effects data frame shows the significances."
- [readme] scCODA purpose statement: "scCODA allows for identification of compositional changes in high-throughput sequencing count data, especially cell compositions from scRNA-seq."
- [discussion] No response-variability modeling: "scCODA does not model the response variability within a condition and thus cannot detect heterogeneities between samples in response to treatment or donor variability"
