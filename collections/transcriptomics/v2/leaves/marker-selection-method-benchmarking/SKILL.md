---
name: marker-selection-method-benchmarking
description: Use when you need to compare how well different marker selection methods
  pick k gene markers from scRNA-seq AnnData (e.g., .h5ad from CITE-seq, Mouse Brain,
  Paul, Zeisel) by classification accuracy/f1 or by ℓ2/ℓ1 full-transcriptome reconstruction
  loss.
when_to_use_negative:
- The goal is a single method's marker list rather than a comparative evaluation across
  selection methods — use the simple quick-start train/evaluate flow instead of the
  benchmark harness
- Input data is not cell-by-gene scRNA-seq with cell type labels in AnnData form (e.g.,
  bulk RNA-seq or already-aggregated feature tables)
- The comparison target requires methods outside the repository's benchmark set without
  an AcmeWrapper/benchmarkerFunctional implementation
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MarkerMap
  role: Provides the benchmarked marker selection models (supervised/mixed/unsupervised)
    and the benchmarking utilities (getBenchmarker, benchmark, split_data, new_model_metrics,
    plot_benchmarks)
  repo: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- name: RandomBaseline
  role: Control method that selects k markers at random, included in every benchmark
    as a floor baseline
  repo: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- name: scanpy
  role: Reads datasets into AnnData objects and provides the highly-variable-genes
    baseline for unsupervised marker selection
  repo: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- name: persist
  role: Deep-learning gene selection baseline (unsupervised PERSIST) benchmarked against
    MarkerMap; must be installed separately
  repo: https://github.com/iancovert/persist/
- name: numpy
  role: Array handling for dense gene-count matrices and index concatenation for train/validation/test
    splits
provenance:
  source_task_ids:
  - task_005
  - task_008
  source_papers:
  - doi: 10.1038/s41540-024-00339-3
    title: 'MarkerMap: nonlinear marker selection for single-cell studies'
  source_content_hash: sha256:e775e7c4dc0249c10d64f10eca3836166b86b19e6084b01ab6b002b8680d0d48
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
  tools:
  - MarkerMap
  - RandomBaseline
  - matplotlib
  - numpy
  - persist
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# marker-selection-method-benchmarking

## Summary

Benchmark marker selection methods (supervised/mixed/unsupervised MarkerMap, RandomBaseline, Scanpy HVGs, PERSIST, RankCorr, COSG) on single-cell RNA-seq data using the markermap package's getBenchmarker/benchmark/plot_benchmarks utilities, evaluating selected k markers by classifier performance or transcriptome reconstruction loss.

## When to use

Use when you need to compare how well different marker selection methods pick k gene markers from scRNA-seq AnnData (e.g., .h5ad from CITE-seq, Mouse Brain, Paul, Zeisel) by classification accuracy/f1 or by ℓ2/ℓ1 full-transcriptome reconstruction loss.

## When NOT to use

- The goal is a single method's marker list rather than a comparative evaluation across selection methods — use the simple quick-start train/evaluate flow instead of the benchmark harness
- Input data is not cell-by-gene scRNA-seq with cell type labels in AnnData form (e.g., bulk RNA-seq or already-aggregated feature tables)
- The comparison target requires methods outside the repository's benchmark set without an AcmeWrapper/benchmarkerFunctional implementation

## Inputs

- Single-cell RNA-seq AnnData objects (.h5ad, e.g., data/cite_seq/CITEseq.h5ad) with gene counts in adata.X and cell labels in adata.obs['annotation'] (group_by)
- Model dictionary from getBenchmarker for each method (MarkerMap variants, RandomBaseline, Scanpy HVGs, PERSIST)
- Parameters: k (number of markers, e.g., 50), loss_tradeoff (0, 0.5, 1.0), z_size=16, hidden_layer_size (~10% of input dim, e.g., 64 for CITE-seq's 500 genes), batch_size=64, max_epochs

## Outputs

- benchmark results object mapping each method to metrics across the benchmark range (k values or label-error percentages)
- Benchmark plots produced by plot_benchmarks (e.g., label_error_benchmark_results.png)
- Per-run metrics from new_model_metrics: misclassification rate, weighted f1-score (test_rep['weighted avg']['f1-score']), confusion matrix
- Averaged ℓ2/ℓ1 reconstruction losses per dataset-method combination (e.g., reconstruction_losses.csv)

## How to apply

Load each dataset via the markermap loaders (get_citeseq, get_mouse_brain, get_paul, get_zeisel) into scanpy AnnData objects, converting sparse matrices to dense with adata.X = adata.X.toarray(). Split data 70%/10%/20% into train/validation/test with split_data. Build the model set with each model's getBenchmarker function (supervised, mixed, and unsupervised MarkerMap via loss_tradeoff = 0, 0.5, 1.0, plus baselines like RandomBaseline that selects k markers at random), parameterized by k (e.g., k=50) and max_epochs. Run the benchmark by passing all models as a dictionary, the number of runs, the data and labels, the benchmark type (e.g., label_error over corruption percentages), and the range of k or corruption values to vary. Evaluate each marker set with new_model_metrics, which trains a simple classifier such as a RandomForestClassifier on the training data restricted to the k markers and measures misclassification rate and weighted f1-score on the held-out test set; for reconstruction benchmarks, fit a linear regression from the 50 markers to the full transcriptome and report average ℓ2 and ℓ1 loss over 10 runs. Plot results with plot_benchmarks (mode='accuracy' or f1) and save the resulting figure/CSV for comparison against published tables.

## Related tools

- **MarkerMap** (Provides the benchmarked marker selection models (supervised/mixed/unsupervised) and the benchmarking utilities (getBenchmarker, benchmark, split_data, new_model_metrics, plot_benchmarks)) — https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- **RandomBaseline** (Control method that selects k markers at random, included in every benchmark as a floor baseline) — https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- **scanpy** (Reads datasets into AnnData objects and provides the highly-variable-genes baseline for unsupervised marker selection) — https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- **persist** (Deep-learning gene selection baseline (unsupervised PERSIST) benchmarked against MarkerMap; must be installed separately) — https://github.com/iancovert/persist/
- **numpy** (Array handling for dense gene-count matrices and index concatenation for train/validation/test splits)

## Examples

```
from markermap.utils import split_data; train_indices, val_indices, test_indices = split_data(adata.X, adata.obs[group_by], [0.7, 0.1, 0.2])
```

## Evaluation signals

- Splits are 70%/10%/20% (train/validation/test) via split_data, and models are evaluated only on the held-out 20% test set
- RandomBaseline accuracy/loss should sit at or below the top supervised methods (supervised MarkerMap, LassoNet-type baselines) for all k values
- Unsupervised MarkerMap (loss_tradeoff=1.0) is expected to remain flat in label-error benchmarks, since it does not depend on input labels
- Reconstruction losses averaged over 10 runs should be comparable to Table 2 values, e.g., unsupervised MarkerMap (0.711, 0.657) ℓ2/ℓ1 on CITE-seq and (0.854, 0.721) on SSv4 with k=50
- Output plots/CSV must contain a complete grid of dataset × method × benchmark-range entries with no missing runs

## Limitations

- Smashpy comparison requires an older tensorflow (==2.5.0) that creates installation problems, so it is not required to install it to use the package
- On Google Colab, the runtime may need restarting after installation because scanpy installs a newer version of matplotlib than the default one
- Label-error corruption applies to a percentage of the training + validation set, so evaluation classifier performance depends on how the corruption scope is defined (marker-selection stage only vs. both stages)
- Benchmarks require multiple repeated runs (e.g., 10 splits) to be comparable to published averaged results, so single-run results may deviate

## Evidence

- [readme] The new_model_metrics function trains a simple model such as a RandomForestClassifer on the training data restricted to the k markers, and then evaluates it on the testing data.: "The new_model_metrics function trains a simple model such as a RandomForestClassifer on the training data restricted to the k markers, and then evaluates it on the testing data."
- [methods] Each model in this repository comes with a function `getBenchmarker` where we specify all the parameters used for constructing the model: "Each model in this repository comes with a function `getBenchmarker` where we specify all the parameters used for constructing the model"
- [readme] We then split the data into training, validation, and test sets with a 70%, 10%, 20% split.: "We then split the data into training, validation, and test sets with a 70%, 10%, 20% split."
- [readme] we run the benchmark by passing in all the models as a dictionary, the number of times to run the model, the data and labels, the type of benchmark, and the range of values we are benchmarking over: "we run the benchmark by passing in all the models as a dictionary, the number of times to run the model, the data and labels, the type of benchmark, and the range of values we are benchmarking over"
- [readme] This model selects k markers at random, and it is always a good idea to include this one: "This model selects k markers at random, and it is always a good idea to include this one"
