---
name: pytorch-deep-clustering-training-scripting
description: Use when implementing an end-to-end deep clustering model in PyTorch
  for multimodal single-cell data (CITE-seq or SMAGE-seq) where autoencoder reconstruction,
  KL loss, and K-means clustering losses must be jointly optimized in one training
  loop.
when_to_use_negative:
- Data is single-omics scRNA-seq only — methods like SC3, scDCC, or Tscan (which operate
  on one modality) are the appropriate baselines, not a multimodal autoencoder
- The user cannot supply the number of clusters K — scMDC provides no automatic estimation
  of K
- No GPU is available (all scMDC experiments in the study were run on an Nvidia Tesla
  P100 16G GPU)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMDC
  role: 'Model architecture being scripted: multimodal denoising autoencoder with
    ZINB loss, KL loss, and deep K-means optimized end-to-end'
  repo: xianglin226/scMDC
- name: PyTorch
  role: Framework for building the encoder/decoder modules, ZINB output layers, Gaussian
    noise corruption, and the joint training loop
- name: scikit-learn
  role: K-means used to initialize the K centroids on the pretrained latent space
    before the clustering stage
- name: h5py
  role: Reads the h5-format input containing X1, X2, optional labels, and the Batch
    indicator
- name: NumPy
  role: Array manipulation of count matrices and one-hot batch indicators
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:6a014dec5644cfe9885ef15f0087901e37492bed44426f8570831266d5c21d70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xianglin226/scMDC
  tools:
  - NumPy
  - PyTorch
  - h5py
  - scMDC
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# pytorch-deep-clustering-training-scripting

## Summary

Script a PyTorch-based end-to-end deep clustering pipeline (as in scMDC): a denoising autoencoder with ZINB reconstruction loss, t-distribution KL loss, and deep K-means, all optimized simultaneously on multimodal single-cell data loaded from h5.

## When to use

Use when implementing an end-to-end deep clustering model in PyTorch for multimodal single-cell data (CITE-seq or SMAGE-seq) where autoencoder reconstruction, KL loss, and K-means clustering losses must be jointly optimized in one training loop.

## When NOT to use

- Data is single-omics scRNA-seq only — methods like SC3, scDCC, or Tscan (which operate on one modality) are the appropriate baselines, not a multimodal autoencoder
- The user cannot supply the number of clusters K — scMDC provides no automatic estimation of K
- No GPU is available (all scMDC experiments in the study were run on an Nvidia Tesla P100 16G GPU)

## Inputs

- h5 file containing X1 (RNA counts), X2 (ADT or ATAC gene-activity counts), optional Y cell-type labels, and a Batch indicator
- User-specified number of clusters K
- Preprocessed/normalized mRNA and ADT matrices (SCANPY: filter zero-count features, size-factor normalization, log transform, scaling)

## Outputs

- Trained PyTorch autoencoder with latent embedding
- Cluster assignments for each cell from the deep K-means stage
- Jointly optimized loss traces (ZINB reconstruction, KL divergence, deep K-means)

## How to apply

Load the h5 input with h5py, extracting X1 (RNA), X2 (ADT or ATAC), optional Y labels, and the Batch indicator. Build the PyTorch module with a shared encoder and modality-specific decoders — e.g. CITE-seq RNA layers [256,64,32,16], ADT decoder [16,64,256], shared high-level encoder [16,20] — applying Gaussian denoising corruption with sigma1=2.5 on RNA and sigma2=1.5 (CITE-seq) or 2.5 (SMAGE-seq) on ADT, and zero-inflated negative binomial (ZINB) output layers as reconstruction loss. Extend the input with a one-hot batch indicator for the conditional batch-correcting variant on multi-batch data. Pretrain for 400 epochs (reconstruction only for the first 200, KL loss with coefficient phi1/phi2=0.001 for CITE-seq or 0.005 for SMAGE-seq added for the remaining 200), then initialize K centroids via scikit-learn K-means on the pretrained latent space and run the clustering stage with deep K-means coefficient gamma=0.1, update_interval=1, maxiter=10000, stopping when the fraction of changed labels falls below tol=0.001. Smoke-test on ~5000 cells (about 3 minutes on an Nvidia Tesla P100 16G GPU).

## Related tools

- **scMDC** (Model architecture being scripted: multimodal denoising autoencoder with ZINB loss, KL loss, and deep K-means optimized end-to-end) — xianglin226/scMDC
- **PyTorch** (Framework for building the encoder/decoder modules, ZINB output layers, Gaussian noise corruption, and the joint training loop)
- **scikit-learn** (K-means used to initialize the K centroids on the pretrained latent space before the clustering stage)
- **h5py** (Reads the h5-format input containing X1, X2, optional labels, and the Batch indicator)
- **NumPy** (Array manipulation of count matrices and one-hot batch indicators)

## Examples

```
import h5py; f = h5py.File('data.h5','r'); X1, X2, Batch = f['X1'][()], f['X2'][()], f['Batch'][()]
```

## Evaluation signals

- Training converges: the percentage of changed cluster labels drops below tol=0.001 within maxiter=10000 with update_interval=1
- All three losses (ZINB reconstruction, KL divergence, deep K-means) optimize jointly without divergence
- Smoke test on a dataset of ~5000 cells runs end-to-end in ~3 minutes on an Nvidia Tesla P100 16G GPU
- Number of K-means-initialized centroids equals the user-specified K and matches the one-hot batch indicator dimensionality of the conditional variant
- Latent space assignments separate known cell types on benchmark data (e.g. BMNC/GSE128639) when labels are held out

## Limitations

- The number of clusters K must be specified by the user; no automatic estimation is provided
- All experiments require GPU hardware (Nvidia Tesla P100 16G), which may limit reproducibility on CPU-only systems
- Hyperparameters differ by platform: CITE-seq uses encoder [256,64,32,16], sigma2=1.5, phi=0.001, while SMAGE-seq uses [256,128,64], sigma2=2.5, phi=0.005

## Evidence

- [abstract] scMDC is an end-to-end deep model that explicitly characterizes different data sources and jointly learns latent features: "scMDC is an end-to-end deep model that explicitly characterizes different data sources and jointly learns latent features"
- [discussion] Pretrain 400 epochs before clustering stage: "We pretrain the autoencoders for 400 epochs before entering the clustering stage."
- [discussion] K-means centroid initialization on pretrained latent space: "we initialize K centroids by implementing K-means algorithm on the pretrained latent space."
- [discussion] ZINB as reconstruction loss for all omics: "For all the omics of data, we employ the zero-inflated negative binomial (ZINB) models as the reconstruction loss"
- [discussion] Users must specify K after pretraining: "After pretraining, the users need to specify the number of clusters (K)."
- [discussion] GPU requirement for all experiments: "All experiments of scMDC in this study are conducted on Nvidia Tesla P100 (16 G) GPU."
- [methods] h5 input format requirement: "1) Prepare the input data in h5 format. (See readme in 'dataset' folder)"
- [readme] scMDC version from README: "Single Cell Multi-omics deep clustering (scMDC v1.0.1)"
