---
name: scrnaseq-principal-component-analysis
description: Use when evaluating whether denoising or imputation of scRNA-seq data
  preserves biological structure, by comparing principal component subspaces of original
  vs. perturbed/imputed matrices.
when_to_use_negative:
- When the goal is imputation itself rather than evaluating signal preservation of
  an imputation method
- When input matrices are not cell-by-gene normalized expression matrices (e.g., already
  a reduced-dimension embedding or feature table)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ALRA
  role: Imputation method whose signal preservation is evaluated; computes rank-k
    approximation via randomized SVD and adaptively thresholds entries
  repo: https://github.com/KlugerLab/ALRA
- name: Seurat
  role: Provides the jackstraw method to select the number of significant signal PCs
    (p-value <= 1e-5) retained from the original dataset
  repo: https://github.com/satijalab/seurat
- name: pracma
  role: R package providing the subspace function used to compute principal angles
    between PC subspaces of original and downsampled/imputed data
- name: rpca-mkl (fastRPCA)
  role: MKL-accelerated randomized SVD used for fast low-rank approximation of large
    expression matrices
  repo: https://github.com/KlugerLab/rpca-mkl
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:3c7872f39f538fa019c7ba2464416a24e4a21bfd123f446cc481edf0924305e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KlugerLab/ALRA
  tools:
  - ALRA
  - Seurat
  - pracma
  - rpca-mkl (fastRPCA)
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# scrnaseq-principal-component-analysis

## Summary

Compute principal components of scRNA-seq expression matrices and quantify how well an imputation method (e.g., ALRA) preserves the biological signal, by measuring the subspace angle between the top-k signal PCs of original vs. downsampled data before and after imputation.

## When to use

Use when evaluating whether denoising or imputation of scRNA-seq data preserves biological structure, by comparing principal component subspaces of original vs. perturbed/imputed matrices.

## When NOT to use

- When the goal is imputation itself rather than evaluating signal preservation of an imputation method
- When input matrices are not cell-by-gene normalized expression matrices (e.g., already a reduced-dimension embedding or feature table)

## Inputs

- Original normalized scRNA-seq expression matrix (cells as rows, genes as columns)
- Downsampled scRNA-seq expression matrices
- ALRA-imputed downsampled expression matrices
- Number of leading signal PCs per dataset (chosen via Seurat jackstraw, p <= 1e-5)

## Outputs

- Table of subspace angles between original and downsampled PCs before and after imputation (subspace_angles.csv)
- Jackstraw-derived PC count per dataset

## How to apply

For each original scRNA-seq dataset (cells as rows, genes as columns), compute PCA and select the number of signal PCs to retain using Seurat's jackstraw method, keeping PCs with an assigned p-value of 1e-5 or smaller. Compute PCA of each perturbed matrix (e.g., downsampled raw and downsampled ALRA-imputed versions), retaining the same number of PCs determined from the corresponding original dataset. Use the subspace function from the R package pracma to compute the subspace angle between the original dataset's PCs and the downsampled PCs for each condition. A correct application shows the angle decreasing after ALRA imputation (e.g., Baron 0.34->0.25, Chen 0.30->0.17, La Manno 0.36->0.24, Zeisel 0.21->0.15), demonstrating ALRA preserves signal despite denoising; ALRA itself computes a rank-k approximation via SVD and thresholds entries to restore biological zeros.

## Related tools

- **ALRA** (Imputation method whose signal preservation is evaluated; computes rank-k approximation via randomized SVD and adaptively thresholds entries) — https://github.com/KlugerLab/ALRA
- **Seurat** (Provides the jackstraw method to select the number of significant signal PCs (p-value <= 1e-5) retained from the original dataset) — https://github.com/satijalab/seurat
- **pracma** (R package providing the subspace function used to compute principal angles between PC subspaces of original and downsampled/imputed data)
- **rpca-mkl (fastRPCA)** (MKL-accelerated randomized SVD used for fast low-rank approximation of large expression matrices) — https://github.com/KlugerLab/rpca-mkl

## Examples

```
library(ALRA); library(pracma); k_choice <- choose_k(A_norm); A_norm_completed <- alra(A_norm, k=k_choice$k)[[3]]; angle <- subspace(prcomp(A_norm)$x[,1:k], prcomp(A_norm_completed)$x[,1:k])
```

## Evaluation signals

- Subspace angle must decrease after ALRA imputation for all four datasets (Baron 0.34->0.25, Chen 0.30->0.17, La Manno 0.36->0.24, Zeisel 0.21->0.15)
- Same number of PCs (from original-dataset jackstraw, p <= 1e-5) must be used for both raw and imputed comparisons per dataset
- Output table subspace_angles.csv contains angles for all four datasets under both conditions (downsampled raw and downsampled imputed)
- PCA is computed on matrices with cells as rows and genes as columns, matching ALRA's required orientation

## Limitations

- ALRA requires the matrix oriented with cells as rows and genes as columns; wrong orientation invalidates results
- ALRA is primarily intended for scRNA-seq data assumed non-negative, low-rank, with many biological zeros; other data types may violate assumptions
- The automatic choice of rank k is based on singular value spacings and may warrant manual inspection in some cases

## Evidence

- [results] ALRA decreases the subspace angle between the signal PCs of the original and downsampled datasets: from 0.34 to 0.25 for Baron et al, 0.30 to 0.17 for Chen et al, 0.36 to 0.24 for La Manno et al, and 0.21 to 0.15 for Zeisel et al: [evidence span withheld — anchor status: not_found]
- [intro] The first step of ALRA is to compute a low-rank approximation of the observed matrix using the singular vector decomposition (SVD): "The first step of ALRA is to compute a low-rank approximation of the observed matrix using the singular vector decomposition (SVD)"
- [readme] Please be sure to pass ALRA a matrix where the cells are rows and genes are columns.: "Please be sure to pass ALRA a matrix where the cells are rows and genes are columns."
- [readme] The k is chosen based on the spacings between the singular values, as it can be quite hard to identify the ``beginning of noise'' from just looking at the spectrum itself.: "The k is chosen based on the spacings between the singular values, as it can be quite hard to identify the ``beginning of noise''"
