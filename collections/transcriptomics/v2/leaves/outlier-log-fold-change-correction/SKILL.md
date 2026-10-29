---
name: outlier-log-fold-change-correction
description: Use when comparing single-cell vs bulk differential expression log fold
  changes after DCA denoising and isolated genes show extreme LFC outliers in the
  denoised data that were not present in raw data or bulk samples.
when_to_use_negative:
- The input log fold changes come from raw (non-denoised) single-cell data where the
  outlier arose from dropout rather than from denoising artifacts
- The comparison is between denoised data and a ground truth other than bulk samples,
  since the article notes defining ground truth for real data is challenging
- Denoising was performed with hyperparameters producing overimputation (e.g., too
  low-dimensional bottleneck layer), which may itself create distorted gene-level
  values
tools:
- name: DCA
  role: Denoises the scRNA-seq count matrix (ZINB loss) whose output LFCs are checked
    and corrected for outliers such as LEFTY1
  repo: https://github.com/theislab/dca
- name: Scanpy
  role: Python API through which the DCA 'dca' preprocessing method can be invoked
    to produce the denoised matrix before DE analysis
- name: DESeq2
  role: Computes bulk and single-cell log fold changes between H1 and DEC used to
    detect and correct outlier genes
- name: anndata
  role: Data container (h5ad-formatted HDF5) holding counts and denoised expression
    for the correction workflow
  repo: https://github.com/theislab/anndata
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:de842b0d9622fd7be4c4828a3bce017f0c9e8e813fecd8ab4b98620abd59147c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/dca
  tools:
  - DCA
  - DESeq2
  - Keras
  - Scanpy
  - anndata
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# outlier-log-fold-change-correction

## Summary

A corrective step applied to differential expression log fold changes computed on DCA-denoised single-cell data, where individual genes (e.g., LEFTY1 in the Chu et al. H1 vs DEC data) show spurious outlier LFCs after denoising that would otherwise deflate correspondence with bulk LFCs.

## When to use

Use when comparing single-cell vs bulk differential expression log fold changes after DCA denoising and isolated genes show extreme LFC outliers in the denoised data that were not present in raw data or bulk samples.

## When NOT to use

- The input log fold changes come from raw (non-denoised) single-cell data where the outlier arose from dropout rather than from denoising artifacts
- The comparison is between denoised data and a ground truth other than bulk samples, since the article notes defining ground truth for real data is challenging
- Denoising was performed with hyperparameters producing overimputation (e.g., too low-dimensional bottleneck layer), which may itself create distorted gene-level values

## Inputs

- Raw Chu et al. GSE75748 single-cell count matrix (H1 and DEC populations)
- Bulk RNA-seq samples from H1 and DEC populations
- Denoised single-cell expression matrix (counts replaced with predicted NB mean from DCA)
- DESeq2 log fold change tables for bulk and single-cell (raw and denoised) comparisons

## Outputs

- Corrected single-cell log fold change table with outliers (e.g., LEFTY1) adjusted
- Per-bootstrap Pearson correlations between single-cell and bulk log fold changes across 100 resamples
- Method ranking of denoising/imputation approaches (DCA, MAGIC, SAVER, scImpute) by bulk-vs-single-cell agreement

## How to apply

After denoising the single-cell count matrix by replacing counts with the predicted negative binomial mean from the DCA deep count autoencoder (zero-inflated negative binomial loss), run DESeq2 separately on bulk samples and on the sampled single cells to obtain H1 vs DEC log fold changes. Inspect the denoised single-cell LFC distribution for genes whose LFC departs strongly from both the raw single-cell and bulk values, exemplified by the LEFTY1 outlier in the Chu et al. GSE75748 dataset. Correct these outlier LFCs before computing correspondence statistics. Then bootstrap 100 times, sampling 20 cells per population (H1 and definitive endoderm) per resample, and compute the Pearson correlation between single-cell and bulk log fold changes per method (DCA, MAGIC, SAVER, scImpute).

## Related tools

- **DCA** (Denoises the scRNA-seq count matrix (ZINB loss) whose output LFCs are checked and corrected for outliers such as LEFTY1) — https://github.com/theislab/dca
- **Scanpy** (Python API through which the DCA 'dca' preprocessing method can be invoked to produce the denoised matrix before DE analysis)
- **DESeq2** (Computes bulk and single-cell log fold changes between H1 and DEC used to detect and correct outlier genes)
- **anndata** (Data container (h5ad-formatted HDF5) holding counts and denoised expression for the correction workflow) — https://github.com/theislab/anndata

## Examples

```
import scanpy as sc; sc.pp.dca(adata)  # denoise counts with ZINB loss, then run DESeq2 and correct outlier LFCs (e.g., LEFTY1) before correlating with bulk
```

## Evaluation signals

- The LEFTY1 log fold change outlier observed in denoised single-cell data is corrected after adjustment
- Across 100 bootstrap iterations (20 cells per population per resample), DCA-denoised data achieve the highest Pearson correlation with bulk log fold changes of all tested methods (DCA, MAGIC, SAVER, scImpute)
- Correlations computed on corrected LFCs should not be lower than correlations computed on the uncorrected denoised LFCs
- The denoised matrix was generated by replacing original counts with the negative binomial component mean, not by imputing zeros in place
- The same genes (restricted, e.g., to highly variable genes) are compared across bulk and single-cell LFC vectors

## Limitations

- Evaluation of denoising is difficult because the definition of a ground truth can be challenging for real data
- Overimputation may occur with inadequate hyperparameter choices (e.g., too low-dimensional bottleneck layer and hence data manifold)
- Hyperparameters were not fine-tuned for any of the analyses described in the manuscript, so outlier behavior may depend on default settings

## Evidence

- [discussion] denoised output replaces counts with NB mean: "The denoised matrix is generated by replacing the original count values with the mean of the negative binomial component"
- [abstract] bulk-vs-single-cell correspondence rationale: "We demonstrate that DCA denoising improves a diverse set of typical scRNA-seq data analyses using simulated and real datasets."
- [readme] outlier correction motivated by denoising artifacts: "remove the dropout effect by taking the count structure, overdispersed nature and sparsity of the data into account"
- [discussion] ground-truth limitation: "The evaluation of denoising is difficult because the definition of a ground truth can be challenging for real data."
