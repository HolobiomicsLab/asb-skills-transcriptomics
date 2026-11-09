---
name: cell-subpopulation-identification-in-2d-embedding
description: Use when scRNA-seq data contain known or suspected heterogeneous cell
  populations (e.g., developmental stages E3–E7 or lineage compartments like ICM/PE/EPI
  vs TE) that should be recovered as separable clusters in a low-dimensional 2D visualization.
when_to_use_negative:
- The input is bulk RNA-seq or already a low-dimensional feature table — VASC's Gumbel-based
  dropout modeling targets single-cell data where zero and near-zero dropout events
  dominate.
- The goal is a precise, deterministic coordinate output for downstream quantitative
  comparison rather than exploratory visualization — stochastic optimization introduces
  run-to-run variation in the dimension reduction.
- Prior knowledge is too limited to choose the embedding dimension — the determination
  of the optimal dimension is described as a tricky task without prior knowledge.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0199
tools:
- name: VASC
  role: deep variational autoencoder trained via vasc.py to produce the 2D representation
    in which sub-populations are identified
  repo: github.com/wang-research/VASC
- name: tensorflow
  role: 1.1.0 backend on which the VASC model is trained
- name: keras
  role: 2.0.6 backend used alongside tensorflow to build and fit the autoencoder
  repo: https://github.com/keras-team/keras
- name: numpy
  role: 1.12.1 — loads and preprocesses the expression matrix before training
- name: h5py
  role: 2.7.0 — reads the expression matrix files
- name: sklearn
  role: 0.18.1 — supporting preprocessing/comparison utilities for the embedding workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2018.08.003
    title: 'VASC: Dimension Reduction and Visualization of Single-Cell RNA-Seq Data
      by Deep Variational Autoencoder'
  source_content_hash: sha256:2fbd5eaf5a2d19e75b9f16a28bc58f49ecfed3a4a619930dae355587f2b38b24
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# cell-subpopulation-identification-in-2d-embedding

## Summary

Identify and visually validate cell sub-populations and lineages by inspecting the 2D embedding produced by VASC, a deep variational autoencoder that models scRNA-seq dropout events (zero and near-zero values) via a Gumbel distribution. The 2D representation recovers known populations more precisely than PCA, ZIFA, and SIMLR and more completely than t-SNE.

## When to use

Use when scRNA-seq data contain known or suspected heterogeneous cell populations (e.g., developmental stages E3–E7 or lineage compartments like ICM/PE/EPI vs TE) that should be recovered as separable clusters in a low-dimensional 2D visualization.

## When NOT to use

- The input is bulk RNA-seq or already a low-dimensional feature table — VASC's Gumbel-based dropout modeling targets single-cell data where zero and near-zero dropout events dominate.
- The goal is a precise, deterministic coordinate output for downstream quantitative comparison rather than exploratory visualization — stochastic optimization introduces run-to-run variation in the dimension reduction.
- Prior knowledge is too limited to choose the embedding dimension — the determination of the optimal dimension is described as a tricky task without prior knowledge.

## Inputs

- scRNA-seq gene expression matrix (e.g., 1529 human pre-implantation embryo cells from E-MTAB-3929)
- per-cell annotation labels (developmental stage, lineage such as ICM/PE/EPI vs TE) for coloring and validation
- vasc.py containing the VASC class and vase function with helpers.py auxiliary functions

## Outputs

- 2D embedding coordinates for every cell from the trained variational autoencoder
- matplotlib-style 2D visualization with cells colored by sub-population/lineage annotation and pseudo-time trajectory overlaid
- qualitative assessment of sub-population recovery (e.g., E3–E7 ordering, pre-lineage vs TE separation, mural/polar TE split)

## How to apply

Load the scRNA-seq expression matrix (e.g., the 1529-cell human pre-implantation embryo dataset from ArrayExpress E-MTAB-3929) with numpy 1.12.1 and h5py 2.7.0, then train the VASC model using the VASC class in vasc.py on the tensorflow 1.1.0 / keras 2.0.6 backend, exploiting the Gumbel-distribution modeling of zero and near-zero dropout events to learn a nonlinear hierarchical representation. Extract the 2D embedding of all cells from the trained variational autoencoder. Plot the embedding colored by external annotations (e.g., developmental stage E3–E7 and lineage labels ICM/PE/EPI vs TE) and overlay the inferred pseudo-time trajectory. Because stochastic optimization introduces run-to-run variation, repeat the VASC run and confirm the sub-population structure is stable across runs.

## Related tools

- **VASC** (deep variational autoencoder trained via vasc.py to produce the 2D representation in which sub-populations are identified) — github.com/wang-research/VASC
- **tensorflow** (1.1.0 backend on which the VASC model is trained)
- **keras** (2.0.6 backend used alongside tensorflow to build and fit the autoencoder) — https://github.com/keras-team/keras
- **numpy** (1.12.1 — loads and preprocesses the expression matrix before training)
- **h5py** (2.7.0 — reads the expression matrix files)
- **sklearn** (0.18.1 — supporting preprocessing/comparison utilities for the embedding workflow)

## Examples

```
python demo.py
```

## Evaluation signals

- The 2D visualization recovers the known E3–E7 developmental stage ordering along pseudo-time (Figure 5 layout).
- Pre-lineage cells (ICM/PE/EPI) separate from TE cells in the 2D embedding, and the TE compartment splits into mural and polar sub-populations.
- Rare cell populations are visibly represented in the 2D plot rather than merged into other clusters.
- Qualitative match against baselines: more precise than PCA, ZIFA, and SIMLR, and more complete than t-SNE (t-SNE misplaced E3 cells outside the trajectory).
- Repeated VASC runs agree qualitatively on cluster layout, confirming stability despite stochastic optimization.

## Limitations

- The stochastic optimization algorithms used in VASC model learning introduce run-to-run variations in the dimension reduction; repeated runs are recommended for a consensus.
- VASC model learning costs several hours on a desktop computer with a single GPU card for datasets with several thousand cells.
- Determination of the optimal latent dimension is a tricky task when prior knowledge is limited, and for small sample sizes it is better to fix the co-variance matrix.

## Evidence

- [abstract] VASC successfully re-establishes the cell dynamics in pre-implantation embryos and identifies several candidate marker genes associated with early embryo development: "VASC successfully re-establishes the cell dynamics in pre-implantation embryos and identifies several candidate marker genes associated with early embryo development"
- [methods] vasc.py class and function: "vasc.py: contains a class VASC and a function vase"
- [intro] effective low-dimensional representation as basis for downstream analysis: "Finding an effective low-dimensional representation of the scRNA-seq data is the basic step for the data visualization and the downstream analysis"
- [discussion] repeated runs recommended: "The stochastic optimization algorithms, used in the VASC model learning, introduce variations in the dimension reduction. Repeated runs are thus recommended"
- [readme] README code layout and nonlinear representation claim: "It can do nonlinear hierarchical feature representations and model the dropout events of scRNA-seq data."
