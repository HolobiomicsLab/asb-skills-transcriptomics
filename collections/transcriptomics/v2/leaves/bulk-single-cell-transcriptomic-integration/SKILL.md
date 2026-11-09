---
name: bulk-single-cell-transcriptomic-integration
description: Use when evaluating whether scRNA-seq denoising brings single-cell expression
  profiles into closer agreement with matched bulk RNA-seq data, e.g. comparing imputation
  methods by LFC correlation on a dataset with paired bulk and single-cell samples
  from the same populations.
when_to_use_negative:
- The dataset has no matched bulk RNA-seq samples for the same cell populations, so
  no bulk-vs-single-cell LFC correlation can be computed.
- The goal is denoising for visualization/clustering rather than quantitative validation
  against a bulk reference; the article notes it 'may be difficult to determine when
  denoising improves scRNA-seq data' in general.
- Data are full-length bulk-like or already aggregated profiles rather than sparse
  single-cell counts, where ZINB/NB denoising assumptions do not apply.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: DCA
  role: Denoises the single-cell count matrix via a deep count autoencoder with ZINB
    loss; denoised matrix replaces counts with the predicted negative binomial mean
  repo: https://github.com/theislab/dca
- name: Scanpy
  role: Python API compatible with DCA; users can call the 'dca' method in the preprocessing
    package
- name: DESeq2
  role: Runs differential expression separately on bulk and single-cell data to obtain
    H1-vs-DEC log fold changes
- name: anndata
  role: h5ad-formatted HDF5 data container used to hold the count matrices
  repo: https://github.com/theislab/anndata
- name: Keras
  role: Backend framework in which DCA is implemented (Python 3, TensorFlow backend)
- name: MAGIC
  role: Comparison denoising method benchmarked against DCA in the bootstrap correlation
    analysis
  repo: https://github.com/pkathail/magic
- name: SAVER
  role: Comparison imputation method benchmarked against DCA in the bootstrap correlation
    analysis
  repo: https://github.com/mohuangx/SAVER
- name: scImpute
  role: Comparison imputation method benchmarked against DCA in the bootstrap correlation
    analysis
  repo: https://github.com/Vivianstats/scImpute
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:d7d2b02dfda238e676f48883b70514018f735c9bbedbc2abc58f4a530dab425b
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
  - scImpute
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# bulk-single-cell-transcriptomic-integration

## Summary

Reproduces the DCA paper's bulk-vs-single-cell differential expression agreement analysis on the Chu et al. H1 vs definitive endoderm (DEC) dataset, testing whether denoising scRNA-seq data (DCA, MAGIC, SAVER, scImpute) improves the Pearson correlation between single-cell and bulk DESeq2 log fold changes across 100 bootstrap resamples.

## When to use

Use when evaluating whether scRNA-seq denoising brings single-cell expression profiles into closer agreement with matched bulk RNA-seq data, e.g. comparing imputation methods by LFC correlation on a dataset with paired bulk and single-cell samples from the same populations.

## When NOT to use

- The dataset has no matched bulk RNA-seq samples for the same cell populations, so no bulk-vs-single-cell LFC correlation can be computed.
- The goal is denoising for visualization/clustering rather than quantitative validation against a bulk reference; the article notes it 'may be difficult to determine when denoising improves scRNA-seq data' in general.
- Data are full-length bulk-like or already aggregated profiles rather than sparse single-cell counts, where ZINB/NB denoising assumptions do not apply.

## Inputs

- Chu et al. GSE75748 gene expression data restricted to single cells and bulk samples from H1 and DEC populations
- Raw single-cell count matrix (e.g. h5ad/anndata or count table)
- Bulk RNA-seq sample count table with H1/DEC labels

## Outputs

- Log fold changes (H1 vs DEC) from DESeq2 for bulk and for raw/denoised single-cell data
- Per-method Pearson correlations between single-cell and bulk LFCs across 100 bootstrap iterations (20 cells per population each)
- Ranking of denoising/imputation methods (DCA, MAGIC, SAVER, scImpute) by bulk-LFC agreement
- Corrected outlier LFC list (e.g. LEFTY1) for denoised data

## How to apply

Load the Chu et al. GSE75748 data restricted to single cells and bulk samples from the H1 and DEC populations. Denoise the single-cell count matrix with the DCA deep count autoencoder using the zero-inflated negative binomial (ZINB) loss, replacing original counts with the predicted negative binomial mean. Run DESeq2 differential expression separately on bulk samples and on single-cell data (raw and denoised) to obtain H1-vs-DEC log fold changes, and correct outlier LFCs introduced by denoising (e.g. the LEFTY1 outlier). Bootstrap 100 times, sampling 20 cells per population per resample, and compute the Pearson correlation between single-cell and bulk log fold changes for each method (DCA, MAGIC, SAVER, scImpute). Validation: DCA should achieve the highest Pearson correlation with bulk LFCs across the 100 bootstraps, with LEFTY1's LFC outlier corrected in the denoised data.

## Related tools

- **DCA** (Denoises the single-cell count matrix via a deep count autoencoder with ZINB loss; denoised matrix replaces counts with the predicted negative binomial mean) — https://github.com/theislab/dca
- **Scanpy** (Python API compatible with DCA; users can call the 'dca' method in the preprocessing package)
- **DESeq2** (Runs differential expression separately on bulk and single-cell data to obtain H1-vs-DEC log fold changes)
- **anndata** (h5ad-formatted HDF5 data container used to hold the count matrices) — https://github.com/theislab/anndata
- **Keras** (Backend framework in which DCA is implemented (Python 3, TensorFlow backend))
- **MAGIC** (Comparison denoising method benchmarked against DCA in the bootstrap correlation analysis) — https://github.com/pkathail/magic
- **SAVER** (Comparison imputation method benchmarked against DCA in the bootstrap correlation analysis) — https://github.com/mohuangx/SAVER
- **scImpute** (Comparison imputation method benchmarked against DCA in the bootstrap correlation analysis) — https://github.com/Vivianstats/scImpute

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('GSE75748_h1_dec.h5ad'); sc.pp.dca(adata); r = pearsonr(adata.varm['lfc_sc'], lfc_bulk)
```

## Evaluation signals

- DCA-denoised single-cell data achieves the highest Pearson correlation with bulk log fold changes among all tested methods across 100 bootstrap iterations.
- The LEFTY1 log fold change outlier observed after denoising is corrected in the denoised data.
- Each bootstrap resample contains exactly 20 cells per population (H1 and DEC), and correlations are computed over all genes between single-cell and bulk LFCs.
- Denoised matrix values correspond to the predicted negative binomial means (not the raw input counts), per the DCA denoising procedure.

## Limitations

- Evaluation of denoising is difficult because defining a ground truth for real data is challenging.
- Overimputation may occur with inadequate hyperparameter choices, such as too low-dimensional a bottleneck layer; hyperparameters were not fine-tuned for any analysis in the manuscript.
- Overfitting/overimputation is a general, not yet extensively treated issue of imputation methods.
- No DOI, data accession IDs, or preserved archive (e.g., Zenodo) is provided for datasets or code, which hampers exact reproducibility.

## Evidence

- [results] Bootstrap design: [evidence span withheld — anchor status: not_found]
- [discussion] Denoising procedure: "The denoised matrix is generated by replacing the original count values with the mean of the negative binomial component"
- [discussion] Dataset restriction: "The gene expression data from Chu et al.39 was restricted to single cells and bulk samples from H1 and DEC"
- [intro] Scanpy integration: [evidence span withheld — anchor status: not_found]
- [intro] Benchmarked methods: "MAGIC20 and SAVER21, on the other hand, denoise single-cell gene expression data and generate a denoised output for each gene and cell entry"
- [readme] README description: "A deep count autoencoder network to denoise scRNA-seq data and remove the dropout effect by taking the count structure, overdispersed nature and sparsity of the data into account"
