---
name: variable-gene-selection-scrnaseq
description: Use when preparing a raw single-cell RNA-seq count matrix for zero-inflated
  or latent-factor dimensionality reduction (ZINB-WaVE, ZIFA, PCA) and you must restrict
  the feature space to the most informative genes first.
when_to_use_negative:
- When the analysis targets genome-wide statistics such as differential expression
  on all genes, rather than low-dimensional representation of cell-to-cell structure
- When the count matrix already contains only a small feature set (a few hundred to
  a few thousand genes), where subsampling blocks or top-variable filtering adds little
  benefit
- When unwanted technical variation is unaccounted for, since gene selection alone
  cannot prevent the inferred low-rank matrix W from capturing confounding effects
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scRNAseq
  role: Bioconductor package used to load the Tasic V1 mouse visual cortex dataset
    whose genes are ranked and subsetted
- name: zinbwave
  role: Fits the zero-inflated negative binomial model on the selected most-variable
    genes; requires R (>=3.4) and Bioconductor (>=3.6); version 0.1.1 reproduces the
    paper exactly
  repo: https://github.com/drisso/zinbwave
- name: ZIFA
  role: Comparison dimensionality-reduction method run on the same filtered/normalized
    gene subset; block_ZIFA recommended for datasets with more than a few thousand
    genes
  repo: https://github.com/epierson9/ZIFA
- name: scater
  role: Used to explore correlation of estimated latent factors with QC measures (detection
    rate, total reads) in the spirit of the scater QC workflow
- name: matrixStats
  role: R package listed as a dependency of the ZINB-WaVE paper analysis, used for
    computing per-gene variability statistics
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:e4d7543be0cc2e46667cdc936a8b50528d81a9f26ee2b2a104dc95b4e6dbb712
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drisso/zinbwave
  tools:
  - Bioconductor (>=3.6)
  - R (>=3.4)
  - ZIFA
  - matrixStats
  - scRNAseq
  - scater
  - zinbwave
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# variable-gene-selection-scrnaseq

## Summary

Preprocesses a scRNA-seq count matrix (e.g., the Tasic V1 mouse visual cortex dataset from the scRNAseq Bioconductor package) by subsetting to the ~1000 most variable genes before dimensionality reduction with ZINB-WaVE, PCA, or ZIFA, reducing computational cost of the non-convex ZINB likelihood optimization while retaining the biological signal.

## When to use

Use when preparing a raw single-cell RNA-seq count matrix for zero-inflated or latent-factor dimensionality reduction (ZINB-WaVE, ZIFA, PCA) and you must restrict the feature space to the most informative genes first.

## When NOT to use

- When the analysis targets genome-wide statistics such as differential expression on all genes, rather than low-dimensional representation of cell-to-cell structure
- When the count matrix already contains only a small feature set (a few hundred to a few thousand genes), where subsampling blocks or top-variable filtering adds little benefit
- When unwanted technical variation is unaccounted for, since gene selection alone cannot prevent the inferred low-rank matrix W from capturing confounding effects

## Inputs

- Raw scRNA-seq gene-by-cell count matrix (e.g., Tasic V1 mouse visual cortex dataset from the scRNAseq Bioconductor package)
- Layer-of-origin / cluster label annotations per cell (for downstream silhouette evaluation)

## Outputs

- Subset count matrix restricted to the 1000 most variable genes
- Total-count (TC) normalized count matrix for the selected genes, fed to zinbwave/PCA/ZIFA

## How to apply

Load the count matrix (e.g., the Tasic V1 dataset via the scRNAseq Bioconductor package) and rank genes by a variability measure computed on the raw or total-count (TC) normalized counts, then retain only the top 1000 most variable genes as input to downstream models. This subset is then TC-normalized and passed to zinbwave, PCA, and ZIFA to extract the first two low-dimensional components. Feature reduction is essential here because ZINB-WaVE requires numerical optimization of a non-convex likelihood, so fitting on all ~20,000+ genes is prohibitively expensive. For ZIFA specifically, the README recommends log-transforming counts (Y = log2(1 + count_data)) and, for datasets with more than a few thousand genes, using block_ZIFA, which subsamples genes in blocks to increase efficiency.

## Related tools

- **scRNAseq** (Bioconductor package used to load the Tasic V1 mouse visual cortex dataset whose genes are ranked and subsetted)
- **zinbwave** (Fits the zero-inflated negative binomial model on the selected most-variable genes; requires R (>=3.4) and Bioconductor (>=3.6); version 0.1.1 reproduces the paper exactly) — https://github.com/drisso/zinbwave
- **ZIFA** (Comparison dimensionality-reduction method run on the same filtered/normalized gene subset; block_ZIFA recommended for datasets with more than a few thousand genes) — https://github.com/epierson9/ZIFA
- **scater** (Used to explore correlation of estimated latent factors with QC measures (detection rate, total reads) in the spirit of the scater QC workflow)
- **matrixStats** (R package listed as a dependency of the ZINB-WaVE paper analysis, used for computing per-gene variability statistics)

## Examples

```
library(scRNAseq); library(matrixStats); se <- Tasic2016V1(); counts <- assay(se); top1000 <- head(order(rowVars(log1p(counts)), decreasing=TRUE), 1000); counts_f <- counts[top1000, ]
```

## Evaluation signals

- The retained feature set contains exactly the intended number of genes (e.g., 1000 most variable) and the original cell-barcode/annotation metadata is preserved
- Downstream ZINB-WaVE on the selected genes achieves the highest average silhouette width against layer-of-origin labels among ZINB-WaVE, PCA, and ZIFA
- The first two ZINB-WaVE components show lower absolute correlation with detection rate and total reads than the corresponding PCA and ZIFA components
- ZIFA is applied on log-transformed counts (log2(1 + count_data)) while ZINB-WaVE models raw counts thanks to the sample-level intercept normalization
- Results (silhouette widths and QC correlations) tabulated in tasic_v1_benchmark_results.csv are consistent with the paper's finding of a more stable low-dimensional representation

## Limitations

- ZINB-WaVE's better performance versus PCA comes at a computational cost: non-convex likelihood optimization, so full-transcriptome fits are expensive — motivating gene pre-selection
- There is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant if unwanted technical variation is unaccounted for
- No compelling examples were found where adding a gene-level covariate improves signal extraction, so gene-level filtering/covariate choices should not be assumed beneficial by default

## Evidence

- [other] Load the Tasic V1 mouse visual cortex dataset with the scRNAseq Bioconductor package and subset it to the 1000 most variable genes.: [evidence span withheld — anchor status: not_found]
- [discussion] Non-convex optimization cost of ZINB-WaVE motivates reducing the feature space.: "The better performance of ZINB-WaVE with respect to PCA comes at a computational cost, as we need to numerically optimize a non-convex likelihood function."
- [readme] ZIFA README guidance on preprocessing count data for high-gene-count datasets.: [evidence span withheld — anchor status: external_doc]
- [readme] For datasets with more than a few thousand genes, block_ZIFA subsamples genes in blocks.: [evidence span withheld — anchor status: external_doc]
- [discussion] zinbwave can model raw counts with no prior normalization via the cell-level intercept.: "the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data, with no need for prior normalization"
- [methods] zinbwave version requirement for reproducibility.: "Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6)."
