---
name: single-cell-rna-seq-dimension-reduction
description: Use when noisy scRNA-seq count matrices (with high dropout) need unsupervised
  dimension reduction to 2D for cell-type visualization, k-means clustering benchmarks,
  or lineage/pseudo-time recovery.
when_to_use_negative:
- The goal is traditional bulk RNA-seq analysis or the data are already a low-dimensional
  feature table rather than a raw scRNA-seq count matrix
- Optimal latent dimension must be determined strictly without any prior knowledge,
  since 'the determination of the optimal dimension is a tricky task if prior knowledge
  is limited'
- Compute budget cannot tolerate several hours of single-GPU model learning for datasets
  with several thousand cells
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3315
tools:
- name: VASC
  role: Deep variational autoencoder (vasc.py, class VASC and function vase) performing
    dropout-aware dimension reduction to 2D
  repo: https://github.com/wang-research/VASC
- name: tensorflow
  role: Backend framework (v1.1.0) for training the VASC autoencoder
- name: keras
  role: Keras 2.0.6 API on top of TensorFlow used to build and train the VASC model
- name: sklearn
  role: Runs PCA/t-SNE baselines, k-means clustering on 2D representations, and NMI/ARI
    computation (sklearn.metrics)
- name: h5py
  role: Loads scRNA-seq expression matrices and datasets into numpy arrays
- name: numpy
  role: Array manipulation of expression matrices and embeddings
- name: Anaconda
  role: Recommended distribution for installing Python 3.5+ and the required packages
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_004
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2018.08.003
    title: 'VASC: Dimension Reduction and Visualization of Single-Cell RNA-Seq Data
      by Deep Variational Autoencoder'
  source_content_hash: sha256:f5c94746f1e122ea5ce3dd542bf09abdd5b40388914f0fb13958471f1dc29d6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wang-research/VASC
  tools:
  - Anaconda
  - Python
  - VASC
  - h5py
  - keras
  - numpy
  - sklearn
  - tensorflow
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# single-cell-rna-seq-dimension-reduction

## Summary

Apply VASC, a deep variational autoencoder that models scRNA-seq dropout events with the Gumbel distribution, to learn a 2D nonlinear latent representation of single-cell RNA-seq data for visualization, clustering (NMI/ARI), and developmental trajectory analysis.

## When to use

Use when noisy scRNA-seq count matrices (with high dropout) need unsupervised dimension reduction to 2D for cell-type visualization, k-means clustering benchmarks, or lineage/pseudo-time recovery.

## When NOT to use

- The goal is traditional bulk RNA-seq analysis or the data are already a low-dimensional feature table rather than a raw scRNA-seq count matrix
- Optimal latent dimension must be determined strictly without any prior knowledge, since 'the determination of the optimal dimension is a tricky task if prior knowledge is limited'
- Compute budget cannot tolerate several hours of single-GPU model learning for datasets with several thousand cells

## Inputs

- scRNA-seq expression matrix (count matrix) loadable via h5py/numpy, e.g. Hemberg-group datasets, Pollen dataset, Biase demo dataset
- Known cell-type labels per dataset (for NMI/ARI evaluation)
- Petropoulos human pre-implantation embryo expression matrix, 1529 cells (ArrayExpress E-MTAB-3929)

## Outputs

- 2D latent embedding of cells from the trained variational autoencoder
- NMI and ARI benchmark table vs PCA, t-SNE, ZIFA, SIMLR (e.g. nmi_ari_benchmark.csv)
- 2D visualization colored by stage/lineage with pseudo-time overlay (e.g. loss_curve.png, embedding plots)
- Trained model weights (e.g. vasc_model.h5)

## How to apply

Instantiate the VASC class from vasc.py (a 3-layer encoder of 512-128-32 units with ReLU activations and L1 regularization, Gaussian latent sampling via the re-parameterization trick, and a mirrored decoder with sigmoid output). Add the zero-inflation (ZI) layer that models dropout with Gumbel-softmax using temperature tau in 0.5-1, then compile with RMSprop at learning rate 0.0001 on the tensorflow 1.1.0 / keras 2.0.6 backend. Load expression matrices with h5py/numpy, train roughly 100 epochs (loss converges quickly after the first 100 epochs; larger datasets like Pollen show smaller fluctuations), and extract the 2D embedding. Benchmark by running k-means with the known number of cell types on the embedding and computing NMI/ARI against true labels, comparing to PCA, t-SNE, ZIFA, and SIMLR baselines. Because stochastic optimization introduces run-to-run variation, repeat runs and, for small samples, consider fixing the co-variance matrix.

## Related tools

- **VASC** (Deep variational autoencoder (vasc.py, class VASC and function vase) performing dropout-aware dimension reduction to 2D) — https://github.com/wang-research/VASC
- **tensorflow** (Backend framework (v1.1.0) for training the VASC autoencoder)
- **keras** (Keras 2.0.6 API on top of TensorFlow used to build and train the VASC model)
- **sklearn** (Runs PCA/t-SNE baselines, k-means clustering on 2D representations, and NMI/ARI computation (sklearn.metrics))
- **h5py** (Loads scRNA-seq expression matrices and datasets into numpy arrays)
- **numpy** (Array manipulation of expression matrices and embeddings)
- **Anaconda** (Recommended distribution for installing Python 3.5+ and the required packages)

## Examples

```
from vasc import VASC, vase; embedding = vase(X, latent_dim=2, epochs=100, lr=0.0001, tau=0.5)
```

## Evaluation signals

- Training loss exhibits a converged, monotone-decreasing trajectory within the ~100-epoch regime, with consistent behavior across repeated runs
- On the 20 Hemberg-group datasets, VASC attains best NMI on 15 datasets and best ARI on 17 datasets and always ranks in the top two methods
- On the Petropoulos embryo dataset, the 2D visualization reproduces the Figure 5 layout: E3-E7 stage ordering along pseudo-time, pre-lineage vs TE separation, and mural/polar TE split
- Down-sampling sanity check: NMI/ARI remain comparable on the Pollen dataset when cell sampling exceeds 50%, with degradation only at very low read counts
- Clustering uses k-means with the known number of cell types, and NMI/ARI are computed via sklearn.metrics against true labels

## Limitations

- Stochastic optimization introduces run-to-run variation in the dimension reduction; repeated runs are recommended for consensus
- Determining the optimal latent dimension is difficult without prior knowledge; when the sample size is small it is better to fix the co-variance matrix
- VASC model learning takes several hours on a single-GPU desktop for datasets with several thousand cells
- More complex variational distribution families remain untested

## Evidence

- [intro] it uses the Gumbel distribution to better model the zero and near-zero dropout events: "In addition, it uses the Gumbel distribution to better model the zero and near-zero dropout events."
- [methods] VASC class and vase function in vasc.py: "vasc.py: contains a class VASC and a function vase"
- [methods] Python and package prerequisites: "+ Python 3.5+ + numpy 1.12.1 + h5py 2.7.0 + sklearn 0.18.1 + tensorflow 1.1.0 + keras 2.0.6"
- [readme] README description of VASC: "We proposed VASC (deep Variational Autoencoder for SCRNA-seq data), a deep multi-layer generative model, for the dimension reduction and visualization."
- [readme] README prerequisites: "We recommend to install the newest Anaconda from https://www.continuum.io/downloads."
- [discussion] Stochastic variation and repeated runs: "The stochastic optimization algorithms, used in the VASC model learning, introduce variations in the dimension reduction. Repeated runs are thus recommended"
