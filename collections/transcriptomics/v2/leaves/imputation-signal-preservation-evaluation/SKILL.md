---
name: imputation-signal-preservation-evaluation
description: Use when comparing an imputed scRNA-seq matrix to its original counterpart
  to quantify whether imputation preserved the dominant biological structure captured
  by the principal components.
when_to_use_negative:
- When the imputation method under study outputs a matrix in which every gene is expressed
  in every cell (e.g., methods that treat all zeros as missing data such as MAGIC/DCA-style
  dense output), since zero-preservation is not the property being measured
- When no original (undownsampled) reference matrix exists to define the target signal
  subspace
- When the goal is to evaluate imputation accuracy of individual entries rather than
  preservation of the global PC structure
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ALRA
  role: Performs the zero-preserving low-rank imputation of the downsampled matrices
    being evaluated
  repo: https://github.com/KlugerLab/ALRA
- name: Seurat
  role: Jackstraw-based selection of the number of significant PCs (p <= 1e-5) from
    the original datasets
  repo: https://github.com/satijalab/seurat
- name: pracma
  role: Provides the subspace function used to compute angles between PC subspaces
- name: fastRPCA (rpca-mkl)
  role: Optional MKL-accelerated randomized SVD backend for ALRA (use.mkl flag)
  repo: https://github.com/KlugerLab/rpca-mkl
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:0d39b70a94f211340c363634045dee84804d65b37cb0fdb146fba386af74510e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KlugerLab/ALRA
  tools:
  - ALRA
  - Seurat
  - fastRPCA (rpca-mkl)
  - pracma
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# imputation-signal-preservation-evaluation

## Summary

Evaluates whether ALRA's zero-preserving low-rank imputation of downsampled scRNA-seq matrices preserves the biological signal, by measuring the subspace angle between the top-k signal PCs of original versus downsampled/imputed data.

## When to use

Use when comparing an imputed scRNA-seq matrix to its original counterpart to quantify whether imputation preserved the dominant biological structure captured by the leading principal components.

## When NOT to use

- When the imputation method under study outputs a matrix in which every gene is expressed in every cell (e.g., methods that treat all zeros as missing data such as MAGIC/DCA-style dense output), since zero-preservation is not the property being measured
- When no original (undownsampled) reference matrix exists to define the target signal subspace
- When the goal is to evaluate imputation accuracy of individual entries rather than preservation of the global PC structure

## Inputs

- Original (full-depth) normalized scRNA-seq expression matrices (cells x genes)
- Downsampled scRNA-seq expression matrices
- ALRA-imputed versions of the downsampled matrices (e.g., A_norm_completed from alra())
- Number of signal PCs per dataset selected by Seurat jackstraw (p <= 1e-5)

## Outputs

- subspace_angles.csv: table of subspace angles for each dataset and condition (downsampled raw vs. downsampled imputed)
- Per-dataset angle comparisons, e.g., Baron 0.34->0.25, Chen 0.30->0.17, La Manno 0.36->0.24, Zeisel 0.21->0.15

## How to apply

Load the original and downsampled scRNA-seq matrices for each dataset (e.g., Baron et al., Chen et al., La Manno et al., Zeisel et al.) together with their ALRA-imputed downsampled versions. For each original dataset, run PCA and use Seurat's jackstraw method to select the number of signal PCs, keeping PCs with assigned p-value <= 1e-5. Compute PCA of each downsampled matrix before and after ALRA imputation (rank-k via randomized SVD with k chosen by singular value spacings), retaining the same number of PCs as the corresponding original. Use the subspace function from the R package pracma to compute the principal angles between the original PCs and each downsampled/imputed PC subspace, then compile the angles into subspace_angles.csv. Correct application requires the angle to decrease after imputation for every dataset, indicating ALRA denoised toward the original signal rather than distorting it.

## Related tools

- **ALRA** (Performs the zero-preserving low-rank imputation of the downsampled matrices being evaluated) — https://github.com/KlugerLab/ALRA
- **Seurat** (Jackstraw-based selection of the number of significant PCs (p <= 1e-5) from the original datasets) — https://github.com/satijalab/seurat
- **pracma** (Provides the subspace function used to compute angles between PC subspaces)
- **fastRPCA (rpca-mkl)** (Optional MKL-accelerated randomized SVD backend for ALRA (use.mkl flag)) — https://github.com/KlugerLab/rpca-mkl

## Examples

```
library(ALRA); k_choice <- choose_k(A_norm); A_norm_completed <- alra(A_norm, k=k_choice$k)[[3]]
```

## Evaluation signals

- The subspace angle between original and downsampled PCs must decrease after ALRA imputation for all four datasets
- Computed angles must match the reported reductions: 0.34->0.25 (Baron), 0.30->0.17 (Chen), 0.36->0.24 (La Manno), 0.21->0.15 (Zeisel)
- The same number of PCs (jackstraw p <= 1e-5 from the original dataset) must be retained when computing angles for raw and imputed conditions
- subspace_angles.csv must contain one row per dataset per condition with finite angle values in [0, 90] degrees

## Limitations

- ALRA requires the expression matrix with cells as rows and genes as columns; transposed input will produce incorrect results
- The choice of rank k is critical: ALRA's choose_k relies on spacings between singular values, and identifying the 'beginning of noise' from the spectrum alone can be hard
- Results improve when using the entire dataset rather than a subset, so subsampled evaluations may understate performance

## Evidence

- [results] ALRA decreases the subspace angle between the signal PCs of the original and downsampled datasets: from 0.34 to 0.25 for Baron et al: [evidence span withheld — anchor status: not_found]
- [intro] The first step of ALRA is to compute a low-rank approximation of the observed matrix using the singular vector decomposition (SVD): "The first step of ALRA is to compute a low-rank approximation of the observed matrix using the singular vector decomposition (SVD)"
- [intro] the last step of ALRA is to restore the biological zeros in the matrix by thresholding its entries: "the last step of ALRA is to restore the biological zeros in the matrix by thresholding its entries"
- [readme] Please be sure to pass ALRA a matrix where the cells are rows and genes are columns.: "Please be sure to pass ALRA a matrix where the cells are rows and genes are columns."
- [readme] The k is chosen based on the spacings between the singular values: "The k is chosen based on the spacings between the singular values"
