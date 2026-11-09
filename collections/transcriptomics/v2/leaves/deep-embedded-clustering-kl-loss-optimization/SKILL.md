---
name: deep-embedded-clustering-kl-loss-optimization
description: Use when you must jointly cluster paired single-cell multi-omics data
  (e.g., CITE-seq RNA+ADT or SMAGE-seq RNA+ATAC stored as h5 with X1/X2 and Batch
  indicator) with a fixed deep architecture and joint loss optimization.
when_to_use_negative:
- You have no paired multimodal data — scMDC is designed for multimodal (CITE-seq/SMAGE-seq)
  inputs, not single-modality-only problems
- 'You cannot specify K: the number of clusters must be supplied by the user, with
  no automatic estimation provided'
- You only need a joint embedding for downstream visualization (e.g., Cobolt, scMM,
  totalVI-style latent learning) rather than clustering-optimized embeddings
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scMDC
  role: The multimodal denoising autoencoder with ZINB, KL, and deep K-means losses
    being reconstructed and trained
  repo: https://github.com/xianglin226/scMDC
- name: PyTorch
  role: Framework for implementing the encoder/decoder modules and joint loss optimization
- name: h5py
  role: Loads the h5 input file containing X1, X2, Y, and Batch indicator
- name: scikit-learn
  role: Provides K-means for initializing the K centroids on the pretrained latent
    space
- name: IDEC
  role: Prior deep embedded clustering framework (t-distribution KL loss on latent
    layer) that scMDC's KL loss builds on
  repo: https://github.com/XifengGuo/IDEC
- name: SymSim
  role: R package used to simulate multi-omics data with signal, dropout, and batch
    effects for validation experiments
  repo: https://github.com/YosefLab/SymSim
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:1ce382ed8df0c49dd65df44fdd790fd3a36679fada51d5509c1126e86833c6d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xianglin226/scMDC
  tools:
  - IDEC
  - NumPy
  - PyTorch
  - SymSim
  - h5py
  - scMDC
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# deep-embedded-clustering-kl-loss-optimization

## Summary

Reconstruct and train the scMDC end-to-end multimodal deep clustering model: a denoising autoencoder with one shared encoder, modality-specific ZINB decoders, a t-distribution KL-divergence loss, and deep K-means, all optimized simultaneously for multimodal single-cell clustering.

## When to use

Use when you must jointly cluster paired single-cell multi-omics data (e.g., CITE-seq RNA+ADT or SMAGE-seq RNA+ATAC stored as h5 with X1/X2 and Batch indicator) with a fixed deep architecture and joint loss optimization.

## When NOT to use

- You have no paired multimodal data — scMDC is designed for multimodal (CITE-seq/SMAGE-seq) inputs, not single-modality-only problems
- You cannot specify K: the number of clusters must be supplied by the user, with no automatic estimation provided
- You only need a joint embedding for downstream visualization (e.g., Cobolt, scMM, totalVI-style latent learning) rather than clustering-optimized embeddings

## Inputs

- h5 file containing X1 (RNA count matrix), X2 (ADT or ATAC/gene-activity matrix), optional Y cell-type labels, and a Batch indicator
- Number of clusters K specified by the user
- Raw CITE-seq or SMAGE-seq data preprocessed with SCANPY (zero-count genes/ADTs filtered, size-factor normalization, log transform, scaling)

## Outputs

- Trained scMDC PyTorch module with jointly optimized autoencoder, KL, and deep K-means losses
- Cluster assignments (cell label vector) and low-dimensional latent embedding per cell
- Trained K cluster centroids in the latent space

## How to apply

Load the h5 input (X1 RNA, X2 ADT/ATAC, optional Y labels, Batch indicator) with h5py, then build the PyTorch module with modality-specific low-level RNA encoder layers [256,64,32,16] for CITE-seq (or [256,128,64] for SMAGE-seq), ADT decoder layers [16,64,256] (or [64,128,256]), and a shared high-level encoder [16,20] (or [64,128,256]), applying Gaussian denoising corruption with sigma1=2.5 on RNA and sigma2=1.5 (CITE-seq) or 2.5 (SMAGE-seq) on ADT. Use zero-inflated negative binomial (ZINB) output layers as the reconstruction loss for all omics, and optionally append a one-hot batch indicator to the input for the conditional batch-correcting variant. Implement the t-distribution KL loss on the latent layer with coefficients phi1/phi2 = 0.001 (CITE-seq) or 0.005 (SMAGE-seq), and the deep K-means loss with coefficient gamma=0.1, initializing K centroids by K-means on the pretrained latent space. Pretrain for 400 epochs, then run the clustering stage up to maxiter=10000 with update_interval=1, stopping when the fraction of changed labels falls below tol=0.001; the whole model (autoencoder, KL loss, deep K-means) is optimized simultaneously end-to-end.

## Related tools

- **scMDC** (The multimodal denoising autoencoder with ZINB, KL, and deep K-means losses being reconstructed and trained) — https://github.com/xianglin226/scMDC
- **PyTorch** (Framework for implementing the encoder/decoder modules and joint loss optimization)
- **h5py** (Loads the h5 input file containing X1, X2, Y, and Batch indicator)
- **scikit-learn** (Provides K-means for initializing the K centroids on the pretrained latent space)
- **IDEC** (Prior deep embedded clustering framework (t-distribution KL loss on latent layer) that scMDC's KL loss builds on) — https://github.com/XifengGuo/IDEC
- **SymSim** (R package used to simulate multi-omics data with signal, dropout, and batch effects for validation experiments) — https://github.com/YosefLab/SymSim

## Examples

```
import h5py; f = h5py.File('data/bmnc.h5','r'); X1, X2, batch = f['X1'][:], f['X2'][:], f['batch'][:]  # then train scMDC: pretrain 400 epochs, K-means init, clustering to tol=0.001
```

## Evaluation signals

- Module and training script run end-to-end without error on a ~5000-cell dataset (~3 minutes on an Nvidia Tesla P100 16G GPU)
- All three loss terms (ZINB reconstruction, KL divergence, deep K-means) decrease jointly during training
- Training converges when the percentage of changed labels drops below tol=0.001 during the clustering stage
- Latent-space K-means initialization yields K centroids that remain stable across update_interval=1 iterations
- Cluster assignments achieve ARI/NMI comparable to or better than competing methods (Seurat V4 WNN, BREM-SC, CiteFuse, Specter) on benchmark datasets like GSE128639 BMNC or SLN208/SLN111

## Limitations

- The number of clusters K must be specified by the user; no automatic estimation is provided
- All experiments require GPU hardware (Nvidia Tesla P100 16G), which may limit reproducibility on CPU-only systems
- Layer sizes and loss coefficients differ by modality pair: CITE-seq uses RNA [256,64,32,16], phi=0.001, sigma2=1.5, while SMAGE-seq uses [256,128,64], phi=0.005, sigma2=2.5 — using the wrong configuration degrades performance

## Evidence

- [intro] End-to-end joint optimization: "The whole model, including the autoencoder, the KL-loss, and the deep K-means clustering, are optimized simultaneously."
- [discussion] Pretraining protocol: "We pretrain the autoencoders for 400 epochs before entering the clustering stage."
- [discussion] Centroid initialization: "we initialize K centroids by implementing K-means algorithm on the pretrained latent space."
- [discussion] ZINB reconstruction: "For all the omics of data, we employ the zero-inflated negative binomial (ZINB) models as the reconstruction loss function"
- [readme] scMDC description: "scMDC is an end-to-end deep model that explicitly characterizes different data sources and jointly learns latent features of deep embedding for clustering analysis."
- [methods] h5 input format: "1) Prepare the input data in h5 format. (See readme in 'dataset' folder)"
- [discussion] User-specified K: "After pretraining, the users need to specify the number of clusters (K)."
