---
name: classifier-metric-evaluation
description: Use when comparing marker selection methods by cell type classification
  performance across a sweep of marker counts k (e.g. accuracy-vs-k curves on scRNA-seq
  data like Zeisel), especially to test performance in the low-marker regime.
when_to_use_negative:
- Input is not cell-labeled scRNA-seq data in AnnData form with a cell type label
  column — the benchmark trains a supervised classifier on labels
- You need de novo marker discovery for a single fixed k rather than comparative evaluation
  across a k sweep
- Comparison targets require installing unavailable methods (e.g. Smashpy pins tensorflow==2.5.0,
  which the package does not require by default)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: markermap
  role: Provides getBenchmarker models (supervised/mixed/unsupervised with loss_tradeoff
    0/0.5/1.0), the benchmark function over a k range, plot_benchmarks, and new_model_metrics
    for classifier-based evaluation
  repo: https://github.com/Computational-Morphogenomics-Group/MarkerMap
- name: scanpy
  role: Reads the .h5ad dataset into an AnnData object whose adata.X and obs labels
    feed the benchmark
- name: numpy
  role: Dense array handling (adata.X.toarray()) and index concatenation (np.concatenate
    of train+val indices)
- name: anndata
  role: AnnData container for expression counts and cell annotations used in the sweep
- name: matplotlib
  role: Backend used by plot_benchmarks to render accuracy-vs-k curves; note scanpy
    installs a newer version than default
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41540-024-00339-3
    title: 'MarkerMap: nonlinear marker selection for single-cell studies'
  source_content_hash: sha256:c02b4e57d11ff07dd032d915ca4a74cc4efa091a096a6f16885c6f3b12d799a6
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# classifier-metric-evaluation

## Summary

Benchmark cell-type prediction accuracy and f1-score as a function of the number of selected markers k using the markermap package's benchmark/plot_benchmarks framework, training a simple classifier (e.g. RandomForestClassifier) on data restricted to the k markers.

## When to use

Use when comparing marker selection methods by cell type classification performance across a sweep of marker counts k (e.g. accuracy-vs-k curves on scRNA-seq data like Zeisel), especially to test performance in the low-marker regime.

## When NOT to use

- Input is not cell-labeled scRNA-seq data in AnnData form with a cell type label column — the benchmark trains a supervised classifier on labels
- You need de novo marker discovery for a single fixed k rather than comparative evaluation across a k sweep
- Comparison targets require installing unavailable methods (e.g. Smashpy pins tensorflow==2.5.0, which the package does not require by default)

## Inputs

- scanpy AnnData object with dense expression matrix in adata.X and cell type labels in adata.obs['annotation'] (group_by-controlled)
- Train/validation/test index splits (70%, 10%, 20%) from markermap.utils.split_data
- k range for the benchmark sweep (e.g. [10, 25, 50])
- num_times: number of random train/test splits (e.g. 10)

## Outputs

- results: accuracy and f1 score for each model at each k
- benchmark_label and benchmark_range describing the swept axis
- Accuracy-vs-k curves via plot_benchmarks(..., mode='accuracy')
- Per-run metrics from new_model_metrics: misclass_rate, weighted-avg f1-score, confusion matrix

## How to apply

Load the dataset (e.g. Zeisel via markermap's get_zeisel loader) into a scanpy AnnData object, converting adata.X to a dense array with adata.X.toarray() if sparse. Define benchmarker models with MarkerMap.getBenchmarker (z_size=16, k overwritten by the sweep, loss_tradeoff=0 for supervised, 0.5 for mixed, 1.0 for unsupervised) plus RandomBaseline.getBenchmarker as a control. Run the benchmark function with benchmark='k' over a k range such as [10, 25, 50] and num_times set to the number of random train/test splits (10 runs for averaging). The returned results, benchmark_label, and benchmark_range report accuracy and f1 score per model at each k; visualize with plot_benchmarks(..., mode='accuracy'). Underlying evaluation uses new_model_metrics, which trains a simple classifier such as a RandomForestClassifier on the training data restricted to the k markers and evaluates it on the held-out test data.

## Related tools

- **markermap** (Provides getBenchmarker models (supervised/mixed/unsupervised with loss_tradeoff 0/0.5/1.0), the benchmark function over a k range, plot_benchmarks, and new_model_metrics for classifier-based evaluation) — https://github.com/Computational-Morphogenomics-Group/MarkerMap
- **scanpy** (Reads the .h5ad dataset into an AnnData object whose adata.X and obs labels feed the benchmark)
- **numpy** (Dense array handling (adata.X.toarray()) and index concatenation (np.concatenate of train+val indices))
- **anndata** (AnnData container for expression counts and cell annotations used in the sweep)
- **matplotlib** (Backend used by plot_benchmarks to render accuracy-vs-k curves; note scanpy installs a newer version than default)

## Examples

```
results, benchmark_label, benchmark_range = benchmark(models, num_times, adata[train_val_indices, :], adata[train_val_indices, :].obs[group_by], benchmark='k', benchmark_range=[10, 25, 50]); plot_benchmarks(results, benchmark_label, benchmark_range, mode='accuracy')
```

## Evaluation signals

- Accuracy-vs-k curves show MarkerMap variants (loss_tradeoff 0, 0.5, 1.0) outperforming RandomBaseline at every k
- The advantage over baselines is most pronounced in the low selected marker regime (fewer than 10% of genes selected as markers)
- new_model_metrics returns a finite misclass_rate and a weighted-avg f1-score, plus a confusion matrix whose class list matches adata.obs[group_by].unique()
- Averaging over num_times=10 random train/test splits reduces variance in the reported accuracy at each k
- adata.X is dense before benchmarking (sparse matrices converted via adata.X.toarray())

## Limitations

- On Google Colab, a runtime restart may be required after installing because scanpy installs a newer version of matplotlib than the default one
- Smashpy creates installation problems due to tensorflow==2.5.0 and is not required by the package, so it may be excluded from the benchmark comparison
- The RandomBaseline control selects k markers at random and should always be included as a floor; benchmarking requires the k parameter in getBenchmarker create_kwargs to be overwritten by the sweep

## Evidence

- [methods] The new_model_metrics function trains a simple model such as a RandomForestClassifer on the training data restricted to the k markers, and then evaluates it on the testing data.: "The new_model_metrics function trains a simple model such as a RandomForestClassifer on the training data restricted to the k markers, and then evaluates it on the testing data."
- [methods] To convert, merely do `adata.X = adata.X.toarray()`: "To convert, merely do `adata.X = adata.X.toarray()`"
- [methods] we run the benchmark by passing in all the models as a dictionary, the number of times to run the model, the data and labels, the type of benchmark, and the range of values we are benchmarking over: "we run the benchmark by passing in all the models as a dictionary, the number of times to run the model, the data and labels, the type of benchmark, and the range of values we are benchmarking over"
- [methods] This model selects k markers at random, and it is always a good idea to include this one: "This model selects k markers at random, and it is always a good idea to include this one"
- [readme] you may have to restart the runtime after installing because `scanpy` installs a newer version of matplotlib than the default one.: "you may have to restart the runtime after installing because `scanpy` installs a newer version of matplotlib than the default one."
