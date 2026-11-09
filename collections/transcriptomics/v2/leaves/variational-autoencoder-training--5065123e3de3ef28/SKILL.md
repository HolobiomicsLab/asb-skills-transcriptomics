---
name: variational-autoencoder-training
description: Use when selecting a small set of gene markers (e.g. k=50) from scRNA-seq
  data in AnnData/h5ad form, needing cell-type-informative markers with whole-transcriptome
  reconstruction via a nonlinear VAE.
when_to_use_negative:
- The goal is marker selection for FISH target design with binarized expression —
  PERSIST is the purpose-built method, not raw MarkerMap training
- The task is fast differential-expression marker ranking without a generative model
  — scanpy DE tests or COSG are simpler choices
- Data cannot be held out for validation (single-group, no labels and unsupervised
  reconstruction only is not the target use case for supervised loss_tradeoff=0)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: markermap
  role: Provides the MarkerMap VAE class, train_model, prepareData, split_data, new_model_metrics
    used to train and evaluate the marker-selection model
  repo: https://pypi.org/project/markermap/
- name: scanpy
  role: Reads the h5ad data into an AnnData object consumed by MarkerMap.prepareData
- name: numpy
  role: Handles dense array conversion (adata.X.toarray()) and index concatenation
    for train/val splits
- name: persist
  role: Optional baseline dependency installed separately from https://github.com/iancovert/persist/
  repo: https://github.com/iancovert/persist/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41540-024-00339-3
    title: 'MarkerMap: nonlinear marker selection for single-cell studies'
  source_content_hash: sha256:934036ec2d07e2b526e14de73ad6edb3be1cb4a350c6a6d41ca0160e90a2991b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - markermap
  - numpy
  - persist
  - pip
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# variational-autoencoder-training

## Summary

Train a MarkerMap variational autoencoder on single-cell gene expression data to select k maximally informative gene markers while jointly reconstructing the whole transcriptome. Skill covers defining the VAE (z_size=16, hidden_layer_size, k, loss_tradeoff), preparing dataloaders, and training with a train/validation split.

## When to use

Use when selecting a small set of gene markers (e.g. k=50) from scRNA-seq data in AnnData/h5ad form, needing cell-type-informative markers with whole-transcriptome reconstruction via a nonlinear VAE.

## When NOT to use

- The goal is marker selection for FISH target design with binarized expression — PERSIST is the purpose-built method, not raw MarkerMap training
- The task is fast differential-expression marker ranking without a generative model — scanpy DE tests or COSG are simpler choices
- Data cannot be held out for validation (single-group, no labels and unsupervised reconstruction only is not the target use case for supervised loss_tradeoff=0)

## Inputs

- AnnData object from an .h5ad file (e.g. data/cite_seq/CITEseq.h5ad) with gene counts in adata.X and cell labels in adata.obs['annotation'] or adata.obs['names']
- Train/val/test index arrays from split_data with proportions [0.7, 0.1, 0.2]
- Hyperparameters: z_size=16, hidden_layer_size (e.g. 64), k (e.g. 50), batch_size=64, loss_tradeoff in {0, 0.5, 1}

## Outputs

- Trained MarkerMap VAE model exposing .markers() — a tensor of k selected gene indices
- Predictive performance metrics (misclassification rate, weighted-avg f1-score) and confusion matrix via new_model_metrics

## How to apply

Read the dataset (e.g. CITEseq.h5ad, or Zeisel/Mouse Brain/Paul/SSv4 loaders) with scanpy into an AnnData object, converting sparse adata.X to dense via adata.X.toarray(). Split cells 70/10/20 into train/validation/test with split_data, then build train_dataloader and val_dataloader with MarkerMap.prepareData. Instantiate MarkerMap with input_size = adata.X.shape[1], hidden_layer_size (~10% of input dimension, e.g. 64 for the 500-column CITE-seq data), z_size=16, num_classes from adata.obs[group_by].unique(), k=50, and choose loss_tradeoff (0 = supervised, 0.5 = mixed, 1 = unsupervised). Train with train_model(supervised_marker_map, train_dataloader, val_dataloader) and extract selected markers from supervised_marker_map.markers().

## Related tools

- **markermap** (Provides the MarkerMap VAE class, train_model, prepareData, split_data, new_model_metrics used to train and evaluate the marker-selection model) — https://pypi.org/project/markermap/
- **scanpy** (Reads the h5ad data into an AnnData object consumed by MarkerMap.prepareData)
- **numpy** (Handles dense array conversion (adata.X.toarray()) and index concatenation for train/val splits)
- **persist** (Optional baseline dependency installed separately from https://github.com/iancovert/persist/) — https://github.com/iancovert/persist/

## Examples

```
supervised_marker_map = MarkerMap(adata.X.shape[1], 64, 16, len(adata.obs['annotation'].unique()), 50, loss_tradeoff=0); train_model(supervised_marker_map, train_dataloader, val_dataloader)
```

## Evaluation signals

- new_model_metrics returns a misclassification rate and test_rep['weighted avg']['f1-score']; these should be competitive with baselines and above the RandomBaseline control
- Selected marker count from supervised_marker_map.markers() equals k (e.g. 50)
- Confusion matrix from plot_confusion_matrix shows diagonal-dominant classification across adata.obs[group_by].unique() classes
- Training uses a proper 70/10/20 split with train/val dataloaders built by MarkerMap.prepareData before train_model is invoked
- MarkerMap accuracy/F1 over 10 random splits should reproduce the paper's Results tables within normal run-to-run variation

## Limitations

- Smashpy is not installed as a dependency (it pins tensorflow==2.5.0), so comparisons against that method require manual installation
- On Google Colab, runtime restart after install may be required because scanpy installs a newer matplotlib than the default
- The Windows installation path is not yet provided; hidden_layer_size must be tuned per dataset (~10% of input dimension)

## Evidence

- [abstract] MarkerMap, a generative model for selecting minimal gene sets which are maximally informative of cell type origin and enable whole transcriptome reconstruction.: "Here we introduce MarkerMap, a generative model for selecting minimal gene sets which are maximally informative of cell type origin and enable whole transcriptome reconstruction."
- [readme] split_data 70/10/20 and prepareData dataloaders: "We then split the data into training, validation, and test sets with a 70%, 10%, 20% split. We can use MarkerMap.prepareData to construct the train_dataloader and val_dataloader"
- [readme] loss_tradeoff levels 0 / 0.5 / 1: "In general, we have 3 levels, 0 (supervised only), 0.5 (mixed supervised-unsupervised) and 1 (unsupervised only)."
- [methods] new_model_metrics trains a classifier on the k markers: "The new_model_metrics function trains a simple model such as a RandomForestClassifer on the training data restricted to the k markers, and then evaluates it on the testing data."
- [methods] dense conversion of sparse matrices: "To convert, merely do `adata.X = adata.X.toarray()`"
