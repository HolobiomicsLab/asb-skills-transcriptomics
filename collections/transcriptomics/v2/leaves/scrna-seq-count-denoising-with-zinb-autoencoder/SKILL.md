---
name: scrna-seq-count-denoising-with-zinb-autoencoder
description: Use when a sparse, overdispersed scRNA-seq count matrix shows dropout
  and downstream tasks (DE, visualization, clustering) suffer from noise, and counts
  are the input (not TPM/CPM).
when_to_use_negative:
- Input is normalized/transformed expression (TPM, CPM, log-transformed) rather than
  raw integer counts — DCA's NB/ZINB likelihood assumes count structure, overdispersion
  and sparsity
- The distinction between true (celltype-specific) and false (dropout) zeros is the
  exact quantity you need validated with ground truth — evaluation of denoising on
  real data lacks a defined ground truth
- Data are so large that fitting is needed but no GPU is available and hyperparameters
  were not tuned — DCA scales linearly but training cost still matters
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: DCA
  role: Deep count autoencoder network trained with ZINB loss that denoises scRNA-seq
    counts and outputs the predicted NB mean matrix
  repo: https://github.com/theislab/dca
- name: Scanpy
  role: Preprocessing integration — 'dca' method in the preprocessing package and
    filter_genes_dispersion for highly variable gene selection
- name: anndata
  role: h5ad-formatted HDF5 storage of count matrices passed into and out of DCA
  repo: https://github.com/theislab/anndata
- name: Keras
  role: Implementation framework for the DCA autoencoder with TensorFlow backend
- name: MAGIC
  role: Comparison imputation/denoising method benchmarked against DCA (version 0.1.0)
  repo: https://github.com/pkathail/magic
- name: SAVER
  role: Comparison imputation method benchmarked against DCA (version 0.3.0)
  repo: https://github.com/mohuangx/SAVER
- name: scImpute
  role: Comparison mixture-model imputation method benchmarked against DCA (version
    0.0.5)
  repo: https://github.com/Vivianstats/scImpute
- name: kopt
  role: Hyperparameter optimization for Keras models used to tune DCA configurations
  repo: https://github.com/Avsecz/kopt
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:2e1111c640e3f23f867bc200cfae68adf71813594cebdcd34121a26eecb693f2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - DCA
  - DESeq2
  - Keras
  - MAGIC
  - SAVER
  - Scanpy
  - anndata
  - kopt
  - scImpute
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 20
---

# scrna-seq-count-denoising-with-zinb-autoencoder

## Summary

Denoise scRNA-seq count matrices with DCA, a deep count autoencoder trained with a zero-inflated negative binomial (ZINB) loss, replacing raw counts with the predicted negative binomial mean. Improves downstream analyses such as differential expression agreement with bulk data, outperforming MAGIC, SAVER, and scImpute.

## When to use

Use when a sparse, overdispersed scRNA-seq count matrix shows dropout and downstream tasks (DE, visualization, clustering) suffer from noise, and counts are the input (not TPM/CPM).

## When NOT to use

- Input is normalized/transformed expression (TPM, CPM, log-transformed) rather than raw integer counts — DCA's NB/ZINB likelihood assumes count structure, overdispersion and sparsity
- The distinction between true (celltype-specific) and false (dropout) zeros is the exact quantity you need validated with ground truth — evaluation of denoising on real data lacks a defined ground truth
- Data are so large that fitting is needed but no GPU is available and hyperparameters were not tuned — DCA scales linearly but training cost still matters

## Inputs

- Raw scRNA-seq count matrix (cells x genes), e.g. h5ad-formatted HDF5 via anndata or Scanpy-readable formats
- Cell type/cluster labels for downstream differential expression evaluation (e.g. Chu et al. H1 vs DEC populations)
- Optional matched bulk RNA-seq samples for validating denoising (bulk DESeq2 log fold changes)

## Outputs

- Denoised count matrix in which each original count is replaced by the predicted negative binomial mean from the DCA model
- Trained DCA autoencoder model (Keras/TensorFlow) with learned noise model parameters
- Downstream analysis outputs on denoised data (e.g. DESeq2 log fold changes, Pearson correlations with bulk LFC across 100 bootstrap iterations of 20 cells per population)

## How to apply

Load raw counts into an anndata object and first remove cells and genes with zero expression; optionally select the top 1000 highly variable genes with Scanpy's filter_genes_dispersion. Train DCA with a ZINB (or NB) loss — default three hidden layers with 64, 32, 64 neurons — using RMSProp at learning rate 0.001, reducing the rate 0.1-fold after 20 epochs without validation loss improvement and stopping early after 25 epochs. Define reconstruction error as the likelihood of the noise-model distribution rather than reconstructing the input, so the compressed bottleneck shares information across features and captures nonlinear gene-gene dependencies. Generate the denoised matrix by replacing original counts with the mean of the negative binomial component. Benchmark against MAGIC, SAVER, and scImpute when quality claims are needed (e.g., Pearson correlation of single-cell vs bulk DESeq2 log fold changes over bootstrap resamples). Optionally tune the bottleneck dimension via hyperopt/kopt with 1000 configurations, since too low-dimensional a bottleneck risks overimputation.

## Related tools

- **DCA** (Deep count autoencoder network trained with ZINB loss that denoises scRNA-seq counts and outputs the predicted NB mean matrix) — https://github.com/theislab/dca
- **Scanpy** (Preprocessing integration — 'dca' method in the preprocessing package and filter_genes_dispersion for highly variable gene selection)
- **anndata** (h5ad-formatted HDF5 storage of count matrices passed into and out of DCA) — https://github.com/theislab/anndata
- **Keras** (Implementation framework for the DCA autoencoder with TensorFlow backend)
- **MAGIC** (Comparison imputation/denoising method benchmarked against DCA (version 0.1.0)) — https://github.com/pkathail/magic
- **SAVER** (Comparison imputation method benchmarked against DCA (version 0.3.0)) — https://github.com/mohuangx/SAVER
- **scImpute** (Comparison mixture-model imputation method benchmarked against DCA (version 0.0.5)) — https://github.com/Vivianstats/scImpute
- **kopt** (Hyperparameter optimization for Keras models used to tune DCA configurations) — https://github.com/Avsecz/kopt

## Examples

```
dca adata, e.g.: from dca.api import dca; dca(adata, verbose=True)  # adata is raw counts in anndata h5ad format
```

## Evaluation signals

- On Chu et al. H1 vs DEC data, DCA-denoised single-cell DESeq2 log fold changes should achieve the highest Pearson correlation with bulk LFC across 100 bootstrap iterations (20 cells per population) compared with MAGIC, SAVER, and scImpute
- Known LFC outliers in the raw data (e.g. LEFTY1) should be corrected in the denoised data
- The denoised matrix should contain smoothed non-negative values derived from the NB mean, with the count structure preserved rather than clipped to a gene-mean constant
- Training should converge with early stopping after 25 epochs without validation loss improvement; reconstruction error (NB/ZINB likelihood) should decrease monotonically on validation data
- Model fit check: the chosen NB or ZINB noise model should minimize binary cross entropy against the mean-dropout rate dependence of the dataset

## Limitations

- It may be difficult to determine when denoising improves scRNA-seq data; evaluation is hard because a ground truth for real data is challenging to define
- Inadequate hyperparameter choices such as too low-dimensional a bottleneck layer can lead to overimputation
- Hyperparameters were not fine-tuned for any of the analyses described in the manuscript; overfitting/overimputation is a general, not yet extensively treated issue of imputation methods
- Classical imputation methods with defined missing values may be unsuitable because of the non-trivial distinction between true and false zero counts

## Evidence

- [abstract] DCA takes the count distribution, overdispersion and sparsity of the data into account using a negative binomial noise model with or without zero-inflation: "DCA takes the count distribution, overdispersion and sparsity of the data into account using a negative binomial noise model with or without zero-inflation"
- [discussion] Denoised matrix is the NB mean: "The denoised matrix is generated by replacing the original count values with the mean of the negative binomial component"
- [discussion] Training optimizer and early stopping: "We used RMSProp for optimization with learning rate 0.001. Learning rate is multiplied by 0.1 if validation loss does not improve for 20 epochs."
- [intro] Likelihood-based reconstruction error rationale: "The trick is to define the reconstruction error as the likelihood of the distribution of the noise model instead of reconstructing the input data itself"
- [discussion] Zero-expression filter before denoising: "First, cells and genes with zero expression are removed from the count matrix."
- [discussion] Overimputation failure mode: "this may also lead to overimputation in case of inadequate hyperparameter choices such as too low-dimensional bottleneck layer and hence data manifold"
- [discussion] DCA implemented in Keras: "DCA is implemented in Python 3 using Keras53 and its TensorFlow54 backend."
- [intro] Scanpy integration: [evidence span withheld — anchor status: not_found]
