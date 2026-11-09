---
name: cell-type-classification-evaluation
description: Use when you have a set of k selected marker genes on a labeled scRNA-seq
  AnnData object and need to quantify how well those markers discriminate cell types
  via classifier performance.
when_to_use_negative:
- You need to evaluate whole-transcriptome reconstruction quality rather than cell-type
  discriminative power of a marker subset — MarkerMap also supports reconstruction,
  but this skill only covers classification metrics.
- Your data has no cell type labels — supervised evaluation via new_model_metrics
  requires adata.obs[group_by] annotations.
- You are comparing marker selection methods at very small k where a simple classifier
  on k markers is not the paper's evaluation regime (the benchmark is defined over
  a range of k values, e.g. k=50).
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: markermap
  role: Provides new_model_metrics, split_data, benchmark, plot_benchmarks, and getBenchmarker
    used to run the classification benchmark on selected markers
  repo: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- name: scanpy
  role: Reads .h5ad datasets into AnnData objects for splitting and marker evaluation
- name: anndata
  role: AnnData container holding adata.X expression and adata.obs['annotation'] labels
- name: numpy
  role: Dense array handling and index concatenation (np.concatenate of train/val
    indices)
- name: persist
  role: One of the benchmarked marker selection baselines evaluated with the classifier
    protocol
  repo: https://github.com/iancovert/persist/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41540-024-00339-3
    title: 'MarkerMap: nonlinear marker selection for single-cell studies'
  source_content_hash: sha256:1fdb195b4727d9ba16ced1ab9482766e9d4028b8feeb5fef3959026ca04cc9ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
  tools:
  - anndata
  - markermap
  - numpy
  - persist
  - pip
  - scanpy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cell-type-classification-evaluation

## Summary

Evaluates selected gene marker sets by training simple classifiers (Random Forest / nearest-neighbor) restricted to the k markers and measuring accuracy and weighted F1 on held-out test cells, following the MarkerMap benchmarking protocol for scRNA-seq data.

## When to use

Use when you have a set of k selected marker genes on a labeled scRNA-seq AnnData object and need to quantify how well those markers discriminate cell types via classifier performance.

## When NOT to use

- You need to evaluate whole-transcriptome reconstruction quality rather than cell-type discriminative power of a marker subset — MarkerMap also supports reconstruction, but this skill only covers classification metrics.
- Your data has no cell type labels — supervised evaluation via new_model_metrics requires adata.obs[group_by] annotations.
- You are comparing marker selection methods at very small k where a simple classifier on k markers is not the paper's evaluation regime (the benchmark is defined over a range of k values, e.g. k=50).

## Inputs

- AnnData object (.h5ad) with gene expression in adata.X and cell type labels in adata.obs['annotation']
- Dense gene expression array (sparse adata.X converted via adata.X.toarray())
- Selected marker gene indices per model (k markers, e.g. k=50)
- Train/val/test index splits (70%, 10%, 20%)

## Outputs

- Misclassification rate and weighted F1 score per model per run (from new_model_metrics)
- Confusion matrix (plot_confusion_matrix over adata.obs[group_by].unique())
- Averaged accuracy and weighted-F1 benchmark tables across 10 runs, e.g. saved to benchmark_results.csv

## How to apply

Load each dataset (CITE-seq via data/cite_seq/CITEseq.h5ad, plus Zeisel, Mouse Brain, Paul, SSv4 loaders) with scanpy into AnnData and convert sparse adata.X to dense with adata.X.toarray(). Split cells into 70%/10%/20% train/validation/test partitions with split_data, wrapping each marker-selection model (MarkerMap variants with loss_tradeoff 0/0.5/1.0, LassoNet, Concrete VAE, Global-Gumbel VAE, SMaSH, RankCorr, Scanpy DE, COSG, PERSIST, RandomBaseline) via getBenchmarker with k=50. Run the benchmark with num_times=10 so every model is trained on 10 random splits; for each run use new_model_metrics, which trains a RandomForestClassifier on the training data restricted to the k markers and evaluates misclassification rate and f1-score on the test set. Average accuracy and weighted-F1 ('weighted avg' f1-score from the classification report) over the 10 runs per model-dataset pair and assemble the Results-style tables, using RandomBaseline as a lower-reference control.

## Related tools

- **markermap** (Provides new_model_metrics, split_data, benchmark, plot_benchmarks, and getBenchmarker used to run the classification benchmark on selected markers) — https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- **scanpy** (Reads .h5ad datasets into AnnData objects for splitting and marker evaluation)
- **anndata** (AnnData container holding adata.X expression and adata.obs['annotation'] labels)
- **numpy** (Dense array handling and index concatenation (np.concatenate of train/val indices))
- **persist** (One of the benchmarked marker selection baselines evaluated with the classifier protocol) — https://github.com/iancovert/persist/

## Examples

```
misclass_rate, test_rep, cm = new_model_metrics(adata[train_val_indices, :].X, adata[train_val_indices, :].obs[group_by], adata[test_indices, :].X, adata[test_indices, :].obs[group_by], markers=supervised_marker_map.markers().clone().cpu().detach().numpy())
```

## Evaluation signals

- benchmark_results.csv contains an accuracy and a weighted-F1 entry for every model-dataset combination (11+ models x 5-6 datasets)
- Reported MarkerMap accuracy/F1 values reproduce the paper's Results tables within normal run-to-run variation across the 10 random 70/10/20 splits
- RandomBaseline consistently yields the lowest or near-lowest accuracy/F1, acting as a lower-reference control
- new_model_metrics returns a finite misclass_rate and a test_rep dict containing test_rep['weighted avg']['f1-score'] for every run
- Averaged metrics are stable across num_times=10 repetitions (no single-split outliers dominating the mean)

## Limitations

- The benchmark protocol is stochastic — 10 random train/validation/test splits mean results vary run to run; single runs are not directly comparable to published tables.
- Smashpy specifies an older tensorflow (==2.5.0) causing installation problems, so it is optional and not required to run the comparison.
- On Google Colab, the runtime may need restarting after install because scanpy installs a newer matplotlib than the default.
- The classifier is deliberately simple (e.g., RandomForest) restricted to k markers; performance reflects marker quality, not a tuned classifier.

## Evidence

- [methods] The new_model_metrics function trains a simple model such as a RandomForestClassifer on the training data restricted to the k markers, and then evaluates it on the testing data.: "The new_model_metrics function trains a simple model such as a RandomForestClassifer on the training data restricted to the k markers, and then evaluates it on the testing data."
- [readme] We then split the data into training, validation, and test sets with a 70%, 10%, 20% split.: "We then split the data into training, validation, and test sets with a 70%, 10%, 20% split."
- [results] Across five scRNA-seq datasets with 50 selected markers, supervised MarkerMap performs competitively with baselines like LassoNet and SMaSH: [evidence span withheld — anchor status: not_found]
- [methods] To convert, merely do `adata.X = adata.X.toarray()`: "To convert, merely do `adata.X = adata.X.toarray()`"
- [readme] This model selects k markers at random, and it is always a good idea to include this one: "This model selects k markers at random, and it is always a good idea to include this one"
