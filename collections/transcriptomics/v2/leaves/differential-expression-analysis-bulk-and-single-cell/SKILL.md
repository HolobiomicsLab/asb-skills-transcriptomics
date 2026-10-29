---
name: differential-expression-analysis-bulk-and-single-cell
description: Use when comparing single-cell vs bulk differential expression agreement
  for a two-population scRNA-seq dataset (e.g. H1 vs DEC), or when benchmarking denoising
  methods (DCA, MAGIC, SAVER, scImpute) by bulk-LFC correspondence.
when_to_use_negative:
- The analysis goal is quantifying denoising quality on datasets without matched bulk
  samples — correlation with bulk LFC cannot be computed.
- The comparison requires more than two populations with complex contrasts — the H1
  vs DEC two-group DESeq2 design does not directly extend.
- The input is already a normalized/imputed expression matrix incompatible with DESeq2's
  raw-count expectation for the raw-data arm of the analysis.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DCA
  role: Denoises the single-cell count matrix using ZINB loss and replaces counts
    with the predicted NB mean before single-cell DE
  repo: https://github.com/theislab/dca
- name: Scanpy
  role: Python API for DCA ('dca' method in preprocessing) and for selecting the top
    1000 highly variable genes via filter_genes_dispersion
- name: anndata
  role: H5AD-formatted data container for count matrices passed through the denoising
    workflow
  repo: https://github.com/theislab/anndata
- name: DESeq2
  role: Differential expression (log fold changes) on bulk samples and single-cell
    data for H1 vs DEC
- name: MAGIC
  role: Comparison denoising method benchmarked by bulk-LFC correlation
  repo: https://github.com/pkathail/magic
- name: SAVER
  role: Comparison imputation method benchmarked by bulk-LFC correlation
  repo: https://github.com/mohuangx/SAVER
- name: scImpute
  role: Comparison imputation method benchmarked by bulk-LFC correlation
  repo: https://github.com/Vivianstats/scImpute
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:aa660431cdffafed225cf20f906f3e9a0b4e3d2798e4cccb9d91376f4927dddf
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# differential-expression-analysis-bulk-and-single-cell

## Summary

Reproduce the DCA paper's Fig. 5 benchmark on the Chu et al. H1 vs definitive endoderm (DEC) data: run DESeq2 log fold change analysis on bulk samples and on raw vs denoised single cells, then bootstrap-correlate single-cell LFCs against bulk LFCs to rank denoising methods.

## When to use

Use when comparing single-cell vs bulk differential expression agreement for a two-population scRNA-seq dataset (e.g. H1 vs DEC), or when benchmarking denoising methods (DCA, MAGIC, SAVER, scImpute) by bulk-LFC correspondence.

## When NOT to use

- The analysis goal is quantifying denoising quality on datasets without matched bulk samples — correlation with bulk LFC cannot be computed.
- The comparison requires more than two populations with complex contrasts — the H1 vs DEC two-group DESeq2 design does not directly extend.
- The input is already a normalized/imputed expression matrix incompatible with DESeq2's raw-count expectation for the raw-data arm of the analysis.

## Inputs

- Chu et al. GSE75748 gene expression matrix (counts) with single-cell and bulk samples annotated by population (H1, DEC)
- Cell-type/population labels distinguishing H1 from definitive endoderm cells and bulk replicates
- Denoised count/expr matrices from DCA, MAGIC, SAVER, and scImpute for method comparison

## Outputs

- Bulk and single-cell log fold change tables (H1 vs DEC) per method
- 100-iteration bootstrap distribution of Pearson correlations between single-cell and bulk LFCs per method
- Ranked comparison of DCA, MAGIC, SAVER, scImpute by bulk-LFC correspondence, with DCA expected highest

## How to apply

Load the Chu et al. GSE75748 matrix restricted to single cells and bulk samples from the H1 and DEC populations, and optionally restrict to the top 1000 highly variable genes selected with Scanpy's 'filter_genes_dispersion' (removing cells and genes with zero expression first). Denoise the single-cell counts with DCA, which fits a zero-inflated negative binomial model via Keras/TensorFlow and generates the denoised matrix by replacing the original count values with the mean of the negative binomial component (Scanpy users can call sc.pp.dca directly). Run DESeq2 differential expression separately on bulk samples and on single-cell data (raw and each denoised variant) to obtain H1-vs-DEC log fold changes, correcting outlier LFCs such as the LEFTY1 outlier seen after denoising. Bootstrap 100 times, sampling 20 cells per population per resample, and compute the Pearson correlation between single-cell and bulk LFCs for each method; DCA should achieve the highest correlation, indicating increased agreement with the bulk data manifold.

## Related tools

- **DCA** (Denoises the single-cell count matrix using ZINB loss and replaces counts with the predicted NB mean before single-cell DE) — https://github.com/theislab/dca
- **Scanpy** (Python API for DCA ('dca' method in preprocessing) and for selecting the top 1000 highly variable genes via filter_genes_dispersion)
- **anndata** (H5AD-formatted data container for count matrices passed through the denoising workflow) — https://github.com/theislab/anndata
- **DESeq2** (Differential expression (log fold changes) on bulk samples and single-cell data for H1 vs DEC)
- **MAGIC** (Comparison denoising method benchmarked by bulk-LFC correlation) — https://github.com/pkathail/magic
- **SAVER** (Comparison imputation method benchmarked by bulk-LFC correlation) — https://github.com/mohuangx/SAVER
- **scImpute** (Comparison imputation method benchmarked by bulk-LFC correlation) — https://github.com/Vivianstats/scImpute

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('chu_gse75748.h5ad'); sc.pp.dca(adata)  # then run DESeq2 on bulk vs denoised cells
```

## Evaluation signals

- Across the 100 bootstraps (20 cells per population per resample), DCA-denoised data yields the highest Pearson correlation with bulk log fold changes of all tested methods.
- The LEFTY1 LFC outlier observed after denoising is corrected in the denoised-data results.
- Each bootstrap computes a Pearson correlation between single-cell and bulk LFC vectors, producing 100 values per method for ranking.
- The denoised matrix equals the predicted negative binomial mean (not raw counts) when DESeq2 is run on the DCA arm.

## Limitations

- It may be difficult to determine in general when denoising improves scRNA-seq data; evaluation is hard because ground truth for real data is challenging to define.
- Inadequate hyperparameters (e.g. too low-dimensional bottleneck layer) can cause overimputation, distorting downstream LFCs.
- Hyperparameters were not fine-tuned for any of the analyses in the manuscript, and no DOI/archived accession for code or data is provided.

## Evidence

- [discussion] Chu et al. dataset restriction: "The gene expression data from Chu et al.39 was restricted to single cells and bulk samples from H1 and DEC"
- [discussion] Denoised output definition: "The denoised matrix is generated by replacing the original count values with the mean of the negative binomial component"
- [discussion] HVG filtering: "the top 1000 highly variable genes are selected using “filter_genes_dispersion” function of Scanpy"
- [intro] Scanpy API for DCA: [evidence span withheld — anchor status: not_found]
- [discussion] DCA implementation stack: "DCA is implemented in Python 3 using Keras53 and its TensorFlow54 backend."
