---
name: deep-clustering-model-training
description: Use when you have paired multi-omic count matrices (X1=RNA, X2=ADT or
  ATAC) in h5 format and want an end-to-end deep clustering with ARI/NMI/AMI evaluation,
  including multi-batch data needing batch-effect correction.
when_to_use_negative:
- The input is single-modality data only (RNA-only or ATAC-only) — use the single-modality
  variants scMDC-RNA/-ADT/-ATAC or scDeepCluster instead, which this model significantly
  outperforms only when both modalities are available
- The goal is a joint embedding for a purpose other than clustering — the paper notes
  joint-embedding methods like Cobolt and scMM are not designed and optimized for
  clustering, and scMDC is purpose-built for clustering, not general embedding tasks
- No GPU is available and the dataset is large — all experiments in the study were
  run on an Nvidia Tesla P100 (16 G), so CPU-only reproduction may be infeasible for
  big multimodal datasets
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMDC
  role: 'Primary model being trained: multimodal denoising autoencoder with ZINB losses,
    KL loss, and deep K-means clustering (v1.0.1)'
  repo: https://github.com/xianglin226/scMDC
- name: PyTorch
  role: Deep learning framework used to implement and train the scMDC autoencoder
    and variant sub-models
- name: Scanpy
  role: Preprocessing and separate normalization of mRNA and ADT/ATAC data before
    model training
- name: SKlearn (scikit-learn)
  role: K-means initialization of centroids on the pretrained latent space and AMI/NMI/ARI
    metric computation
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:44f7c1b707425e102fad3b0d8b5c69ae73ec82a9e107be50f9a94d3929d7a648
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xianglin226/scMDC
  tools:
  - BREM-SC
  - CiteFuse
  - Cobolt
  - NumPy
  - PyTorch
  - SC3
  - SKlearn
  - SKlearn (scikit-learn)
  - Scanpy
  - Seurat
  - Seurat V4
  - Specter
  - SymSim
  - Tscan
  - h5py
  - scDCC
  - scMDC
  - scMM
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 27
---

# deep-clustering-model-training

## Summary

Train the scMDC end-to-end multimodal deep clustering model — a denoising autoencoder with ZINB reconstruction losses on RNA plus a second omic (ADT/ATAC), jointly optimized with a KL loss and deep K-means — to cluster paired CITE-seq or SMAGE-seq cells without requiring labels.

## When to use

Use when you have paired multi-omic count matrices (X1=RNA, X2=ADT or ATAC) in h5 format and want an end-to-end deep clustering with ARI/NMI/AMI evaluation, including multi-batch data needing batch-effect correction.

## When NOT to use

- The input is single-modality data only (RNA-only or ATAC-only) — use the single-modality variants scMDC-RNA/-ADT/-ATAC or scDeepCluster instead, which this model significantly outperforms only when both modalities are available
- The goal is a joint embedding for a purpose other than clustering — the paper notes joint-embedding methods like Cobolt and scMM are not designed and optimized for clustering, and scMDC is purpose-built for clustering, not general embedding tasks
- No GPU is available and the dataset is large — all experiments in the study were run on an Nvidia Tesla P100 (16 G), so CPU-only reproduction may be infeasible for big multimodal datasets

## Inputs

- h5-format file with X1 (RNA count matrix), X2 (ADT or ATAC count matrix), Y (cell-type labels, optional for training), and Batch indicators for multi-batch data
- Preprocessed/normalized mRNA and ADT/ATAC matrices (Scanpy: zero-count feature filtering, size-factor normalization, log transform, scaling)
- SMAGE-seq ATAC reads first mapped to gene regions via Signac GeneActivity

## Outputs

- Cluster assignment vector per cell (predictions written via --prediction_file)
- Trained scMDC model weights (pretrained autoencoder plus jointly optimized clustering stage)
- Estimated number of clusters K (when n_clusters=-1, resolution=0.2, n_neighbors=30)
- Per-cell latent deep embeddings for downstream AMI/NMI/ARI evaluation

## How to apply

Prepare the input in h5 format (X1=RNA, X2=ADT/ATAC, Y=labels), preprocessing mRNA and ADT/ATAC separately with Scanpy: filter zero-count genes and ADTs, size-factor normalize, log-transform, and scale. Train the denoising autoencoder with zero-inflated negative binomial (ZINB) reconstruction losses for 400 pretraining epochs (reconstruction-only for the first 200, KL loss added for the remaining 200), using the SMAGE-seq defaults encodeLayer [256,128,64], sigma2=2.5, phi=0.005, gamma=0.1, tol=0.001 on a GPU. For CITE-seq variant training use the low-level encoder layers [256,64,32,16] with sigma1=2.5 (RNA noise) and sigma2=1.5 (ADT) or 2.5 (ATAC), and a maximum of 10000 clustering epochs. After pretraining, initialize K centroids by running K-means on the pretrained latent space (K can be estimated by scMDC via n_clusters=-1 with resolution=0.2 and n_neighbors=30, or specified by the user), then jointly optimize the autoencoder, KL loss, and deep K-means losses until convergence at tol=0.001; save cluster assignments with the --prediction_file option. The conditional autoencoder framework also corrects batch effects when the multi-batch data (e.g., PBMC13K with Batch indicators) is supplied.

## Related tools

- **scMDC** (Primary model being trained: multimodal denoising autoencoder with ZINB losses, KL loss, and deep K-means clustering (v1.0.1)) — https://github.com/xianglin226/scMDC
- **PyTorch** (Deep learning framework used to implement and train the scMDC autoencoder and variant sub-models)
- **Scanpy** (Preprocessing and separate normalization of mRNA and ADT/ATAC data before model training)
- **SKlearn (scikit-learn)** (K-means initialization of centroids on the pretrained latent space and AMI/NMI/ARI metric computation)

## Examples

```
python scMDC.py --X1 pbmc13k.h5 --prediction_file pred.csv --pretrain_epochs 400 --gamma 0.1 --tol 0.001 --n_clusters -1 --resolution 0.2 --filter1 --filter2
```

## Evaluation signals

- Clustering metrics (AMI, NMI, ARI) computed against reference cell-type labels should show scMDC ranking best across all metrics on SMAGE-seq datasets, with Cobolt second-best for AMI and NMI
- One-sided paired t-tests across datasets should show statistically significant improvements over competing multimodal methods (Cobolt, scMM, Seurat V4 WNN, K-means+PCA)
- Training must pass through 400 pretraining epochs (200 reconstruction-only, 200 with KL loss) before the clustering stage begins, and convergence is declared at tol=0.001
- For the ablation, scMDC should outperform scMDC-RNA/-ADT/-ATAC/-Concat and scDeepCluster on every CITE-seq and SMAGE-seq dataset (ARI and NMI), with the only exception scMDC-ATAC at P=0.07 due to small n=4
- On multi-batch PBMC13K, the integrated clustering should reflect batch-effect correction from the conditional autoencoder framework

## Limitations

- After pretraining, the number of clusters (K) must be specified by the user; no fully automatic estimation is provided in the original design
- All experiments were conducted on an Nvidia Tesla P100 (16 G) GPU, which may limit reproducibility on CPU-only systems
- Statistical significance of ablation comparisons can lack power for small numbers of datasets (scMDC vs. scMDC-ATAC: P=0.07 with n=4 SMAGE-seq datasets)
- No explicit limitations or caveats subsection is present in the provided text; no changelog found

## Evidence

- [abstract] we develop a novel multimodal deep learning method, scMDC, for single-cell multi-omics data clustering analysis: "we develop a novel multimodal deep learning method, scMDC, for single-cell multi-omics data clustering analysis"
- [intro] The whole model, including the autoencoder, the KL-loss, and the deep K-means clustering, are optimized simultaneously.: "The whole model, including the autoencoder, the KL-loss, and the deep K-means clustering, are optimized simultaneously."
- [discussion] For all the omics of data, we employ the zero-inflated negative binomial (ZINB) models as the reconstruction loss function28: "For all the omics of data, we employ the zero-inflated negative binomial (ZINB) models as the reconstruction loss function"
- [discussion] We pretrain the autoencoders for 400 epochs before entering the clustering stage.: "We pretrain the autoencoders for 400 epochs before entering the clustering stage."
- [discussion] we initialize K centroids by implementing K-means algorithm on the pretrained latent space.: "we initialize K centroids by implementing K-means algorithm on the pretrained latent space."
- [discussion] After pretraining, the users need to specify the number of clusters (K).: "After pretraining, the users need to specify the number of clusters (K)."
- [methods] --prediction_file option: "--filter1: if do feature selection on Genes. Default: No."
- [readme] Single Cell Multi-omics deep clustering (scMDC v1.0.1): "Single Cell Multi-omics deep clustering (**scMDC v1.0.1**)"
