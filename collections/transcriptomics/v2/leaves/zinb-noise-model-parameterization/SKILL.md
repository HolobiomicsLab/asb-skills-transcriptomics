---
name: zinb-noise-model-parameterization
description: Use when denoising or imputing sparse, overdispersed scRNA-seq count
  matrices where dropout zero counts must be modeled explicitly and classical imputation
  with defined missing values is unsuitable.
when_to_use_negative:
- Input is already normalized, log-transformed, or non-count expression data — the
  ZINB/NB likelihood loss assumes raw sparse integer counts
- The goal is bulk RNA-seq or other non-single-cell data where the count structure,
  overdispersion and sparsity assumptions of scRNA-seq do not hold
- A defined ground-truth missing-value structure is required for validation; denoising
  methods like DCA operate without explicit missing-value labels
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: DCA
  role: Deep count autoencoder package that implements the ZINB/NB parameterization,
    loss, and denoised-mean output
  repo: https://github.com/theislab/dca
- name: Keras
  role: Framework in which the DCA autoencoder and its ZINB output heads are implemented
- name: TensorFlow
  role: Backend for the Keras DCA implementation
- name: anndata
  role: Reader for h5ad-formatted HDF5 count-matrix input files
  repo: https://github.com/theislab/anndata
- name: Scanpy
  role: Python API compatibility layer; its normalize_total library-size normalization
    defines mean_norm.tsv scaling, and filter_genes_dispersion pre-filters to the
    top 1000 highly variable genes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:c69171707e96f8aa2a12c598d3b97c7127346d91ef3cddc46323f504744eaeed
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
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# zinb-noise-model-parameterization

## Summary

Parameterize a deep count autoencoder's output heads with a zero-inflated negative binomial (ZINB) distribution — mu, pi (dropout probability), and theta (dispersion) — so that reconstruction error is the likelihood of the noise model rather than reconstruction of the raw input counts, and the inferred mean serves as the denoised scRNA-seq output.

## When to use

Use when denoising or imputing sparse, overdispersed scRNA-seq count matrices where dropout zero counts must be modeled explicitly and classical imputation with defined missing values is unsuitable.

## When NOT to use

- Input is already normalized, log-transformed, or non-count expression data — the ZINB/NB likelihood loss assumes raw sparse integer counts
- The goal is bulk RNA-seq or other non-single-cell data where the count structure, overdispersion and sparsity assumptions of scRNA-seq do not hold
- A defined ground-truth missing-value structure is required for validation; denoising methods like DCA operate without explicit missing-value labels

## Inputs

- Raw scRNA-seq count matrix (h5ad-formatted HDF5 file read via anndata, or CSV/TSV with genes in rows and cells in columns)
- Cell and gene labels (mandatory for the CLI)

## Outputs

- Denoised count matrix (mean.tsv / mean_norm.tsv): mean parameter of the ZINB distribution, gene x cell, same dimensions as input minus zero-expression genes/cells
- Dropout probability matrix (pi.tsv) per cell and gene
- Dispersion matrix (dispersion.tsv) per cell and gene
- 32-dimensional bottleneck hidden representation per cell (reduced.tsv)

## How to apply

Build an autoencoder with three hidden layers of 64, 32, 64 neurons and a 32-dimensional bottleneck, then attach three output heads: mu with exponential activation, pi (dropout probability) with sigmoid activation, and theta as the dispersion parameter. Define the reconstruction error as the ZINB negative-log-likelihood of the raw counts (never as input reconstruction), adding a ridge prior on pi, and compile with RMSProp at learning rate 0.001. Train with early stopping after 25 epochs without validation-loss improvement and reduce the learning rate by 0.1 if validation loss does not improve for 20 epochs. To denoise, replace each original count with the mean of the negative binomial component, taking the count distribution, overdispersion and sparsity into account. Because the compression shares information across features, gene-gene dependencies are accounted for and the learned mean output is a denoised, imputed expression matrix of the same dimensions as the input.

## Related tools

- **DCA** (Deep count autoencoder package that implements the ZINB/NB parameterization, loss, and denoised-mean output) — https://github.com/theislab/dca
- **Keras** (Framework in which the DCA autoencoder and its ZINB output heads are implemented)
- **TensorFlow** (Backend for the Keras DCA implementation)
- **anndata** (Reader for h5ad-formatted HDF5 count-matrix input files) — https://github.com/theislab/anndata
- **Scanpy** (Python API compatibility layer; its normalize_total library-size normalization defines mean_norm.tsv scaling, and filter_genes_dispersion pre-filters to the top 1000 highly variable genes)

## Examples

```
dca matrix.csv results
```

## Evaluation signals

- Training terminates via early stopping after 25 epochs without validation-loss improvement, and learning rate drops by 0.1 after 20 epochs without improvement
- The denoised output (mean.tsv) matches the input matrix dimensions (gene x cell), excluding only zero-expression genes/cells
- Every cell-gene entry has been replaced by the NB-component mean; pi and dispersion matrices exist with the same dimensions as mean.tsv
- reduced.tsv contains a 32-dimensional hidden representation (activations of bottleneck neurons) for each cell
- Reconstruction error during training is the ZINB likelihood loss (with ridge prior on pi), not an input-reconstruction MSE

## Limitations

- It may be difficult to determine when denoising improves scRNA-seq data, since defining ground truth for real data is challenging
- Inadequate hyperparameter choices, such as a too low-dimensional bottleneck layer, may lead to overimputation; hyperparameters were not fine-tuned for any of the manuscript's analyses
- Cells and genes with zero expression are removed from the count matrix, so the output covers only the retained features/cells

## Evidence

- [intro] The trick is to define the reconstruction error as the likelihood of the distribution of the noise model instead of reconstructing the input data itself: "The trick is to define the reconstruction error as the likelihood of the distribution of the noise model instead of reconstructing the input data itself"
- [discussion] Denoised matrix via NB mean: "The denoised matrix is generated by replacing the original count values with the mean of the negative binomial component"
- [abstract] NB noise model with or without zero-inflation: "DCA takes the count distribution, overdispersion and sparsity of the data into account using a negative binomial noise model with or without zero-inflation"
- [readme] mean.tsv is the main output: "`mean.tsv` is the main output of the method which represents the mean parameter of the ZINB distribution."
- [discussion] RMSProp, lr 0.001, early stopping 25 epochs: "We used RMSProp for optimization with learning rate 0.001. Learning rate is multiplied by 0.1 if validation loss does not improve for 20 epochs."
