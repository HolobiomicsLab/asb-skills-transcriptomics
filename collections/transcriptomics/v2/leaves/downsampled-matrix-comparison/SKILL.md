---
name: downsampled-matrix-comparison
description: Use when comparing normalized scRNA-seq matrices across downsampling
  or imputation conditions and you need a rotation-invariant measure (subspace angle
  of signal PCs) of whether multivariate structure is preserved.
when_to_use_negative:
- The goal is entry-level matrix comparison (e.g., RMSE or correlation of imputed
  values) rather than preservation of low-rank subspace structure
- The data has not been normalized (ALRA expects library- and log-normalized input
  with cells as rows and genes as columns), so PCA/jackstraw-based comparison would
  be confounded by library size
- No matched original/downsampled pair exists — subspace angles require the same gene
  space and a common reference PC basis
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: ALRA
  role: Imputes the downsampled normalized expression matrix via adaptively-thresholded
    low-rank approximation, producing the imputed condition whose subspace angle is
    compared to the raw downsampled condition
  repo: https://github.com/KlugerLab/ALRA
- name: Seurat
  role: Provides the jackstraw procedure used on the original datasets to choose the
    number of significant leading PCs (p ≤ 1e-5) retained for the subspace-angle comparison
  repo: https://github.com/satijalab/seurat
- name: pracma
  role: Supplies the subspace function used to compute the angle between the leading
    PC subspaces of original and downsampled/imputed matrices
- name: rpca-mkl / RSVD
  role: Optional randomized-SVD backend (use.mkl flag in ALRA) accelerating the rank-k
    approximation underlying imputation
  repo: https://github.com/KlugerLab/rpca-mkl
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:07c703cf78019a069d1291a2331982639a6ddf2b40569c734bacf06980724045
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KlugerLab/ALRA
  tools:
  - ALRA
  - Seurat
  - pracma
  - rpca-mkl / RSVD
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# downsampled-matrix-comparison

## Summary

Quantifies whether ALRA preserves biological signal in downsampled scRNA-seq data by computing the subspace angle between the top-k principal components (k chosen via Seurat jackstraw, p ≤ 1e-5) of original vs. downsampled matrices, before and after ALRA imputation.

## When to use

Use when comparing normalized scRNA-seq matrices across downsampling or imputation conditions and you need a rotation-invariant measure (subspace angle of leading signal PCs) of whether multivariate structure is preserved.

## When NOT to use

- The goal is entry-level matrix comparison (e.g., RMSE or correlation of imputed values) rather than preservation of low-rank subspace structure
- The data has not been normalized (ALRA expects library- and log-normalized input with cells as rows and genes as columns), so PCA/jackstraw-based comparison would be confounded by library size
- No matched original/downsampled pair exists — subspace angles require the same gene space and a common reference PC basis

## Inputs

- Original normalized scRNA-seq expression matrices (cells × genes) for Baron, Chen, La Manno, and Zeisel datasets
- Downsampled versions of those matrices
- ALRA-imputed downsampled matrices (from alra() applied to the downsampled normalized data)
- Number of signal PCs selected by Seurat jackstraw (p ≤ 1e-5) per original dataset

## Outputs

- Table of subspace angles (radians) between original-dataset signal PCs and downsampled PCs, per dataset and condition (raw vs. imputed), written to subspace_angles.csv
- Per-dataset PC counts selected by jackstraw

## How to apply

Load the original and downsampled expression matrices (Baron, Chen, La Manno, Zeisel) plus their ALRA-imputed downsampled versions. For each original dataset, run PCA and use Seurat's jackstraw to retain leading PCs with assigned p-value ≤ 1e-5, fixing that PC count for the matching downsampled condition. Compute PCA of the downsampled matrices before and after ALRA imputation, then use the subspace function from the R package pracma to measure the angle between original and downsampled PCs. Rationale: ALRA computes a rank-k approximation via randomized SVD and restores biological zeros by thresholding, so the subspace angle between signal PCs tests whether this denoising distorts the low-rank biological structure rather than only comparing entries. Expected result: the angle decreases after ALRA imputation for all four datasets (Baron 0.34→0.25, Chen 0.30→0.17, La Manno 0.36→0.24, Zeisel 0.21→0.15).

## Related tools

- **ALRA** (Imputes the downsampled normalized expression matrix via adaptively-thresholded low-rank approximation, producing the imputed condition whose subspace angle is compared to the raw downsampled condition) — https://github.com/KlugerLab/ALRA
- **Seurat** (Provides the jackstraw procedure used on the original datasets to choose the number of significant leading PCs (p ≤ 1e-5) retained for the subspace-angle comparison) — https://github.com/satijalab/seurat
- **pracma** (Supplies the subspace function used to compute the angle between the leading PC subspaces of original and downsampled/imputed matrices)
- **rpca-mkl / RSVD** (Optional randomized-SVD backend (use.mkl flag in ALRA) accelerating the rank-k approximation underlying imputation) — https://github.com/KlugerLab/rpca-mkl

## Examples

```
library(ALRA); A_norm_completed <- alra(normalize_data(A_downsampled), k=k_choice$k)[[3]]; angle <- pracma::subspace(prcomp(A_norm)$x[,1:k], prcomp(A_norm_completed)$x[,1:k])
```

## Evaluation signals

- Subspace angle must decrease after ALRA imputation for all four datasets (Baron 0.34→0.25, Chen 0.30→0.17, La Manno 0.36→0.24, Zeisel 0.21→0.15)
- The number of PCs retained for the downsampled conditions equals the jackstraw-selected count (p ≤ 1e-5) from the matching original dataset
- subspace_angles.csv contains one raw and one imputed angle per dataset, computed with pracma::subspace against the same original PC basis
- ALRA was run on normalized input with cells as rows and genes as columns, and output taken as result.completed[[3]]

## Limitations

- ALRA's randomized-SVD rank-k choice can require manual inspection; the README notes k is chosen from singular value spacings and 'in some cases you might want to take a closer look'
- Downsampled-data results may be weaker than full-dataset results: the README notes 'the results improve when using the entire dataset'
- rpca-mkl is 'still under development and is not on CRAN, so it is not a required package'; MKL pre-compiled libraries are only included for Linux and OS X

## Evidence

- [results] ALRA decreases the subspace angle between the signal PCs of the original and downsampled datasets: from 0.34 to 0.25 for Baron et al, 0.30 to 0.17 for Chen et al, 0.36 to 0.24 for La Manno et al, and 0.21 to 0.15 for Zeisel et al: [evidence span withheld — anchor status: not_found]
- [methods] selecting PCs via Seurat jackstraw with p <= 1e-5: [evidence span withheld — anchor status: not_found]
- [methods] computing the subspace angle with pracma: [evidence span withheld — anchor status: not_found]
- [readme] ALRA computes its rank-k approximation via randomized SVD: "ALRA first computes its rank-k approximation using randomized SVD. Next, each row (gene) is thresholded by the magnitude of the most negative value"
- [intro] the true biological zeros are symmetrically distributed around zero: "the entries of this matrix that correspond to the true biological zeros are symmetrically distributed around zero"
