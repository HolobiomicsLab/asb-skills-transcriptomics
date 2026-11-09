---
name: variational-autoencoder-dimension-reduction
description: Use when raw single-cell UMI/expression matrices (e.g. 10x Genomics PBMC3k,
  ~2700 cells with high dropout rate) need nonlinear 2D embedding to separate cell
  types or resolve rare sub-populations like FCGR3A+ vs CD14+ monocytes.
when_to_use_negative:
- The input is bulk RNA-seq or already a low-dimensional feature embedding — VASC
  targets single-cell data dominated by dropout events
- You need deterministic, reproducible embeddings from a single run — stochastic optimization
  introduces run-to-run variation
- The sample size is very small and the co-variance matrix cannot be reliably estimated
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: VASC
  role: Core deep variational autoencoder (vasc.py, class VASC and function vase)
    trained on the log2-transformed matrix to produce the 2D embedding
  repo: github.com/wang-research/VASC
- name: h5py
  role: Loads the 10x Genomics PBMC3k UMI count matrix (2700 cells x 1158 genes) from
    the deposited 10x data
- name: keras
  role: Deep-learning API used to build and train the VASC model; run via demo.py
    and config.py following the README prerequisites
  repo: keras-team/keras
- name: tensorflow
  role: GPU backend for the Keras VASC model training
- name: sklearn
  role: Prerequisite library for auxiliary processing/scaling in the VASC pipeline
- name: numpy
  role: Array operations for normalization and log2 transformation of the count matrix
- name: Anaconda
  role: Recommended Python environment for installing the pinned prerequisites (Python
    3.5+, numpy 1.12.1, h5py 2.7.0, sklearn 0.18.1, tensorflow 1.1.0, keras 2.0.6)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.gpb.2018.08.003
    title: 'VASC: Dimension Reduction and Visualization of Single-Cell RNA-Seq Data
      by Deep Variational Autoencoder'
  source_content_hash: sha256:e165a100b64364e3e29ed7cab928c70ff73b492934d70938ae85cbc0694cf0c3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Anaconda
  - Python
  - VASC
  - h5py
  - keras
  - numpy
  - sklearn
  - tensorflow
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# variational-autoencoder-dimension-reduction

## Summary

VASC is a deep variational autoencoder that performs unsupervised dimension reduction and 2D visualization of scRNA-seq data by learning nonlinear hierarchical representations while explicitly modeling zero/near-zero dropout events with the Gumbel distribution.

## When to use

Use when raw single-cell UMI/expression matrices (e.g. 10x Genomics PBMC3k, ~2700 cells with high dropout rate) need nonlinear 2D embedding to separate cell types or resolve rare sub-populations like FCGR3A+ vs CD14+ monocytes.

## When NOT to use

- The input is bulk RNA-seq or already a low-dimensional feature embedding — VASC targets single-cell data dominated by dropout events
- You need deterministic, reproducible embeddings from a single run — stochastic optimization introduces run-to-run variation
- The sample size is very small and the co-variance matrix cannot be reliably estimated

## Inputs

- Raw UMI count matrix from 10x Genomics (e.g. PBMC3k: 2700 cells x 1158 selected genes, loaded via h5py)
- TPM-like normalized, log2-transformed scRNA-seq expression matrix (rows = cells, columns = genes)
- Small demonstration dataset such as the included Biase dataset

## Outputs

- 2D latent embedding of all cells from the trained VASC encoder
- 2D scatter visualization distinguishing cell types (B cells, CD4+ T cells, CD8+ T cells, NK cells) and monocyte sub-populations (FCGR3A+ vs CD14+)
- Candidate low-dimensional representation for downstream cell lineage and sub-population analysis

## How to apply

Load the UMI count matrix (e.g. the 10x Genomics PBMC3k matrix of 2700 cells x 1158 selected genes) via h5py, apply a UMI-to-TPM-like per-cell normalization, then log2-transform to stabilize the dropout-dominated distribution before training. Train the VASC model (class VASC and function vase in vasc.py, driven by demo.py and config.py) on the log2-transformed matrix using Keras with the TensorFlow backend; the Gumbel distribution in the loss models zero and near-zero dropout events while the multi-layer encoder learns the nonlinear hierarchy. If a sub-population (e.g. the monocyte cluster) needs finer resolution, re-run VASC restricted to those cells and extract the 2D latent embedding from the trained encoder. Judgment is on the 2D scatter: major cell types should separate, and the monocyte re-run should resolve FCGR3A+ and CD14+ sub-clusters as in Figure 6 of the paper.

## Related tools

- **VASC** (Core deep variational autoencoder (vasc.py, class VASC and function vase) trained on the log2-transformed matrix to produce the 2D embedding) — github.com/wang-research/VASC
- **h5py** (Loads the 10x Genomics PBMC3k UMI count matrix (2700 cells x 1158 genes) from the deposited 10x data)
- **keras** (Deep-learning API used to build and train the VASC model; run via demo.py and config.py following the README prerequisites) — keras-team/keras
- **tensorflow** (GPU backend for the Keras VASC model training)
- **sklearn** (Prerequisite library for auxiliary processing/scaling in the VASC pipeline)
- **numpy** (Array operations for normalization and log2 transformation of the count matrix)
- **Anaconda** (Recommended Python environment for installing the pinned prerequisites (Python 3.5+, numpy 1.12.1, h5py 2.7.0, sklearn 0.18.1, tensorflow 1.1.0, keras 2.0.6))

## Examples

```
python demo.py  # trains VASC (vasc.py, class VASC) on the demo data using demo.py/config.py with Keras + TensorFlow backend
```

## Evaluation signals

- The 2D visualization separates the major PBMC cell types (B cells, CD4+ T cells, CD8+ T cells, NK cells) as in the paper's Figure 6
- A second VASC run restricted to the monocyte cluster resolves FCGR3A+ and CD14+ sub-populations
- Output embedding is exactly 2D (one latent coordinate pair per input cell, e.g. 2700 points for PBMC3k)
- Pre-processing invariants hold: per-cell TPM-like normalization followed by log2 transformation before training
- VASC is trained on more than 20 datasets' worth of evidence for superiority over four state-of-the-art dimension reduction methods — compare against PCA/t-SNE baselines where possible

## Limitations

- Determining the optimal latent dimension is tricky without prior knowledge
- With small sample sizes it is better to fix the co-variance matrix
- Stochastic optimization introduces run-to-run variation in the dimension reduction; repeated runs are recommended for consensus
- VASC model learning can take several hours on a single-GPU desktop for datasets with several thousand cells
- More complex variational distribution families remain untested

## Evidence

- [abstract] abstract: "we proposed the deep variational autoencoder for scRNA-seq data (VASC), a deep multi-layer generative model, for the unsupervised dimension reduction and visualization of scRNA-seq data"
- [intro] intro: "In addition, it uses the Gumbel distribution to better model the zero and near-zero dropout events."
- [methods] methods: "We gave a demo.py and config.py to demonstrate the use of VASC."
- [readme] readme: "vasc.py: contains a class VASC and a function vase"
- [discussion] discussion: "The stochastic optimization algorithms, used in the VASC model learning, introduce variations in the dimension reduction. Repeated runs are thus recommended"
