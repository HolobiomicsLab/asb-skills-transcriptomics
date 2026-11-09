---
name: cell-population-2d-visualization
description: Use when scRNA-seq UMI count data with high dropout (e.g. PBMC3k 2700
  cells x 1158 genes) needs unsupervised dimension reduction to a 2D visualization
  that distinguishes cell types or resolves sub-populations such as FCGR3A+ vs CD14+
  monocytes.
when_to_use_negative:
- Input is already a low-dimensional embedding or a feature/marker table rather than
  a raw or normalized expression matrix — VASC operates on the expression matrix to
  learn its own nonlinear hierarchical representation
- The goal is bulk RNA-seq or traditional low-dropout data analysis where conventional
  dimension reduction (PCA, t-SNE) suffices; VASC is designed for noisy scRNA-seq
  data with technical dropout events
- Fast, deterministic 2D coordinates are required for small samples without GPU access
  — VASC model learning takes several hours on a single-GPU desktop for several-thousand-cell
  datasets and stochastic optimization introduces run-to-run variation
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: VASC
  role: Deep variational autoencoder (vasc.py, class VASC and function vase) that
    performs unsupervised dimension reduction and models dropout events with a Gumbel
    distribution to produce the 2D embedding
  repo: https://github.com/wang-research/VASC
- name: keras
  role: Deep learning framework used to train the VASC autoencoder (version 2.0.6
    in the published configuration)
  repo: https://github.com/keras-team/keras
- name: tensorflow
  role: Backend for Keras (version 1.1.0) used for GPU-accelerated VASC model learning
- name: h5py
  role: Reads the deposited 10x Genomics UMI count matrix (h5 format) into numpy arrays
- name: sklearn
  role: Auxiliary machine-learning utilities (version 0.18.1) for handling embeddings
- name: Anaconda
  role: Recommended Python distribution/environment manager for installing the required
    package versions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2018.08.003
    title: 'VASC: Dimension Reduction and Visualization of Single-Cell RNA-Seq Data
      by Deep Variational Autoencoder'
  source_content_hash: sha256:bed773bbd737996d4ada1a44c33abc1999b6fb7acf78008f66e53df961cad7fd
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# cell-population-2d-visualization

## Summary

Apply VASC, a deep variational autoencoder that explicitly models dropout events with a Gumbel distribution, to obtain a 2D latent embedding of scRNA-seq data that separates cell types and sub-populations, as demonstrated on the 10x Genomics PBMC3k dataset.

## When to use

Use when scRNA-seq UMI count data with high dropout (e.g. PBMC3k 2700 cells x 1158 genes) needs unsupervised dimension reduction to a 2D visualization that distinguishes cell types or resolves sub-populations such as FCGR3A+ vs CD14+ monocytes.

## When NOT to use

- Input is already a low-dimensional embedding or a feature/marker table rather than a raw or normalized expression matrix — VASC operates on the expression matrix to learn its own nonlinear hierarchical representation
- The goal is bulk RNA-seq or traditional low-dropout data analysis where conventional dimension reduction (PCA, t-SNE) suffices; VASC is designed for noisy scRNA-seq data with technical dropout events
- Fast, deterministic 2D coordinates are required for small samples without GPU access — VASC model learning takes several hours on a single-GPU desktop for several-thousand-cell datasets and stochastic optimization introduces run-to-run variation

## Inputs

- Raw scRNA-seq UMI count matrix (e.g. 10x Genomics PBMC3k: 2700 cells x 1158 selected genes, loaded via h5py from the deposited 10x data)
- Optional per-cell labels (B cells, CD4+ T cells, CD8+ T cells, NK cells, monocytes) for coloring/validating the 2D scatter

## Outputs

- 2D latent embedding of all cells from the trained VASC encoder
- 2D scatter visualization showing separated cell types (B, CD4+ T, CD8+ T, NK cells) and, on re-runs, FCGR3A+ vs CD14+ monocyte sub-clusters (Figure 6-style result)

## How to apply

Load the raw UMI count matrix (e.g. the 10x Genomics PBMC3k matrix of 2700 cells x 1158 selected genes) with h5py in an Anaconda environment, then apply a UMI-to-TPM-like per-cell normalization and log2-transform the result to stabilize the dropout-dominated distribution before training. Train the VASC model (vasc.py, class VASC; a deep multi-layer generative variational autoencoder using the Gumbel distribution to model zero and near-zero dropout events) with Keras on a TensorFlow backend, GPU recommended, following the demo.py/config.py execution pattern; expect several hours of training on a single-GPU desktop for datasets with several thousand cells. Extract the 2D latent embedding from the trained encoder for all cells and render it as a 2D scatter plot colored by known labels where available. To resolve finer structure, re-run VASC restricted to a subset of cells (e.g. monocytes only) and visualize the new 2D embedding. Accept the result when major cell types are separated in the 2D plot and, on re-runs, sub-populations (FCGR3A+ vs CD14+ monocytes) form distinct clusters.

## Related tools

- **VASC** (Deep variational autoencoder (vasc.py, class VASC and function vase) that performs unsupervised dimension reduction and models dropout events with a Gumbel distribution to produce the 2D embedding) — https://github.com/wang-research/VASC
- **keras** (Deep learning framework used to train the VASC autoencoder (version 2.0.6 in the published configuration)) — https://github.com/keras-team/keras
- **tensorflow** (Backend for Keras (version 1.1.0) used for GPU-accelerated VASC model learning)
- **h5py** (Reads the deposited 10x Genomics UMI count matrix (h5 format) into numpy arrays)
- **sklearn** (Auxiliary machine-learning utilities (version 0.18.1) for handling embeddings)
- **Anaconda** (Recommended Python distribution/environment manager for installing the required package versions)

## Examples

```
import h5py, numpy as np; f = h5py.File('pbmc3k.h5', 'r'); X = np.log2(normalize_umi(f['matrix'][:])); from vasc import VASC; emb = VASC().fit_transform(X)
```

## Evaluation signals

- The 2D scatter of the encoder embedding clearly separates major PBMC cell types: B cells, CD4+ T cells, CD8+ T cells, and NK cells
- Re-running VASC on the monocyte-only subset resolves two distinct clusters corresponding to FCGR3A+ and CD14+ monocytes, consistent with Figure 6
- The input matrix matches expected dimensions (2700 cells x 1158 selected genes for PBMC3k) after h5py loading and UMI-to-TPM-like normalization plus log2 transformation
- Training completes without dropout-model errors, confirming the Gumbel-distribution dropout layer was applied on the log2-transformed matrix
- Because stochastic optimization introduces run-to-run variation, repeated runs should yield broadly consistent cluster structure in the 2D visualization

## Limitations

- Stochastic optimization algorithms used in VASC model learning introduce variations in the dimension reduction; repeated runs are recommended for consensus
- VASC model learning costs several hours on a desktop-level computer with a single GPU card for datasets with several thousand cells
- Determining the optimal latent dimension is difficult without prior knowledge; when the sample size is small, it is better to fix the co-variance matrix
- More complex variational distribution families remain untested

## Evidence

- [abstract] VASC clearly distinguishes most cell types and further separates monocytes into FCGR3A+ and CD14+ sub-populations: "VASC also performs well on a 10× Genomics dataset with more cells and higher dropout rate"
- [methods] vasc.py contains the class VASC used to train the autoencoder: "vasc.py: contains a class VASC and a function vase"
- [intro] Gumbel distribution models dropout events in the VASC model: "In addition, it uses the Gumbel distribution to better model the zero and near-zero dropout events."
- [discussion] Training cost warning for several-thousand-cell datasets on single-GPU desktops: "it costs several hours for the VASC model learning using a desktop-level computer with single GPU card"
- [discussion] Repeated runs recommended due to stochastic optimization variation: "The stochastic optimization algorithms, used in the VASC model learning, introduce variations in the dimension reduction. Repeated runs are thus recommended"
- [readme] README prerequisites list the exact package versions for reproducing VASC runs: "+ Python 3.5+
+ numpy 1.12.1
+ h5py 2.7.0
+ sklearn 0.18.1
+ tensorflow 1.1.0
+ keras 2.0.6"
