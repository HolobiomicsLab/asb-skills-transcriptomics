---
name: dropout-event-modeling-with-gumbel-distribution
description: Use when input is a noisy single-cell RNA-seq expression matrix with
  many zero/near-zero values that would mislead PCA or t-SNE, and the goal is dropout-aware
  low-dimensional embedding or 2D visualization of cell populations.
when_to_use_negative:
- The data are bulk RNA-seq or otherwise lack the large fraction of zero/near-zero
  dropout events that Gumbel modeling targets — traditional dimension reduction may
  suffice
- The dataset is very small and the co-variance matrix cannot be reliably estimated;
  the article notes it is better to fix the co-variance matrix when sample size is
  small
- Optimal embedding dimension must be chosen without prior knowledge — the article
  flags this determination as a tricky task
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: VASC
  role: Deep variational autoencoder whose Gumbel-distributed dropout component performs
    the dropout-event modeling; provides the VASC class and vase function in vasc.py
  repo: github.com/wang-research/VASC
- name: tensorflow 1.1.0
  role: Backend for training the VASC generative model
- name: keras 2.0.6
  role: High-level deep-learning API over TensorFlow used to implement the VASC autoencoder
  repo: keras-team/keras
- name: numpy 1.12.1
  role: Loading and preprocessing the expression matrix
- name: h5py 2.7.0
  role: Reading h5py-format expression data files
- name: sklearn 0.18.1
  role: Auxiliary preprocessing/analysis functions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2018.08.003
    title: 'VASC: Dimension Reduction and Visualization of Single-Cell RNA-Seq Data
      by Deep Variational Autoencoder'
  source_content_hash: sha256:985d47a57441a607d0c312c27d5ee1d0d651dde7aa6f7f7f2dc15856f36f4bdd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Anaconda
  - VASC
  - h5py
  - h5py 2.7.0
  - keras
  - keras 2.0.6
  - numpy
  - numpy 1.12.1
  - sklearn
  - sklearn 0.18.1
  - tensorflow
  - tensorflow 1.1.0
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# dropout-event-modeling-with-gumbel-distribution

## Summary

Explicitly model zero and near-zero dropout events in scRNA-seq data by placing a Gumbel distribution on dropout events inside a deep variational autoencoder (VASC), so the learned low-dimensional representation is not distorted by single-cell technical dropouts. This yields nonlinear hierarchical feature representations suitable for 2D visualization and downstream lineage/sub-population analysis.

## When to use

Use when input is a noisy single-cell RNA-seq expression matrix with many zero/near-zero values that would mislead PCA or t-SNE, and the goal is dropout-aware low-dimensional embedding or 2D visualization of cell populations.

## When NOT to use

- The data are bulk RNA-seq or otherwise lack the large fraction of zero/near-zero dropout events that Gumbel modeling targets — traditional dimension reduction may suffice
- The dataset is very small and the co-variance matrix cannot be reliably estimated; the article notes it is better to fix the co-variance matrix when sample size is small
- Optimal embedding dimension must be chosen without prior knowledge — the article flags this determination as a tricky task

## Inputs

- Single-cell RNA-seq gene expression matrix (e.g., h5py-loadable expression values), such as the Petropoulos human pre-implantation embryo dataset (1529 cells, ArrayExpress E-MTAB-3929) or the small Biase demo dataset
- Optional cell metadata for coloring the embedding (developmental stage labels E3–E7, lineage annotations ICM/PE/EPI vs TE)

## Outputs

- 2D embedding of cells from the trained variational autoencoder (for visualization colored by stage/lineage)
- Nonlinear hierarchical low-dimensional representation of the input scRNA-seq data
- Candidate marker genes associated with the modeled cell dynamics (e.g., early embryo development)

## How to apply

Set up the VASC environment (Python 3.5+, numpy 1.12.1, h5py 2.7.0, sklearn 0.18.1, tensorflow 1.1.0, keras 2.0.6, preferably via Anaconda). Load the raw single-cell gene expression matrix (e.g., the 1529-cell human pre-implantation embryo data, or the included Biase demo dataset) with numpy/h5py, then instantiate the VASC class from vasc.py and train the deep multi-layer generative model; the Gumbel distribution is used to model the zero and near-zero dropout events while the variational autoencoder captures nonlinear variations and learns a hierarchical representation. Extract the 2D embedding and visualize it (e.g., coloring cells by developmental stage E3–E7 or lineage annotation), optionally overlaying a pseudo-time trajectory. Because stochastic optimization introduces run-to-run variation in the dimension reduction, repeat the run and verify the qualitative layout is consistent.

## Related tools

- **VASC** (Deep variational autoencoder whose Gumbel-distributed dropout component performs the dropout-event modeling; provides the VASC class and vase function in vasc.py) — github.com/wang-research/VASC
- **tensorflow 1.1.0** (Backend for training the VASC generative model)
- **keras 2.0.6** (High-level deep-learning API over TensorFlow used to implement the VASC autoencoder) — keras-team/keras
- **numpy 1.12.1** (Loading and preprocessing the expression matrix)
- **h5py 2.7.0** (Reading h5py-format expression data files)
- **sklearn 0.18.1** (Auxiliary preprocessing/analysis functions)

## Examples

```
pip install numpy==1.12.1 h5py==2.7.0 sklearn==0.18.1 tensorflow==1.1.0 keras==2.0.6; python demo.py
```

## Evaluation signals

- The 2D embedding recovers the known developmental-stage ordering E3–E7 along pseudo-time (e.g., the Petropoulos embryo dataset layout, outperforming PCA, ZIFA, SIMLR, and t-SNE, with t-SNE misplacing E3 cells outside the trajectory)
- Pre-lineage cells (ICM/PE/EPI) separate from TE cells, and the TE compartment splits into mural and polar sub-populations in the visualization
- Repeat the stochastic run and confirm the dimension-reduction result is qualitatively stable, since repeated runs are recommended
- Very rare cell populations are still represented in the 2D visualization rather than being absorbed by dropout noise

## Limitations

- The stochastic optimization algorithms introduce run-to-run variations in the dimension reduction; repeated runs are recommended
- VASC model learning takes several hours on a desktop-level computer with a single GPU for datasets with several thousand cells
- The determination of the optimal embedding dimension is tricky if prior knowledge is limited
- When the sample size is small, it is better to fix the co-variance matrix; more complex variational distribution families remain untested

## Evidence

- [intro] Gumbel dropout modeling rationale: "In addition, it uses the Gumbel distribution to better model the zero and near-zero dropout events."
- [abstract] Dropout modeling plus hierarchical representation: "VASC can explicitly model the dropout events and find the nonlinear hierarchical feature representations of the original data"
- [discussion] Run-to-run variation caveat: "The stochastic optimization algorithms, used in the VASC model learning, introduce variations in the dimension reduction. Repeated runs are thus recommended"
- [methods] Implementation entry points: "vasc.py: contains a class VASC and a function vase"
- [readme] README confirmation of dropout modeling: "It can do nonlinear hierarchical feature representations and model the dropout events of scRNA-seq data."
