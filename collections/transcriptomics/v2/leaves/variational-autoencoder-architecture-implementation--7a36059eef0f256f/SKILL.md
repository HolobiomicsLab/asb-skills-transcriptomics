---
name: variational-autoencoder-architecture-implementation
description: Use when building or reproducing an end-to-end marker selection model
  for single-cell RNA-seq (.h5ad / AnnData) data that must select exactly k genes
  and support supervised (loss_tradeoff=0), unsupervised (=1), or joint (=0.5) objectives.
when_to_use_negative:
- You only need a fast baseline marker ranking without a trained generative model
  — the repository offers RandomBaseline (selects k markers at random) and RankCorr
  (rank-correlation-based selection) instead
- Input data is not single-cell expression in an AnnData/h5ad structure with a categorical
  group_by label column, so prepareData cannot build the dataloaders
- You need reproducible marker gene names for FISH panel design with binarized expression
  — PERSIST is designed for that spatial transcriptomics setting, not MarkerMap
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: MarkerMap
  role: The pip-installable generative VAE model being implemented; provides MarkerMap,
    train_model, prepareData, getBenchmarker, and the markers() accessor
  repo: https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- name: scanpy
  role: Reads the CITEseq.h5ad file into an AnnData object used as the input to prepareData
- name: numpy
  role: Handles dense arrays (e.g. np.concatenate of train/val indices) and the adata.X
    matrix passed to the model
- name: pip
  role: Installs markermap (`pip install markermap`) or editable for development (`pip
    install -e .`)
  repo: https://pypi.org/project/markermap/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41540-024-00339-3
    title: 'MarkerMap: nonlinear marker selection for single-cell studies'
  source_content_hash: sha256:ffb9bf188f9dacde13f81350436c8869fb097cc1bd2bfe78bfcfd6ef4591abfa
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
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# variational-autoencoder-architecture-implementation

## Summary

Implements the MarkerMap VAE-based marker selector: a Gumbel-Softmax layer differentiably picks k markers from a d-dimensional logit vector, a VAE branch reconstructs the full transcriptome from the selected markers, and a classification branch on the 16-dimensional latent space is combined with reconstruction via a loss_tradeoff-weighted convex loss.

## When to use

Use when building or reproducing an end-to-end marker selection model for single-cell RNA-seq (.h5ad / AnnData) data that must select exactly k genes and support supervised (loss_tradeoff=0), unsupervised (=1), or joint (=0.5) objectives.

## When NOT to use

- You only need a fast baseline marker ranking without a trained generative model — the repository offers RandomBaseline (selects k markers at random) and RankCorr (rank-correlation-based selection) instead
- Input data is not single-cell expression in an AnnData/h5ad structure with a categorical group_by label column, so prepareData cannot build the dataloaders
- You need reproducible marker gene names for FISH panel design with binarized expression — PERSIST is designed for that spatial transcriptomics setting, not MarkerMap

## Inputs

- AnnData object from an .h5ad file (e.g. data/cite_seq/CITEseq.h5ad) with gene expression in adata.X and cell labels in adata.obs[group_by] (e.g. 'annotation')
- Hyperparameters: input_size (number of genes, e.g. 500 for CITE-seq), hidden_layer_size (e.g. 64), z_size (16), num_classes, k (e.g. 50), loss_tradeoff (0 / 0.5 / 1), batch_size (e.g. 64)

## Outputs

- Trained MarkerMap VAE model whose markers() method returns exactly k=50 marker gene indices
- Trainable PyTorch module importable as `from markermap.vae_models import MarkerMap, train_model`
- Train/val dataloaders constructed by MarkerMap.prepareData

## How to apply

Implement a variational auto-encoder whose constructor takes input_size, hidden_layer_size, z_size (set to 16), num_classes, and k, mirroring the MarkerMap signature. Add a Gumbel-Softmax selector layer over a d-dimensional logit vector so that k markers are differentiably selected, and expose them via a markers() method returning marker indices. Build the VAE reconstruction branch and a classification branch on the latent space, and combine their losses with a convex combination controlled by loss_tradeoff: 0 for supervised only, 0.5 for mixed, 1.0 for unsupervised only. Provide a prepareData classmethod that builds train/val dataloaders from an AnnData object using a group_by label column and batch_size (e.g. 64), and a train_model function taking the model plus both dataloaders. Choose hidden_layer_size as roughly 10% of the input dimension (64 for the 500-column CITE-seq data) and smoke-test on data/cite_seq/CITEseq.h5ad read with scanpy, converting adata.X to dense with adata.X.toarray() if sparse, and splitting 70%/10%/20%.

## Related tools

- **MarkerMap** (The pip-installable generative VAE model being implemented; provides MarkerMap, train_model, prepareData, getBenchmarker, and the markers() accessor) — https://github.com/Computational-Morphogenomics-Group/MarkerMap.git
- **scanpy** (Reads the CITEseq.h5ad file into an AnnData object used as the input to prepareData)
- **numpy** (Handles dense arrays (e.g. np.concatenate of train/val indices) and the adata.X matrix passed to the model)
- **pip** (Installs markermap (`pip install markermap`) or editable for development (`pip install -e .`)) — https://pypi.org/project/markermap/

## Examples

```
from markermap.vae_models import MarkerMap, train_model; mm = MarkerMap(adata.X.shape[1], 64, 16, len(adata.obs['annotation'].unique()), 50, loss_tradeoff=0); train_model(mm, train_dataloader, val_dataloader)
```

## Evaluation signals

- The module imports successfully: `from markermap.vae_models import MarkerMap, train_model` runs without error
- train_model completes on the CITE-seq train/val dataloaders built from the 70%/10%/20% split
- markers() returns exactly k=50 marker indices that are usable by the RandomForest-based new_model_metrics evaluation
- Setting loss_tradeoff=0, 0.5, and 1.0 yields the supervised, mixed, and unsupervised variants respectively, matching the paper's three levels
- z_size equals 16 in the latent layer and hidden_layer_size approximates 10% of the input dimension (64 for 500-gene CITE-seq data)

## Limitations

- On Google Colab the runtime may need restarting after install because scanpy installs a newer version of matplotlib than the default
- Smashpy pins an older tensorflow (==2.5.0) causing installation problems, so it is not required unless benchmarking against that method
- hidden_layer_size is data-dependent (rule of thumb ~10% of input dimension), and k may require domain knowledge, though it can be benchmarked over a range

## Evidence

- [readme] MarkerMap is a generative model for selecting the most informative gene markers by projecting cells into a shared, interpretable embedding without sacrificing accuracy.: "MarkerMap is a generative model for selecting the most informative gene markers by projecting cells into a shared, interpretable embedding without sacrificing accuracy."
- [readme] z_size is the dimension of the latent space in the variational auto-encoder. We always use 16: "z_size is the dimension of the latent space in the variational auto-encoder. We always use 16"
- [readme] In general, we have 3 levels, 0 (supervised only), 0.5 (mixed supervised-unsupervised) and 1 (unsupervised only).: "In general, we have 3 levels, 0 (supervised only), 0.5 (mixed supervised-unsupervised) and 1 (unsupervised only)."
- [methods] We then split the data into training, validation, and test sets with a 70%, 10%, 20% split.: "We then split the data into training, validation, and test sets with a 70%, 10%, 20% split."
- [methods] To convert, merely do `adata.X = adata.X.toarray()`: "To convert, merely do `adata.X = adata.X.toarray()`"
- [methods] The new_model_metrics function trains a simple model such as a RandomForestClassifer on the training data restricted to the k markers, and then evaluates it on the testing data.: "The new_model_metrics function trains a simple model such as a RandomForestClassifer on the training data restricted to the k markers, and then evaluates it on the testing data."
