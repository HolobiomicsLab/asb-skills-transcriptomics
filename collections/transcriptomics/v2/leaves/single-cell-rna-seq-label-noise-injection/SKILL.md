---
name: single-cell-rna-seq-label-noise-injection
description: Use when you must quantify sensitivity of marker gene selection or cell-type
  classifiers to mislabeled training cells in scRNA-seq data (e.g. .h5ad AnnData with
  a group_by annotation column), at controlled corruption percentages for k selected
  markers.
when_to_use_negative:
- You need to assess robustness of a fully unsupervised clustering or marker method
  to label noise — unsupervised MarkerMap does not depend on input labels, so the
  corruption has no effect and the experiment is uninformative for it beyond confirming
  a flat curve
- Your data lack cell-type annotations (no group_by labels in adata.obs); uniform
  label corruption requires labels to corrupt
- You are evaluating marker stability with respect to sample size or k rather than
  annotation quality — use the standard benchmark over a range of k values instead
  of label_error
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: markermap
  role: Provides the label_error benchmark function, getBenchmarker, split_data, new_model_metrics,
    and plot_benchmarks used to inject noise and evaluate marker robustness
  repo: https://github.com/Computational-Morphogenomics-Group/MarkerMap
- name: scanpy
  role: Reads the .h5ad datasets into AnnData objects whose X and obs[group_by] supply
    expression and labels for corruption
- name: numpy
  role: Concatenates train/val indices (np.concatenate) and handles array data during
    splitting and evaluation
- name: matplotlib
  role: Renders the accuracy degradation curves saved via plot_benchmarks
- name: anndata
  role: Underlying AnnData structure for gene expression (adata.X) and cell labels
    used by the benchmark
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41540-024-00339-3
    title: 'MarkerMap: nonlinear marker selection for single-cell studies'
  source_content_hash: sha256:f31640a8c8afbd667e5c03f902b957b5136e25d496ba222885fad78c5918a06e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Computational-Morphogenomics-Group/MarkerMap
  tools:
  - anndata
  - markermap
  - matplotlib
  - numpy
  - scanpy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# single-cell-rna-seq-label-noise-injection

## Summary

A benchmarking procedure that injects uniformly random label noise into single-cell RNA-seq training labels (label_error benchmark) to measure how robustly marker selection methods (MarkerMap variants, LassoNet-type baselines, RankCorr, COSG, RandomBaseline) select informative gene sets as annotation quality degrades.

## When to use

Use when you must quantify sensitivity of marker gene selection or cell-type classifiers to mislabeled training cells in scRNA-seq data (e.g. .h5ad AnnData with a group_by annotation column), at controlled corruption percentages for k selected markers.

## When NOT to use

- You need to assess robustness of a fully unsupervised clustering or marker method to label noise — unsupervised MarkerMap does not depend on input labels, so the corruption has no effect and the experiment is uninformative for it beyond confirming a flat curve
- Your data lack cell-type annotations (no group_by labels in adata.obs); uniform label corruption requires labels to corrupt
- You are evaluating marker stability with respect to sample size or k rather than annotation quality — use the standard benchmark over a range of k values instead of label_error

## Inputs

- Single-cell RNA-seq datasets as scanpy AnnData objects (.h5ad, e.g. data/cite_seq/CITEseq.h5ad) with cell-type labels in adata.obs['names'] mapped to adata.obs['annotation'] (group_by)
- Benchmark model set from getBenchmarker (supervised/mixed/unsupervised MarkerMap with loss_tradeoff 0, 0.5, 1.0; LassoNet-type baselines; RandomBaseline)
- Range of label corruption percentages for the label_error benchmark
- Number of markers k (e.g. k=50), z_size=16, hidden_layer_size (e.g. 64 for the 500-column CITE-seq data), batch_size=64

## Outputs

- Misclassification rate and weighted f1-score per method per corruption level (from new_model_metrics on the 20% test set)
- Accuracy degradation curves across label-noise percentages saved as label_error_benchmark_results.png (via plot_benchmarks)
- Per-method confusion matrices (plot_confusion_matrix) if computed for a fixed corruption level

## How to apply

Load each single-cell dataset (Zeisel, CITE-seq via CITEseq.h5ad, Mouse Brain, Paul) with the markermap loaders get_zeisel, get_citeseq, get_mouse_brain, and get_paul so scanpy returns an AnnData object with labels in adata.obs['annotation']. Define the model set with getBenchmarker for supervised, mixed, and unsupervised MarkerMap (loss_tradeoff 0, 0.5, 1.0), LassoNet-type baselines, and the RandomBaseline control, parameterized by k=50. Split data 70%/10%/20% with split_data, then run the benchmark function with benchmark='label_error' over a range of corruption percentages twice: once corrupting labels across the full training + validation set (affecting both marker selection and the evaluation classifier) and once restricting noisy labels to the marker-selection stage only. The label_error procedure picks that percent of points in the training + validation set and sets their label to a random label; at each corruption level, evaluate markers with new_model_metrics, which trains a RandomForestClassifier on the k selected markers and reports misclassification rate and weighted f1-score on the held-out 20% test set. Plot accuracy degradation curves with plot_benchmarks and save label_error_benchmark_results.png; accept the run if supervised MarkerMap and LassoNet degrade only mildly with noise, naive supervised baselines degrade sharply, and unsupervised MarkerMap remains flat because it does not depend on input labels.

## Related tools

- **markermap** (Provides the label_error benchmark function, getBenchmarker, split_data, new_model_metrics, and plot_benchmarks used to inject noise and evaluate marker robustness) — https://github.com/Computational-Morphogenomics-Group/MarkerMap
- **scanpy** (Reads the .h5ad datasets into AnnData objects whose X and obs[group_by] supply expression and labels for corruption)
- **numpy** (Concatenates train/val indices (np.concatenate) and handles array data during splitting and evaluation)
- **matplotlib** (Renders the accuracy degradation curves saved via plot_benchmarks)
- **anndata** (Underlying AnnData structure for gene expression (adata.X) and cell labels used by the benchmark)

## Examples

```
results, benchmark_label, benchmark_range = benchmark(models, n_runs, adata[train_val_indices].X, adata[train_val_indices].obs[group_by], benchmark='label_error', benchmark_range=[0, 0.1, 0.2, 0.3, 0.4], k=50)
```

## Evaluation signals

- Top supervised methods (supervised MarkerMap, LassoNet) show only mild accuracy degradation as label corruption percentage increases
- Naive supervised baselines degrade sharply with increasing label noise while unsupervised MarkerMap remains flat across all corruption levels
- Restricting noise to the marker-selection stage only should produce a milder degradation curve than corrupting train + validation labels used by both selection and the evaluation classifier
- RandomBaseline control stays near chance-level misclassification rate and weighted f1-score at all corruption levels
- The run is accepted only if the qualitative trend matches the paper's label-error degradation figure (plot saved as label_error_benchmark_results.png)

## Limitations

- Noise is uniformly random over labels — the benchmark does not cover structured or systematic mislabeling regimes
- Unsupervised MarkerMap is unaffected by label noise by construction, so its flat curve verifies independence rather than providing a robustness comparison
- MarkerMap training with max_epochs limits may leave models under-converged; Smashpy comparisons require an older tensorflow (==2.5.0) that causes installation conflicts and is not required

## Evidence

- [methods] label_error: Given a range of percentages, pick that percent of points in the training + validation set and set their label to a random label: "label_error: Given a range of percentages, pick that percent of points in the training + validation set and set their label to a random label"
- [readme] The new_model_metrics function trains a simple model such as a RandomForestClassifer on the training data restricted to the k markers, and then evaluates it on the testing data.: "The new_model_metrics function trains a simple model such as a RandomForestClassifer on the training data restricted to the k markers, and then evaluates it on the testing data."
- [readme] We then split the data into training, validation, and test sets with a 70%, 10%, 20% split.: "We then split the data into training, validation, and test sets with a 70%, 10%, 20% split."
- [readme] In general, we have 3 levels, 0 (supervised only), 0.5 (mixed supervised-unsupervised) and 1 (unsupervised only).: "In general, we have 3 levels, 0 (supervised only), 0.5 (mixed supervised-unsupervised) and 1 (unsupervised only)."
- [methods] This model selects k markers at random, and it is always a good idea to include this one: "This model selects k markers at random, and it is always a good idea to include this one"
