---
name: count-matrix-denoising
description: Use when a raw scRNA-seq count matrix (h5ad/CSV/TSV) is sparse and dropout-heavy
  and the goal is a denoised count matrix with the same gene x cell dimensions plus
  optional 32-dim latent representations.
when_to_use_negative:
- Input is already normalized, log-transformed, or not raw counts — DCA's ZINB/NB
  loss is defined on sparse count data
- The goal is classical imputation with explicitly defined missing values — the paper
  notes the non-trivial distinction between true (celltype-specific) and false (dropout)
  zero counts makes such approaches unsuitable for scRNA-seq
- Very small datasets or inadequate hyperparameter choices (e.g., a too low-dimensional
  bottleneck layer), where overimputation is a risk
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: DCA
  role: Deep count autoencoder that trains the ZINB/NB-loss network and produces the
    denoised mean matrix; installed via pip or conda (bioconda) and runnable from
    the command line
  repo: https://github.com/theislab/dca
- name: anndata
  role: Reader for h5ad-formatted HDF5 count matrices used as DCA input
  repo: https://github.com/theislab/anndata
- name: Scanpy
  role: Provides filter_genes_dispersion for highly variable gene selection and normalize_total
    for the default library size normalization applied to DCA output; the dca method
    is exposed in the Scanpy preprocessing package
  repo: https://scanpy.readthedocs.io/en/stable/api/scanpy.pp.normalize_total.html
- name: Keras / TensorFlow
  role: Framework in which DCA is implemented and trained (autoencoder/network.py,
    autoencoder/loss.py)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:353ab6d6ff973be859e7707a87d77e0b3c605174ebd3033c875e888385ef736e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - DCA
  - Keras
  - Keras / TensorFlow
  - Scanpy
  - TensorFlow
  - anndata
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# count-matrix-denoising

## Summary

Denoise raw scRNA-seq count matrices with the deep count autoencoder (DCA), which trains an autoencoder under a zero-inflated negative binomial (ZINB/NB) count loss and outputs the inferred NB mean as the denoised matrix, jointly handling dropout, overdispersion and sparsity.

## When to use

Use when a raw scRNA-seq count matrix (h5ad/CSV/TSV) is sparse and dropout-heavy and the goal is a denoised count matrix with the same gene x cell dimensions plus optional 32-dim latent representations.

## When NOT to use

- Input is already normalized, log-transformed, or not raw counts — DCA's ZINB/NB loss is defined on sparse count data
- The goal is classical imputation with explicitly defined missing values — the paper notes the non-trivial distinction between true (celltype-specific) and false (dropout) zero counts makes such approaches unsuitable for scRNA-seq
- Very small datasets or inadequate hyperparameter choices (e.g., a too low-dimensional bottleneck layer), where overimputation is a risk

## Inputs

- Raw scRNA-seq count matrix as h5ad-formatted HDF5 file (read via anndata) or CSV/TSV file with genes in rows, cells in columns, and mandatory cell and gene labels
- Count matrix pre-filtered to remove cells and genes with zero expression (optionally restricted to top 1000 or 5000 highly variable genes selected with Scanpy filter_genes_dispersion)

## Outputs

- Denoised matrix (mean.tsv): NB/ZINB mean parameter per gene x cell entry, same dimensions as input excluding zero-expression genes/cells
- mean_norm.tsv: library size-normalized expressions (Scanpy normalize_total)
- pi.tsv and dispersion.tsv: dropout probability and dispersion per cell and gene
- reduced.tsv: 32-dimensional bottleneck (hidden) representation per cell

## How to apply

Load the raw count matrix from an h5ad-formatted HDF5 file with anndata (or CSV/TSV via the dca CLI, genes x cells with mandatory cell and gene labels), after removing cells and genes with zero expression. Build the default network with three hidden layers of 64, 32, 64 neurons and a 32-dimensional bottleneck, and attach three ZINB output heads: mu with exponential activation, dropout probability pi with sigmoid activation, and the dispersion parameter theta. Define reconstruction error as the likelihood of the noise model (ZINB negative log-likelihood in autoencoder/loss.py) with a ridge prior on pi, and optimize with RMSProp at learning rate 0.001, multiplying the learning rate by 0.1 after 20 epochs without validation-loss improvement and applying early stopping after 25 epochs without improvement. Generate the denoised matrix by replacing each original count with the mean of the negative binomial component; the main output file represents the mean parameter of the ZINB distribution, and library-size-normalized versions use Scanpy's normalize_total method.

## Related tools

- **DCA** (Deep count autoencoder that trains the ZINB/NB-loss network and produces the denoised mean matrix; installed via pip or conda (bioconda) and runnable from the command line) — https://github.com/theislab/dca
- **anndata** (Reader for h5ad-formatted HDF5 count matrices used as DCA input) — https://github.com/theislab/anndata
- **Scanpy** (Provides filter_genes_dispersion for highly variable gene selection and normalize_total for the default library size normalization applied to DCA output; the dca method is exposed in the Scanpy preprocessing package) — https://scanpy.readthedocs.io/en/stable/api/scanpy.pp.normalize_total.html
- **Keras / TensorFlow** (Framework in which DCA is implemented and trained (autoencoder/network.py, autoencoder/loss.py))

## Examples

```
dca matrix.csv results
```

## Evaluation signals

- Training terminates via the early-stopping criterion (25 epochs without validation loss improvement), with learning rate reduced by 0.1 after 20 epochs without improvement
- Denoised output has the same dimensions as the input matrix (except zero-expression genes/cells removed), with the NB-mean imputation applied to every cell-gene entry
- Accompanying pi.tsv and dispersion.tsv matrices match the dimensions of mean.tsv and the input file
- reduced.tsv contains per-cell activations of the 32 bottleneck neurons (default 32-dimensional space)
- Validation of noise model choice fits NB and ZINB models to the mean–dropout rate dependence by minimizing binary cross entropy; hyperparameter search (with --hyper) selects the configuration with the lowest reconstruction error

## Limitations

- It may be difficult to determine when denoising improves scRNA-seq data, since defining a ground truth for real data is challenging
- Overimputation may occur with inadequate hyperparameter choices such as a too low-dimensional bottleneck layer
- Hyperparameters were not fine-tuned for any of the analyses in the manuscript; no DOI or preserved archive (e.g., Zenodo) is provided for datasets or code

## Evidence

- [intro] The trick is to define the reconstruction error as the likelihood of the distribution of the noise model instead of reconstructing the input data itself: "The trick is to define the reconstruction error as the likelihood of the distribution of the noise model instead of reconstructing the input data itself"
- [discussion] Denoising by replacing original counts with the predicted NB mean: "The denoised matrix is generated by replacing the original count values with the mean of the negative binomial component"
- [discussion] Training configuration: "We used RMSProp for optimization with learning rate 0.001. Learning rate is multiplied by 0.1 if validation loss does not improve for 20 epochs."
- [readme] mean.tsv is the main output representing the ZINB mean parameter: "mean.tsv` is the main output of the method which represents the mean parameter of the ZINB distribution."
- [discussion] DCA implementation stack: "DCA is implemented in Python 3 using Keras53 and its TensorFlow54 backend."
