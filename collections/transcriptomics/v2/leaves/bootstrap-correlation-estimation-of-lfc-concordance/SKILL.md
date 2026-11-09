---
name: bootstrap-correlation-estimation-of-lfc-concordance
description: Use when evaluating whether a denoising/imputation method (DCA, MAGIC,
  SAVER, scImpute) brings single-cell log fold changes closer to matched bulk RNA-seq
  LFCs, e.g., on the Chu et al. H1 vs definitive endoderm data.
when_to_use_negative:
- No matched bulk RNA-seq data exist for the same cell populations, so bulk LFCs cannot
  serve as a reference manifold
- The question is clustering, visualization, or trajectory quality rather than differential
  expression agreement with bulk data
- Populations have fewer than ~20 cells per group, making the 20-cells-per-population
  bootstrap resampling infeasible
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: DCA
  role: Denoises the single-cell count matrix with a ZINB loss, replacing counts with
    the predicted negative binomial mean before DE testing
  repo: https://github.com/theislab/dca
- name: Scanpy
  role: Python API used to run DCA denoising ('dca' method in the preprocessing package)
    within the analysis workflow
- name: DESeq2
  role: Computes log fold changes between H1 and DEC separately on bulk samples and
    on raw/denoised single-cell resamples
- name: MAGIC
  role: Comparison denoising method whose LFC concordance with bulk is benchmarked
    against DCA
  repo: https://github.com/pkathail/magic
- name: SAVER
  role: Comparison imputation method benchmarked in the bootstrap correlation analysis
  repo: https://github.com/mohuangx/SAVER
- name: scImpute
  role: Comparison imputation method benchmarked in the bootstrap correlation analysis
  repo: https://github.com/Vivianstats/scImpute
- name: anndata
  role: h5ad-formatted data container used to hold count matrices and cell labels
    during the workflow
  repo: https://github.com/theislab/anndata
- name: Keras
  role: Backend framework in which DCA is implemented and trained
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:49325260ec9c17248a56a662920c53c9f9381bc7477b52dd7a3d166e8f873bc7
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# bootstrap-correlation-estimation-of-lfc-concordance

## Summary

Assesses whether scRNA-seq denoising improves agreement between single-cell and bulk differential expression by bootstrap-sampling 20 cells per population, running DESeq2 to obtain log fold changes (e.g., H1 vs DEC), and computing Pearson correlation with bulk LFCs across 100 resamples per method.

## When to use

Use when evaluating whether a denoising/imputation method (DCA, MAGIC, SAVER, scImpute) brings single-cell log fold changes closer to matched bulk RNA-seq LFCs, e.g., on the Chu et al. H1 vs definitive endoderm data.

## When NOT to use

- No matched bulk RNA-seq data exist for the same cell populations, so bulk LFCs cannot serve as a reference manifold
- The question is clustering, visualization, or trajectory quality rather than differential expression agreement with bulk data
- Populations have fewer than ~20 cells per group, making the 20-cells-per-population bootstrap resampling infeasible

## Inputs

- Single-cell gene expression count matrix with cell-type labels (e.g., Chu et al. GSE75748 H1 and DEC cells)
- Matched bulk RNA-seq expression samples for the same populations (H1 and DEC)
- Denoised/imputed single-cell matrices produced by DCA (ZINB NB-mean output), MAGIC, SAVER, or scImpute

## Outputs

- Per-bootstrap Pearson correlation values between single-cell and bulk log fold changes for each method (100 iterations)
- Ranked comparison of bulk-LFC concordance across denoising/imputation methods (expected: DCA highest)
- List of outlier LFC genes corrected after denoising (e.g., LEFTY1)

## How to apply

Load the Chu et al. GSE75748 data restricted to single cells and bulk samples from H1 and definitive endoderm (DEC) populations, and run DESeq2 on the bulk samples to obtain reference LFCs. Denoise the single-cell count matrix with DCA using the zero-inflated negative binomial (ZINB) loss, replacing counts with the predicted NB mean (optionally correcting outlier LFCs such as LEFTY1 after denoising). For each of 100 bootstrap iterations, sample 20 cells per population, run DESeq2 on raw and each denoised/imputed matrix, and compute the Pearson correlation between single-cell and bulk LFCs. Compare the correlation distributions across methods (DCA, MAGIC, SAVER, scImpute); correctness is indicated when DCA achieves the highest Pearson correlation with bulk LFCs across the bootstraps, confirming increased agreement with the bulk data manifold.

## Related tools

- **DCA** (Denoises the single-cell count matrix with a ZINB loss, replacing counts with the predicted negative binomial mean before DE testing) — https://github.com/theislab/dca
- **Scanpy** (Python API used to run DCA denoising ('dca' method in the preprocessing package) within the analysis workflow)
- **DESeq2** (Computes log fold changes between H1 and DEC separately on bulk samples and on raw/denoised single-cell resamples)
- **MAGIC** (Comparison denoising method whose LFC concordance with bulk is benchmarked against DCA) — https://github.com/pkathail/magic
- **SAVER** (Comparison imputation method benchmarked in the bootstrap correlation analysis) — https://github.com/mohuangx/SAVER
- **scImpute** (Comparison imputation method benchmarked in the bootstrap correlation analysis) — https://github.com/Vivianstats/scImpute
- **anndata** (h5ad-formatted data container used to hold count matrices and cell labels during the workflow) — https://github.com/theislab/anndata
- **Keras** (Backend framework in which DCA is implemented and trained)

## Examples

```
from dca.api import dca; dca(adata, mode='denoise')  # then per bootstrap: DESeq2 on 20 sampled H1/DEC cells vs bulk, Pearson r of LFCs
```

## Evaluation signals

- Pearson correlation between single-cell and bulk LFCs is computed identically for 100 bootstrap iterations with 20 cells sampled per population each time
- DCA-denoised data achieve the highest Pearson correlation with bulk log fold changes compared to MAGIC, SAVER, and scImpute
- The LEFTY1 log fold change outlier observed after denoising is corrected in the denoised data
- Raw (undenoised) single-cell LFCs serve as a baseline showing lower bulk-LFC correspondence than DCA
- Correlation distributions across bootstraps are reproducible and rank stably across resamples

## Limitations

- Evaluation of denoising is difficult because defining a ground truth for real data is challenging
- It may be difficult to determine in general when denoising improves scRNA-seq data
- Overimputation may occur with inadequate hyperparameter choices (e.g., too low-dimensional bottleneck layer), which would distort LFC estimates
- Hyperparameters were not fine-tuned for any of the analyses described in the manuscript
- No DOI, data accession IDs, or preserved archive (e.g., Zenodo) is provided for datasets or code

## Evidence

- [discussion] Denoising by replacing original counts with the predicted negative binomial mean: "The denoised matrix is generated by replacing the original count values with the mean of the negative binomial component"
- [abstract] ZINB noise model: "using a negative binomial noise model with or without zero-inflation, and nonlinear gene-gene dependencies are"
- [discussion] Chu et al. dataset restriction: "The gene expression data from Chu et al.39 was restricted to single cells and bulk samples from H1 and DEC"
- [intro] Comparison methods: "MAGIC20 and SAVER21, on the other hand, denoise single-cell gene expression data and generate a denoised output for each gene and cell entry"
- [discussion] DCA implementation: "DCA is implemented in Python 3 using Keras53 and its TensorFlow54 backend."
