---
name: gpu-deep-embedding-training
description: Use when raw CITE-seq (RNA+ADT) H5 files must be converted into a low-dimensional
  latent embedding whose clusters separate cell types and mix batches, and GPU-accelerated
  deep training is feasible.
when_to_use_negative:
- Data is single-modal scRNA-seq only with no paired ADT/second modality — scMDC is
  designed for multimodal sources
- Only CPU hardware is available and the dataset is large — the study's experiments
  require an Nvidia Tesla P100 (16 G) GPU
- A quick exploratory visualization is needed rather than clustering — embedding-focused
  methods like SCVIS may suffice without deep clustering training
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scMDC
  role: PyTorch deep model trained end-to-end (autoencoder + KL loss + deep K-means)
    to learn the latent embedding and cluster assignments on GPU
  repo: https://github.com/xianglin226/scMDC
- name: h5py
  role: Reads the X1/X2/Y/Batch structures from the input H5 files before training
- name: Scanpy
  role: Preprocessing and normalization of mRNA and ADT data prior to training
- name: Pytorch
  role: Deep-learning framework in which the scMDC autoencoder is implemented and
    trained on GPU
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:c1231c5513578e1f6e82404c21f23e7e7baf14c314db4f16bfa1a5e8611182c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xianglin226/scMDC
  tools:
  - IDEC
  - Pytorch
  - SCVIS
  - Scanpy
  - Seurat
  - h5py
  - scMDC
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# gpu-deep-embedding-training

## Summary

Trains the scMDC multimodal deep autoencoder (PyTorch, GPU) on CITE-seq H5 inputs to produce a latent embedding that jointly separates cell types and removes batch effects, using ZINB reconstruction losses and joint optimization with KL and deep K-means losses.

## When to use

Use when raw CITE-seq (RNA+ADT) H5 files must be converted into a low-dimensional latent embedding whose clusters separate cell types and mix batches, and GPU-accelerated deep training is feasible.

## When NOT to use

- Data is single-modal scRNA-seq only with no paired ADT/second modality — scMDC is designed for multimodal sources
- Only CPU hardware is available and the dataset is large — the study's experiments require an Nvidia Tesla P100 (16 G) GPU
- A quick exploratory visualization is needed rather than clustering — embedding-focused methods like SCVIS may suffice without deep clustering training

## Inputs

- H5-format multimodal input with X1 (RNA counts), X2 (ADT counts), Y (cell type labels), Batch labels
- SCANPY-normalized mRNA and ADT matrices (size-factor normalization, log transform, scaling)
- User-specified number of clusters K

## Outputs

- Latent embedding file (--embedding_file) containing the scMDC latent representation Z per cell
- Predicted cluster label file (--prediction_file) per cell
- Trained scMDC autoencoder/clustering model state

## How to apply

Load the H5 file with h5py, reading X1 (RNA), X2 (ADT), Y (cell type labels), and Batch structures, after preprocessing/normalizing mRNA and ADT separately with SCANPY (filter zero-count genes, size-factor normalization, log transform, scaling). Run scMDC with CITE-Seq defaults: encodeLayer [256,64,32,16], decodeLayer1 [16,64,256], decodeLayer2 [16,20], pretrain_epochs=400 (reconstruction for the first 200 epochs, KL loss added for the remaining 200), maxiter=10000, gamma=0.1, update_interval=1, tol=0.001, sigma1=2.5, sigma2=1.5, using ZINB reconstruction losses per modality. After pretraining, initialize K centroids by K-means on the pretrained latent space — the number of clusters K must be specified by the user — then jointly optimize the autoencoder, KL-loss, and deep K-means losses in the clustering stage. Enable the --embedding_file and --prediction_file flags to persist the latent representation Z and predicted cluster labels; multi-batch data is handled by the conditional autoencoder framework for batch-effect correction. All experiments in the study ran on an Nvidia Tesla P100 (16 G) GPU, so plan for CUDA hardware.

## Related tools

- **scMDC** (PyTorch deep model trained end-to-end (autoencoder + KL loss + deep K-means) to learn the latent embedding and cluster assignments on GPU) — https://github.com/xianglin226/scMDC
- **h5py** (Reads the X1/X2/Y/Batch structures from the input H5 files before training)
- **Scanpy** (Preprocessing and normalization of mRNA and ADT data prior to training)
- **Pytorch** (Deep-learning framework in which the scMDC autoencoder is implemented and trained on GPU)

## Examples

```
python main.py --x1 X1 --x2 X2 --batch Batch --encodeLayer 256 64 32 16 --decodeLayer1 16 64 256 --decodeLayer2 16 20 --pretrain_epochs 400 --maxiter 10000 --gamma 0.1 --update_interval 1 --tol 0.001 --sigma1 2.5 --sigma2 1.5 --embedding_file z.csv --prediction_file pred.csv
```

## Evaluation signals

- The --embedding_file and --prediction_file outputs exist and contain one entry per input cell
- t-SNE of the latent Z colored by marker genes (LYZ, CD8A, NKG7), predicted labels, batch IDs, and true labels renders all required overlay panels
- Most cell types are spatially separated in the latent embedding (qualitative concordance with published Figures 5-6)
- Cells from the two batches are mixed within most clusters, indicating batch-effect removal by the conditional autoencoder
- Convergence is reached within maxiter=10000 at tol=0.001 with pretraining completing 400 epochs

## Limitations

- The number of clusters K must be specified by the user; scMDC provides no automatic estimation
- All experiments were conducted on an Nvidia Tesla P100 (16 G) GPU, which may limit reproducibility on CPU-only systems
- Competing baselines compared in the study (IDEC, SCVIS, Seurat) did not achieve simultaneous cell-type partition and batch-effect removal, so embeddings from those methods should not be expected to match scMDC behavior

## Evidence

- [intro] End-to-end joint optimization with pretraining schedule: "The whole model, including the autoencoder, the KL-loss, and the deep K-means clustering, are optimized simultaneously."
- [discussion] Pretraining epochs split: "We pretrain the autoencoders for 400 epochs before entering the clustering stage."
- [discussion] K-means centroid initialization and user-specified K: "we initialize K centroids by implementing K-means algorithm on the pretrained latent space."
- [discussion] ZINB reconstruction losses per modality: "For all the omics of data, we employ the zero-inflated negative binomial (ZINB) models as the reconstruction loss function28"
- [discussion] GPU requirement: "All experiments of scMDC in this study are conducted on Nvidia Tesla P100 (16 G) GPU."
- [discussion] User must specify K: "After pretraining, the users need to specify the number of clusters (K)."
- [methods] H5 input preparation: "1) Prepare the input data in h5 format. (See readme in 'dataset' folder)"
