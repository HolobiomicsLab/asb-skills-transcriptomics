---
name: pearson-correlation-distance-computation
description: 'Use when you need a continuous cell ordering (pseudo-time) from a binary/transformed
  eeSNV matrix: Pearson correlation distance between cells feeds a minimum spanning
  tree that must recapitulate a primary-to-metastasis trajectory and expose subgroup
  ramifications.'
when_to_use_negative:
- Input is a raw SNV or gene expression matrix that has not been reduced by SSrGE
  — the correlation distance should be computed on LASSO-selected eeSNV features,
  not noisy GE or unfiltered SNVs
- Cells lack the primary (PDX-pRCC) subgroup needed to root the MST at the primary-to-metastasis
  origin
- The goal is subpopulation classification accuracy rather than trajectory/pseudo-time
  reconstruction
edam_operation: http://edamontology.org/operation_0531
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SSrGE
  role: Produces the eeSNV feature matrix (X_r via fit + transform with LASSO penalization)
    on which the Pearson correlation between cells is computed
  repo: https://github.com/lanagarmire/SSrGE.git
- name: Numpy
  role: Computes the pairwise Pearson correlation matrix between cells on eeSNV features
- name: Scipy
  role: Converts the correlation to a distance matrix and builds the minimum spanning
    tree via scipy.sparse.csgraph.minimum_spanning_tree
- name: scikit-learn
  role: Backend library (version 0.18 in the original study) underlying the SSrGE
    LASSO models that generate the eeSNV features
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:483b688ee4ae57fb515c3995c93a7ccbaf5a72a0ad46681a714390bf0e79f9a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lanagarmire/SSrGE.git
  tools:
  - Numpy
  - Python
  - SSrGE
  - Scipy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0531
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# pearson-correlation-distance-computation

## Summary

Computes a pairwise Pearson correlation matrix between single cells on their eeSNV feature vectors and converts it to a correlation distance (e.g., 1 − r), serving as the cell-to-cell dissimilarity input for Monocle-style minimum spanning tree pseudo-time reconstruction of the Kim primary-to-metastasis (PDX-pRCC → Pt-mRCC → PDX-mRCC) trajectory.

## When to use

Use when you need a continuous cell ordering (pseudo-time) from a binary/transformed eeSNV matrix: Pearson correlation distance between cells feeds a minimum spanning tree that must recapitulate a primary-to-metastasis trajectory and expose subgroup ramifications.

## When NOT to use

- Input is a raw SNV or gene expression matrix that has not been reduced by SSrGE — the correlation distance should be computed on LASSO-selected eeSNV features, not noisy GE or unfiltered SNVs
- Cells lack the primary (PDX-pRCC) subgroup needed to root the MST at the primary-to-metastasis origin
- The goal is subpopulation classification accuracy rather than trajectory/pseudo-time reconstruction

## Inputs

- SSrGE-transformed eeSNV feature matrix X_r (n_cells x n_eeSNVs) from ssrge.transform(X) after ssrge.fit(X, Y)
- Subgroup labels per cell distinguishing PDX-pRCC, Pt-mRCC, and PDX-mRCC

## Outputs

- Pairwise Pearson correlation matrix between cells (n_cells x n_cells)
- Correlation distance matrix (e.g., 1 − r) with zero diagonal
- Rooted minimum spanning tree over cells yielding a continuous primary-to-metastasis pseudo-time ordering (written to mst_ordering.csv)

## How to apply

Take the reduced SSrGE matrix X_r returned by ssrge.transform(X) after ssrge.fit(X, Y) — rows are cells (Kim dataset PDX-pRCC, Pt-mRCC, PDX-mRCC), columns are eeSNVs selected by LASSO penalization. Compute the pairwise Pearson correlation between cells over the eeSNV features using NumPy (np.corrcoef) or SciPy, then convert to a correlation distance matrix (e.g., 1 − correlation), ensuring a symmetric matrix with zero diagonal. Build the minimum spanning tree with scipy.sparse.csgraph.minimum_spanning_tree (Monocle-style ordering), root it at the primary (PDX-pRCC) cells, and traverse to derive a continuous primary-to-metastasis ordering. The rationale is that eeSNV features outperform gene expression for separating subpopulations (even a few features suffice — '8 eeSNV features have decent separations for Kim dataset'), so a correlation distance on eeSNVs gives a low-noise genotype-based geometry for the tree.

## Related tools

- **SSrGE** (Produces the eeSNV feature matrix (X_r via fit + transform with LASSO penalization) on which the Pearson correlation between cells is computed) — https://github.com/lanagarmire/SSrGE.git
- **Numpy** (Computes the pairwise Pearson correlation matrix between cells on eeSNV features)
- **Scipy** (Converts the correlation to a distance matrix and builds the minimum spanning tree via scipy.sparse.csgraph.minimum_spanning_tree)
- **scikit-learn** (Backend library (version 0.18 in the original study) underlying the SSrGE LASSO models that generate the eeSNV features)

## Examples

```
import numpy as np; from scipy.sparse.csgraph import minimum_spanning_tree; D = 1 - np.corrcoef(X_r); np.fill_diagonal(D, 0); T = minimum_spanning_tree(D)
```

## Evaluation signals

- The MST must place primary (PDX-pRCC) cells at the root with Pt-mRCC and PDX-mRCC cells ramifying downstream into separable branches
- The correlation distance matrix must be symmetric with values in [0, 2] and a zero diagonal
- Subgroup-specific eeSNVs reported by ssrge.rank_features_for_a_subgroup must be significantly enriched in their corresponding tree branch
- The reduced matrix X_r must have shape (n_cells, n_eeSNVs) consistent with the LASSO-selected eeSNV count (e.g., nb_ranked_features or alpha-driven selection)

## Limitations

- SSrGE may miss some driver mutations due to incomplete DNA coverage of scRNA-seq reads, so eeSNV-based distances may omit relevant genetic variation
- LASSO may arbitrarily select one of highly correlated SNV features corresponding to gene expression, affecting which eeSNVs define the correlation geometry
- Explanations for eeSNV accuracy advantage over gene expression remain speculative (not experimentally verified)

## Evidence

- [readme] This procedure can be used as  a stand-alone procedure to reduce any SNV matrix (raw:single-cell, col: SNV (binary)), using a gene expression matrix: "This procedure can be used as  a stand-alone procedure to reduce any SNV matrix (raw:single-cell, col: SNV (binary)), using a gene expression matrix"
- [discussion] LASSO penalization works as a feature selection method and minimizes the spurious SNVs: "Secondly, LASSO penalization works as a feature selection method and minimizes the spurious SNVs (false positive) from the filtered set of eeSNVs."
- [discussion] 8 eeSNV features have decent separations for Kim dataset: "(e.g., 8 eeSNV features have decent separations for Kim dataset)"
- [results] the tree must place primary (PDX-pRCC) cells at the root with Pt-mRCC and PDX-mRCC cells ramifying downstream: [evidence span withheld — anchor status: not_found]
