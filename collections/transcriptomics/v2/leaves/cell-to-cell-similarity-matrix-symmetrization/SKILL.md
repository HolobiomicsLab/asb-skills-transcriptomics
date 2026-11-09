---
name: cell-to-cell-similarity-matrix-symmetrization
description: 'Use when a scRNA-seq clustering pipeline needs a dense, undirected cell-to-cell
  similarity matrix: the ADMM-learned coefficient matrix is sparse and asymmetric,
  and zero entries must be imputed (Weighted Adamic/Adar over KNN) before spectral
  clustering.'
when_to_use_negative:
- The input is already a dense, symmetric affinity matrix produced by another framework
  (e.g., MPSSC's Gaussian-kernel doubly stochastic affinity matrices) — no sparse-subspace
  learning or symmetrization step is needed.
- 'Very large-scale scRNA-seq datasets: SSRE is ''relatively time-consuming in large-scale
  datasets'' and parallel acceleration is only proposed as future work, so runtime
  may be prohibitive.'
- 'Single-cell multi-omics integration tasks: the paper only proposes multi-view clustering
  as a possible future trend, not an implemented capability.'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: Python (SSRE implementation)
  role: Runs the SSRE pipeline that solves the sparse self-expression problem with
    ADMM, symmetrizes the coefficient matrix, applies Weighted Adamic/Adar KNN imputation,
    and writes esimsparse.csv.
  repo: https://github.com/CSUBioGroup/SSRE
- name: MATLAB (SSRE implementation)
  role: Reference implementation of the same sparse-subspace representation and similarity-enhancement
    procedure for reproducing the SSRE cell-to-cell similarity matrix.
  repo: https://github.com/CSUBioGroup/SSRE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:19d5e74bbd762ae469306f1774cc67fe40bed316e65a48dd8ae294f8f3b7aa7c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSUBioGroup/SSRE
  tools:
  - MATLAB
  - MATLAB (SSRE implementation)
  - Python
  - Python (SSRE implementation)
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cell-to-cell-similarity-matrix-symmetrization

## Summary

Builds the SSRE cell-to-cell similarity matrix for scRNA-seq data: the sparse self-expression coefficient matrix learned via ADMM is symmetrized into an undirected sparse similarity (simsparse), then enhanced with Pearson correlation, Jaccard, and cosine similarities plus modified Weighted Adamic/Adar imputation over K-nearest neighbors to recover marked missing entries.

## When to use

Use when a scRNA-seq clustering pipeline needs a dense, undirected cell-to-cell similarity matrix: the ADMM-learned coefficient matrix is sparse and asymmetric, and zero entries must be imputed (Weighted Adamic/Adar over KNN) before spectral clustering.

## When NOT to use

- The input is already a dense, symmetric affinity matrix produced by another framework (e.g., MPSSC's Gaussian-kernel doubly stochastic affinity matrices) — no sparse-subspace learning or symmetrization step is needed.
- Very large-scale scRNA-seq datasets: SSRE is 'relatively time-consuming in large-scale datasets' and parallel acceleration is only proposed as future work, so runtime may be prohibitive.
- Single-cell multi-omics integration tasks: the paper only proposes multi-view clustering as a possible future trend, not an implemented capability.

## Inputs

- Preprocessed, gene-selected scRNA-seq expression matrix (cells x genes; loaded via NumPy/pandas)
- KNN neighborhood size K and the ADMM convergence criterion/tolerance for the sparse-subspace solver (method parameters)

## Outputs

- Enhanced symmetric cell-to-cell similarity matrix Esimsparse (n_cells x n_cells, no missing values), written to esimsparse.csv

## How to apply

Load the preprocessed, gene-selected scRNA-seq expression matrix (cells as observations, genes as features) in Python with NumPy/pandas. Formulate the sparse-subspace self-expression objective of Equations 1-3 and solve it with the Alternating Direction Method of Multipliers (ADMM), alternating coefficient-matrix and dual-variable updates until the convergence criterion is met; then symmetrize the learned coefficient matrix to obtain the undirected sparse similarity simsparse. Compute the three default pairwise similarities (Pearson correlation, Jaccard, cosine), restrict each cell to its K-nearest neighbors, and impute marked missing (zero) entries with the modified Weighted Adamic/Adar formula of Equation 5, combining them per Equation 6 into the enhanced matrix Esimsparse = Isimsparse + IsimsparseT + simsparse. Write Esimsparse to esimsparse.csv and verify it is square (n_cells x n_cells), symmetric, and contains no missing values before feeding it, combined with spectral clustering, to detect cell types.

## Related tools

- **Python (SSRE implementation)** (Runs the SSRE pipeline that solves the sparse self-expression problem with ADMM, symmetrizes the coefficient matrix, applies Weighted Adamic/Adar KNN imputation, and writes esimsparse.csv.) — https://github.com/CSUBioGroup/SSRE
- **MATLAB (SSRE implementation)** (Reference implementation of the same sparse-subspace representation and similarity-enhancement procedure for reproducing the SSRE cell-to-cell similarity matrix.) — https://github.com/CSUBioGroup/SSRE

## Examples

```
python -c "import pandas as pd; E=pd.read_csv('esimsparse.csv',index_col=0); assert E.shape[0]==E.shape[1] and (E.values==E.values.T).all() and not E.isna().any().any()"
```

## Evaluation signals

- esimsparse.csv is a square matrix whose rows and columns equal the number of input cells.
- The output matrix is symmetric: Esimsparse equals its transpose, confirming an undirected similarity.
- No missing values remain in the output after Weighted Adamic/Adar imputation of the marked zero entries.
- Spectral clustering on Esimsparse reveals block structure consistent with known cell types.
- Downstream clustering metrics (e.g., NMI/ARI) on the ten real and five simulated benchmark datasets match or exceed state-of-the-art single-cell clustering baselines as reported for SSRE.

## Limitations

- SSRE is relatively time-consuming on large-scale datasets; parallel computing is proposed as a possible acceleration strategy but is not implemented in this work.
- Users can choose one or two of the three pairwise similarities, but all three are used by default — deviating from the default changes the enhanced similarity matrix.
- Incorporating additional biological information (gene functions, gene regulatory relationships) is only proposed as future work, not realized here.
- Multi-view clustering for single-cell multi-omics integration is described only as a possible future trend.

## Evidence

- [abstract] SSRE implementations and repository: "The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE."
- [intro] Sparse subspace similarity basis: "SSRE computes the linear representation between cells based on sparse subspace theory, and thus generates a sparse representation of cell-to-cell similarity"
- [intro] Block structure validation: "The experimental results show that when combined with spectral clustering, the learned similarities by SSRE can reveal the block structure of scRNA-seq data reliably."
- [discussion] Pairwise similarity defaults: "users can choose one or two of the three pairwise similarities mentioned in this study to perform gene selection and similarity enhancement procedures, and all three are used by default"
- [discussion] Scalability limitation: "One limitation of SSRE is relatively time-consuming in large-scale datasets; therefore, parallel computing is a possible strategy to accelerate the framework"
- [abstract] Benchmark scope for evaluation: "Tested on ten real scRNA-seq datasets and five simulated datasets, SSRE achieved the superior performance in most cases compared to several state-of-the-art single-cell clustering methods."
