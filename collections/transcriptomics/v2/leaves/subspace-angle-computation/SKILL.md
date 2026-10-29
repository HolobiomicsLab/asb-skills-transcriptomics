---
name: subspace-angle-computation
description: Use when comparing low-dimensional structure (top-k PCA subspaces) between
  an original scRNA-seq matrix and its downsampled or imputed version, to test whether
  imputation preserves the signal.
when_to_use_negative:
- Input is not a pair/matched set of original vs. downsampled (or imputed) scRNA-seq
  expression matrices — the angle is only meaningful between comparable PC subspaces
- The number of signal PCs cannot be determined from the original dataset (e.g., no
  jackstraw p-values available), since the same k must be used across conditions
- The goal is imputation quality of individual entries (false-zero recovery) rather
  than preservation of global low-dimensional structure
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ALRA
  role: Produces the rank-k (SVD) imputed downsampled matrices whose PC subspaces
    are compared against the original dataset to test signal preservation
  repo: https://github.com/KlugerLab/ALRA
- name: Seurat
  role: Jackstraw-based selection of the number of signal PCs (p <= 1e-5) retained
    from the original dataset
  repo: https://github.com/satijalab/seurat
- name: pracma
  role: Provides the subspace() function used to compute the angle between PC subspaces
    of original and downsampled/imputed data
- name: rpca-mkl (fastRPCA)
  role: Optional MKL-accelerated randomized SVD backend for large-matrix PCA via ALRA's
    use.mkl flag
  repo: https://github.com/KlugerLab/rpca-mkl
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:113563f317e3f8b7d922d2e1ec6c84e0337c74a524b0616b1321e524c8db171b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KlugerLab/ALRA
  tools:
  - ALRA
  - Seurat
  - pracma
  - rpca-mkl (fastRPCA)
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# subspace-angle-computation

## Summary

Quantifies preservation of biological signal in downsampled scRNA-seq data by computing the subspace angle between the top-k signal PCs of an original dataset and the PCs of the downsampled dataset, before and after ALRA imputation, for Baron, Chen, La Manno, and Zeisel et al.

## When to use

Use when comparing low-dimensional structure (top-k PCA subspaces) between an original scRNA-seq matrix and its downsampled or imputed version, to test whether imputation preserves the signal.

## When NOT to use

- Input is not a pair/matched set of original vs. downsampled (or imputed) scRNA-seq expression matrices — the angle is only meaningful between comparable PC subspaces
- The number of signal PCs cannot be determined from the original dataset (e.g., no jackstraw p-values available), since the same k must be used across conditions
- The goal is imputation quality of individual entries (false-zero recovery) rather than preservation of global low-dimensional structure

## Inputs

- Original normalized scRNA-seq expression matrix (cells x genes) for Baron et al., Chen et al., La Manno et al., and Zeisel et al.
- Downsampled expression matrix for each dataset
- ALRA-imputed downsampled matrix for each dataset
- Number of signal PCs per dataset, chosen by Seurat jackstraw (p <= 1e-5)

## Outputs

- Table of subspace angles per dataset for downsampled-raw and downsampled-imputed conditions (subspace_angles.csv)

## How to apply

Load the original and downsampled scRNA-seq expression matrices (cells as rows, genes as columns) plus the ALRA-imputed downsampled versions. For each original dataset, run PCA and select the number of leading PCs via Seurat's jackstraw, keeping PCs with p-value <= 1e-5. Compute PCA of each downsampled matrix before and after ALRA imputation, retaining the same number of PCs as determined from the corresponding original dataset. Use the subspace function from the R package pracma to compute the angle between the original dataset's PCs and the downsampled (raw vs. imputed) PCs. ALRA's rank-k SVD approximation denoises while preserving true zeros, so the expected behavior is that the subspace angle decreases after imputation (e.g., Baron 0.34->0.25, Chen 0.30->0.17, La Manno 0.36->0.24, Zeisel 0.21->0.15).

## Related tools

- **ALRA** (Produces the rank-k (SVD) imputed downsampled matrices whose PC subspaces are compared against the original dataset to test signal preservation) — https://github.com/KlugerLab/ALRA
- **Seurat** (Jackstraw-based selection of the number of signal PCs (p <= 1e-5) retained from the original dataset) — https://github.com/satijalab/seurat
- **pracma** (Provides the subspace() function used to compute the angle between PC subspaces of original and downsampled/imputed data)
- **rpca-mkl (fastRPCA)** (Optional MKL-accelerated randomized SVD backend for large-matrix PCA via ALRA's use.mkl flag) — https://github.com/KlugerLab/rpca-mkl

## Examples

```
pca_orig <- prcomp(A_norm); pca_ds <- prcatomponent <- prcomp(A_ds); theta <- pracma::subspace(pca_orig$x[,1:k], pca_ds$x[,1:k])
```

## Evaluation signals

- Subspace angle must decrease after ALRA imputation for all four datasets (Baron 0.34->0.25, Chen 0.30->0.17, La Manno 0.36->0.24, Zeisel 0.21->0.15)
- The number of PCs retained must match the jackstraw-derived count (p <= 1e-5) from the corresponding original dataset for both raw and imputed conditions
- Output table subspace_angles.csv contains one angle per dataset per condition (downsampled raw and downsampled imputed)
- PCs are computed on normalized matrices with cells as rows and genes as columns, as ALRA requires

## Limitations

- The number of signal PCs is inherited from the original dataset's jackstraw; if k is chosen differently per condition, angles are not comparable
- ALRA's randomized SVD introduces stochasticity; results can vary and the README notes results improve when using the entire dataset rather than subsets
- rpca-mkl (the optional fast backend) is still under development, not on CRAN, and its pre-compiled MKL libraries only support Linux and OS X

## Evidence

- [readme] ALRA first computes its rank-k approximation using randomized SVD.: "Given a scRNA-seq expression matrix, ALRA first computes its rank-k approximation using randomized SVD."
- [intro] The first step of ALRA is to compute a low-rank approximation of the observed matrix using the singular vector decomposition (SVD): "The first step of ALRA is to compute a low-rank approximation of the observed matrix using the singular vector decomposition (SVD)"
- [intro] the entries of this matrix that correspond to the true biological zeros are symmetrically distributed around zero: "the entries of this matrix that correspond to the true biological zeros are symmetrically distributed around zero"
- [readme] Please be sure to pass ALRA a matrix where the cells are rows and genes are columns.: "Please be sure to pass ALRA a matrix where the cells are rows and genes are columns."
