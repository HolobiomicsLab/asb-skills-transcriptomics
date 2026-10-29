---
name: convex-supervised-unsupervised-loss-balancing
description: Use when training MarkerMap on single-cell (e.g. CITE-seq .h5ad) data
  and you must choose between supervised, unsupervised, or joint marker selection
  within one differentiable-sampling VAE model.
when_to_use_negative:
- When you need markers from a non-neural baseline first — RankCorr, PERSIST, COSG,
  or RandomBenchmark may be more appropriate comparisons, and RandomBenchmark (random
  k markers) should always accompany evaluations
- When the task is plain clustering annotation without a marker-count constraint k;
  loss_tradeoff tuning presumes you want exactly k differentiably selected markers
- When labels are unreliable and reconstruction is the only goal — still valid, but
  set loss_tradeoff=1 and num_classes=None rather than mixing in a noisy classification
  term without benchmarking
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: MarkerMap
  role: The VAE model whose loss_tradeoff parameter implements the convex combination
    of reconstruction and classification losses; provides getBenchmarker, prepareData,
    and markers()
  repo: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- name: scanpy
  role: Reads the CITE-seq .h5ad into an AnnData object whose .X and .obs[group_by]
    feed the dataloaders
- name: numpy
  role: Handles array data, e.g. np.concatenate of train and validation indices before
    evaluation
- name: pip
  role: Install markermap via `pip install markermap` (editable `pip install -e .`
    for development)
  repo: https://pypi.org/project/markermap/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41540-024-00339-3
    title: 'MarkerMap: nonlinear marker selection for single-cell studies'
  source_content_hash: sha256:46edfec4e5414629cf4b727e5131768b75bcf786eddc53df5e95a21d996cb8f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
  tools:
  - MarkerMap
  - numpy
  - pip
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# convex-supervised-unsupervised-loss-balancing

## Summary

MarkerMap's technique for balancing marker selection objectives by combining a VAE reconstruction loss and a supervised classification loss as a convex combination controlled by the loss_tradeoff parameter (0 = supervised only, 0.5 = mixed, 1.0 = unsupervised only).

## When to use

Use when training MarkerMap on single-cell (e.g. CITE-seq .h5ad) data and you must choose between supervised, unsupervised, or joint marker selection within one differentiable-sampling VAE model.

## When NOT to use

- When you need markers from a non-neural baseline first — RankCorr, PERSIST, COSG, or RandomBenchmark may be more appropriate comparisons, and RandomBenchmark (random k markers) should always accompany evaluations
- When the task is plain clustering annotation without a marker-count constraint k; loss_tradeoff tuning presumes you want exactly k differentiably selected markers
- When labels are unreliable and reconstruction is the only goal — still valid, but set loss_tradeoff=1 and num_classes=None rather than mixing in a noisy classification term without benchmarking

## Inputs

- AnnData object (.h5ad) with gene expression in adata.X and cell labels in adata.obs[group_by] (e.g. CITEseq.h5ad with 500 gene columns)
- Train/val dataloaders built by MarkerMap.prepareData with a group_by column and batch_size (e.g. 64)
- Hyperparameters: hidden_layer_size (e.g. 64, ~10% of input dimension), z_size (16), k (e.g. 50), loss_tradeoff in {0, 0.5, 1}

## Outputs

- A trained MarkerMap model whose markers() method returns exactly k selected marker indices
- Misclassification rate, weighted-avg f1-score, and confusion matrix from new_model_metrics when the markers are evaluated with a RandomForestClassifier
- Benchmark results over ranges of k or loss_tradeoff values via the benchmark/plot_benchmarks functions

## How to apply

Implement the MarkerMap variational auto-encoder with input_size, hidden_layer_size, z_size (conventionally 16) and num_classes, plus a Gumbel-Softmax selector layer that differentiably selects k markers exposed via a markers() method. Add a VAE reconstruction branch on the latent space and a supervised classification branch, then combine their losses with the convex combination controlled by loss_tradeoff: 0 for supervised-only, 0.5 for mixed supervised-unsupervised, and 1.0 for unsupervised-only (where num_classes can be set to None). Prepare data from an AnnData object using split_data (70%, 10%, 20% split) and MarkerMap.prepareData with a group_by label column, then call train_model on the train and val dataloaders. To compare the three regimes fairly, use MarkerMap.getBenchmarker to build supervised (loss_tradeoff=0), mixed (0.5), and unsupervised variants with a shared max_epochs training limit, and evaluate the returned markers() with new_model_metrics, which trains a RandomForestClassifier restricted to the k markers.

## Related tools

- **MarkerMap** (The VAE model whose loss_tradeoff parameter implements the convex combination of reconstruction and classification losses; provides getBenchmarker, prepareData, and markers()) — https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- **scanpy** (Reads the CITE-seq .h5ad into an AnnData object whose .X and .obs[group_by] feed the dataloaders)
- **numpy** (Handles array data, e.g. np.concatenate of train and validation indices before evaluation)
- **pip** (Install markermap via `pip install markermap` (editable `pip install -e .` for development)) — https://pypi.org/project/markermap/

## Examples

```
from markermap.vae_models import MarkerMap, train_model; supervised_marker_map = MarkerMap(adata.X.shape[1], 64, 16, len(adata.obs['annotation'].unique()), 50, loss_tradeoff=0); train_model(supervised_marker_map, train_dataloader, val_dataloader)
```

## Evaluation signals

- markers() returns exactly k=50 marker indices (matching the k constructor argument) usable by the RandomForest evaluation
- Module imports MarkerMap and train_model successfully and train_model completes on CITE-seq train/val dataloaders
- loss_tradeoff=0, 0.5, and 1.0 each train without error; unsupervised variant accepts num_classes=None
- new_model_metrics reports a finite misclassification rate and weighted-avg f1-score on the 20% test split, and the confusion matrix plots against adata.obs[group_by].unique()
- Supervised/mixed variants should beat the RandomBenchmark (random k markers) control on the benchmark over a range of k values

## Limitations

- Smashpy pins tensorflow==2.5.0 causing installation problems, so it is not required for the package but needed only for that comparison
- On Google Colab you may have to restart the runtime after installing because scanpy installs a newer version of matplotlib than the default one
- The persist package must be installed separately from https://github.com/iancovert/persist/
- Sparse adata.X must be converted to dense via adata.X = adata.X.toarray() before use
- hidden_layer_size is data-dependent (rule of thumb ~10% of input dimension; 64 for the 500-column CITE-seq data), so defaults may not transfer

## Evidence

- [readme] Loss_tradeoff is also important, see the paper for a further discussion. In general, we have 3 levels, 0 (supervised only), 0.5 (mixed supervised-unsupervised) and 1 (unsupervised only).: "Loss_tradeoff is also important, see the paper for a further discussion. In general, we have 3 levels, 0 (supervised only), 0.5 (mixed supervised-unsupervised) and 1 (unsupervised only)."
- [abstract] Here we introduce MarkerMap, a generative model for selecting minimal gene sets which are maximally informative of cell type origin and enable whole transcriptome reconstruction.: "Here we introduce MarkerMap, a generative model for selecting minimal gene sets which are maximally informative of cell type origin and enable whole transcriptome reconstruction."
- [methods] We then split the data into training, validation, and test sets with a 70%, 10%, 20% split.: "We then split the data into training, validation, and test sets with a 70%, 10%, 20% split."
- [methods] The new_model_metrics function trains a simple model such as a RandomForestClassifer on the training data restricted to the k markers, and then evaluates it on the testing data.: "The new_model_metrics function trains a simple model such as a RandomForestClassifer on the training data restricted to the k markers, and then evaluates it on the testing data."
