---
name: negative-binomial-autoencoder-denoising
description: Use when the input is a raw scRNA-seq count matrix (e.g. .h5ad from 10X
  Genomics) with dropout-affected counts that must be denoised/imputed before correlation,
  visualization, or differential analyses, or when benchmarking denoising methods.
when_to_use_negative:
- 'When a well-defined ground truth or well-calibrated denoising objective is required
  for real data: the article notes it may be difficult to determine when denoising
  improves scRNA-seq data and that ground truth definition is challenging'
- 'When overimputation is unacceptable without careful model selection: inadequate
  hyperparameter choices such as a too-low-dimensional bottleneck layer can cause
  overimputation'
- 'When hyperparameters were tuned on downstream analyses: the authors warn that hyperparameters
  were not fine-tuned for any analysis in the manuscript, so blindly tuning may inflate
  apparent gains'
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DCA
  role: Deep count autoencoder network that performs the NB/ZINB-loss denoising and
    imputation in one step
  repo: https://github.com/theislab/dca
- name: Scanpy
  role: Python API compatible with DCA; provides the dca preprocessing method and
    filter_genes_dispersion for HVG selection
- name: anndata
  role: Reads/writes the h5ad-formatted HDF5 count matrices used as DCA input/output
  repo: https://github.com/theislab/anndata
- name: kopt
  role: Hyper-parameter optimization for Keras (with hyperopt, TPE) used for optional
    DCA hyperparameter search
  repo: https://github.com/Avsecz/kopt
- name: MAGIC
  role: Competing denoising method benchmarked for runtime and quality against DCA
  repo: https://github.com/pkathail/magic
- name: SAVER
  role: Competing imputation method benchmarked for runtime and quality against DCA
  repo: https://github.com/mohuangx/SAVER
- name: scImpute
  role: Competing imputation method benchmarked for runtime and quality against DCA
  repo: https://github.com/Vivianstats/scImpute
provenance:
  source_task_ids:
  - task_008
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:3ef99d810db2e78a69566ba0534c574f075d991f353ca22f5632f9bf020871cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - DCA
  - DCA (deep count autoencoder network)
  - Keras
  - MAGIC
  - SAVER
  - Scanpy
  - TensorFlow
  - anndata
  - kopt
  - scImpute
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# negative-binomial-autoencoder-denoising

## Summary

Denoise single-cell RNA-seq count matrices with a deep count autoencoder (DCA) that uses a negative binomial (or zero-inflated NB) loss so the reconstruction error is the likelihood of a count noise model, imputing dropouts while preserving overdispersion and gene-gene dependencies.

## When to use

Use when the input is a raw scRNA-seq count matrix (e.g. .h5ad from 10X Genomics) with dropout-affected counts that must be denoised/imputed before correlation, visualization, or differential analyses, or when benchmarking denoising methods.

## When NOT to use

- When a well-defined ground truth or well-calibrated denoising objective is required for real data: the article notes it may be difficult to determine when denoising improves scRNA-seq data and that ground truth definition is challenging
- When overimputation is unacceptable without careful model selection: inadequate hyperparameter choices such as a too-low-dimensional bottleneck layer can cause overimputation
- When hyperparameters were tuned on downstream analyses: the authors warn that hyperparameters were not fine-tuned for any analysis in the manuscript, so blindly tuning may inflate apparent gains

## Inputs

- Raw scRNA-seq count matrix in h5ad-formatted HDF5 (anndata) or 10X format
- Optional: ADT protein measurements (CITE-seq) paired per cell as ground truth
- Optional: large count matrix for downsampling-based runtime scaling (e.g. 1.3 million cells, 10X Genomics single-cell-3prime-paper)

## Outputs

- Denoised expression matrix (original counts replaced by predicted negative binomial means)
- Per-gene/per-cell denoised values for downstream Spearman correlation, detection-rate, or clustering analyses
- Benchmark tables (e.g. protein_mrna_correlations.csv, runtime_scaling_results.csv) with fitted metrics such as log-log scaling slope

## How to apply

Load the count matrix with anndata/Scanpy and run DCA with its negative binomial noise model (default architecture: three hidden layers of 64-32-64 neurons) using Keras with a TensorFlow backend, RMSProp optimization at learning rate 0.001 (multiplied by 0.1 if validation loss does not improve for 20 epochs, early stopping after 25 epochs), without size-factor normalization. The key trick is to define the reconstruction error as the likelihood of the noise-model distribution rather than reconstructing the input itself, so the compressed bottleneck shares information across features and captures nonlinear gene-gene dependencies. Generate the denoised matrix by replacing the original counts with the predicted negative binomial mean. Optionally select the noise model (NB vs ZINB) by fitting both to the mean-dropout-rate dependence while minimizing binary cross entropy; hyperparameter search with 1000 TPE-sampled configurations via hyperopt/kopt is available but was not needed for the paper's analyses. For runtime benchmarking, downsample large matrices (e.g. the 1.3M-cell 10X dataset to 100-100,000 cells, top 1000 highly variable genes via Scanpy filter_genes_dispersion, removing cells/genes with zero expression), run each method on CPU, and fit a log-log regression of wall-clock time versus cell count.

## Related tools

- **DCA** (Deep count autoencoder network that performs the NB/ZINB-loss denoising and imputation in one step) — https://github.com/theislab/dca
- **Scanpy** (Python API compatible with DCA; provides the dca preprocessing method and filter_genes_dispersion for HVG selection)
- **anndata** (Reads/writes the h5ad-formatted HDF5 count matrices used as DCA input/output) — https://github.com/theislab/anndata
- **kopt** (Hyper-parameter optimization for Keras (with hyperopt, TPE) used for optional DCA hyperparameter search) — https://github.com/Avsecz/kopt
- **MAGIC** (Competing denoising method benchmarked for runtime and quality against DCA) — https://github.com/pkathail/magic
- **SAVER** (Competing imputation method benchmarked for runtime and quality against DCA) — https://github.com/mohuangx/SAVER
- **scImpute** (Competing imputation method benchmarked for runtime and quality against DCA) — https://github.com/Vivianstats/scImpute

## Examples

```
import scanpy as sc; sc.pp.filter_genes_dispersion(adata, n_top_genes=1000); import dca; dca.dca(adata, mode='denoise')
```

## Evaluation signals

- After CITE-seq denoising, median Spearman correlation across protein-mRNA pairs (e.g. CD3, CD4, CD8, CD127) should increase relative to original counts; CD3E detection should rise from ~80% to ~99.9% of T cells, matching the CD3 protein ground truth
- Runtime benchmark: fitted log-log slope of DCA CPU runtime versus cell count should be close to 1 (paper: 0.66), and DCA should denoise 100,000 cells in minutes versus hours for MAGIC/SAVER/scImpute
- Confirm the denoised matrix was produced by replacing original counts with the predicted negative binomial component mean, not raw reconstructions
- Training diagnostics: validation loss plateau triggers learning-rate reduction by 0.1 after 20 non-improving epochs and early stopping after 25 epochs
- Output object should be an anndata-compatible (h5ad) matrix restricted to the filtered gene set (e.g. top 1000 highly variable genes) with zero-expression cells/genes removed

## Limitations

- It may be difficult to determine when denoising improves scRNA-seq data, and evaluation is hard because defining ground truth for real data is challenging
- Inadequate hyperparameter choices such as a too-low-dimensional bottleneck layer and hence data manifold may lead to overimputation
- Overfitting/overimputation is a general, not yet extensively treated issue of imputation methods; hyperparameters were not fine-tuned for any analysis in the manuscript
- Hardware specifics are only partially specified (scalability analysis on a server with two Intel Xeon E5-2620 2.40 GHz CPUs), and no DOI/Zeodo archive is provided for datasets or code

## Evidence

- [intro] NB loss likelihood trick: "The trick is to define the reconstruction error as the likelihood of the distribution of the noise model instead of reconstructing the input data itself"
- [discussion] Denoising by NB mean replacement: "The denoised matrix is generated by replacing the original count values with the mean of the negative binomial component"
- [intro] Default architecture and scalability: "The deep learning framework (by default three hidden layers with 64, 32, 64 neurons) of DCA enables the capturing of the complexity and non-linearity in scRNA-seq data."
- [discussion] Training schedule: "We used RMSProp for optimization with learning rate 0.001. Learning rate is multiplied by 0.1 if validation loss does not improve for 20 epochs."
- [abstract] Scalability finding: "Our method scales linearly with the number of cells and can, therefore, be applied to datasets of millions of cells."
- [discussion] Overimputation caveat: "this may also lead to overimputation in case of inadequate hyperparameter choices such as too low-dimensional bottleneck layer"
- [readme] README install: "A deep count autoencoder network to denoise scRNA-seq data and remove the dropout effect by taking the count structure, overdispersed nature and sparsity of the data into account"
