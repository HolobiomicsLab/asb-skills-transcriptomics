---
name: cell-type-proportion-computation-per-condition
description: Use when comparing cell-type composition of an AI-generated (semi-profiled)
  cohort against real-profiled single-cell ground truth across defined conditions,
  e.g. COVID-19 severity states (Healthy, Asymptomatic, Mild, Moderate, Severe, Critical).
when_to_use_negative:
- You only have bulk data and no single-cell reference or annotated representative
  samples — scSemiProfiler's single-cell level deconvolution requires a single-cell
  reference to guide inference and classifier training
- The goal is generic bulk deconvolution at cell-type resolution without single-cell
  inference — deconvolution tools like CIBERSORTx, Bisque, DWLS, MuSiC, or NNLS are
  the conventional alternative
- Input data is already a feature/proportion table with no per-cell annotations —
  recompute proportions only after cell-type assignment via the trained classifier
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scSemiProfiler
  role: Provides the cell-type-proportion utility that computes per-sample and per-condition
    composition from the annotated semi-profiled and real-profiled cohorts
  repo: https://github.com/mcgilldinglab/scSemiProfiler
- name: scikit-learn
  role: Supplies the MLP classifier trained on annotated representative single-cell
    data to assign cell types to semi-profiled cells before proportion computation
- name: SciPy
  role: scipy.stats.pearsonr computes the per-condition Pearson correlation between
    semi-profiled and real-profiled proportion vectors
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:972671116081df25059006aa77bd2d13e71a846d818f1ace3b99f4f5e352f5d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scSemiProfiler
  tools:
  - SciPy
  - scSemiProfiler
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cell-type-proportion-computation-per-condition

## Summary

Computes per-sample and per-condition cell-type proportion vectors from annotated single-cell data (semi-profiled and real-profiled) using the scSemiProfiler cell-type-proportion utility, then quantifies cohort similarity with per-condition Pearson correlations.

## When to use

Use when comparing cell-type composition of an AI-generated (semi-profiled) cohort against real-profiled single-cell ground truth across defined conditions, e.g. COVID-19 severity states (Healthy, Asymptomatic, Mild, Moderate, Severe, Critical).

## When NOT to use

- You only have bulk data and no single-cell reference or annotated representative samples — scSemiProfiler's single-cell level deconvolution requires a single-cell reference to guide inference and classifier training
- The goal is generic bulk deconvolution at cell-type resolution without single-cell inference — deconvolution tools like CIBERSORTx, Bisque, DWLS, MuSiC, or NNLS are the conventional alternative
- Input data is already a feature/proportion table with no per-cell annotations — recompute proportions only after cell-type assignment via the trained classifier

## Inputs

- Semi-profiled single-cell cohort produced by the upstream scSemiProfiler VAE-GAN inference step
- Real-profiled ground-truth single-cell dataset (e.g. the COVID-19 cohort with 124 samples, preprocessed as detailed by the source study)
- Annotated representative single-cell data used to train the scikit-learn MLP cell-type classifier
- Condition metadata (severity: Healthy, Asymptomatic, Mild, Moderate, Severe, Critical) for sample-conditional merging

## Outputs

- Cell-type proportion vectors per sample and per condition for both cohorts
- Per-condition Pearson correlation coefficients (scipy.stats.pearsonr) between semi-profiled and real-profiled proportions
- similarity_metrics.csv with exactly 6 condition rows and columns for the cell-type proportion Pearson and the IFN pathway Pearson

## How to apply

Load the semi-profiled cohort from the upstream scSemiProfiler inference step alongside the real-profiled ground-truth single-cell dataset, merging samples on severity condition. Annotate semi-profiled cells by training a multi-layer perceptron (MLP) classifier from scikit-learn on the annotated representative single-cell data and applying it to the inferred cells. Compute cell-type proportions per sample and per condition with the scSemiProfiler cell-type-proportion utility, then use scipy.stats.pearsonr to correlate the semi-profiled vs real-profiled proportion vectors within each of the 6 severity conditions. Write all metrics to similarity_metrics.csv with exactly 6 condition rows; correctness is judged by every per-condition cell-type proportion Pearson exceeding 0.95 (observed: 0.987 Healthy, 0.970 Asymptomatic, 0.996 Mild, 0.992 Moderate, 0.978 Severe, 0.989 Critical).

## Related tools

- **scSemiProfiler** (Provides the cell-type-proportion utility that computes per-sample and per-condition composition from the annotated semi-profiled and real-profiled cohorts) — https://github.com/mcgilldinglab/scSemiProfiler
- **scikit-learn** (Supplies the MLP classifier trained on annotated representative single-cell data to assign cell types to semi-profiled cells before proportion computation)
- **SciPy** (scipy.stats.pearsonr computes the per-condition Pearson correlation between semi-profiled and real-profiled proportion vectors)

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(semi_props_per_condition, real_props_per_condition)
```

## Evaluation signals

- similarity_metrics.csv exists with exactly 6 rows (one per severity condition) and the required metric columns
- All 6 per-condition cell-type proportion Pearson correlations exceed 0.95 (expected values: 0.987, 0.970, 0.996, 0.992, 0.978, 0.989)
- Merged cohorts cover all 6 severity conditions: Healthy, Asymptomatic, Mild, Moderate, Severe, Critical
- Stacked bar plots of proportions show the semi-profiled cohort has nearly identical cell type proportions across disease conditions vs the real-profiled ground truth
- The companion IFN pathway Pearson falls within 0.75–0.85 as a cross-check of downstream similarity

## Limitations

- Requires a single-cell reference (annotated representative samples); the tool's deep generative model enables single-cell level deconvolution only 'provided there is a single-cell reference'
- Annotation of inferred cells depends on classifier quality trained on representative data; de novo annotation or reference-trained classifiers are the two supported routes
- No software versions for scSemiProfiler or dependencies are stated in the discussed section, complicating reproducibility pinning

## Evidence

- [readme] Stacked bar plots indicate that the semi-profiled cohort has nearly identical cell type proportions across disease conditions compared to the real-profiled ground truth.: [evidence span withheld — anchor status: external_doc]
- [readme] This includes de novo annotation or utilizing a classifier trained on the reference data.: [evidence span withheld — anchor status: external_doc]
- [discussion] The striking resemblance between semi-profiled and real-profiled data in facets like UMAP visualization and cell type proportions ascertains the robustness of scSemiProfiler.: "The striking resemblance between semi-profiled and real-profiled data in facets like UMAP visualization and cell type proportions ascertains the robustness of scSemiProfiler."
- [discussion] The tool’s deep generative model enables “single-cell level deconvolution” across various studies, provided there is a single-cell reference.: "The tool’s deep generative model enables “single-cell level deconvolution” across various studies, provided there is a single-cell reference."
