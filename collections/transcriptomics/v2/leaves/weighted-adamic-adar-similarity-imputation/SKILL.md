---
name: weighted-adamic-adar-similarity-imputation
description: Use when a sparse cell-to-cell similarity matrix learned from scRNA-seq
  data (e.g., via sparse subspace representation) has zero/missing entries that must
  be imputed before spectral clustering of cell types.
when_to_use_negative:
- Input is already a dense, complete similarity matrix with no missing (zero) similarity
  entries to impute
- The task requires large-scale datasets where SSRE is known to be time-consuming
  and parallel acceleration has not been implemented
- The clustering target does not involve single-cell RNA-seq cell-type identification
  or similarity-learning-based spectral clustering
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Python
  role: Implementation language for imputing sparse similarity entries with the modified
    Weighted Adamic/Adar formula and writing esimsparse.csv
  repo: https://github.com/CSUBioGroup/SSRE
- name: MATLAB
  role: Alternative implementation language for the SSRE similarity-enhancement procedure
  repo: https://github.com/CSUBioGroup/SSRE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:06a5fe93c898e5f3e5e238605a3711ba06aa0dcf50c6ec9c822e0eecff81f560
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSUBioGroup/SSRE
  tools:
  - MATLAB
  - Python
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# weighted-adamic-adar-similarity-imputation

## Summary

A similarity-enhancement procedure from SSRE that imputes missing (zero) entries in a sparse cell-to-cell similarity matrix using a modified Weighted Adamic/Adar measure over K-nearest neighbors, then combines the result with the symmetrized sparse similarity and three classical pairwise similarities (Pearson correlation, Jaccard, cosine) into an enhanced matrix E_sim_sparse.

## When to use

Use when a sparse cell-to-cell similarity matrix learned from scRNA-seq data (e.g., via sparse subspace representation) has zero/missing entries that must be imputed before spectral clustering of cell types.

## When NOT to use

- Input is already a dense, complete similarity matrix with no missing (zero) similarity entries to impute
- The task requires large-scale datasets where SSRE is known to be time-consuming and parallel acceleration has not been implemented
- The clustering target does not involve single-cell RNA-seq cell-type identification or similarity-learning-based spectral clustering

## Inputs

- Preprocessed (gene-selected) scRNA-seq expression matrix with cells as observations and genes as features (NumPy/pandas)
- Sparse cell-to-cell similarity matrix (sim_sparse) obtained from ADMM-solved sparse subspace representation
- Pairwise similarity matrices: Pearson correlation, Jaccard, and cosine similarity among cells
- K-nearest-neighbor sets per cell

## Outputs

- Enhanced cell-to-cell similarity matrix E_sim_sparse (square, symmetric, no missing values) written to esimsparse.csv

## How to apply

First solve the sparse-subspace self-expression objective (SSRE Equations 1-3) with ADMM, alternating coefficient-matrix and dual-variable updates until convergence, and symmetrize the coefficient matrix into sim_sparse. For each cell, restrict to its K-nearest neighbors and apply the modified Weighted Adamic/Adar formula (Equation 5) over common neighbors to impute marked missing similarity values, combining with the three default pairwise similarities (Pearson correlation, Jaccard, cosine) as in Equation 6. Finally, assemble the enhanced matrix E_sim_sparse = I_sim_sparse + I_sim_sparse^T + sim_sparse and write it to esimsparse.csv. Users may choose one or two of the three pairwise similarities instead, but all three are used by default.

## Related tools

- **Python** (Implementation language for imputing sparse similarity entries with the modified Weighted Adamic/Adar formula and writing esimsparse.csv) — https://github.com/CSUBioGroup/SSRE
- **MATLAB** (Alternative implementation language for the SSRE similarity-enhancement procedure) — https://github.com/CSUBioGroup/SSRE

## Examples

```
E_sim_sparse = I_sim_sparse + I_sim_sparse.T + sim_sparse; pd.DataFrame(E_sim_sparse).to_csv('esimsparse.csv')
```

## Evaluation signals

- Output matrix E_sim_sparse is square with rows/columns equal to the number of input cells
- Output matrix is symmetric (E_sim_sparse == E_sim_sparse.T) and contains no missing (NaN) values
- Imputed entries are populated only for K-nearest-neighbor pairs, reflecting the restricted-neighborhood Weighted Adamic/Adar computation
- Visual inspection confirms expected block structure of the enhanced similarity matrix consistent with known cell types
- Downstream spectral clustering on E_sim_sparse recovers cell-type clusters better than on the raw sparse similarity alone (SSRE reported superior performance on ten real and five simulated scRNA-seq datasets)

## Limitations

- SSRE is relatively time-consuming on large-scale datasets; parallel computing is proposed but not implemented as an acceleration strategy
- The three pairwise similarities are used by default; selecting one or two changes the imputation combination and must be justified per dataset
- Incorporation of additional biological information (gene functions, gene regulatory relationships) is only proposed as future work, not implemented in this enhancement step

## Evidence

- [intro] SSRE computes linear representation between cells based on sparse subspace theory to generate a sparse representation of cell-to-cell similarity: "SSRE computes the linear representation between cells based on sparse subspace theory, and thus generates a sparse representation of cell-to-cell similarity"
- [discussion] Users may choose one or two of three pairwise similarities: "users can choose one or two of the three pairwise similarities mentioned in this study to perform gene selection and similarity enhancement procedures, and all three are used by default"
- [abstract] MATLAB and Python implementations available: "The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE."
- [abstract] SSRE performance claim on real and simulated datasets: "Tested on ten real scRNA-seq datasets and five simulated datasets, SSRE achieved the superior performance in most cases compared to several state-of-the-art single-cell clustering methods."
