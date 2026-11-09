---
name: multi-batch-cite-seq-clustering
description: Use when input is multi-batch multimodal single-cell data (H5 files with
  X1 RNA, X2 ADT, Y cell-type labels, Batch), and the goal is simultaneous cell-type
  clustering plus batch-effect removal in a latent embedding.
when_to_use_negative:
- When data are single-modal (RNA only) and multimodal modeling is unnecessary — methods
  like SC3 or Tscan suffice
- When clusters/cell-type labels are already available and only batch integration
  (e.g., Seurat V4 WNN-based integration) is needed, not joint clustering
- On CPU-only systems when runtime is critical, since scMDC experiments in the study
  were run on an Nvidia Tesla P100 GPU
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scMDC
  role: 'Primary model: end-to-end multimodal deep clustering with ZINB loss and conditional
    autoencoder for batch correction; produces latent embedding and cluster labels'
  repo: https://github.com/xianglin226/scMDC
- name: Scanpy
  role: Preprocessing/normalization of mRNA and ADT data and t-SNE visualization of
    the latent embedding
- name: h5py
  role: Reading the H5 input files containing X1, X2, Y, and Batch structures
- name: Pytorch
  role: Implementation framework for running scMDC training and inference
- name: IDEC
  role: Competing single-modal deep clustering baseline for embedding/clustering comparison
  repo: https://github.com/XifengGuo/IDEC
- name: SCVIS
  role: Competing dimension-reduction baseline for comparison
  repo: https://github.com/shahcompbio/scvis
- name: Seurat
  role: Competing multimodal clustering baseline (KNN graph in PCA space; V4 WNN)
    and source of the BMNC 'bmcite' reference data
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:71aab5741abb669f560736f35aba3667ad732b438ef7aeb2e9445af39e93cf87
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# multi-batch-cite-seq-clustering

## Summary

Joint clustering and latent-embedding learning for multi-batch CITE-seq data using scMDC, a multimodal deep model with ZINB reconstruction loss and a conditional autoencoder for batch-effect correction, followed by t-SNE visualization of cell-type separation and batch mixing.

## When to use

Use when input is multi-batch multimodal single-cell data (H5 files with X1 RNA, X2 ADT, Y cell-type labels, Batch), and the goal is simultaneous cell-type clustering plus batch-effect removal in a latent embedding.

## When NOT to use

- When data are single-modal (RNA only) and multimodal modeling is unnecessary — methods like SC3 or Tscan suffice
- When clusters/cell-type labels are already available and only batch integration (e.g., Seurat V4 WNN-based integration) is needed, not joint clustering
- On CPU-only systems when runtime is critical, since scMDC experiments in the study were run on an Nvidia Tesla P100 GPU

## Inputs

- H5 files containing X1 (RNA counts), X2 (ADT counts), Y (cell type labels), and Batch structures (e.g., BMNC/GSE128639, SLN111 & SLN208/GSE150599, CBMN/GSE100866)
- User-specified number of clusters K
- Marker gene names for visualization (e.g., LYZ, CD8A, NKG7)

## Outputs

- Latent embedding matrix Z (saved via --embedding_file)
- Predicted cluster labels per cell (saved via --prediction_file)
- t-SNE panels colored by marker expression, predicted labels, batch IDs, and true cell types (Figures 5-6 layouts)

## How to apply

Load the H5 inputs with h5py, reading X1 (RNA), X2 (ADT), Y, and Batch, and preprocess each modality with SCANPY (filter zero-count genes/ADTs, size-factor normalization, log transform, scaling). Run scMDC with CITE-Seq defaults — encodeLayer [256,64,32,16], decodeLayer1 [16,64,256], decodeLayer2 [16,20], pretrain_epochs=400, maxiter=10000, gamma=0.1, update_interval=1, tol=0.001, sigma1=2.5, sigma2=1.5 — enabling --embedding_file and --prediction_file to save the latent Z and predicted labels; autoencoders are pretrained 400 epochs (reconstruction first 200, KL loss added for the remaining 200) and K-means initializes K centroids on the pretrained latent space before joint optimization. You must supply the number of clusters K explicitly since no automatic estimation exists. Embed Z in 2D with t-SNE via Scanpy and color by marker genes (LYZ, CD8A, NKG7), predicted labels, batch IDs, and true labels. Optionally compare against IDEC, SCVIS, or Seurat baselines; on multi-batch SLN111 and BMNC, scMDC is reported as the only method achieving both superior cell-type partition and batch-effect removal.

## Related tools

- **scMDC** (Primary model: end-to-end multimodal deep clustering with ZINB loss and conditional autoencoder for batch correction; produces latent embedding and cluster labels) — https://github.com/xianglin226/scMDC
- **Scanpy** (Preprocessing/normalization of mRNA and ADT data and t-SNE visualization of the latent embedding)
- **h5py** (Reading the H5 input files containing X1, X2, Y, and Batch structures)
- **Pytorch** (Implementation framework for running scMDC training and inference)
- **IDEC** (Competing single-modal deep clustering baseline for embedding/clustering comparison) — https://github.com/XifengGuo/IDEC
- **SCVIS** (Competing dimension-reduction baseline for comparison) — https://github.com/shahcompbio/scvis
- **Seurat** (Competing multimodal clustering baseline (KNN graph in PCA space; V4 WNN) and source of the BMNC 'bmcite' reference data) — https://github.com/satijalab/seurat

## Examples

```
python run_scMDC.py --data SLN111.h5 --encodeLayer 256 64 32 16 --decodeLayer1 16 64 256 --decodeLayer2 16 20 --pretrain_epochs 400 --gamma 0.1 --embedding_file --prediction_file
```

## Evaluation signals

- Both --embedding_file and --prediction_file outputs were written successfully
- t-SNE plot renders all required overlay panels: marker gene expression (LYZ, CD8A, NKG7), predicted labels, batch IDs, and true cell types
- Qualitative cluster concordance with true labels, matching the published Figure 5-6 panel layouts on BMNC/SLN111
- Cells from the two batches are mixed within most clusters, indicating batch-effect removal
- Loss converges to tol=0.001 within maxiter=10000 during the clustering stage

## Limitations

- The number of clusters K must be specified by the user; scMDC provides no automatic estimation
- All study experiments required GPU hardware (Nvidia Tesla P100, 16 GB), which may limit CPU-only reproducibility
- Cell types may not be fully separated in every cluster; the paper reports separation of 'most' cell types with batch mixing in 'most' clusters

## Evidence

- [intro] conditional autoencoder batch correction: "by employing a conditional autoencoder framework, scMDC can correct batch effects when analyzing multi-batch data."
- [discussion] pretraining protocol: "We pretrain the autoencoders for 400 epochs before entering the clustering stage."
- [discussion] K-means centroid initialization: "we initialize K centroids by implementing K-means algorithm on the pretrained latent space."
- [discussion] K must be user-specified: "After pretraining, the users need to specify the number of clusters (K)."
- [discussion] zero-count filtering: "Specifically, the genes and ADTs with no count are filtered out."
- [discussion] SCANPY preprocessing: "The raw CITE-seq data is preprocessed and normalized by the Python package SCANPY48. mRNA and ADT data are normalized separately but using the same method."
- [readme] scMDC purpose: "We develop a novel multimodal deep learning method, scMDC, for single-cell multi-omics data clustering analysis."
- [discussion] GPU requirement: "All experiments of scMDC in this study are conducted on Nvidia Tesla P100 (16 G) GPU."
