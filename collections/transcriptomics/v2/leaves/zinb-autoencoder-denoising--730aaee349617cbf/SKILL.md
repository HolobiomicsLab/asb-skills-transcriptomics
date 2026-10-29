---
name: zinb-autoencoder-denoising
description: Use when a raw scRNA-seq count matrix (e.g. gene x cell CSV/TSV or AnnData
  h5ad) is sparse with dropouts and you need a denoised/imputed matrix that preserves
  count structure for downstream visualization or clustering.
when_to_use_negative:
- Input is already normalized or log-transformed expression rather than raw counts
  — the ZINB loss requires count structure
- Data contains no meaningful count signal to model (e.g. feature tables or bulk data),
  where MSE autoencoders or standard pipelines suffice
- When the goal is to evaluate an already-denoised matrix rather than perform denoising/imputation
  itself
edam_operation: http://edamontology.org/operation_3947
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: DCA
  role: 'Core denoiser: deep count autoencoder with ZINB/NB loss applied to raw count
    matrices; also available as ''dca'' in Scanpy''s preprocessing package'
  repo: https://github.com/theislab/dca
- name: Scanpy
  role: Filtering (e.g. filter_genes_dispersion for highly variable gene selection)
    and downstream PCA/visualization of denoised output
- name: anndata
  role: Load simulated/real count matrices and ground-truth celltype labels into AnnData
    h5ad objects consumed by DCA
  repo: https://github.com/theislab/anndata
- name: kopt
  role: Hyperparameter optimization for the Keras-based autoencoder (TPE search over
    1000 configurations)
  repo: https://github.com/Avsecz/kopt
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:2fd5e5f9604914b866513c7fa8fa1cb5a287983a819a03474ca779cc59b2c1f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - DCA
  - Scanpy
  - anndata
  - kopt
  edam_operation: http://edamontology.org/operation_3947
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# zinb-autoencoder-denoising

## Summary

Denoise scRNA-seq count matrices with DCA, a deep count autoencoder trained with a zero-inflated negative binomial (ZINB) reconstruction loss, so that dropout and overdispersion are handled by the noise model rather than an MSE objective.

## When to use

Use when a raw scRNA-seq count matrix (e.g. gene x cell CSV/TSV or AnnData h5ad) is sparse with dropouts and you need a denoised/imputed matrix that preserves count structure for downstream visualization or clustering.

## When NOT to use

- Input is already normalized or log-transformed expression rather than raw counts — the ZINB loss requires count structure
- Data contains no meaningful count signal to model (e.g. feature tables or bulk data), where MSE autoencoders or standard pipelines suffice
- When the goal is to evaluate an already-denoised matrix rather than perform denoising/imputation itself

## Inputs

- Raw scRNA-seq count matrix as CSV/TSV (genes in rows, cells in columns, with cell and gene labels)
- AnnData h5ad object containing count matrix and celltype labels
- Simulated count matrices with dropout plus no-dropout ground truth (e.g. Splatter two- or six-celltype simulations, 200 genes, 2000 cells)

## Outputs

- mean.tsv — denoised matrix (ZINB negative-binomial mean), same dimensions as input excluding zero-expression genes/cells
- mean_norm.tsv — library size-normalized denoised expression
- pi.tsv and dispersion.tsv — dropout probability and dispersion per cell and gene
- reduced.tsv — 32-dimensional bottleneck (hidden) representation per cell
- Comparison metrics (silhouette, ARI vs ground truth) and PCA plots for validation

## How to apply

Load raw counts as an AnnData h5ad or CSV/TSV gene x cell matrix with mandatory cell and gene labels; DCA first removes cells and genes with zero expression. Train the default network (three hidden layers with 64, 32, 64 neurons) with the ZINB loss, optimizing with RMSProp at learning rate 0.001, reducing the learning rate by 0.1 after 20 epochs without validation-loss improvement, and early stopping after 25 epochs. Denoising replaces original counts with the mean of the negative binomial component; outputs mean.tsv (and mean_norm.tsv with Scanpy normalize_total-style library-size normalization), pi.tsv, dispersion.tsv, and reduced.tsv with the 32-dimensional bottleneck activations. Run hyperparameter search with the --hyper option (1000 configurations optimized via TPE, selecting lowest reconstruction error) when defaults underperform. Validate by PCA colored by celltype labels and comparing silhouette scores / ARI against ground truth or an MSE-autoencoder baseline — DCA's count-aware loss should recover structure the MSE baseline cannot.

## Related tools

- **DCA** (Core denoiser: deep count autoencoder with ZINB/NB loss applied to raw count matrices; also available as 'dca' in Scanpy's preprocessing package) — https://github.com/theislab/dca
- **Scanpy** (Filtering (e.g. filter_genes_dispersion for highly variable gene selection) and downstream PCA/visualization of denoised output)
- **anndata** (Load simulated/real count matrices and ground-truth celltype labels into AnnData h5ad objects consumed by DCA) — https://github.com/theislab/anndata
- **kopt** (Hyperparameter optimization for the Keras-based autoencoder (TPE search over 1000 configurations)) — https://github.com/Avsecz/kopt

## Examples

```
dca matrix.csv results
```

## Evaluation signals

- Output folder contains mean.tsv, mean_norm.tsv, pi.tsv, dispersion.tsv, and reduced.tsv with dimensions matching the input minus zero-expression genes/cells
- Denoised matrix recovers celltype structure: silhouette scores and clustering agreement (ARI) versus ground truth at least as high as an MSE-autoencoder baseline
- PCA of DCA-denoised cells colored by simulated celltype separates groups matching the no-dropout ground truth (Fig. 2-style validation)
- Training converges under the specified schedule: early stopping after 25 epochs without improvement and learning rate drops by 0.1 after 20 stagnant epochs
- reduced.tsv yields a 32-dimensional representation per cell consistent with the default bottleneck

## Limitations

- It may be difficult to determine when denoising improves scRNA-seq data, since defining a ground truth for real data is challenging
- Overimputation can occur with inadequate hyperparameter choices such as a too low-dimensional bottleneck layer; overfitting/overimputation is a general, not yet extensively treated issue of imputation methods
- Hyperparameters were not fine-tuned for any of the analyses described in the manuscript; no DOI or archived (e.g. Zenodo) data accession is provided for datasets or code

## Evidence

- [abstract] We propose a deep count autoencoder network (DCA) to denoise scRNA-seq datasets.: "We propose a deep count autoencoder network (DCA) to denoise scRNA-seq datasets."
- [discussion] Training schedule and architecture: "We used RMSProp for optimization with learning rate 0.001. Learning rate is multiplied by 0.1 if validation loss does not improve for 20 epochs."
- [discussion] Denoised output definition: "The denoised matrix is generated by replacing the original count values with the mean of the negative binomial component"
- [readme] README output files: "`mean.tsv` is the main output of the method which represents the mean parameter of the ZINB distribution."
- [readme] README CLI usage: "`dca matrix.csv results`

where `matrix.csv` is a CSV/TSV-formatted raw count matrix with genes in rows and cells in columns. Cell and gene labels are mandatory."
- [intro] Architecture: "The deep learning framework (by default three hidden layers with 64, 32, 64 neurons) of DCA enables the capturing of the complexity and non-linearity in scRNA-seq data."
