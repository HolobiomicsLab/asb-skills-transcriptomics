---
name: single-cell-expression-denoising
description: Use when a sparse scRNA-seq or dropout-simulated count matrix needs denoising/imputation,
  or when comparing DCA vs MAGIC/SAVER/scImpute recovery of known gene associations.
when_to_use_negative:
- The data are bulk RNA-seq without dropout and no denoising benchmark is intended
  — DCA's count/ZINB noise model targets sparse scRNA-seq data
- The input is already normalized/transformed expression rather than raw counts, since
  DCA models overdispersed count structure
- A ground-truth-free assessment is required and no reference signal (e.g. known gene-time
  correlations) exists — the paper notes evaluating denoising is difficult because
  defining a ground truth for real data is challenging
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: DCA
  role: 'Primary denoising method: deep count autoencoder with NB/ZINB loss, default
    64-32-64 hidden layers, RMSProp lr 0.001, early stopping 25 epochs'
  repo: https://github.com/theislab/dca
- name: MAGIC
  role: Comparison denoising method (version 0.1.0) run on the dropout-perturbed matrix
  repo: https://github.com/pkathail/magic
- name: SAVER
  role: Comparison imputation method (version 0.3.0) run on the dropout-perturbed
    matrix
  repo: https://github.com/mohuangx/SAVER
- name: scImpute
  role: Comparison imputation method (version 0.0.5) run on the dropout-perturbed
    matrix
  repo: https://github.com/Vivianstats/scImpute
- name: anndata
  role: Loads/stores count matrices in h5ad-formatted HDF5 for the denoising workflow
  repo: https://github.com/theislab/anndata
- name: Scanpy
  role: Python API entry point ('dca' method in the preprocessing package) and gene
    filtering (filter_genes_dispersion) in the article's workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:54b245da7df356db3ea9911430b0aea390bd117deca2a2a3b90dedd447d89f87
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - DCA
  - MAGIC
  - SAVER
  - Scanpy
  - anndata
  - scImpute
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# single-cell-expression-denoising

## Summary

Denoise and impute single-cell RNA-seq count matrices using DCA, a deep count autoencoder with ZINB/NB loss, and benchmark it against MAGIC, SAVER and scImpute by measuring recovery of signal (e.g. gene-time correlations) from dropout-perturbed data.

## When to use

Use when a sparse scRNA-seq or dropout-simulated count matrix needs denoising/imputation, or when comparing DCA vs MAGIC/SAVER/scImpute recovery of known gene associations.

## When NOT to use

- The data are bulk RNA-seq without dropout and no denoising benchmark is intended — DCA's count/ZINB noise model targets sparse scRNA-seq data
- The input is already normalized/transformed expression rather than raw counts, since DCA models overdispersed count structure
- A ground-truth-free assessment is required and no reference signal (e.g. known gene-time correlations) exists — the paper notes evaluating denoising is difficult because defining a ground truth for real data is challenging

## Inputs

- Raw or dropout-perturbed gene expression count matrix (genes x samples/cells), e.g. CSV or h5ad-formatted HDF5 via anndata
- Optional covariate vector for recovery evaluation (e.g. developmental time for 206 Francesconi et al. samples)

## Outputs

- Denoised count matrix per method (DCA, MAGIC, SAVER, scImpute) with original count values replaced by imputed/denoised expression
- Per-method Pearson correlation distributions (noisy, DCA, MAGIC, SAVER, scImpute) for the top 500 time-correlated genes, e.g. as a Fig. 4d-style recovery plot

## How to apply

Load the count matrix (e.g. the Francesconi et al. C. elegans bulk time-course of 206 samples) via pandas/anndata, removing cells and genes with zero expression. To benchmark, simulate single-cell dropout by subtracting gene-wise exponentially distributed values until 80% of entries are zeros. Run DCA (Python 3/Keras with TensorFlow backend) with its default 64-32-64 three-hidden-layer autoencoder, negative binomial or ZINB loss, RMSProp at learning rate 0.001 (reduced 0.1x after 20 epochs without validation-loss improvement) and early stopping after 25 epochs; the denoised matrix replaces original counts with the predicted negative binomial mean. In parallel, denoise copies of the corrupted matrix with MAGIC 0.1.0, SAVER 0.3.0 and scImpute 0.0.5. Evaluate by computing Pearson correlations between each gene's expression and time, restricting to the top 500 time-correlated genes in the original data, and comparing per-method correlation distributions against the noisy baseline. Rationale: DCA defines reconstruction error as the likelihood of a count noise model and shares information across features through the bottleneck, capturing nonlinear gene-gene dependencies.

## Related tools

- **DCA** (Primary denoising method: deep count autoencoder with NB/ZINB loss, default 64-32-64 hidden layers, RMSProp lr 0.001, early stopping 25 epochs) — https://github.com/theislab/dca
- **MAGIC** (Comparison denoising method (version 0.1.0) run on the dropout-perturbed matrix) — https://github.com/pkathail/magic
- **SAVER** (Comparison imputation method (version 0.3.0) run on the dropout-perturbed matrix) — https://github.com/mohuangx/SAVER
- **scImpute** (Comparison imputation method (version 0.0.5) run on the dropout-perturbed matrix) — https://github.com/Vivianstats/scImpute
- **anndata** (Loads/stores count matrices in h5ad-formatted HDF5 for the denoising workflow) — https://github.com/theislab/anndata
- **Scanpy** (Python API entry point ('dca' method in the preprocessing package) and gene filtering (filter_genes_dispersion) in the article's workflow)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('dropoutsim.h5ad'); sc.pp.dca(adata)
```

## Evaluation signals

- The correlation-distribution plot is generated for all five conditions (noisy, DCA, MAGIC, SAVER, scImpute)
- DCA's Pearson correlation distribution for the top 500 time-correlated genes shows recovery toward the original data relative to the noisy baseline, qualitatively matching published Fig. 4d (the paper reports no numeric threshold)
- The dropout simulation yields ~80% zero entries in the perturbed matrix before denoising
- DCA training terminates appropriately: learning rate reduced 0.1x after 20 epochs without validation-loss improvement, early stopping after 25 epochs
- The denoised output matrix has the same gene x sample dimensions as the input, with values drawn from the predicted negative binomial mean

## Limitations

- It may be difficult to determine when denoising improves scRNA-seq data, and defining ground truth for real data is challenging
- Overimputation can occur with inadequate hyperparameter choices such as a too low-dimensional bottleneck layer; overfitting/overimputation is a general, not yet extensively treated issue of imputation methods
- Hyperparameters were not fine-tuned for any of the analyses described in the manuscript; no DOI or preserved archive (e.g. Zenodo) is provided for datasets or code

## Evidence

- [abstract] DCA takes the count distribution, overdispersion and sparsity of the data into account using a negative binomial noise model with or without zero-inflation: "DCA takes the count distribution, overdispersion and sparsity of the data into account using a negative binomial noise model with or without zero-inflation"
- [intro] The deep learning framework of DCA enables capturing complexity and non-linearity in scRNA-seq data and is scalable: "The deep learning framework (by default three hidden layers with 64, 32, 64 neurons) of DCA enables the capturing of the complexity and non-linearity in scRNA-seq data."
- [discussion] We used RMSProp for optimization with learning rate 0.001. Learning rate is multiplied by 0.1 if validation loss does not improve for 20 epochs.: "We used RMSProp for optimization with learning rate 0.001. Learning rate is multiplied by 0.1 if validation loss does not improve for 20 epochs."
- [discussion] The denoised matrix is generated by replacing the original count values with the mean of the negative binomial component: "The denoised matrix is generated by replacing the original count values with the mean of the negative binomial component"
- [discussion] Francesconi et al36. data set contained 206 samples covering a 12-hour time-course.: "Francesconi et al36. data set contained 206 samples covering a 12-hour time-course."
- [readme] A deep count autoencoder network to denoise scRNA-seq data and remove the dropout effect: "A deep count autoencoder network to denoise scRNA-seq data and remove the dropout effect by taking the count structure, overdispersed nature and sparsity of the data into account"
