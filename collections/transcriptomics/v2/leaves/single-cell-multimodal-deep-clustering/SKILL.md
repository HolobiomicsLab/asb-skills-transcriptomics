---
name: single-cell-multimodal-deep-clustering
description: Use when you need to jointly cluster RNA+ADT (CITE-seq) data and assess
  whether a learned latent space separates cell types while removing batch effects
  across batches.
when_to_use_negative:
- Single-modal scRNA-seq data without an ADT/second modality, where single-modal methods
  like IDEC, Tscan, or SC3 are the appropriate baselines
- When you need automatic estimation of the number of clusters — scMDC requires the
  user to specify K after pretraining
- Data already reduced to a low-dimensional embedding, where clustering should be
  applied directly rather than relearning a joint latent space
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: scMDC
  role: End-to-end multimodal deep model that jointly learns the latent embedding
    and cluster assignments from RNA and ADT data
  repo: https://github.com/xianglin226/scMDC
- name: Scanpy
  role: Preprocessing/normalization of mRNA and ADT data and t-SNE embedding of the
    scMDC latent representation
- name: h5py
  role: Reading the H5 input files containing X1 (RNA), X2 (ADT), Y labels, and Batch
    structures
- name: PyTorch
  role: Framework in which the scMDC model is implemented and trained (GPU recommended)
- name: Seurat
  role: Competing multimodal baseline (WNN-based) for embedding/clustering comparison
  repo: https://github.com/satijalab/seurat
- name: IDEC
  role: Single-modal deep clustering baseline for comparison
  repo: https://github.com/XifengGuo/IDEC
- name: SCVIS
  role: Single-cell dimension-reduction baseline for comparison
  repo: https://github.com/shahcompbio/scvis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:43bb7906e3a1ffeef17b445255395460b1630dec692abf222a4f446ad0381424
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xianglin226/scMDC
  tools:
  - IDEC
  - PyTorch
  - SCVIS
  - Scanpy
  - Seurat
  - h5py
  - scMDC
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# single-cell-multimodal-deep-clustering

## Summary

Runs the scMDC multimodal deep clustering model (PyTorch) on multi-batch CITE-seq datasets such as BMNC and SLN111, exports its latent embedding and predicted cluster labels, and visualizes them with Scanpy t-SNE to show cell-type separation and batch mixing.

## When to use

Use when you need to jointly cluster RNA+ADT (CITE-seq) data and assess whether a learned latent space separates cell types while removing batch effects across batches.

## When NOT to use

- Single-modal scRNA-seq data without an ADT/second modality, where single-modal methods like IDEC, Tscan, or SC3 are the appropriate baselines
- When you need automatic estimation of the number of clusters — scMDC requires the user to specify K after pretraining
- Data already reduced to a low-dimensional embedding, where clustering should be applied directly rather than relearning a joint latent space

## Inputs

- H5 files with X1 (RNA counts), X2 (ADT counts), Y (cell type labels), and Batch structures (e.g., BMNC GSE128639, SLN111 GSE150599)
- Scanpy-preprocessed and normalized mRNA and ADT matrices
- User-specified number of clusters K

## Outputs

- Latent embedding matrix Z from scMDC (--embedding_file)
- Predicted cluster labels per cell (--prediction_file)
- t-SNE 2D embedding rendered as panels colored by LYZ/CD8A/NKG7 expression, predicted labels, batch IDs, and true cell type labels

## How to apply

Load the H5 inputs with h5py, reading X1 (RNA), X2 (ADT), Y (cell type labels), and Batch structures, after normalizing mRNA and ADT separately with Scanpy (filter zero-count genes, size-factor normalization, log transform, scaling). Run scMDC with the CITE-Seq default settings: encodeLayer [256,64,32,16], decodeLayer1 [16,64,256], decodeLayer2 [16,20], pretrain_epochs=400, maxiter=10000, gamma=0.1, update_interval=1, tol=0.001, sigma1=2.5, sigma2=1.5, enabling the --embedding_file and --prediction_file flags to save the latent Z and predicted labels. The model pretrains the ZINB-reconstruction autoencoder (KL loss added for the last 200 of 400 epochs), initializes K centroids via K-means on the pretrained latent space, and then jointly optimizes autoencoder, KL-loss, and deep K-means losses; you must specify the number of clusters K yourself. Embed Z in 2D with Scanpy t-SNE and color by marker-gene expression (LYZ, CD8A, NKG7), predicted cluster labels, batch IDs, and true cell type labels, reproducing the Figure 5-6 panel layout. Validate qualitatively: most cell types should partition into distinct clusters while cells from the two batches mix within clusters.

## Related tools

- **scMDC** (End-to-end multimodal deep model that jointly learns the latent embedding and cluster assignments from RNA and ADT data) — https://github.com/xianglin226/scMDC
- **Scanpy** (Preprocessing/normalization of mRNA and ADT data and t-SNE embedding of the scMDC latent representation)
- **h5py** (Reading the H5 input files containing X1 (RNA), X2 (ADT), Y labels, and Batch structures)
- **PyTorch** (Framework in which the scMDC model is implemented and trained (GPU recommended))
- **Seurat** (Competing multimodal baseline (WNN-based) for embedding/clustering comparison) — https://github.com/satijalab/seurat
- **IDEC** (Single-modal deep clustering baseline for comparison) — https://github.com/XifengGuo/IDEC
- **SCVIS** (Single-cell dimension-reduction baseline for comparison) — https://github.com/shahcompbio/scvis

## Examples

```
python main.py --dataset SLN111 --encodeLayer 256 64 32 16 --decodeLayer1 16 64 256 --decodeLayer2 16 20 --pretrain_epochs 400 --gamma 0.1 --tol 0.001 --embedding_file --prediction_file
```

## Evaluation signals

- The --embedding_file and --prediction_file outputs are written and contain the latent representation and predicted cluster labels for every cell
- The t-SNE plot renders all required overlay panels: marker expression (LYZ, CD8A, NKG7), predicted labels, batch IDs, and true cell type labels
- On multi-batch SLN111 (and BMNC), most cell types form distinct clusters in the t-SNE of the latent space
- Cells from the two batches mix within most clusters, indicating batch-effect removal alongside cell-type partition
- Cluster concordance with published Figures 5-6 can be checked qualitatively against the paper

## Limitations

- The number of clusters K must be specified by the user; scMDC provides no automatic estimation
- All experiments in the study were conducted on an Nvidia Tesla P100 (16 G) GPU, which may limit reproducibility on CPU-only systems
- It is the only method achieving both superior cell type partition and batch effect removal on SLN111, so baselines like IDEC, SCVIS, or Seurat may not jointly satisfy both criteria in comparisons

## Evidence

- [discussion] Preprocessing and normalization of mRNA and ADT data with SCANPY: "The raw CITE-seq data is preprocessed and normalized by the Python package SCANPY48."
- [methods] h5 input format: "1) Prepare the input data in h5 format. (See readme in 'dataset' folder)"
- [intro] batch correction via conditional autoencoder: "by employing a conditional autoencoder framework, scMDC can correct batch effects when analyzing multi-batch data."
- [readme] scMDC purpose: "We develop a novel multimodal deep learning method, scMDC, for single-cell multi-omics data clustering analysis."
- [abstract] joint latent learning: "scMDC is an end-to-end deep model that explicitly characterizes different data sources and jointly learns latent features of deep embedding for clustering analysis"
- [discussion] K must be specified: "After pretraining, the users need to specify the number of clusters (K)."
- [discussion] GPU requirement: "All experiments of scMDC in this study are conducted on Nvidia Tesla P100 (16 G) GPU."
