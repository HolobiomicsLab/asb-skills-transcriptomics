---
name: latent-dimension-sensitivity-analysis
description: 'Use when a VASC (or similar deep variational autoencoder) latent representation
  must be validated for robustness: you have labeled scRNA-seq data (e.g., Pollen),
  need to pick a latent dimension, and must show NMI/ARI stability across sampling
  fractions and read depths.'
when_to_use_negative:
- The dataset has no known cell labels, so NMI/ARI clustering quality cannot be computed
  against ground truth.
- You need to determine the optimal dimension de novo on a small sample without prior
  knowledge — the paper warns this is 'a tricky task'; use a fixed covariance and
  repeated runs instead.
- Input is already a reduced/latent representation and the goal is visualization only
  rather than dimension robustness assessment.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: VASC
  role: Deep variational autoencoder whose latent dimension is swept; trained via
    the VASC class and vase function in vasc.py on each down-sampled data variant
  repo: github.com/wang-research/VASC
- name: sklearn
  role: Clustering of cells in the learned low-dimensional space and NMI/ARI computation
    against known labels
- name: tensorflow
  role: Backend for training the VASC deep variational autoencoder on each sensitivity-analysis
    configuration
- name: keras
  role: Keras backend used with TensorFlow to build and fit the VASC autoencoder models
  repo: https://github.com/keras-team/keras
- name: numpy
  role: Numerical handling of count matrices and generation of down-sampled data variants
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2018.08.003
    title: 'VASC: Dimension Reduction and Visualization of Single-Cell RNA-Seq Data
      by Deep Variational Autoencoder'
  source_content_hash: sha256:c04dc0fb8613ddf546c9ae66a927c58f4234bdd5da508e1b7ca3fb2f6f96c28a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
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
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5789473684210527
n_claims: 19
---

# latent-dimension-sensitivity-analysis

## Summary

A sensitivity analysis skill that sweeps the VASC autoencoder's latent dimension (e.g., 2, 4, ..., 20) alongside cell-sampling fractions and read-depth down-sampling on a scRNA-seq dataset, scoring each configuration by NMI/ARI clustering quality against known cell labels to determine the smallest adequate latent space.

## When to use

Use when a VASC (or similar deep variational autoencoder) latent representation must be validated for robustness: you have labeled scRNA-seq data (e.g., Pollen), need to pick a latent dimension, and must show NMI/ARI stability across sampling fractions and read depths.

## When NOT to use

- The dataset has no known cell labels, so NMI/ARI clustering quality cannot be computed against ground truth.
- You need to determine the optimal dimension de novo on a small sample without prior knowledge — the paper warns this is 'a tricky task'; use a fixed covariance and repeated runs instead.
- Input is already a reduced/latent representation and the goal is visualization only rather than dimension robustness assessment.

## Inputs

- scRNA-seq count matrix (e.g., Pollen dataset) with per-cell labels
- VASC pipeline code: vasc.py (class VASC, function vase), helpers.py
- Python environment: Python 3.5+, numpy 1.12.1, h5py 2.7.0, sklearn 0.18.1, tensorflow 1.1.0, keras 2.0.6

## Outputs

- Summary table of NMI and ARI scores across sampling fractions, read depths, and latent dimensions (pollen_downsampling_nmi_ari.csv)
- Sensitivity conclusion on minimum adequate latent dimension and minimum cell fraction for stable clustering

## How to apply

Load the scRNA-seq count matrix and cell labels, then preprocess as in the VASC pipeline (vasc.py provides the VASC class and vase function). Generate down-sampled variants by sampling 10%, 20%, ..., 100% of cells and by down-sampling reads to 5000, 10000, ..., 300000 counts per cell. For each variant, train the VASC deep variational autoencoder (TensorFlow 1.1.0 / Keras 2.0.6 backend) while sweeping the latent dimension over 2, 4, ..., 20. Cluster cells in the learned low-dimensional space with scikit-learn and evaluate against known labels using NMI and ARI. Judge success by comparing the summary table against expectations: stable NMI/ARI above 50% cell sampling, degradation only at very low read counts, and no improvement beyond a latent dimension of 2 — consistent with the paper's note that determining the optimal dimension is tricky without prior knowledge.

## Related tools

- **VASC** (Deep variational autoencoder whose latent dimension is swept; trained via the VASC class and vase function in vasc.py on each down-sampled data variant) — github.com/wang-research/VASC
- **sklearn** (Clustering of cells in the learned low-dimensional space and NMI/ARI computation against known labels)
- **tensorflow** (Backend for training the VASC deep variational autoencoder on each sensitivity-analysis configuration)
- **keras** (Keras backend used with TensorFlow to build and fit the VASC autoencoder models) — https://github.com/keras-team/keras
- **numpy** (Numerical handling of count matrices and generation of down-sampled data variants)

## Examples

```
python demo.py  # trains VASC via vasc.py (VASC class / vase function) with the configured latent dimension, then sweep latent_dim in [2,4,...,20] and score NMI/ARI with sklearn
```

## Evaluation signals

- NMI/ARI values on the down-sampled Pollen dataset remain comparable when the percentage of sampled cells is above 50%.
- Performance degradation appears only at very low read counts (low end of the 5000–300000 counts-per-cell sweep), not across the mid-to-high range.
- Increasing the latent dimension beyond 2 yields no improvement in NMI/ARI, confirming 2 as the sufficient dimension.
- The summary table pollen_downsampling_nmi_ari.csv is produced with rows covering all combinations of sampling fraction, read depth, and latent dimension (2, 4, ..., 20).
- Repeated runs agree, since stochastic optimization in VASC model learning introduces run-to-run variation and consensus across repeats is expected.

## Limitations

- Determining the optimal latent dimension is difficult without prior knowledge.
- Stochastic optimization in VASC model learning introduces run-to-run variation; repeated runs are recommended for consensus.
- When the sample size is small, it is better to fix the co-variance matrix.
- VASC model learning takes several hours on a single-GPU desktop for datasets with several thousand cells, so a full sweep is compute-intensive.

## Evidence

- [readme] vasc.py: contains a class VASC and a function vase: "vasc.py: contains a class VASC and a function vase"
- [discussion] Optimal dimension difficult without prior knowledge: "The determination of the optimal dimension is a tricky task if prior knowledge is limited."
- [discussion] Stochastic optimization variation; repeated runs recommended: "The stochastic optimization algorithms, used in the VASC model learning, introduce variations in the dimension reduction."
- [discussion] Pollen dataset identifies distinct transcriptome cell populations: "This penalty is very useful to identify the cell populations with distinct transcriptomes (for examples, the Pollen dataset)."
- [discussion] Fix covariance for small samples: "When the sample size is small, it is better to fix the co-variance matrix."
