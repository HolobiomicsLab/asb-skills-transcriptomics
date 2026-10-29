---
name: alra-low-rank-imputation
description: Use when a droplet-based scRNA-seq count matrix (e.g. Drop-seq) contains
  abundant false zeros and you need zero-preserving imputation, or when validating
  imputation against RNA FISH Gini-coefficient correlations.
when_to_use_negative:
- The input contains no zeros to impute, e.g. it is already a completed or FISH-scale
  measurement matrix rather than a sparse scRNA-seq count matrix
- Preserving exact zero values is not required and non-zero-preservation methods such
  as MAGIC/DCA (which output a matrix in which every gene is expressed in every cell)
  are acceptable
- The dataset is so large that runtime matters and rpca-mkl is unavailable; the article
  notes SAVER and scImpute 'may be extremely slow on scRNA-seq datasets with over
  100,000 cells', but very large matrices still benefit from the MKL-accelerated path
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: ALRA
  role: 'Core imputation method: computes rank-k randomized-SVD approximation, thresholds
    each gene row by its most negative value to restore biological zeros, and rescales
    the matrix'
  repo: https://github.com/KlugerLab/ALRA
- name: rsvd
  role: Randomized SVD R package required by ALRA to compute the low-rank approximation
    (install.packages('rsvd'))
  repo: https://github.com/KlugerLab/ALRA
- name: rpca-mkl
  role: Optional Intel MKL out-of-core randomized SVD backend; enabled via ALRA's
    use.mkl flag for dramatic speedups (not on CRAN)
  repo: https://github.com/KlugerLab/rpca-mkl
- name: SAVER
  role: Alternative zero-preserving imputation method used as a comparison baseline
    (Gini correlation ~0.6 vs ~0.8 for ALRA)
- name: Seurat
  role: ALRA is integrated into Seurat as the RunALRA() function; alraSeurat2.R provides
    a wrapper for Seurat v2 objects
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:ec589af0e661a14997fb4da14c9be27a821dc3456a9141a02633950e1d881d8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KlugerLab/ALRA
  tools:
  - ALRA
  - SAVER
  - Seurat
  - rpca-mkl
  - rsvd
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# alra-low-rank-imputation

## Summary

ALRA (Adaptively thresholded Low-Rank Approximation) imputes technical zeros ('dropouts') in scRNA-seq expression matrices via randomized SVD low-rank approximation while preserving true biological zeros through per-gene adaptive thresholding. This skill applies ALRA to normalized single-cell data and validates imputation quality against orthogonal measurements such as RNA FISH Gini coefficients.

## When to use

Use when a droplet-based scRNA-seq count matrix (e.g. Drop-seq) contains abundant false zeros and you need zero-preserving imputation, or when validating imputation against RNA FISH Gini-coefficient correlations.

## When NOT to use

- The input contains no zeros to impute, e.g. it is already a completed or FISH-scale measurement matrix rather than a sparse scRNA-seq count matrix
- Preserving exact zero values is not required and non-zero-preservation methods such as MAGIC/DCA (which output a matrix in which every gene is expressed in every cell) are acceptable
- The dataset is so large that runtime matters and rpca-mkl is unavailable; the article notes SAVER and scImpute 'may be extremely slow on scRNA-seq datasets with over 100,000 cells', but very large matrices still benefit from the MKL-accelerated path

## Inputs

- Normalized scRNA-seq expression matrix with cells as rows and genes as columns (e.g. Drop-seq melanoma data, 8640 cells x 32,287 genes, filtered to 8498 cells x 12,241 genes)
- Optional orthogonal validation data: RNA FISH measurements of 26 genes in the same cell line (16 genes shared with Drop-seq after filtering)
- Rank parameter k (e.g. k=14) or use ALRA's automatic k selection via singular value spacings

## Outputs

- Completed (imputed) expression matrix with technical zeros imputed and biological zeros preserved at zero
- Pearson correlation between FISH and Drop-seq (raw and ALRA-imputed) Gini coefficients for the 15 shared genes, written to gini_fish_correlation.csv

## How to apply

Obtain the raw scRNA-seq matrix (e.g. Drop-seq melanoma data from Torre et al. 2018: 8640 cells, 32,287 genes) and filter cells with library size greater than 20,000 or less than 500 and genes with mean expression less than 0.1, yielding 8498 cells and 12,241 genes. Remove cells in the bottom and top tenth percentiles of GAPDH expression and normalize all genes by GAPDH expression. Pass ALRA a matrix where cells are rows and genes are columns; it computes a rank-k approximation via randomized SVD (rsvd package; k=14 for this dataset, or use choose_k() based on singular value spacings), then restores biological zeros by thresholding each row (gene) by the magnitude of that gene's most negative value, and finally rescales the matrix. Compute Gini coefficients of each shared gene in the FISH data, raw Drop-seq data, and ALRA-imputed data, then calculate the Pearson correlation between FISH and Drop-seq Gini coefficients across the 15 shared genes (excluding GAPDH) and write all values to gini_fish_correlation.csv. Users with rpca-mkl installed can set the use.mkl flag for dramatic speedups.

## Related tools

- **ALRA** (Core imputation method: computes rank-k randomized-SVD approximation, thresholds each gene row by its most negative value to restore biological zeros, and rescales the matrix) — https://github.com/KlugerLab/ALRA
- **rsvd** (Randomized SVD R package required by ALRA to compute the low-rank approximation (install.packages('rsvd'))) — https://github.com/KlugerLab/ALRA
- **rpca-mkl** (Optional Intel MKL out-of-core randomized SVD backend; enabled via ALRA's use.mkl flag for dramatic speedups (not on CRAN)) — https://github.com/KlugerLab/rpca-mkl
- **SAVER** (Alternative zero-preserving imputation method used as a comparison baseline (Gini correlation ~0.6 vs ~0.8 for ALRA))
- **Seurat** (ALRA is integrated into Seurat as the RunALRA() function; alraSeurat2.R provides a wrapper for Seurat v2 objects) — https://github.com/satijalab/seurat

## Examples

```
library(ALRA); result.completed <- alra(A_norm, k=14); A_norm_completed <- result.completed[[3]]
```

## Evaluation signals

- FISH-vs-Drop-seq Gini coefficient Pearson correlation should be approximately 0.8 for ALRA-imputed data (vs approximately 0.3 for raw Drop-seq and approximately 0.6 for SAVER)
- Imputed matrix preserves zeros for biologically non-expressed genes: true biological zeros must remain at zero expression levels after ALRA
- Percent-expressed check on a known subset: after ALRA, NCAM1 should be expressed in essentially all NK cells (4% in raw data) and CR2 in all B cells (1% in raw data)
- Singular value spectrum check: k chosen via choose_k() should correspond to the 'beginning of noise' in singular value spacings (k=14 for the Drop-seq melanoma dataset)
- Output CSV gini_fish_correlation.csv contains Gini values and Pearson correlations for all 15 shared genes (excluding GAPDH)

## Limitations

- ALRA's rank-k approximation assumes the true expression matrix is non-negative, low-rank, and contains many biological zeros; data deviating from this structure may be poorly imputed
- The choice of k can be difficult — it is based on spacings between singular values, and results may vary if k is misspecified
- Other methods like MAGIC and DCA treat all zeros as missing data and output a matrix in which every gene is expressed in every cell, so they are not zero-preserving alternatives; SAVER/scImpute preserve biological zeros but may be extremely slow on datasets with over 100,000 cells
- rpca-mkl pre-compiled MKL libraries are only included for Linux and OS X; the installation will not work for Windows without compiling from source with Intel MKL installed

## Evidence

- [intro] ALRA computes a low-rank approximation of the observed matrix via SVD, then restores biological zeros by thresholding entries using the symmetry of entries corresponding to true biological zeros around zero: [evidence span withheld — anchor status: not_found]
- [readme] ALRA first computes its rank-k approximation using randomized SVD. Next, each row (gene) is thresholded by the magnitude of the most negative value of that gene. Finally, the matrix is rescaled.: "ALRA first computes its rank-k approximation using randomized SVD. Next, each row (gene) is thresholded by the magnitude of the most negative value of that gene. Finally, the matrix is rescaled."
- [readme] Please be sure to pass ALRA a matrix where the cells are rows and genes are columns.: "Please be sure to pass ALRA a matrix where the cells are rows and genes are columns."
- [readme] The k is chosen based on the spacings between the singular values, as it can be quite hard to identify the ``beginning of noise'' from just looking at the spectrum itself.: "The k is chosen based on the spacings between the singular values, as it can be quite hard to identify the ``beginning of noise'' from just looking at the spectrum itself."
- [intro] the fraction of zeros in a typical scRNA-seq expression matrix is much higher, even exceeding 90% with recently developed droplet-based techniques: "the fraction of zeros in a typical scRNA-seq expression matrix is much higher, even exceeding 90% with recently developed droplet-based techniques"
- [readme] The functions now have a flag `use.mkl` for users who have installed [rpca-mkl]: "The functions now have a flag `use.mkl` for users who have installed [rpca-mkl]"
