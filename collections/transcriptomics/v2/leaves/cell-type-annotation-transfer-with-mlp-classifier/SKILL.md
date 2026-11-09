---
name: cell-type-annotation-transfer-with-mlp-classifier
description: Use when semi-profiled single-cell data (e.g. scSemiProfiler output for
  a COVID-19 cohort with Healthy/Asymptomatic/Mild/Moderate/Severe/Critical conditions)
  is unlabeled, and annotated representative single-cell reference data exists to
  train a classifier for label transfer.
when_to_use_negative:
- No annotated single-cell reference is available — scSemiProfiler's deconvolution
  and classifier-based annotation require a single-cell reference sample
- Input is already fully annotated bulk deconvolution output where only cell-type-level
  resolution is needed (conventional methods such as CIBERSORTx or DWLS suffice)
- Data modalities beyond scRNA-seq (e.g. proteomics) are involved — extension to other
  modalities is not yet implemented
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scSemiProfiler
  role: Generates the semi-profiled single-cell data to annotate and provides the
    cell-type-proportion utility for computing annotated proportions per sample/condition
  repo: https://github.com/mcgilldinglab/scSemiProfiler
- name: scikit-learn
  role: Supplies the MLP classifier trained on annotated representative single-cell
    data and applied to the semi-profiled cells; also used for PCA dimensionality
    reduction in preprocessing
- name: SciPy
  role: scipy.stats.pearsonr computes the per-condition Pearson correlation between
    semi-profiled and real-profiled cell-type proportion vectors
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:18d2c1784e5ec247d94d475e7d22e08a90aa8c15132ebcfa06a6ff02601a215c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scSemiProfiler
  tools:
  - SciPy
  - scSemiProfiler
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cell-type-annotation-transfer-with-mlp-classifier

## Summary

Transfers cell-type labels from annotated representative single-cell data to semi-profiled (AI-inferred) single-cell data by training a scikit-learn MLP classifier on the reference annotations, enabling per-sample and per-condition cell-type proportion comparison between semi-profiled and real-profiled cohorts (e.g. COVID-19 severity conditions).

## When to use

Use when semi-profiled single-cell data (e.g. scSemiProfiler output for a COVID-19 cohort with Healthy/Asymptomatic/Mild/Moderate/Severe/Critical conditions) is unlabeled, and annotated representative single-cell reference data exists to train a classifier for label transfer.

## When NOT to use

- No annotated single-cell reference is available — scSemiProfiler's deconvolution and classifier-based annotation require a single-cell reference sample
- Input is already fully annotated bulk deconvolution output where only cell-type-level resolution is needed (conventional methods such as CIBERSORTx or DWLS suffice)
- Data modalities beyond scRNA-seq (e.g. proteomics) are involved — extension to other modalities is not yet implemented

## Inputs

- Annotated representative single-cell dataset (cell types per cell) from selected representatives
- Semi-profiled single-cell data produced by the scSemiProfiler deep generative (VAE-GAN) in silico inference step
- Disease condition metadata per sample (e.g. COVID-19 severity: Healthy, Asymptomatic, Mild, Moderate, Severe, Critical)

## Outputs

- Cell-type labels for every semi-profiled cell
- Per-sample and per-condition cell-type proportion table
- Per-condition Pearson correlation values between semi-profiled and real-profiled proportions (e.g. similarity_metrics.csv with 6 condition rows)

## How to apply

Train a scikit-learn MLP (multi-layer perceptron) classifier on the annotated representative single-cell data generated during the semi-profiling process (representatives chosen by active learning or global mode Silhouette-score clustering). Apply the trained classifier to predict cell types for each cell in the semi-profiled samples. With the predicted annotations, compute cell-type proportions per sample and per condition using the scSemiProfiler cell-type-proportion utility, then merge semi-profiled and real-profiled data on severity condition and calculate Pearson correlation coefficients between proportion vectors with scipy.stats.pearsonr. Write the per-condition metrics (e.g. cell-type proportion and interferon pathway Pearson correlations) to similarity_metrics.csv with one row per condition; successful transfer yields correlations exceeding 0.95 (Healthy 0.987, Asymptomatic 0.970, Mild 0.996, Moderate 0.992, Severe 0.978, Critical 0.989 in the COVID-19 validation).

## Related tools

- **scSemiProfiler** (Generates the semi-profiled single-cell data to annotate and provides the cell-type-proportion utility for computing annotated proportions per sample/condition) — https://github.com/mcgilldinglab/scSemiProfiler
- **scikit-learn** (Supplies the MLP classifier trained on annotated representative single-cell data and applied to the semi-profiled cells; also used for PCA dimensionality reduction in preprocessing)
- **SciPy** (scipy.stats.pearsonr computes the per-condition Pearson correlation between semi-profiled and real-profiled cell-type proportion vectors)

## Examples

```
from sklearn.neural_network import MLPClassifier; clf = MLPClassifier(); clf.fit(representative_cells, representative_labels); preds = clf.predict(semiprofiled_cells)
```

## Evaluation signals

- similarity_metrics.csv contains exactly 6 condition rows with columns for cell-type proportion Pearson and IFN pathway Pearson
- All per-condition cell-type proportion Pearson correlations exceed 0.95 (reference values: 0.987 Healthy, 0.970 Asymptomatic, 0.996 Mild, 0.992 Moderate, 0.978 Severe, 0.989 Critical)
- IFN pathway Pearson correlation falls within 0.75–0.85
- Stacked-bar cell-type proportions across disease conditions are near-identical between the semi-profiled and real-profiled cohorts

## Limitations

- Requires a single-cell reference to perform single-cell level deconvolution/annotation transfer
- Only de novo annotation or a classifier trained on reference data is supported; no versions stated for scSemiProfiler or dependencies
- Extension to other modalities (proteomics) requires algorithm adjustments and has not been benchmarked

## Evidence

- [readme] classifier-based annotation of semi-profiled cells: [evidence span withheld — anchor status: external_doc]
- [discussion] fidelity of cell type proportions: "The striking resemblance between semi-profiled and real-profiled data in facets like UMAP visualization and cell type proportions"
- [discussion] reference requirement for deconvolution: "The tool’s deep generative model enables “single-cell level deconvolution” across various studies, provided there is a single-cell reference."
- [readme] semi-profiled vs ground-truth comparison: [evidence span withheld — anchor status: external_doc]
