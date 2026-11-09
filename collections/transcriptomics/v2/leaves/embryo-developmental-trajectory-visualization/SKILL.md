---
name: embryo-developmental-trajectory-visualization
description: Use when scRNA-seq data from staged embryos (e.g. E3–E7 pre-implantation
  stages) must be visualized in 2D to check whether developmental stage ordering,
  lineage separation, and pseudo-time structure are recovered, and to compare against
  PCA, t-SNE, ZIFA, or SIMLR.
when_to_use_negative:
- The input is already a low-dimensional embedding or visualization, not a raw scRNA-seq
  expression matrix with dropout events
- The analysis goal is marker-gene identification or differential expression rather
  than 2D trajectory visualization
- The dataset is very small and fixing the covariance matrix would be required; the
  article notes that when the sample size is small it is better to fix the co-variance
  matrix
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: VASC
  role: Deep variational autoencoder trained on the embryo expression matrix to produce
    the 2D embedding and pseudo-time trajectory
  repo: github.com/wang-research/VASC
- name: numpy
  role: Loading and preprocessing the embryo expression matrix
- name: h5py
  role: Reading the h5 expression matrix of embryo cells
- name: tensorflow
  role: Backend for training the VASC model (version 1.1.0)
- name: keras
  role: Backend for the VASC variational autoencoder (version 2.0.6)
  repo: keras-team/keras
- name: sklearn
  role: Supporting preprocessing/comparison utilities (version 0.18.1)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2018.08.003
    title: 'VASC: Dimension Reduction and Visualization of Single-Cell RNA-Seq Data
      by Deep Variational Autoencoder'
  source_content_hash: sha256:c537fbfcdcf2d7b46156cf0b1a0d46d52b0473435fe0230ee0317d2128b07dc0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Anaconda
  - VASC
  - h5py
  - keras
  - numpy
  - sklearn
  - tensorflow
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# embryo-developmental-trajectory-visualization

## Summary

Visualize single-cell RNA-seq from human pre-implantation embryos (e.g. the 1529-cell Petropoulos dataset) as a 2D embedding from the VASC variational autoencoder, colored by developmental stage (E3–E7), lineage (ICM/PE/EPI vs TE), and overlaid pseudo-time to recover developmental trajectory structure.

## When to use

Use when scRNA-seq data from staged embryos (e.g. E3–E7 pre-implantation stages) must be visualized in 2D to check whether developmental stage ordering, lineage separation, and pseudo-time structure are recovered, and to compare against PCA, t-SNE, ZIFA, or SIMLR.

## When NOT to use

- The input is already a low-dimensional embedding or visualization, not a raw scRNA-seq expression matrix with dropout events
- The analysis goal is marker-gene identification or differential expression rather than 2D trajectory visualization
- The dataset is very small and fixing the covariance matrix would be required; the article notes that when the sample size is small it is better to fix the co-variance matrix

## Inputs

- scRNA-seq expression matrix of pre-implantation embryo cells (e.g. 1529 cells from Petropoulos dataset, ArrayExpress E-MTAB-3929)
- Per-cell developmental stage labels (E3–E7)
- Per-cell lineage annotations (ICM/PE/EPI vs TE)

## Outputs

- 2D embedding of all embryo cells from the trained VASC variational autoencoder
- Matplotlib visualization colored by developmental stage (E3–E7) and lineage (ICM/PE/EPI vs TE) with overlaid pseudo-time trajectory
- Qualitative comparison layout against PCA, t-SNE, ZIFA, and SIMLR embeddings

## How to apply

Download the embryo expression matrix (e.g. 1529 cells from ArrayExpress E-MTAB-3929) and load it with numpy and h5py. Train the VASC model (class VASC in vasc.py on a tensorflow 1.1.0 / keras 2.0.6 backend), which uses the Gumbel distribution to model zero and near-zero dropout events while learning a nonlinear hierarchical representation. Extract the 2D embedding of the cells from the trained variational autoencoder and plot it with matplotlib, coloring cells by stage (E3–E7) and lineage (ICM/PE/EPI vs TE) and overlaying the inferred pseudo-time trajectory. Inspect for recovery of the E3–E7 stage ordering along pseudo-time, separation of pre-lineage cells from TE cells, and splitting of the TE compartment into mural and polar sub-populations. Because stochastic optimization introduces run-to-run variation, repeat the VASC run and confirm the 2D visualization qualitatively matches the published Figure 5 layout.

## Related tools

- **VASC** (Deep variational autoencoder trained on the embryo expression matrix to produce the 2D embedding and pseudo-time trajectory) — github.com/wang-research/VASC
- **numpy** (Loading and preprocessing the embryo expression matrix)
- **h5py** (Reading the h5 expression matrix of embryo cells)
- **tensorflow** (Backend for training the VASC model (version 1.1.0))
- **keras** (Backend for the VASC variational autoencoder (version 2.0.6)) — keras-team/keras
- **sklearn** (Supporting preprocessing/comparison utilities (version 0.18.1))

## Examples

```
python demo.py  # train the VASC class from vasc.py (tensorflow 1.1.0 / keras 2.0.6) on the embryo expression matrix and extract the 2D embedding
```

## Evaluation signals

- The 2D embedding recovers the known E3–E7 developmental stage ordering along the pseudo-time axis
- Pre-lineage cells (ICM/PE/EPI) separate from TE cells in the 2D plot
- The TE compartment splits into mural and polar sub-populations in the embedding
- The E3 cells are placed inside the trajectory (unlike t-SNE, which misplaced E3 cells outside the trajectory)
- Repeated VASC runs (stochastic optimization) qualitatively reproduce the Figure 5 layout, confirming stability

## Limitations

- Stochastic optimization in VASC model learning introduces run-to-run variation in the dimension reduction; repeated runs are recommended
- VASC model learning takes several hours on a desktop computer with a single GPU card for datasets with several thousand cells
- The determination of the optimal latent dimension is a tricky task if prior knowledge is limited
- When the sample size is small, it is better to fix the co-variance matrix

## Evidence

- [results] Petropoulos 2D recovery finding: [evidence span withheld — anchor status: not_found]
- [abstract] VASC purpose: "we proposed the deep variational autoencoder for scRNA-seq data (VASC), a deep multi-layer generative model, for the unsupervised dimension reduction and visualization of scRNA-seq data"
- [intro] Gumbel dropout modeling: "In addition, it uses the Gumbel distribution to better model the zero and near-zero dropout events."
- [readme] prerequisites: "+ Python 3.5+
+ numpy 1.12.1
+ h5py 2.7.0
+ sklearn 0.18.1
+ tensorflow 1.1.0
+ keras 2.0.6"
- [discussion] repeated runs needed: "The stochastic optimization algorithms, used in the VASC model learning, introduce variations in the dimension reduction. Repeated runs are thus recommended"
- [discussion] embryo dynamics case study: "The application on a dataset of the human pre-implantation embryo development shows that VASC can re-establish the cell dynamics"
