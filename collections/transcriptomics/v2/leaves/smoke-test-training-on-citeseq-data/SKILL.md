---
name: smoke-test-training-on-citeseq-data
description: Use when implementing or reconstructing the MarkerMap marker-selection
  module and you need to verify it trains end-to-end on a real scRNA-seq/CITE-seq
  AnnData object and returns exactly k marker indices.
when_to_use_negative:
- You only need marker-selection benchmarks against other methods (RankCorr, PERSIST,
  COSG) rather than a reconstruction-fidelity smoke test of the module itself.
- The data is spatial transcriptomics or non-count expression — this smoke test targets
  the CITE-seq AnnData pipeline described in the article.
- You need a method with no training loop (e.g. RandomBaseline, which simply selects
  k markers at random).
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MarkerMap
  role: VAE-based marker selection model under test; provides prepareData classmethod,
    train_model function, and markers() accessor
  repo: https://github.com/Computational-Morphogenomics-Group/MarkerMap
- name: scanpy
  role: Reads CITEseq.h5ad into an AnnData object for the training harness
- name: numpy
  role: Concatenates train/val indices and converts selected marker indices to numpy
    for evaluation
- name: pip
  role: Installs the markermap package (pip install markermap) or editable install
    (pip install -e .) before the smoke test
  repo: https://pypi.org/project/markermap/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41540-024-00339-3
    title: 'MarkerMap: nonlinear marker selection for single-cell studies'
  source_content_hash: sha256:469fc6674c7974c6b0e5c50157bfbe644e60e54ac34692377da40c04a656932a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Computational-Morphogenomics-Group/MarkerMap
  tools:
  - MarkerMap
  - numpy
  - pip
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# smoke-test-training-on-citeseq-data

## Summary

A minimal end-to-end training harness for the MarkerMap VAE-based marker selector on the CITE-seq dataset (CITEseq.h5ad), verifying that the model instantiates, trains, and returns exactly k markers via differentiable Gumbel-Softmax sampling.

## When to use

Use when implementing or reconstructing the MarkerMap marker-selection module and you need to verify it trains end-to-end on a real scRNA-seq/CITE-seq AnnData object and returns exactly k marker indices.

## When NOT to use

- You only need marker-selection benchmarks against other methods (RankCorr, PERSIST, COSG) rather than a reconstruction-fidelity smoke test of the module itself.
- The data is spatial transcriptomics or non-count expression — this smoke test targets the CITE-seq AnnData pipeline described in the article.
- You need a method with no training loop (e.g. RandomBaseline, which simply selects k markers at random).

## Inputs

- data/cite_seq/CITEseq.h5ad (AnnData with gene expression in adata.X and labels in adata.obs['names']/adata.obs['annotation'])
- split fractions [0.7, 0.1, 0.2] for training/validation/test
- Model hyperparameters: hidden_layer_size=64, z_size=16, k=50, batch_size=64, loss_tradeoff in {0, 0.5, 1}

## Outputs

- Trained MarkerMap model instance
- markers() returning exactly k=50 marker gene indices (numpy array after .clone().cpu().detach().numpy())
- train_dataloader and val_dataloader for subsequent benchmarking

## How to apply

Load data/cite_seq/CITEseq.h5ad with scanpy (read_h5ad), converting adata.X to dense via adata.X.toarray() if sparse, and set labels in adata.obs['annotation'] (controlled by group_by). Split the data 70%/10%/20% with split_data, then call MarkerMap.prepareData(adata, train_indices, val_indices, group_by, None, batch_size=64) to build train and val dataloaders. Instantiate MarkerMap with input_size=adata.X.shape[1], hidden_layer_size=64 (~10% of the 500 input columns for CITE-seq), z_size=16, num_classes from the unique labels, k=50, and loss_tradeoff=0 (supervised) for the smoke test, then run train_model(marker_map, train_dataloader, val_dataloader). The model differentiably selects k markers with a Gumbel-Softmax selector; the loss_tradeoff parameter convexly combines reconstruction and classification losses (0=supervised, 0.5=mixed, 1=unsupervised). Correct application means the module imports MarkerMap and train_model, training completes on the CITE-seq dataloaders, and markers() returns exactly k=50 indices usable by a RandomForestClassifier evaluation via new_model_metrics.

## Related tools

- **MarkerMap** (VAE-based marker selection model under test; provides prepareData classmethod, train_model function, and markers() accessor) — https://github.com/Computational-Morphogenomics-Group/MarkerMap
- **scanpy** (Reads CITEseq.h5ad into an AnnData object for the training harness)
- **numpy** (Concatenates train/val indices and converts selected marker indices to numpy for evaluation)
- **pip** (Installs the markermap package (pip install markermap) or editable install (pip install -e .) before the smoke test) — https://pypi.org/project/markermap/

## Examples

```
from markermap.vae_models import MarkerMap, train_model; train_dataloader, val_dataloader = MarkerMap.prepareData(adata, train_indices, val_indices, 'annotation', None, batch_size=64); train_model(MarkerMap(adata.X.shape[1], 64, 16, len(adata.obs['annotation'].unique()), 50, loss_tradeoff=0), train_dataloader, val_dataloader)
```

## Evaluation signals

- from markermap.vae_models import MarkerMap, train_model imports without error
- train_model completes on the CITE-seq train/val dataloaders without exception
- marker_map.markers() returns exactly k=50 marker indices
- The selected markers are usable by new_model_metrics, which trains a RandomForestClassifier on the k markers and evaluates on the 20% test split
- adata.X is dense (call adata.X.toarray() on sparse arrays) before prepareData

## Limitations

- hidden_layer_size is dataset-dependent; ~10% of input dimension is a rule of thumb (64 for the 500-column CITE-seq data)
- Sparse arrays must be converted to dense first; the README notes scanpy may install a newer matplotlib requiring a runtime restart on Google Colab
- For unsupervised variants (loss_tradeoff=1) num_classes can be None, but the smoke test uses loss_tradeoff=0 (supervised only)
- On Windows installation is listed as 'Coming soon!' in the README

## Evidence

- [readme] For the CITE-seq data which has 500 columns, we will use 64: "This is dependent on the data, a good rule of thumb is ~10% of the dimension of the input data. For the CITE-seq data which has 500 columns, we will use 64"
- [readme] train_model invocation: "train_model(supervised_marker_map, train_dataloader, val_dataloader)"
- [methods] split procedure: "We then split the data into training, validation, and test sets with a 70%, 10%, 20% split."
- [methods] dense conversion: "To convert, merely do `adata.X = adata.X.toarray()`"
- [methods] evaluation via RandomForest: "The new_model_metrics function trains a simple model such as a RandomForestClassifer on the training data restricted to the k markers, and then evaluates it on the testing data."
