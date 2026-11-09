---
name: variational-autoencoder-st-data-augmentation
description: Use when training a CeLEry spatial-location model on limited ST training
  data (e.g., one tissue section) and you need to boost robustness to Gaussian/expression
  noise in the scRNA-seq test cells, or when the number of training sections is too
  small for merged-slice training.
when_to_use_negative:
- The scRNA-seq test data come from a small targeted gene panel on a single-cell-resolution
  platform (e.g., small-panel Xenium) — the article states CeLEry is suboptimal on
  such data and awaits transcriptome-wide, single-cell-resolution ST
- You have abundant multi-section training data where merged-slice training already
  provides sufficient signal and augmentation adds little (Scenario 4-style same-brain
  multi-section settings)
- The goal is inference of cell-cell communications or discrete cell-type assignment
  with no spatial coordinate recovery — augmentation is designed for location/domain
  prediction robustness
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CeLEry
  role: Package implementing the VAE-based data augmentation procedure and the downstream
    deep neural network location-prediction training on augmented data
  repo: https://github.com/QihuangZhang/CeLEry
- name: spaGCN
  role: Provides rank_genes_groups to select spatially variable genes that define
    the feature space used by the VAE augmentation
- name: Scanpy
  role: Used for preprocessing (log normalization, z-scoring) of the ST expression
    matrices prior to VAE training
- name: PyTorch
  role: Backend for training the variational autoencoder (16/8/4-channel convolutional
    encoder, 512-dim embedding) used to generate ST replicates
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39895-3
    title: Leveraging spatial transcriptomics data to recover cell locations in single-cell
      RNA-seq with CeLEry
  source_content_hash: sha256:132d879751e608a5f0da84a5df7beb02cfac594b31363b19f580c27390baa6c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QihuangZhang/CeLEry
  tools:
  - CeLEry
  - PyTorch
  - Scanpy
  - Tangram
  - novoSpaRc
  - spaGCN
  - spaOTsc
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# variational-autoencoder-st-data-augmentation

## Summary

Optional CeLEry data augmentation step that trains a variational autoencoder (beta=1e-5, 512-dim embedding, 16/8/4-channel convolutional encoder) on preprocessed spatial transcriptomics data to generate synthetic ST replicates, which are combined with the original training data to improve robustness of location-prediction models to noise in scRNA-seq test data.

## When to use

Use when training a CeLEry spatial-location model on limited ST training data (e.g., one tissue section) and you need to boost robustness to Gaussian/expression noise in the scRNA-seq test cells, or when the number of training sections is too small for merged-slice training.

## When NOT to use

- The scRNA-seq test data come from a small targeted gene panel on a single-cell-resolution platform (e.g., small-panel Xenium) — the article states CeLEry is suboptimal on such data and awaits transcriptome-wide, single-cell-resolution ST
- You have abundant multi-section training data where merged-slice training already provides sufficient signal and augmentation adds little (Scenario 4-style same-brain multi-section settings)
- The goal is inference of cell-cell communications or discrete cell-type assignment with no spatial coordinate recovery — augmentation is designed for location/domain prediction robustness

## Inputs

- Preprocessed spatial transcriptomics training matrix (spots x spatially variable genes) with true 2D spot coordinates or layer labels (e.g., LIBD Visium DLPFC, MERFISH Mouse3_sagittal slices)
- One-hot gene-group matrix from K-means clustering of genes by expression pattern
- Number of ST replicates to generate (hyperparameter, e.g., 2 or 10)

## Outputs

- VAE-generated ST replicate expression matrices (512-dim embedding, decoder-reconstructed spatial gene expression)
- Augmented training set = original ST data combined with K generated replicates, fed into CeLEry location-prediction model retraining
- Replicate count chosen by cross-validation

## How to apply

First preprocess the ST training data (spot-level log transform of UMI counts plus gene-level z-scoring) and select spatially variable genes with rank_genes_groups in spaGCN. Then K-means cluster the genes into groups sharing similar expression patterns, encode the clustering as a one-hot matrix, and train the VAE with KL weight beta=1e-5 and a 512-dimensional embedding using a 16/8/4-channel convolutional encoder. Apply the decoder to reconstruct the spatial gene expression matrix and concatenate these synthetic replicates with the original ST data before retraining the supervised deep neural network (3 hidden layers of 50/10/5 nodes, ReLU, Sigmoid-scaled output). The number of replicates (e.g., 2 or 10 in the benchmarks) is a user-specified hyperparameter that should be tuned via cross-validation; more replicates yield progressively greater robustness as test-data noise increases.

## Related tools

- **CeLEry** (Package implementing the VAE-based data augmentation procedure and the downstream deep neural network location-prediction training on augmented data) — https://github.com/QihuangZhang/CeLEry
- **spaGCN** (Provides rank_genes_groups to select spatially variable genes that define the feature space used by the VAE augmentation)
- **Scanpy** (Used for preprocessing (log normalization, z-scoring) of the ST expression matrices prior to VAE training)
- **PyTorch** (Backend for training the variational autoencoder (16/8/4-channel convolutional encoder, 512-dim embedding) used to generate ST replicates)

## Examples

```
pip install CeLEryPy
```

## Evaluation signals

- Under increasing Gaussian noise on test-spot gene expression, the MSE advantage of augmented (2- or 10-replicate) CeLEry over ordinary (0-replicate) CeLEry widens monotonically with the noise standard deviation (Fig. 5d pattern)
- On LIBD DLPFC Scenario 1, augmented training raises cortical-layer top-1 accuracy from 53.8% to 63.2% and top-2 to 92.1%; Scenario 2 top-1 rises from 44.0% to 51.2%
- VAE hyperparameters match the paper: KL weight beta=1e-5, 512-dimensional embedding, 16/8/4-channel convolutional encoder
- Replicate count is selected by cross-validation rather than fixed a priori, per the authors' recommendation
- Generated replicates reconstruct the spatial gene expression matrix and share the same spatially variable gene set as the original training data before concatenation

## Limitations

- The number of replicates to be generated is a hyperparameter prespecified by users; there is no fixed recommended setting and it must be explored via cross-validation
- Augmentation improves robustness mainly under noisy test conditions and for limited single-section training; its benefit depends on the VAE's ability to reconstruct informative gradient gene expression patterns
- CeLEry (and hence augmentation benefit) is suboptimal on single-cell-resolution ST platforms with small gene panels; performance should improve with transcriptome-wide, single-cell-resolution ST data

## Evidence

- [abstract] Optional data augmentation via a variational autoencoder to improve robustness and overcome noise in scRNA-seq data.: "CeLEry has an optional data augmentation procedure via a variational autoencoder, which improves the method's robustness and allows it to overcome noise in scRNA-seq data."
- [methods] we propose a data augmentation procedure to generate “replicates” of the original ST data: "we propose a data augmentation procedure to generate “replicates” of the original ST data"
- [methods] we apply the decoder to reconstruct the spatial gene expression matrix, which will be combined with the original ST data for model training: "we apply the decoder to reconstruct the spatial gene expression matrix, which will be combined with the original ST data for model training"
- [discussion] The number of replicates to be generated is a hyperparameter prespecified by users: "The number of replicates to be generated is a hyperparameter prespecified by users"
- [readme] README package description and dependencies: "CeLEry optionally generates replicates of the spatial transcriptomic data via variational autoencoder then includes them as the training data together with original spatial transcriptomic data."
