---
name: reference-cell-type-selection
description: Use when running scCODA CompositionalAnalysis on aggregated per-sample
  cell-type counts and you must fix or auto-select the reference cell type before
  HMC sampling, so credible effects (Final Parameter, FDR e.g. 0.2) are interpreted
  relative to that reference.
when_to_use_negative:
- The data are raw per-cell expression profiles rather than aggregated per-sample
  cell-type proportions — scCODA requires cell-type definitions from prior clustering
- The scientific question concerns within-condition response variability or donor-level
  heterogeneity, which scCODA explicitly does not model
- The expected effect is non-log-linear in covariates, since scCODA assumes a log-linear
  relationship between covariates and cell abundance
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scCODA
  role: Provides CompositionalAnalysis model with pre-specified or automatic reference
    selection, sample_hmc() inference, and summary()/credible_effects() for reading
    effects relative to the reference
  repo: https://github.com/theislab/scCODA
- name: scanpy
  role: Ecosystem for generating cell-type annotations and aggregated count data that
    feed from_pandas/from_scanpy compositional dataset construction
- name: TensorFlow
  role: Backend (tensorflow, tensorflow-probability) powering the HMC sampling used
    after reference selection
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-27150-6
    title: scCODA is a Bayesian model for compositional single-cell data analysis
  source_content_hash: sha256:ea6152da911108348dca6e86e80f2ef2c768bc759de53ef488d4d6ccac81eb17
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scCODA
  tools:
  - Python (>=3.8)
  - TensorFlow
  - scCODA
  - scanpy
  - tensorflow-probability (>=0.12)
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# reference-cell-type-selection

## Summary

Selecting the reference cell type for scCODA's Dirichlet-multinomial compositional model, either pre-specified (e.g., CD16+ monocytes in the supercentenarian PBMC analysis) or via scCODA's automatic reference selection (e.g., the UC colon lamina propria/epithelium analyses), so that credible spike-and-slab effects are interpretable relative to an abundant, stable baseline cell type.

## When to use

Use when running scCODA CompositionalAnalysis on aggregated per-sample cell-type counts and you must fix or auto-select the reference cell type before HMC sampling, so credible effects (Final Parameter, FDR e.g. 0.2) are interpreted relative to that reference.

## When NOT to use

- The data are raw per-cell expression profiles rather than aggregated per-sample cell-type proportions — scCODA requires cell-type definitions from prior clustering
- The scientific question concerns within-condition response variability or donor-level heterogeneity, which scCODA explicitly does not model
- The expected effect is non-log-linear in covariates, since scCODA assumes a log-linear relationship between covariates and cell abundance

## Inputs

- Per-sample cell-type count matrix (NxK) as compositional anndata from sccoda.util.cell_composition_data.from_pandas
- Sample covariate annotations in data.obs (e.g., age_group for supercentenarian vs younger-control PBMCs; health status for UC colon samples)
- Optional pre-specified reference cell type name (e.g., CD16+ monocytes)

## Outputs

- CAResult object with credible-effects table (Final Parameter column) relative to the selected reference
- Summary CSV such as sccoda_pbmc_results.csv or sccoda_uc_colon_results.csv listing credible cell-type shifts at FDR 0.2

## How to apply

After building the compositional anndata dataset (NxK count matrix in data.X, covariates such as age_group or health status in data.obs via sccoda.util.cell_composition_data.from_pandas), decide whether to pass a pre-specified reference to CompositionalAnalysis or rely on scCODA's automatic reference selection. Pre-specify a reference when prior biology or FACS validation anchors a stable cell type — e.g., CD16+ monocytes as reference in the supercentenarian PBMC analysis, which let scCODA flag B cells as the sole credible decrease at FDR 0.2. Use automatic selection when no trusted reference exists, as in the SCP259 ulcerative colitis reanalysis of the epithelium (16 cell types) and lamina propria (37 cell types). The model uses a hierarchical Dirichlet-multinomial likelihood with a Logit-normal spike-and-slab prior and log link; inference via sample_hmc() (TensorFlow backend) yields effects whose credibility must be read relative to the chosen reference in the summary()/credible_effects() Final Parameter column.

## Related tools

- **scCODA** (Provides CompositionalAnalysis model with pre-specified or automatic reference selection, sample_hmc() inference, and summary()/credible_effects() for reading effects relative to the reference) — https://github.com/theislab/scCODA
- **scanpy** (Ecosystem for generating cell-type annotations and aggregated count data that feed from_pandas/from_scanpy compositional dataset construction)
- **TensorFlow** (Backend (tensorflow, tensorflow-probability) powering the HMC sampling used after reference selection)

## Examples

```
import sccoda; model = sccoda.model.other_models.CompositionalAnalysis(data, formula='age_group', reference_cell_type='CD16+ monocytes'); results = model.sample_hmc(); results.set_fdr(target_fdr=0.2); results.summary()
```

## Evaluation signals

- Credible-effects table lists B cells as the only cell type with non-zero Final Parameter relative to CD16+ monocytes in the supercentenarian PBMC run (Supplementary Data 1 validation)
- UC lamina propria run reproduces the Treg increase and epithelial/IAF increase (healthy vs inflamed) matching Supplementary Data 3
- The reference cell type itself has a zero Final Parameter and is excluded from credible effects by construction
- FDR control holds at the target level (set_fdr target_fdr=0.2) with spike-and-slab sparsity limiting credible calls
- Results written reproducibly to sccoda_pbmc_results.csv / sccoda_uc_colon_results.csv

## Limitations

- Credible changes must be interpreted relative to the selected reference, so a mis-chosen reference distorts interpretation
- scCODA requires pre-specified cell-type definitions that hinge on statistically sound and biologically meaningful clustering assignments
- scCODA does not model response variability within a condition and cannot detect heterogeneities between samples in response to treatment or donor variability
- scCODA does not model dependency structure among cell compositions beyond those induced by compositional effects

## Evidence

- [intro] auto vs pre-specified reference: "scCODA can automatically select an appropriate cell type as the reference (Methods—“Automatic reference selection”) or uses a pre-specified reference cell type11"
- [intro] model structure: "The model uses a Logit-normal spike-and-slab prior10 with a log-link function to estimate effects of binary (or continuous) covariates on cell-type proportions"
- [methods] model setup: "Using the scCODA model is easiest by generating an instance of ``ana.CompositionalAnalysis``."
- [methods] inference: "Once the model is set up, inference via HMC sampling can be performed via ``sample_hmc()``."
- [methods] result interpretation: "The ``Final Parameter`` column of the effects data frame shows the significances."
