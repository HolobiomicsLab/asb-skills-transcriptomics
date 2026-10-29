---
name: similarity-matrix-benchmark-comparison
description: Use when you have learned decomposition patterns (e.g., ACTIONet archetypes
  or cNMF usage matrices) and need to quantitatively compare them against labeled
  bulk reference profiles via a correlation similarity matrix.
when_to_use_negative:
- There is no independently labeled reference dataset (e.g., bulk cell-sorted profiles)
  available to correlate against — the comparison degrades to a circular check
- The two decomposition methods were run on differently preprocessed matrices — correlations
  then reflect preprocessing, not method performance
- The goal is selecting a single optimal decomposition resolution rather than benchmarking
  against known profiles; the article argues multiple meaningful resolutions exist,
  so pruning to one number of patterns is inappropriate
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ACTIONet (Python package)
  role: Produces the multiresolution archetypal decomposition being benchmarked (filter_adata,
    reduce_kernel, run_ACTIONet)
  repo: https://github.com/shmohammadi86/ACTIONet
- name: cNMF
  role: Produces the competing consensus NMF profiles with pattern count fixed at
    k=12
- name: scanpy
  role: Library-size normalization (normalize_total) and log1p transformation of the
    shared input matrix
- name: Leiden
  role: Graph-clustering over the pattern graph to prune redundant archetypes, keeping
    the lowest-resolution pattern per equivalent class
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-18416-6
    title: A multiresolution framework to characterize single-cell state landscapes
  source_content_hash: sha256:5f5dcd54c165422fbdbff969e887bd9bb7dbf2d11e34a263061289bc99d5a8fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shmohammadi86/ACTIONet
  tools:
  - ACTIONet
  - ACTIONet (Python package)
  - Leiden
  - cNMF
  - scanpy
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# similarity-matrix-benchmark-comparison

## Summary

Benchmark decomposed single-cell pattern profiles against an independent labeled reference by building pattern-vs-reference similarity matrices. In the source workflow, ACTIONet multiresolution archetype profiles (14 dominant patterns) and cNMF consensus profiles (k=12) from 14,311-cell PBMC scRNA-seq data are correlated against cell-sorted bulk PBMC RNA-seq reference profiles to judge which decomposition better recapitulates known cell states.

## When to use

Use when you have learned decomposition patterns (e.g., ACTIONet archetypes or cNMF usage matrices) and need to quantitatively compare them against labeled bulk reference profiles via a correlation similarity matrix.

## When NOT to use

- There is no independently labeled reference dataset (e.g., bulk cell-sorted profiles) available to correlate against — the comparison degrades to a circular check
- The two decomposition methods were run on differently preprocessed matrices — correlations then reflect preprocessing, not method performance
- The goal is selecting a single optimal decomposition resolution rather than benchmarking against known profiles; the article argues multiple meaningful resolutions exist, so pruning to one number of patterns is inappropriate

## Inputs

- Preprocessed scRNA-seq gene-by-cell matrix (10x h5 input, e.g., pbmc_10k_v3.h5 with unique variable names)
- ACTIONet multiresolution archetype/pattern-profile matrix (14 dominant patterns, nonredundant after Leiden pruning)
- cNMF consensus pattern-profile matrix (k=12, fixed per suggested protocol)
- Labeled cell-sorted bulk PBMC RNA-seq reference profiles (from ref. 27)

## Outputs

- Pattern-vs-bulk similarity (correlation) matrix for ACTIONet profiles
- Pattern-vs-bulk similarity (correlation) matrix for cNMF profiles
- Assembled comparison table saved as CSV (fig2c_similarity_matrix.csv)

## How to apply

Preprocess the scRNA-seq matrix identically for both methods being compared: filter with an.pp.filter_adata (min_cells_per_feature=0.01, min_features_per_cell=1000), then sc.pp.normalize_total and sc.pp.log1p, then an.pp.reduce_kernel (randomized SVD low-rank approximation). Run ACTIONet (an.run_ACTIONet) to obtain the multiresolution archetypal decomposition, and prune redundant archetypes with Leiden graph-clustering over the pattern graph, keeping the lowest-resolution pattern per equivalent class; run the competing method (here cNMF with k fixed at 12 per its suggested protocol) on the same preprocessed matrix. Load the labeled bulk reference profiles (the cell-sorted bulk PBMC data set from ref. 27) and compute the correlation of both the ACTIONet and cNMF pattern-profile matrices against these reference profiles. Assemble the two pattern-vs-bulk similarity matrices and save the comparison table (e.g., fig2c_similarity_matrix.csv). Judge success by verifying that the decomposed profiles known to be more biologically granular (heterogeneous myeloid cells, CD4 memory T-cell subtypes) show higher correlation with the matching bulk labels than the comparison method's profiles, reproducing the qualitative pattern of Fig. 2c.

## Related tools

- **ACTIONet (Python package)** (Produces the multiresolution archetypal decomposition being benchmarked (filter_adata, reduce_kernel, run_ACTIONet)) — https://github.com/shmohammadi86/ACTIONet
- **cNMF** (Produces the competing consensus NMF profiles with pattern count fixed at k=12)
- **scanpy** (Library-size normalization (normalize_total) and log1p transformation of the shared input matrix)
- **Leiden** (Graph-clustering over the pattern graph to prune redundant archetypes, keeping the lowest-resolution pattern per equivalent class)

## Examples

```
an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000); sc.pp.normalize_total(adata); sc.pp.log1p(adata); an.pp.reduce_kernel(adata); an.run_ACTIONet(adata)
```

## Evaluation signals

- The similarity matrix rows correspond to nonredundant learned patterns and columns to labeled bulk reference cell types, with valid correlation values in [-1, 1]
- ACTIONet profiles show higher correlations for heterogeneous myeloid cells and high-resolution T-cell subtypes (e.g., CD4 memory cells) than cNMF profiles, reproducing Fig. 2c's qualitative pattern
- cNMF on the same preprocessed matrix recovers fewer cell states (12 patterns at k=12) and lower consistency with known bulk profiles, i.e., lower max correlation per cell type
- The comparison table is exported as fig2c_similarity_matrix.csv and the 14,311-cell PBMC dataset and bulk reference from ref. 27 are traceable

## Limitations

- Matrix decomposition methods require a priori selection of the number of underlying dominant patterns, which is not trivial to determine (cNMF needed k=12 fixed by its suggested protocol)
- Datasets are cited only by reference numbers without accessions or download links; no tool/software versions are stated for ACTIONet, R, Python, UMAP, or Leiden
- The comparison is qualitative/semiquantitative — correlations establish relative recapitulation of expected profiles rather than a statistical significance test

## Evidence

- [discussion] Bulk reference provenance: "The cell-sorted bulk PBMC data set for validation was obtained from ref. 27"
- [discussion] Pattern pruning rationale: "We then applied the Leiden graph-clustering algorithms over this pattern graph to define equivalent patterns classes"
- [methods] Preprocessing parameters: "an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000)"
- [methods] Dimensionality reduction step: "an.pp.reduce_kernel(adata)"
- [intro] A priori pattern count caveat: "A priori selection of the number of underlying dominant patterns to be learned is required but not trivial to determine, nor is the certainty of robustness and reproducibility of the patterns."
- [readme] Benchmark framing: [evidence span withheld — anchor status: not_found]
