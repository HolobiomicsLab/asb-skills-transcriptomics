---
name: pairwise-similarity-computation-scrna-seq
description: Use when clustering scRNA-seq expression data whose high dropout and
  noise make naive two-cell similarity unreliable, and a robust cell-to-cell similarity
  matrix is needed as input to spectral clustering.
when_to_use_negative:
- The analysis goal is direct clustering on raw expression features without similarity
  learning — spectral clustering on the learned similarity is the SSRE pathway
- Very large-scale datasets where the time cost is prohibitive; the authors note SSRE
  is 'relatively time-consuming in large-scale datasets' and parallel acceleration
  was not implemented
- Only a simple two-cell expression comparison is needed — SSRE's value comes from
  incorporating similar neighbors, not pairwise-only computation
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Python
  role: 'Implementation language for the SSRE pipeline: loading the expression matrix,
    ADMM optimization, pairwise similarity computation, and Adamic/Adar-based enhancement'
  repo: https://github.com/CSUBioGroup/SSRE
- name: MATLAB
  role: Alternative implementation language for the SSRE pipeline
  repo: https://github.com/CSUBioGroup/SSRE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:a793bdf3a5fda0a2a9f39e38ac366050a791b4975221943368c7e3681f1b4d5f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSUBioGroup/SSRE
  tools:
  - MATLAB
  - Python
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# pairwise-similarity-computation-scrna-seq

## Summary

Computes cell-to-cell similarity for scRNA-seq clustering by combining a sparse-subspace self-expression matrix (learned via ADMM) with three classical pairwise similarities (Pearson correlation, Jaccard, and cosine), then enhancing the sparse similarity using KNN-based modified Weighted Adamic/Adar imputation. SSRE uses this enhanced similarity with spectral clustering to reveal cell-type block structure.

## When to use

Use when clustering scRNA-seq expression data whose high dropout and noise make naive two-cell similarity unreliable, and a robust cell-to-cell similarity matrix is needed as input to spectral clustering.

## When NOT to use

- The analysis goal is direct clustering on raw expression features without similarity learning — spectral clustering on the learned similarity is the SSRE pathway
- Very large-scale datasets where the time cost is prohibitive; the authors note SSRE is 'relatively time-consuming in large-scale datasets' and parallel acceleration was not implemented
- Only a simple two-cell expression comparison is needed — SSRE's value comes from incorporating similar neighbors, not pairwise-only computation

## Inputs

- Preprocessed gene-selected scRNA-seq expression matrix (cells as observations, genes as features; e.g., in NumPy/pandas or .mat format)
- Number of nearest neighbors K for the similarity-enhancement step
- Sparsity/regularization penalty parameters for the ADMM optimization

## Outputs

- Enhanced cell-to-cell similarity matrix Esimsparse written to esimsparse.csv (square, symmetric, one row/column per cell, no missing values)
- Sparse cell-to-cell similarity matrix simsparse from the symmetrized self-expression coefficients

## How to apply

Load the preprocessed, gene-selected expression matrix (cells x genes) in Python or MATLAB. Solve the sparse-subspace objective (Equations 1-3) with ADMM, alternating coefficient-matrix and dual-variable updates until convergence, then symmetrize the learned coefficient matrix to obtain the undirected sparse similarity simsparse. Compute Pearson correlation, Jaccard, and cosine similarities among cells (all three are used by default; users may select one or two). For each cell, restrict to its K-nearest neighbors and impute missing (zero) sparse-similarity entries with the modified Weighted Adamic/Adar formula (Equation 5) over common neighbors, combining with the three pairwise similarities as in Equation 6 to yield Esimsparse = Isimsparse + Isimsparse^T + simsparse.

## Related tools

- **Python** (Implementation language for the SSRE pipeline: loading the expression matrix, ADMM optimization, pairwise similarity computation, and Adamic/Adar-based enhancement) — https://github.com/CSUBioGroup/SSRE
- **MATLAB** (Alternative implementation language for the SSRE pipeline) — https://github.com/CSUBioGroup/SSRE

## Examples

```
import numpy as np; from scipy.spatial.distance import pdist, squareform; S_cos = 1 - squareform(pdist(X, metric='cosine'))  # cosine term of SSRE's three pairwise similarities
```

## Evaluation signals

- Output Esimsparse is square with rows/columns equal to the number of input cells
- Esimsparse is symmetric (undirected similarity) and contains no missing values
- Visual inspection of the similarity matrix shows block structure consistent with known cell types
- Downstream spectral clustering on Esimsparse recovers cell-type labels comparable to state-of-the-art methods (NMI/Purity/ARI-style benchmarking on real and simulated scRNA-seq datasets)

## Limitations

- SSRE is relatively time-consuming on large-scale datasets; parallel computing is proposed but not implemented
- Dropout-induced missing values are imputed only within K-nearest neighborhoods, so quality depends on the choice of K and neighbor quality
- Users may substitute one or two of the three pairwise similarities, which changes the enhancement behavior relative to the default all-three configuration

## Evidence

- [intro] SSRE computes the linear representation between cells based on sparse subspace theory, and thus generates a sparse representation of cell-to-cell similarity: "SSRE computes the linear representation between cells based on sparse subspace theory, and thus generates a sparse representation of cell-to-cell similarity [19]."
- [results] SSRE imputes marked missing values using modified Weighted Adamic/Adar over common neighbors: [evidence span withheld — anchor status: not_found]
- [discussion] Users can choose subsets of the three pairwise similarities; all three are default: "users can choose one or two of the three pairwise similarities mentioned in this study to perform gene selection and similarity enhancement procedures, and all three are used by default"
- [abstract] MATLAB and Python implementations of SSRE are available: "The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE."
- [intro] Learned similarities combined with spectral clustering reveal block structure: "The experimental results show that when combined with spectral clustering, the learned similarities by SSRE can reveal the block structure of scRNA-seq data reliably."
