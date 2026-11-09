---
name: zinb-reconstruction-loss-modeling
description: Use when training a deep autoencoder (e.g. scMDC-style multimodal denoising
  autoencoder) on raw single-cell count data whose mRNA/ADT/ATAC matrices are zero-inflated
  and over-dispersed, and reconstruction must model dropout rather than use a plain
  Gaussian/MSE loss.
when_to_use_negative:
- The data are not raw zero-inflated counts (e.g. continuous non-single-cell measurements
  without dropout/over-dispersion), so a ZINB likelihood is not an appropriate generative
  model.
- The goal is joint embedding for downstream tasks other than clustering, without
  an end-to-end clustering objective — the article notes naive joint-embedding-then-cluster
  pipelines are not optimized for clustering and omit dropout modeling.
- Only a CPU is available and the dataset is large — the article's experiments all
  require an Nvidia Tesla P100 (16 G) GPU.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3474
tools:
- name: scMDC
  role: Multimodal deep clustering framework whose decoders employ ZINB models as
    the reconstruction loss for all omics; optimized end-to-end with KL and deep K-means
    losses
  repo: https://github.com/xianglin226/scMDC
- name: PyTorch
  role: Implementation framework for the autoencoder module with zero-inflated negative
    binomial output layers and their joint training
- name: SCANPY
  role: Preprocessing and normalization of raw mRNA and ADT counts (filter zero-count
    features, size-factor normalization, log transform, scaling) before ZINB reconstruction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:125132271288001ce182b693a851c0727aa7371a5c2f9563bea1c2e952c8a829
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xianglin226/scMDC
  tools:
  - NumPy
  - PyTorch
  - SCANPY
  - h5py
  - scMDC
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# zinb-reconstruction-loss-modeling

## Summary

Model the reconstruction loss of a single-cell autoencoder with zero-inflated negative binomial (ZINB) output layers so that pervasive dropout events and over-dispersion in count data (scRNA-seq mRNA, CITE-seq ADT, scATAC-derived gene activities) are explicitly characterized during denoising and clustering.

## When to use

Use when training a deep autoencoder (e.g. scMDC-style multimodal denoising autoencoder) on raw single-cell count data whose mRNA/ADT/ATAC matrices are zero-inflated and over-dispersed, and reconstruction must model dropout rather than use a plain Gaussian/MSE loss.

## When NOT to use

- The data are not raw zero-inflated counts (e.g. continuous non-single-cell measurements without dropout/over-dispersion), so a ZINB likelihood is not an appropriate generative model.
- The goal is joint embedding for downstream tasks other than clustering, without an end-to-end clustering objective — the article notes naive joint-embedding-then-cluster pipelines are not optimized for clustering and omit dropout modeling.
- Only a CPU is available and the dataset is large — the article's experiments all require an Nvidia Tesla P100 (16 G) GPU.

## Inputs

- h5 file containing raw count matrices X1 (mRNA) and X2 (ADT or scATAC gene-activity), optional Y labels, and a batch indicator
- SCANPY-preprocessed, size-factor-normalized, log-transformed, scaled mRNA and ADT matrices
- Architecture spec with ZINB output layers per modality (CITE-seq: encoder [256,64,32,16]+[16,20], decoder [16,64,256])

## Outputs

- Trained denoising multimodal autoencoder whose decoders output ZINB parameters (mean, dispersion, dropout probability) per omic
- Reconstruction loss (ZINB negative log-likelihood) used to jointly optimize autoencoder, KL loss, and deep K-means clustering end-to-end
- Latent embeddings and cluster assignments (K specified by the user)

## How to apply

In scMDC, the ZINB loss is not a preprocessing step but the reconstruction objective of the denoising multimodal autoencoder: after the shared encoder takes concatenated modality data (with Gaussian denoising corruption sigma1=2.5 on RNA and sigma2=1.5 for CITE-seq ADT), modality-specific decoders end in ZINB output layers that predict the zero-inflation probability, dispersion, and mean of each omic, and the ZINB negative log-likelihood is minimized jointly with the t-distribution KL loss (phi=0.001 CITE-seq, 0.005 SMAGE-seq) and deep K-means loss (gamma=0.1) in a single end-to-end optimization. Inputs are h5 files with X1 (RNA), X2 (ADT or ATAC gene activity via Signac GeneActivity) preprocessed with SCANPY (filter zero-count genes/ADTs, size-factor normalization, log transform, scaling). The same ZINB reconstruction is used for all omics and also underlies the 400-epoch pretraining phase before centroids are K-means-initialized. Rationale: competing CITE-seq tools do not explicitly model dropout and overdispersion, and ZINB better characterizes single-cell count data, which is why it is the loss of choice for this deep-clustering architecture.

## Related tools

- **scMDC** (Multimodal deep clustering framework whose decoders employ ZINB models as the reconstruction loss for all omics; optimized end-to-end with KL and deep K-means losses) — https://github.com/xianglin226/scMDC
- **PyTorch** (Implementation framework for the autoencoder module with zero-inflated negative binomial output layers and their joint training)
- **SCANPY** (Preprocessing and normalization of raw mRNA and ADT counts (filter zero-count features, size-factor normalization, log transform, scaling) before ZINB reconstruction)

## Examples

```
python main.py --input dataset/input.h5 --pretrain 400 --maxiter 10000 --update_interval 1 --tol 0.001 --gamma 0.1
```

## Evaluation signals

- Decoder output layers are zero-inflated negative binomial (not plain MSE/Gaussian), and the ZINB loss decreases together with the KL and deep K-means losses during joint optimization
- Clustering-stage convergence criterion is met: fraction of changed labels falls below tol=0.001 (update_interval=1, maxiter=10000) after 400 pretraining epochs and K-means centroid initialization
- Runtime is linear and smoke-test-scale: a dataset of about 5000 cells trains in roughly 3 minutes on an Nvidia Tesla P100 16G GPU
- Benchmark comparison shows the ZINB-based model outperforming competing single-modal and multimodal clustering methods that do not explicitly model dropout and overdispersion
- Gaussian denoising corruption is applied before the ZINB reconstruction (sigma1=2.5 on RNA; sigma2=1.5 CITE-seq / 2.5 SMAGE-seq on ADT), and the conditional variant extends the input with a one-hot batch indicator for multi-batch data

## Limitations

- All scMDC experiments are conducted on an Nvidia Tesla P100 (16 G) GPU, so CPU-only environments may not reproduce the reported runtimes or convergence.
- The number of clusters K is not estimated automatically — users must specify it after pretraining.
- ZINB reconstruction is defined for count-like omics; it was designed for zero-inflated, over-dispersed single-cell mRNA/ADT/ATAC data, not arbitrary continuous modalities.

## Evidence

- [discussion] For all the omics of data, we employ the zero-inflated negative binomial (ZINB) models as the reconstruction loss function28: "For all the omics of data, we employ the zero-inflated negative binomial (ZINB) models as the reconstruction loss function28"
- [intro] the pervasive dropout events make single-cell count data to be zero-inflated and over-dispersed.: "the pervasive dropout events make single-cell count data to be zero-inflated and over-dispersed."
- [discussion] These methods do not explicitly model dropout events and overdispersions in mRNA and/or ADT count data.: "These methods do not explicitly model dropout events and overdispersions in mRNA and/or ADT count data."
- [methods] 1) Prepare the input data in h5 format. (See readme in 'dataset' folder): "1) Prepare the input data in h5 format. (See readme in 'dataset' folder)"
