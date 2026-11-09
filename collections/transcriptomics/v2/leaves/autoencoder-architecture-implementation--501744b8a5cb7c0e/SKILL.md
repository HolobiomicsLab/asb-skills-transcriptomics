---
name: autoencoder-architecture-implementation
description: Use when raw scRNA-seq count matrices (h5ad or CSV/TSV) need denoising/imputation
  in one step while respecting count overdispersion, sparsity, and dropout.
when_to_use_negative:
- Input is already normalized/log-transformed or not raw counts — the ZINB/NB noise
  model is defined on sparse count data
- The analysis requires imputing defined missing values (classical imputation); DCA
  instead denoises all entries since true and false zero counts cannot be trivially
  distinguished
- Datasets too large for memory or when a linear, non-parametric smoothing method
  is intended — though DCA itself scales linearly, this skill specifically covers
  the autoencoder architecture setup
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: DCA
  role: The deep count autoencoder package being implemented/invoked; provides network.py
    and loss.py with the ZINB loss and denoised output generation
  repo: https://github.com/theislab/dca
- name: Keras
  role: Deep learning framework in which the autoencoder network and ZINB loss are
    implemented
- name: TensorFlow
  role: Backend for Keras used to train the autoencoder
- name: anndata
  role: Reads h5ad-formatted HDF5 count matrices as input
  repo: https://github.com/theislab/anndata
- name: Scanpy
  role: Python API integration via the 'dca' method in the preprocessing package;
    filter_genes_dispersion used for highly-variable-gene selection
- name: kopt
  role: Hyperparameter search library (with hyperopt, TPE over 1000 configurations)
    optional for tuning the architecture
  repo: https://github.com/Avsecz/kopt
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:f290be0c5b2d792ef74043ce04e4d20f31a05f3b295a08917a657d47ebb0ea13
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - DCA
  - Keras
  - Scanpy
  - TensorFlow
  - anndata
  - kopt
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# autoencoder-architecture-implementation

## Summary

Implements the DCA deep count autoencoder: a fixed network of three hidden layers (64, 32, 64 neurons) with a 32-dimensional bottleneck, ZINB/NB output heads, and a noise-model-likelihood reconstruction loss for denoising scRNA-seq count data.

## When to use

Use when raw scRNA-seq count matrices (h5ad or CSV/TSV) need denoising/imputation in one step while respecting count overdispersion, sparsity, and dropout.

## When NOT to use

- Input is already normalized/log-transformed or not raw counts — the ZINB/NB noise model is defined on sparse count data
- The analysis requires imputing defined missing values (classical imputation); DCA instead denoises all entries since true and false zero counts cannot be trivially distinguished
- Datasets too large for memory or when a linear, non-parametric smoothing method is intended — though DCA itself scales linearly, this skill specifically covers the autoencoder architecture setup

## Inputs

- Raw scRNA-seq count matrix as h5ad-formatted HDF5 file (anndata reader) or CSV/TSV with genes in rows, cells in columns, and mandatory cell and gene labels

## Outputs

- mean.tsv — denoised matrix, the ZINB/NB mean parameter per cell-gene, same dimensions as input (excluding zero-expression genes/cells)
- mean_norm.tsv — library size-normalized expressions (Scanpy normalize_total convention)
- pi.tsv — dropout probability per cell and gene
- dispersion.tsv — dispersion (theta) per cell and gene
- reduced.tsv — 32-dimensional bottleneck activations (hidden representation per cell)

## How to apply

Load the raw count matrix (h5ad-formatted HDF5 via anndata, or CSV/TSV with genes in rows and cells in columns, with mandatory cell and gene labels). Build the network in autoencoder/network.py with three hidden layers of 64, 32, 64 neurons and a 32-dimensional bottleneck; the compression across features is what accounts for gene-gene dependencies. Attach three output heads parameterizing the zero-inflated negative binomial: mu (exponential activation), pi (dropout probability, sigmoid activation), and theta (dispersion). Implement the ZINB negative-log-likelihood loss in autoencoder/loss.py — reconstruction error is defined as the likelihood of the noise-model distribution rather than reconstructing the input — plus a ridge prior on pi; compile with RMSProp at learning rate 0.001. Train with early stopping after 25 epochs without validation-loss improvement, reducing the learning rate by a factor of 0.1 after 20 epochs without improvement. Generate the denoised matrix by replacing each original count with the mean of the negative binomial component.

## Related tools

- **DCA** (The deep count autoencoder package being implemented/invoked; provides network.py and loss.py with the ZINB loss and denoised output generation) — https://github.com/theislab/dca
- **Keras** (Deep learning framework in which the autoencoder network and ZINB loss are implemented)
- **TensorFlow** (Backend for Keras used to train the autoencoder)
- **anndata** (Reads h5ad-formatted HDF5 count matrices as input) — https://github.com/theislab/anndata
- **Scanpy** (Python API integration via the 'dca' method in the preprocessing package; filter_genes_dispersion used for highly-variable-gene selection)
- **kopt** (Hyperparameter search library (with hyperopt, TPE over 1000 configurations) optional for tuning the architecture) — https://github.com/Avsecz/kopt

## Examples

```
dca matrix.csv results
```

## Evaluation signals

- Training terminates via early stopping: 25 epochs without validation-loss improvement
- The denoised matrix (mean.tsv) matches the input matrix dimensions (except zero-expression genes/cells removed), with NB-mean imputation applied to every cell-gene entry
- pi.tsv, dispersion.tsv, and reduced.tsv have the same dimensions as mean.tsv; reduced.tsv has 32 columns by default (bottleneck size)
- Reconstruction error is computed as the ZINB/NB noise-model likelihood, not a per-element input reconstruction error
- Library size normalization of the output follows Scanpy's normalize_total convention (mean_norm.tsv)

## Limitations

- It may be difficult to determine when denoising improves scRNA-seq data; evaluation is hard because ground truth for real data is challenging to define
- Inadequate hyperparameter choices (e.g., too low-dimensional a bottleneck layer) can lead to overimputation; overfitting/overimputation is a general, not yet extensively treated issue of imputation methods
- Hyperparameters were not fine-tuned for any of the analyses in the manuscript; no DOI/accession IDs or preserved archive (e.g., Zenodo) is provided for datasets or code

## Evidence

- [discussion] DCA is implemented in Python 3 using Keras53 and its TensorFlow54 backend.: "DCA is implemented in Python 3 using Keras53 and its TensorFlow54 backend."
- [intro] Default architecture: "The deep learning framework (by default three hidden layers with 64, 32, 64 neurons) of DCA enables the capturing of the complexity and non-linearity in scRNA-seq data."
- [intro] Loss definition as noise-model likelihood: "The trick is to define the reconstruction error as the likelihood of the distribution of the noise model instead of reconstructing the input data itself"
- [discussion] Training and denoising parameters: "We used RMSProp for optimization with learning rate 0.001. Learning rate is multiplied by 0.1 if validation loss does not improve for 20 epochs."
- [discussion] Denoised output is the NB mean: "The denoised matrix is generated by replacing the original count values with the mean of the negative binomial component"
- [readme] mean.tsv is the main output: "`mean.tsv` is the main output of the method which represents the mean parameter of the ZINB distribution."
