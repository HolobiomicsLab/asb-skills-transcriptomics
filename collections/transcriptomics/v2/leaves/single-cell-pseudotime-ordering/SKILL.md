---
name: single-cell-pseudotime-ordering
description: Use when single cells have a sparse binary/eeSNV feature matrix with
  subgroup labels and the goal is a continuous genotype-based ordering (primary-to-metastasis
  trajectory) plus detection of subgroup-specific branches.
when_to_use_negative:
- Input is already a gene-expression feature table rather than eeSNV/SNV-derived features
  — the article reports eeSNVs, not transcript abundance, achieve better subpopulation
  separation
- Cells lack subgroup labels or a known primary/root population, so the tree cannot
  be rooted for directional pseudo-time
- Goal is clustering alone without a continuous trajectory; a subpopulation classifier
  on eeSNVs is the simpler choice
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3173
tools:
- name: SSrGE
  role: Fits sparse LASSO models linking SNVs to gene expression and supplies the
    eeSNV-reduced matrix (X_r) that the MST is built on, plus rank_features_for_a_subgroup
    for branch annotation
  repo: https://github.com/lanagarmire/SSrGE.git
- name: Scipy
  role: Pearson correlation distances between cells and minimum spanning tree construction
    via scipy.sparse.csgraph.minimum_spanning_tree
- name: Numpy
  role: Matrix operations for pairwise correlation computation on eeSNV features
- name: scikit-learn
  role: Backend library on which SSrGE's LASSO/ElasticNet models are built (version
    = 0.18 per README)
- name: Python
  role: Language (2.7.X) in which the SSrGE package and the MST ordering workflow
    are implemented
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-07170-5
    title: Using single nucleotide variations in single-cell RNA-seq to identify subpopulations
      and genotype-phenotype linkage
  source_content_hash: sha256:8b1d953fbc37e935814d43ae61ac47ae77289df3fa7464e8cd399a1327cc1325
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
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# single-cell-pseudotime-ordering

## Summary

Monocle-style minimum spanning tree (MST) pseudo-time reconstruction computed on eeSNV features (the SSrGE-transformed matrix) to order single cells continuously, e.g., from primary (PDX-pRCC) to metastasized (Pt-mRCC, PDX-mRCC) tumors in the Kim dataset, while exposing intra-group ramification heterogeneity.

## When to use

Use when single cells have a sparse binary/eeSNV feature matrix with subgroup labels and the goal is a continuous genotype-based ordering (primary-to-metastasis trajectory) plus detection of subgroup-specific branches.

## When NOT to use

- Input is already a gene-expression feature table rather than eeSNV/SNV-derived features — the article reports eeSNVs, not transcript abundance, achieve better subpopulation separation
- Cells lack subgroup labels or a known primary/root population, so the tree cannot be rooted for directional pseudo-time
- Goal is clustering alone without a continuous trajectory; a subpopulation classifier on eeSNVs is the simpler choice

## Inputs

- SSrGE-transformed eeSNV feature matrix X_r (n_cells x n_eeSNVs) from ssrge.transform(X) after ssrge.fit(X, Y)
- Per-cell subgroup labels distinguishing PDX-pRCC (primary), Pt-mRCC, and PDX-mRCC cells

## Outputs

- Minimum spanning tree over cells with a continuous primary-to-metastasis pseudo-time ordering
- mst_ordering.csv listing cells in pseudo-time order
- Figure of the MST with cells colored by subgroup (reproducing Fig. 6a)
- Subgroup-specific eeSNVs and impacted genes ranked by ssrge.rank_features_for_a_subgroup for each branch

## How to apply

Load the reduced eeSNV matrix X_r returned by ssrge.transform(X) after ssrge.fit(X, Y), together with subgroup labels (PDX-pRCC, Pt-mRCC, PDX-mRCC). Compute pairwise Pearson correlation between cells on the eeSNV features with NumPy/SciPy and convert it to a correlation distance matrix. Build the minimum spanning tree over cells with scipy.sparse.csgraph.minimum_spanning_tree, mirroring the Monocle-style ordering, then root the tree at the primary (PDX-pRCC) cells and read the pseudo-time ordering along tree paths. Annotate each ramification with ssrge.rank_features_for_a_subgroup to find eeSNVs and impacted genes significantly enriched in that branch; draw the MST colored by subgroup to reproduce Fig. 6a and export mst_ordering.csv.

## Related tools

- **SSrGE** (Fits sparse LASSO models linking SNVs to gene expression and supplies the eeSNV-reduced matrix (X_r) that the MST is built on, plus rank_features_for_a_subgroup for branch annotation) — https://github.com/lanagarmire/SSrGE.git
- **Scipy** (Pearson correlation distances between cells and minimum spanning tree construction via scipy.sparse.csgraph.minimum_spanning_tree)
- **Numpy** (Matrix operations for pairwise correlation computation on eeSNV features)
- **scikit-learn** (Backend library on which SSrGE's LASSO/ElasticNet models are built (version = 0.18 per README))
- **Python** (Language (2.7.X) in which the SSrGE package and the MST ordering workflow are implemented)

## Examples

```
from garmire_SSrGE.ssrge import SSrGE; ssrge = SSrGE(); ssrge.fit(X, Y); X_r = ssrge.transform(X); D = 1 - np.corrcoef(X_r); from scipy.sparse.csgraph import minimum_spanning_tree; T = minimum_spanning_tree(D)
```

## Evaluation signals

- The MST places primary (PDX-pRCC) cells at the root, with Pt-mRCC and PDX-mRCC cells ramifying downstream into separable branches
- The tree reproduces a continuous pseudo-time ordering of cells from primary to metastasized tumors along tree paths
- eeSNVs reported by rank_features_for_a_subgroup for a subgroup are significantly enriched in the corresponding tree branch
- The ramification pattern highlights heterogeneity within each subgroup (intra-group branches), consistent with Fig. 6a
- mst_ordering.csv contains all cells exactly once with root-side cells matching the primary subgroup

## Limitations

- SSrGE may miss some driver mutations due to the incomplete DNA coverage of scRNA-seq reads, which propagates into the eeSNV feature space used for the tree
- LASSO may arbitrarily select one of highly correlated SNV features corresponding to gene expression, affecting which eeSNVs define branch structure
- ssrge features may be missed when using nb_ranked_features if it exceeds the number of eeSNVs obtained with the specified alpha

## Evidence

- [discussion] Monocle-style MST ordering on eeSNV features: "The process of selecting eeSNVs linked to GE allows us to identify representative genotype markers for cell subpopulations."
- [discussion] Correlation-based MST built on eeSNV features reconstructs primary-to-metastasis trajectory: "One can also easily rank the eeSNVs and the genes harboring them, for the purpose of identifying robust eeSNVs as genetic markers for a variety of cancers."
- [methods] Subgroup-specific eeSNV/gene annotation API: "subgroup = ssrge.rank_features_for_a_subgroup([0, 1, 2, 3, 4, 5])"
- [readme] Transformed eeSNV matrix used as reduced feature space: "X_r = ssrge.transform(X)"
- [readme] Significance criterion for subgroup-specific eeSNVs: "a given eeSNV is specific to a subgroup if it is signficantly more present amongst the cells from the given subgroup"
