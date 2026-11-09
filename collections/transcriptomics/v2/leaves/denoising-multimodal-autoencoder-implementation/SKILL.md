---
name: denoising-multimodal-autoencoder-implementation
description: Use when clustering paired single-cell multimodal data (CITE-seq RNA+ADT
  or SMAGE-seq RNA+ATAC) with an end-to-end deep model that jointly optimizes reconstruction,
  KL, and clustering losses.
when_to_use_negative:
- When a joint embedding suffices without clustering optimization — the article notes
  joint-embedding methods 'are not designed and optimized for clustering', but if
  clustering is not the goal, a model like scMM or Cobolt is more appropriate
- When the number of clusters K is unknown and cannot be estimated by the user — scMDC
  requires the user to specify K and provides no automatic estimation
- On single-modal scRNA-seq data where single-omic tools (SC3, Tscan, Specter) are
  sufficient
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scMDC
  role: The multimodal deep clustering model being implemented; code at xianglin226/scMDC
- name: PyTorch
  role: Framework for building the autoencoder module with ZINB output layers and
    joint loss optimization
- name: h5py
  role: Loading the h5-format input data (X1, X2, Y, Batch indicator)
- name: SCANPY
  role: Preprocessing and normalization of mRNA and ADT data before training
- name: scikit-learn
  role: K-means initialization of the K centroids on the pretrained latent space
- name: NumPy
  role: Array handling for concatenated modality inputs and batch one-hot encoding
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:d9f26eda4d73141a9a465ab4f9e9792189de14a63c8d1b4aa283497703351e8b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NumPy
  - PyTorch
  - SCANPY
  - h5py
  - scMDC
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# denoising-multimodal-autoencoder-implementation

## Summary

Implement the scMDC denoising multimodal autoencoder in PyTorch: a shared encoder over concatenated modalities, modality-specific decoders, Gaussian corruption, ZINB reconstruction loss, t-distribution KL loss, and a deep K-means head, all optimized end-to-end for single-cell multi-omics clustering.

## When to use

Use when clustering paired single-cell multimodal data (CITE-seq RNA+ADT or SMAGE-seq RNA+ATAC) with an end-to-end deep model that jointly optimizes reconstruction, KL, and clustering losses.

## When NOT to use

- When a joint embedding suffices without clustering optimization — the article notes joint-embedding methods 'are not designed and optimized for clustering', but if clustering is not the goal, a model like scMM or Cobolt is more appropriate
- When the number of clusters K is unknown and cannot be estimated by the user — scMDC requires the user to specify K and provides no automatic estimation
- On single-modal scRNA-seq data where single-omic tools (SC3, Tscan, Specter) are sufficient

## Inputs

- h5 file containing X1 (RNA counts), X2 (ADT or ATAC/gene-activity counts), optional Y cell-type labels, and a Batch indicator
- SCANPY-normalized RNA and ADT count matrices (separately normalized with the same method)
- User-specified number of clusters K

## Outputs

- Trained PyTorch scMDC module with shared encoder latent embedding and ZINB output layers
- Cluster assignments for every cell from the deep K-means stage
- Convergence log when the percentage of changed labels drops below tol=0.001

## How to apply

Load the h5 input (X1 RNA, X2 ADT/ATAC, optional Y labels, Batch indicator) with h5py and preprocess RNA and ADT separately with SCANPY (zero-count filtering, size-factor normalization, log transform, scaling). Build the PyTorch module with modality-specific RNA encoder layers [256,64,32,16] for CITE-seq (or [256,128,64] for SMAGE-seq), decoders [16,64,256] (or [64,128,256]), and a shared high-level encoder to the latent layer, applying Gaussian denoising corruption with sigma1=2.5 on RNA and sigma2=1.5 (CITE-seq) or 2.5 (SMAGE-seq) on the second modality, with zero-inflated negative binomial output layers as the reconstruction loss for all omics. For multi-batch data, append a one-hot batch indicator to the input to form the conditional batch-correcting variant. Implement the t-distribution KL loss on the latent layer with coefficients phi1/phi2 = 0.001 (CITE-seq) or 0.005 (SMAGE-seq), pretrain 400 epochs (reconstruction only for the first 200, KL loss added for the remaining 200), then initialize K centroids by K-means on the pretrained latent space and jointly optimize autoencoder, KL, and deep K-means losses (coefficient gamma=0.1) up to maxiter=10000 with update_interval=1, stopping when the fraction of changed labels falls below tol=0.001.

## Related tools

- **scMDC** (The multimodal deep clustering model being implemented; code at xianglin226/scMDC)
- **PyTorch** (Framework for building the autoencoder module with ZINB output layers and joint loss optimization)
- **h5py** (Loading the h5-format input data (X1, X2, Y, Batch indicator))
- **SCANPY** (Preprocessing and normalization of mRNA and ADT data before training)
- **scikit-learn** (K-means initialization of the K centroids on the pretrained latent space)
- **NumPy** (Array handling for concatenated modality inputs and batch one-hot encoding)

## Examples

```
import h5py; f = h5py.File('data.h5','r'); X1, X2, batch = f['X1'][:], f['X2'][:], f['Batch'][:]; model = scMDC(dims=[256,64,32,16,20]); train(model, X1, X2, batch, K=6, pretrain_epochs=400, maxiter=10000, tol=0.001)
```

## Evaluation signals

- The module and training script run end-to-end without error and all three losses (ZINB reconstruction, KL, deep K-means) decrease during joint optimization
- Training converges when the percentage of changed cluster labels falls below tol=0.001 within maxiter=10000
- Smoke test on a dataset of about 5000 cells completes in roughly 3 minutes on an Nvidia Tesla P100 16G GPU
- Clustering accuracy (e.g., ARI/NMI against provided Y labels) matches or improves on the paper's reported results on benchmark datasets like GSE128639 BMNC
- Latent-space K-means initialization reproduces stable centroids before the clustering stage begins

## Limitations

- The number of clusters K must be specified by the user; no automatic estimation is provided
- All experiments require GPU hardware (Nvidia Tesla P100 16G), which may limit reproducibility on CPU-only systems
- Architecture sizes, noise sigmas, and loss coefficients are dataset-dependent (CITE-seq vs SMAGE-seq presets) and were fixed for these platforms rather than learned

## Evidence

- [full_text] shared encoder taking concatenated data from different modalities and two modality-specific decoders: [evidence span withheld — anchor status: not_found]
- [intro] The whole model optimized simultaneously: "The whole model, including the autoencoder, the KL-loss, and the deep K-means clustering, are optimized simultaneously."
- [discussion] ZINB as reconstruction loss: "For all the omics of data, we employ the zero-inflated negative binomial (ZINB) models as the reconstruction loss function28"
- [discussion] Pretraining and K-means initialization: "We pretrain the autoencoders for 400 epochs before entering the clustering stage."
- [discussion] K specified by user: "After pretraining, the users need to specify the number of clusters (K)."
- [discussion] K-means on pretrained latent space: "we initialize K centroids by implementing K-means algorithm on the pretrained latent space."
- [methods] h5 input preparation: "1) Prepare the input data in h5 format. (See readme in 'dataset' folder)"
- [discussion] SCANPY preprocessing: "The raw CITE-seq data is preprocessed and normalized by the Python package SCANPY48. mRNA and ADT data are normalized separately but using the same method."
