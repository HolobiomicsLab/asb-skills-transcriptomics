---
name: single-cell-rna-seq-marker-selection
description: Use when a scRNA-seq AnnData dataset needs a small panel of marker genes
  (e.g., k=50 or fewer than 10% of genes) that maximize cell type classification accuracy,
  as in spatial transcriptomics panel design or marker benchmarking studies.
when_to_use_negative:
- Input is not cell-by-gene scRNA-seq expression with cell type labels (the method
  relies on supervised classification signal)
- The goal is differential expression per se rather than selecting a minimal k-marker
  set for cell type prediction — Scanpy DE tests or COSG target that directly
- Data is extremely high-dimensional with no ability to densify adata.X — sparse matrices
  must first be converted via adata.X.toarray()
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: markermap
  role: Core VAE-based marker selection model; provides MarkerMap, prepareData, getBenchmarker,
    train_model, new_model_metrics, split_data, benchmark, plot_benchmarks
  repo: https://github.com/Computational-Morphogenomics-Group/MarkerMap
- name: scanpy
  role: Reads .h5ad files into AnnData objects (sc.read_h5ad) and handles the gene
    expression data
- name: anndata
  role: Underlying AnnData container for adata.X expression matrix and adata.obs labels
- name: numpy
  role: Array handling, e.g., concatenating train and validation indices with np.concatenate
- name: persist
  role: Baseline marker selection method benchmarked against MarkerMap; must be installed
    separately from its GitHub repo
  repo: https://github.com/iancovert/persist/
- name: pip
  role: Installation route for the markermap package (`pip install markermap`)
  repo: https://pypi.org/project/markermap/
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1038/s41540-024-00339-3
    title: 'MarkerMap: nonlinear marker selection for single-cell studies'
  source_content_hash: sha256:ae54a1ed0fb76f7d687022b968d9e44eeaf2c77194e0d3a81f8fd8466c8283e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Computational-Morphogenomics-Group/MarkerMap
  tools:
  - anndata
  - markermap
  - matplotlib
  - numpy
  - persist
  - pip
  - scanpy
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# single-cell-rna-seq-marker-selection

## Summary

Selects minimal, cell-type-informative gene marker sets from scRNA-seq data using MarkerMap, a generative VAE-based model with supervised/mixed/unsupervised modes controlled by loss_tradeoff, and benchmarks the resulting k markers against baselines (LassoNet, SMaSH, COSG, PERSIST, RankCorr, RandomBaseline) via nearest-neighbor and Random Forest classification accuracy and weighted F1.

## When to use

Use when a scRNA-seq AnnData dataset needs a small panel of marker genes (e.g., k=50 or fewer than 10% of genes) that maximize cell type classification accuracy, as in spatial transcriptomics panel design or marker benchmarking studies.

## When NOT to use

- Input is not cell-by-gene scRNA-seq expression with cell type labels (the method relies on supervised classification signal)
- The goal is differential expression per se rather than selecting a minimal k-marker set for cell type prediction — Scanpy DE tests or COSG target that directly
- Data is extremely high-dimensional with no ability to densify adata.X — sparse matrices must first be converted via adata.X.toarray()

## Inputs

- AnnData object (adata.X gene expression matrix, dense; labels in adata.obs['annotation'] or adata.obs['names'], e.g., .h5ad files like data/cite_seq/CITEseq.h5ad)
- Number of markers k (e.g., 50) and loss_tradeoff (0, 0.5, or 1.0)
- Group-by label column name (group_by variable)

## Outputs

- Selected marker gene indices (supervised_marker_map.markers())
- Misclassification rate, weighted-average F1-score, and confusion matrix from new_model_metrics
- Benchmark results: averaged accuracy and weighted-F1 per model per dataset (benchmark_results.csv) or accuracy-vs-k curves from plot_benchmarks

## How to apply

Load the dataset (e.g., CITEseq.h5ad, Zeisel, Mouse Brain, Paul, SSv4) with scanpy into an AnnData object, converting sparse adata.X to dense with adata.X.toarray() and storing cell labels in adata.obs['annotation']. Split into 70%/10%/20% train/validation/test with split_data, then build train/val dataloaders via MarkerMap.prepareData with a batch_size such as 64. Train a MarkerMap VAE with z_size=16, hidden_layer_size roughly 10% of input dimension (e.g., 64 for the 500-gene CITE-seq data), and choose k markers and loss_tradeoff (0 = supervised, 0.5 = mixed, 1.0 = unsupervised). Evaluate with new_model_metrics, which trains a RandomForestClassifier on the training data restricted to the k selected markers and reports misclassification rate and weighted F1 on the held-out test set. For benchmarking, wrap each method with getBenchmarker and run the benchmark function num_times=10 over random splits (benchmark='k' over ranges like [10, 25, 50] for k sweeps), then visualize with plot_benchmarks(..., mode='accuracy'), including RandomBaseline as a lower-reference control.

## Related tools

- **markermap** (Core VAE-based marker selection model; provides MarkerMap, prepareData, getBenchmarker, train_model, new_model_metrics, split_data, benchmark, plot_benchmarks) — https://github.com/Computational-Morphogenomics-Group/MarkerMap
- **scanpy** (Reads .h5ad files into AnnData objects (sc.read_h5ad) and handles the gene expression data)
- **anndata** (Underlying AnnData container for adata.X expression matrix and adata.obs labels)
- **numpy** (Array handling, e.g., concatenating train and validation indices with np.concatenate)
- **persist** (Baseline marker selection method benchmarked against MarkerMap; must be installed separately from its GitHub repo) — https://github.com/iancovert/persist/
- **pip** (Installation route for the markermap package (`pip install markermap`)) — https://pypi.org/project/markermap/

## Examples

```
supervised_marker_map = MarkerMap(adata.X.shape[1], 64, 16, len(adata.obs[group_by].unique()), 50, loss_tradeoff=0); train_model(supervised_marker_map, train_dataloader, val_dataloader)
```

## Evaluation signals

- benchmark_results.csv contains an accuracy and a weighted-F1 entry for every model-dataset combination
- Reported MarkerMap accuracy/F1 values reproduce the paper's Results tables within normal run-to-run variation across the 10 random 70/10/20 splits
- RandomBaseline always performs at or below all other models, serving as a lower-reference control
- Accuracy-vs-k curves show MarkerMap variants beating RandomBaseline, with the advantage most pronounced when fewer than 10% of genes are selected
- new_model_metrics returns a valid misclassification rate, test_rep['weighted avg']['f1-score'], and a confusion matrix plottable with plot_confusion_matrix over adata.obs[group_by].unique()

## Limitations

- hidden_layer_size is dataset-dependent (rule of thumb ~10% of input dimension; e.g., 64 for 500-gene CITE-seq), so hyperparameters must be adjusted per dataset
- On Google Colab the runtime may need restarting after installation because scanpy installs a newer matplotlib than the default
- Smashpy comparison requires an older tensorflow (==2.5.0), causing installation conflicts, so Smashpy is not required and not installed by default
- Only MacOS installation is documented; Windows support is 'Coming soon!'
- Markers are learned on 70/10/20 random splits, so single-split results vary; averaging over num_times=10 runs is needed for stable comparison

## Evidence

- [methods] The new_model_metrics function trains a simple model such as a RandomForestClassifer on the training data restricted to the k markers, and then evaluates it on the testing data.: "The new_model_metrics function trains a simple model such as a RandomForestClassifer on the training data restricted to the k markers"
- [readme] In general, we have 3 levels, 0 (supervised only), 0.5 (mixed supervised-unsupervised) and 1 (unsupervised only).: "we have 3 levels, 0 (supervised only), 0.5 (mixed supervised-unsupervised) and 1 (unsupervised only)"
- [methods] Data is handled with numpy and with scanpy which is used for many computational biology datasets.: "Data is handled with numpy and with scanpy which is used for many computational biology datasets."
- [readme] We then split the data into training, validation, and test sets with a 70%, 10%, 20% split.: "We then split the data into training, validation, and test sets with a 70%, 10%, 20% split."
- [methods] This model selects k markers at random, and it is always a good idea to include this one: "This model selects k markers at random, and it is always a good idea to include this one"
