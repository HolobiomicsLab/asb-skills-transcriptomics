---
name: jackstraw-pc-significance-testing
description: Use when you must choose how many PCs to retain in a scRNA-seq PCA objectively
  (jackstraw p-values), or quantify whether imputation/denoising preserved the signal
  subspace via subspace angles.
when_to_use_negative:
- The input is not a scRNA-seq expression matrix but already a dimensionality-reduced
  embedding or feature table — jackstraw significance testing requires raw gene-level
  PCA.
- The question is whether ALRA's imputed values match held-out entries directly (use
  the out-of-bag-error skill instead of subspace-angle comparison).
- Datasets have no downsampled counterpart, since the angle-reduction validation (e.g.,
  Baron 0.34->0.25) is defined only for original/downsampled pairs.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Provides the jackstraw permutation method for assigning significance p-values
    to PCs and selecting how many to retain
  repo: https://github.com/satijalab/seurat
- name: ALRA
  role: Produces the imputed downsampled matrices whose post-imputation PC subspace
    is compared against the original via subspace angles
  repo: https://github.com/KlugerLab/ALRA
- name: pracma
  role: R package whose subspace function computes the angle between original and
    downsampled/imputed PC subspaces
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:f4424989de9a76a365d6f22fd24ebc47249da4323204f54e469e4eec3daae446
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/satijalab/seurat
  tools:
  - ALRA
  - Seurat
  - pracma
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# jackstraw-pc-significance-testing

## Summary

Selects the number of signal-bearing principal components from scRNA-seq PCA using Seurat's jackstraw permutation-based significance testing (retaining PCs with p-value ≤ 1e-5), then measures subspace agreement between PC bases of original, downsampled, and ALRA-imputed matrices with pracma's subspace function.

## When to use

Use when you must choose how many leading PCs to retain in a scRNA-seq PCA objectively (jackstraw p-values), or quantify whether imputation/denoising preserved the signal subspace via subspace angles.

## When NOT to use

- The input is not a scRNA-seq expression matrix but already a dimensionality-reduced embedding or feature table — jackstraw significance testing requires raw gene-level PCA.
- The question is whether ALRA's imputed values match held-out entries directly (use the out-of-bag-error skill instead of subspace-angle comparison).
- Datasets have no downsampled counterpart, since the angle-reduction validation (e.g., Baron 0.34->0.25) is defined only for original/downsampled pairs.

## Inputs

- Original scRNA-seq normalized expression matrices (cells x genes) for Baron et al., Chen et al., La Manno et al., and Zeisel et al.
- Matched downsampled expression matrices (cells x genes)
- ALRA-imputed downsampled matrices from the upstream out-of-bag-error task

## Outputs

- Per-dataset jackstraw PC p-values and the selected number of significant PCs (k)
- Subspace angles between original and downsampled PCs (raw) and original and imputed PCs (post-ALRA)
- subspace_angles.csv summarizing angles per dataset and condition

## How to apply

For each original scRNA-seq matrix, run PCA and apply Seurat's jackstraw method, retaining only PCs assigned a p-value of 1e-5 or smaller to define the signal rank k. Project the same k PCs for matched downsampled matrices, both before and after ALRA imputation (ALRA computes a rank-k SVD approximation and thresholds entries to restore biological zeros, so its output should span a subspace closer to the original). For each dataset and condition, compute the subspace angle between the original dataset's PC vectors and the downsampled/imputed PC vectors using the subspace function from the R package pracma. Compile angles into a table (subspace_angles.csv); correct application is judged by the angle decreasing after ALRA imputation, as ALRA's zero-preserving low-rank completion should recover signal lost to dropout in the downsampled data.

## Related tools

- **Seurat** (Provides the jackstraw permutation method for assigning significance p-values to PCs and selecting how many to retain) — https://github.com/satijalab/seurat
- **ALRA** (Produces the imputed downsampled matrices whose post-imputation PC subspace is compared against the original via subspace angles) — https://github.com/KlugerLab/ALRA
- **pracma** (R package whose subspace function computes the angle between original and downsampled/imputed PC subspaces)

## Examples

```
library(pracma); angle_baron_imputed <- subspace(P_original_k, P_downsampled_alra_k)
```

## Evaluation signals

- Every retained PC from the original datasets has a jackstraw p-value ≤ 1e-5.
- The same number of PCs k is used for the original, downsampled, and imputed comparisons of a given dataset.
- Subspace angles decrease after ALRA imputation for all four datasets: Baron 0.34->0.25, Chen 0.30->0.17, La Manno 0.36->0.24, Zeisel 0.21->0.15.
- subspace_angles.csv is produced with one angle per dataset per condition (downsampled raw and downsampled imputed).
- ALRA inputs follow the README convention of cells as rows and genes as columns, with library and log normalization applied before completion.

## Limitations

- Jackstraw PC selection depends on the original dataset's noise structure; using a different k than the one chosen from the original would break the subspace-angle comparison.
- ALRA is intended for scRNA-seq expression matrices; the README notes results improve when using the entire dataset rather than subsets, so angles estimated on small downsampled matrices may be pessimistic.
- Subspace angle reduction demonstrates signal preservation but not accuracy of individual imputed entries.

## Evidence

- [abstract] We present a method based on low-rank matrix approximation which imputes these values while preserving biologically non-expressed genes (true biological zeros) at zero expression levels.: "We present a method based on low-rank matrix approximation which imputes these values while preserving biologically non-expressed genes (true biological zeros) at zero expression levels."
- [other] For each original dataset, compute PCA and select the number of leading PCs to retain using Seurat's jackstraw method, keeping PCs with an assigned p-value of 1e-5 or smaller.: [evidence span withheld — anchor status: not_found]
- [other] Compute the subspace angle between the original dataset's PCs and the downsampled PCs using the subspace function from the R package pracma.: [evidence span withheld — anchor status: not_found]
- [readme] Please be sure to pass ALRA a matrix where the cells are rows and genes are columns.: "Please be sure to pass ALRA a matrix where the cells are rows and genes are columns."
- [readme] ALRA is integrated into Seurat v3.0 as function RunALRA().: "ALRA is integrated into [Seurat v3.0](https://github.com/satijalab/seurat/tree/release/3.0) (currently at pre-release stage) as function `RunALRA()`."
