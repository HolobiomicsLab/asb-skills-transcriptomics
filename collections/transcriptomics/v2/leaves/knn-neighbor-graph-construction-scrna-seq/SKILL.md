---
name: knn-neighbor-graph-construction-scrna-seq
description: Use when a scRNA-seq expression matrix yields a sparse cell-to-cell similarity
  (e.g., from ADMM-solved sparse subspace representation) whose zero entries must
  be imputed within K-nearest-neighbor neighborhoods to obtain a dense similarity
  for spectral clustering.
when_to_use_negative:
- The input similarity matrix is already dense and contains no missing/zero entries
  to impute
- The data are very large-scale, since SSRE is reported to be relatively time-consuming
  on large datasets
- The task requires single-cell multi-omics integration, which the authors only propose
  as future work
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Python
  role: Implementation language for the SSRE pipeline including KNN-restricted Weighted
    Adamic/Adar imputation and writing esimsparse.csv
  repo: https://github.com/CSUBioGroup/SSRE
- name: MATLAB
  role: Implementation language for the SSRE pipeline including the similarity-enhancement
    module
  repo: https://github.com/CSUBioGroup/SSRE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:946080c711937eafb868bad9f86595cdb38793b0e622338ff85d91ad48470569
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
yes_rate: 0.7777777777777778
n_claims: 9
---

# knn-neighbor-graph-construction-scrna-seq

## Summary

Constructs an enhanced cell-to-cell similarity matrix for scRNA-seq clustering by restricting each cell to its K-nearest neighbors in a sparse subspace-representation similarity (simsparse) and imputing missing (zero) entries with the modified Weighted Adamic/Adar index over common neighbors, then symmetrizing with the three classical pairwise similarities.

## When to use

Use when a scRNA-seq expression matrix yields a sparse cell-to-cell similarity (e.g., from ADMM-solved sparse subspace representation) whose zero entries must be imputed within K-nearest-neighbor neighborhoods to obtain a dense similarity for spectral clustering.

## When NOT to use

- The input similarity matrix is already dense and contains no missing/zero entries to impute
- The data are very large-scale, since SSRE is reported to be relatively time-consuming on large datasets
- The task requires single-cell multi-omics integration, which the authors only propose as future work

## Inputs

- Preprocessed (gene-selected) scRNA-seq expression matrix with cells as observations and genes as features
- Sparse cell-to-cell similarity matrix simsparse obtained from the ADMM-solved sparse subspace representation
- Three pairwise similarity matrices (Pearson correlation, Jaccard, cosine)
- K, the number of nearest neighbors per cell

## Outputs

- Enhanced similarity matrix Esimsparse, written to esimsparse.csv
- Dense, symmetric cell-to-cell similarity matrix suitable as input to spectral clustering

## How to apply

First solve the sparse-subspace representation objective (Equations 1-3) with ADMM, alternating coefficient-matrix and dual-variable updates until convergence, and symmetrize the learned coefficients to obtain the undirected sparse similarity simsparse. Compute the three default pairwise similarities SSRE uses — Pearson correlation, Jaccard, and cosine — among cells. For each cell, restrict to its K-nearest neighbors and apply the modified Weighted Adamic/Adar formula of Equation 5 to impute the missing (zero) similarity entries, combining these with the three pairwise similarities as in Equation 6. Finally, compute the enhanced matrix Esimsparse = Isimsparse + Isimsparse^T + simsparse, which yields a more comprehensive cell-to-cell similarity matrix. Users may select one or two of the three pairwise similarities, but all three are used by default.

## Related tools

- **Python** (Implementation language for the SSRE pipeline including KNN-restricted Weighted Adamic/Adar imputation and writing esimsparse.csv) — https://github.com/CSUBioGroup/SSRE
- **MATLAB** (Implementation language for the SSRE pipeline including the similarity-enhancement module) — https://github.com/CSUBioGroup/SSRE

## Examples

```
import pandas as pd; pd.DataFrame(Esimsparse).to_csv('esimsparse.csv')  # Esimsparse = I_simsparse + I_simsparse.T + simsparse, with KNN-restricted Weighted Adamic/Adar imputation
```

## Evaluation signals

- Output matrix Esimsparse is square with rows and columns equal to the number of input cells
- Esimsparse is symmetric
- Esimsparse contains no missing values (previously zero entries within KNN neighborhoods are imputed)
- Spectral clustering on the learned similarities reveals a block structure consistent with known cell types
- Esimsparse = Isimsparse + Isimsparse^T + simsparse matches the formula in the paper

## Limitations

- SSRE is relatively time-consuming on large-scale datasets; parallel computing is proposed but not implemented as acceleration
- Users can choose one or two of the three pairwise similarities, but deviating from the default of all three may change similarity quality
- Effectiveness depends on the quality of the upstream sparse subspace representation learned via ADMM

## Evidence

- [methods] SSRE imputes marked missing values in the sparse similarity using the modified Weighted Adamic/Adar over common neighbors: [evidence span withheld — anchor status: not_found]
- [methods] For each cell, restrict to its K-nearest neighbors and apply the modified Weighted Adamic/Adar formula of Equation 5: [evidence span withheld — anchor status: not_found]
- [intro] SSRE computes the linear representation between cells based on sparse subspace theory, and thus generates a sparse representation of cell-to-cell similarity: "SSRE computes the linear representation between cells based on sparse subspace theory, and thus generates a sparse representation of cell-to-cell similarity"
- [discussion] users can choose one or two of the three pairwise similarities mentioned in this study to perform gene selection and similarity enhancement procedures, and all three are used by default: "users can choose one or two of the three pairwise similarities mentioned in this study to perform gene selection and similarity enhancement procedures, and all three are used by default"
- [abstract] The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE.: "The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE."
