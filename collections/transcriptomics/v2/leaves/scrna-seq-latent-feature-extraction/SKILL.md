---
name: scrna-seq-latent-feature-extraction
description: Use when scRNA-seq data contain dropout events and you need a nonlinear
  low-dimensional (2D) cell representation for visualization, lineage re-establishment,
  or downstream gene-marker discovery via correlation with latent features.
when_to_use_negative:
- Input is not scRNA-seq with dropout events but a bulk RNA-seq or already-reduced
  feature table — VASC's Gumbel-based dropout modeling and dimension reduction are
  specific to noisy single-cell data
- You need a fixed, deterministic embedding in a single run — stochastic optimization
  introduces run-to-run variation requiring repeated runs
- You have no basis for choosing the latent dimension and cannot fix the co-variance
  matrix for very small sample sizes
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: VASC
  role: Deep variational autoencoder producing the unsupervised 2D dimension reduction
    and latent features of the scRNA-seq data; implemented as a class VASC and function
    vase in vasc.py
  repo: https://github.com/wang-research/VASC
- name: Python
  role: Runtime environment (Python 3.5+) for running VASC and computing Spearman
    correlations
- name: numpy
  role: Used to compute Spearman's rank correlation between gene expression profiles
    and latent features
- name: sklearn
  role: Supports correlation computation and latent-feature handling in the workflow
- name: h5py
  role: Reads the scRNA-seq expression data (h5 format) used as VASC input
- name: keras
  role: Deep-learning backend on which the VASC autoencoder model is built
  repo: https://github.com/keras-team/keras
- name: Anaconda
  role: Recommended installation platform for the Python environment and required
    packages
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2018.08.003
    title: 'VASC: Dimension Reduction and Visualization of Single-Cell RNA-Seq Data
      by Deep Variational Autoencoder'
  source_content_hash: sha256:18e7a10fcebed6cb76aa298a4addc5ffa8274dcd83cebf7bd4af3681bdedf981
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# scrna-seq-latent-feature-extraction

## Summary

Extract low-dimensional latent features from single-cell RNA-seq data using VASC, a deep variational autoencoder that models dropout events, then correlate gene expression against the 2D latent features to identify candidate marker genes (e.g. associated with human pre-implantation embryo development).

## When to use

Use when scRNA-seq data contain dropout events and you need a nonlinear low-dimensional (2D) cell representation for visualization, lineage re-establishment, or downstream gene-marker discovery via correlation with latent features.

## When NOT to use

- Input is not scRNA-seq with dropout events but a bulk RNA-seq or already-reduced feature table — VASC's Gumbel-based dropout modeling and dimension reduction are specific to noisy single-cell data
- You need a fixed, deterministic embedding in a single run — stochastic optimization introduces run-to-run variation requiring repeated runs
- You have no basis for choosing the latent dimension and cannot fix the co-variance matrix for very small sample sizes

## Inputs

- Single-cell RNA-seq gene expression matrix (e.g. Petropoulos human pre-implantation embryo dataset, h5py/h5 format)
- VASC 2D latent representation of the cells produced by the VASC class in vasc.py
- Environment: Anaconda with Python 3.5+, numpy 1.12.1, h5py 2.7.0, sklearn 0.18.1, tensorflow 1.1.0, keras 2.0.6

## Outputs

- 2D latent feature representation of cells (visualization / cell dynamics re-establishment)
- Per-gene Spearman rank correlation values against each of the two latent features
- List of top-correlated genes (rank <= 100 in either VASC feature)
- DAVID 6.8 GO biological process enrichment table written to david_enrichment_results.csv (e.g. carbohydrate derivative metabolic process: 37 genes, q = 5.63E-05; oxidation-reduction process: 32 genes, q = 4.87E-05; lipid metabolic process: 32 genes, q = 4.94E-03)

## How to apply

Load the scRNA-seq expression matrix (e.g. the Petropoulos human pre-implantation embryo dataset) and run the VASC class from vasc.py to obtain a 2D latent representation, using Python 3.5+ with numpy 1.12.1, h5py 2.7.0, sklearn 0.18.1, tensorflow 1.1.0 and keras 2.0.6 installed via Anaconda. VASC explicitly models zero and near-zero dropout events with the Gumbel distribution and learns nonlinear hierarchical feature representations. Next, compute Spearman's rank correlation between every gene's expression profile and each of the two VASC latent features in Python using numpy/sklearn. Rank genes by absolute correlation for each latent feature and retain genes with rank <= 100. Submit the retained top-correlated gene lists to DAVID 6.8 for GO biological process enrichment, recording gene counts and q-values. For small sample sizes, fix the co-variance matrix, and repeat runs to build consensus because stochastic optimization introduces run-to-run variation.

## Related tools

- **VASC** (Deep variational autoencoder producing the unsupervised 2D dimension reduction and latent features of the scRNA-seq data; implemented as a class VASC and function vase in vasc.py) — https://github.com/wang-research/VASC
- **Python** (Runtime environment (Python 3.5+) for running VASC and computing Spearman correlations)
- **numpy** (Used to compute Spearman's rank correlation between gene expression profiles and latent features)
- **sklearn** (Supports correlation computation and latent-feature handling in the workflow)
- **h5py** (Reads the scRNA-seq expression data (h5 format) used as VASC input)
- **keras** (Deep-learning backend on which the VASC autoencoder model is built) — https://github.com/keras-team/keras
- **Anaconda** (Recommended installation platform for the Python environment and required packages)

## Examples

```
python demo.py  # after configuring config.py, using the VASC class from vasc.py to obtain the 2D latent features of the scRNA-seq data
```

## Evaluation signals

- The 2D latent representation re-establishes known embryo pre-implantation cell lineage dynamics (VASC successfully re-establishes cell dynamics in pre-implantation embryos)
- Enrichment table (david_enrichment_results.csv) reproduces the expected terms: carbohydrate derivative metabolic process (37 genes, q = 5.63E-05), oxidation-reduction process (32 genes, q = 4.87E-05), lipid metabolic process (32 genes, q = 4.94E-03)
- Retained gene lists contain only genes with absolute Spearman correlation rank <= 100 in at least one of the two VASC features
- Multiple repeated VASC runs yield a consensus embedding with consistent cell populations, since stochastic optimization introduces variation

## Limitations

- Determination of the optimal latent dimension is tricky without prior knowledge
- When the sample size is small, it is better to fix the co-variance matrix
- Stochastic optimization algorithms introduce variation in the dimension reduction; repeated runs are recommended
- VASC model learning costs several hours on a single-GPU desktop computer for datasets with several thousand cells
- More complex variational distribution families remain untested

## Evidence

- [abstract] VASC uses Gumbel distribution for dropout: "we proposed the deep variational autoencoder for scRNA-seq data (VASC), a deep multi-layer generative model, for the unsupervised dimension reduction and visualization of scRNA-seq data"
- [abstract] Marker gene identification: "VASC successfully re-establishes the cell dynamics in pre-implantation embryos and identifies several candidate marker genes associated with early embryo development"
- [methods] vasc.py contents: "vasc.py: contains a class VASC and a function vase"
- [methods] Prerequisites: "+ Python 3.5+
+ numpy 1.12.1
+ h5py 2.7.0
+ sklearn 0.18.1
+ tensorflow 1.1.0
+ keras 2.0.6"
- [intro] Gumbel dropout: "it uses the Gumbel distribution to better model the zero and near-zero dropout events"
- [discussion] Small-sample covariance: "When the sample size is small, it is better to fix the co-variance matrix."
- [discussion] Run variation: "The stochastic optimization algorithms, used in the VASC model learning, introduce variations in the dimension reduction. Repeated runs are thus recommended"
- [readme] README demo: "We gave a demo.py and config.py to demonstrate the use of VASC."
