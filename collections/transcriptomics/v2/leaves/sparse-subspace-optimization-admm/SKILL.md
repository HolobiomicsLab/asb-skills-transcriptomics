---
name: sparse-subspace-optimization-admm
description: Use when a single-cell RNA-seq analysis needs a robust cell-to-cell similarity
  matrix for spectral clustering of cell types, especially with high-noise, high-dropout
  data where pairwise-only similarities are unreliable.
when_to_use_negative:
- The input is already a high-quality, dense cell-to-cell similarity/affinity matrix
  that does not suffer from dropout-induced sparsity.
- The goal is raw gene-expression feature extraction rather than similarity learning
  for cell-type clustering (e.g. input is an unclustered feature table for differential
  analysis).
- Very large-scale datasets where the time cost is prohibitive and parallel acceleration
  is unavailable — the article notes SSRE is relatively time-consuming on large-scale
  data.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Python
  role: Implementation of SSRE's ADMM-based sparse subspace similarity computation
    and similarity-enhancement module (NumPy/pandas pipeline writing esimsparse.csv)
  repo: https://github.com/CSUBioGroup/SSRE
- name: MATLAB
  role: Alternative implementation of the SSRE ADMM solver and similarity enhancement
    available for reproducibility
  repo: https://github.com/CSUBioGroup/SSRE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:8e81f54420d6d128f1d81add1d51d476ef8d79fb6dfe2f914afc0f0d07cfc731
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
yes_rate: 0.9
n_claims: 10
---

# sparse-subspace-optimization-admm

## Summary

Solve SSRE's sparse subspace representation of scRNA-seq cell-to-cell similarity by expressing each cell as a sparse linear combination of other cells (Equations 1-3) via the Alternating Direction Method of Multipliers (ADMM), then enhance the resulting sparse similarity with KNN-based Weighted Adamic/Adar imputation and three classical pairwise similarities into the final matrix Esim_sparse.

## When to use

Use when a single-cell RNA-seq analysis needs a robust cell-to-cell similarity matrix for spectral clustering of cell types, especially with high-noise, high-dropout data where pairwise-only similarities are unreliable.

## When NOT to use

- The input is already a high-quality, dense cell-to-cell similarity/affinity matrix that does not suffer from dropout-induced sparsity.
- The goal is raw gene-expression feature extraction rather than similarity learning for cell-type clustering (e.g. input is an unclustered feature table for differential analysis).
- Very large-scale datasets where the time cost is prohibitive and parallel acceleration is unavailable — the article notes SSRE is relatively time-consuming on large-scale data.

## Inputs

- Preprocessed, gene-selected scRNA-seq expression matrix (cells x genes, e.g. CSV/CSV read with NumPy/pandas)
- K value for nearest-neighbor restriction
- Regularization/sparsity penalty parameters for the ADMM objective of Equations 1-3

## Outputs

- Sparse cell-to-cell similarity matrix sim_sparse (symmetrized self-expression coefficient matrix)
- Enhanced similarity matrix Esim_sparse written to esimsparse.csv
- Cell-type cluster labels from spectral clustering on the learned similarity

## How to apply

Load the preprocessed, gene-selected scRNA-seq expression matrix (cells as observations, genes as features) with NumPy/pandas, then formulate the sparse-subspace objective of Equations 1-3 so each cell is a sparse linear combination of other cells. Solve the optimization with ADMM, alternating coefficient-matrix updates and dual-variable updates until convergence, and symmetrize the learned coefficient matrix to obtain the undirected sparse similarity sim_sparse. Compute the three default pairwise similarities (Pearson correlation, Jaccard, cosine), restrict to each cell's K-nearest neighbors, and impute missing (zero) similarity entries with the modified Weighted Adamic/Adar formula of Equation 5, combining them as in Equation 6. Write the enhanced similarity matrix Esim_sparse to esimsparse.csv and feed it, combined with spectral clustering, to reveal the block structure of the data.

## Related tools

- **Python** (Implementation of SSRE's ADMM-based sparse subspace similarity computation and similarity-enhancement module (NumPy/pandas pipeline writing esimsparse.csv)) — https://github.com/CSUBioGroup/SSRE
- **MATLAB** (Alternative implementation of the SSRE ADMM solver and similarity enhancement available for reproducibility) — https://github.com/CSUBioGroup/SSRE

## Examples

```
import numpy as np, pandas as pd; Isim = ...; Esim_sparse = Isim + Isim.T + sim_sparse; pd.DataFrame(Esim_sparse).to_csv('esimsparse.csv', index=False)
```

## Evaluation signals

- Output matrix is square with rows/columns equal to the number of input cells.
- Output matrix is symmetric (undirected similarity obtained by symmetrizing the coefficient matrix).
- Output matrix contains no missing values after Weighted Adamic/Adar imputation of zero entries.
- Visual inspection confirms block structure consistent with known cell types when combined with spectral clustering.
- Comparison against state-of-the-art single-cell clustering methods (e.g. via NMI/ARI on ten real and five simulated datasets) shows superior or competitive performance.

## Limitations

- SSRE is relatively time-consuming on large-scale datasets; parallel computing is proposed as a possible acceleration strategy but is not implemented in the framework.
- The three pairwise similarities (Pearson correlation, Jaccard, cosine) are used by default, but users may choose only one or two, which changes the enhancement inputs.
- Incorporation of additional biological information (gene functions, gene regulatory relationships) is only proposed as future work, not part of this method.

## Evidence

- [intro] SSRE computes the linear representation between cells based on sparse subspace theory, and thus generates a sparse representation of cell-to-cell similarity: "SSRE computes the linear representation between cells based on sparse subspace theory, and thus generates a sparse representation of cell-to-cell similarity"
- [abstract] SSRE implementations: "The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE."
- [discussion] SSRE limitation: "One limitation of SSRE is relatively time-consuming in large-scale datasets; therefore, parallel computing is a possible strategy to accelerate the framework [60]."
- [discussion] Pairwise similarity choices: "users can choose one or two of the three pairwise similarities mentioned in this study to perform gene selection and similarity enhancement procedures, and all three are used by default"
