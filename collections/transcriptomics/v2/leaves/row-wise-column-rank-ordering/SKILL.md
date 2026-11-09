---
name: row-wise-column-rank-ordering
description: 'Use when beginning a UniBic/runibic biclustering run on a continuous
  expression matrix: the input matrix must first be converted to an index matrix where
  each row lists column indexes sorted by that row''s increasing values.'
when_to_use_negative:
- The input is already discrete or already a rank/index matrix (BCUnibicD operates
  on discretized data; re-sorting would not add information).
- The goal is downstream bicluster seeding (pairwiseLCS/calculateLCS) or expansion
  (cluster) — those consume the index matrix rather than produce it.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: runibic
  role: provides the unisort function that produces the row-wise sorted column-index
    matrix (Step 1 of UniBic)
  repo: github.com/athril/runibic
- name: UniBic
  role: original algorithm whose Index Matrix Creation step this skill implements;
    index matrix feeds row-pair LCS seeding
  repo: sourceforge.net/projects/unibic
- name: biclust
  role: supplies the BicatYeast sample dataset and Biclust wrapper used to exercise
    unisort in the vignette
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/srep23466
    title: 'UniBic: Sequential row-based biclustering algorithm for analysis of gene
      expression data'
  source_content_hash: sha256:0fe76c0543d7d5e967cce679c64bbbe8256bf123967ee099fdcd6e1faabea247
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - UniBic
  - biclust
  - runibic
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# row-wise-column-rank-ordering

## Summary

Compute the UniBic index matrix: for each row of the input expression matrix, reorder column indexes by increasing entry value. This is Step 1 (Index Matrix Creation) of the UniBic pipeline and the substrate on which its row-pair LCS seeding operates.

## When to use

Use when beginning a UniBic/runibic biclustering run on a continuous expression matrix: the input matrix must first be converted to an index matrix where each row lists column indexes sorted by that row's increasing values.

## When NOT to use

- The input is already discrete or already a rank/index matrix (BCUnibicD operates on discretized data; re-sorting would not add information).
- The goal is downstream bicluster seeding (pairwiseLCS/calculateLCS) or expansion (cluster) — those consume the index matrix rather than produce it.

## Inputs

- numeric gene expression matrix (e.g. BicatYeast from the biclust package, runibic vignette test matrix, or a SummarizedExperiment assay matrix)
- optionally a preprocessed/discretized matrix (runibic::discretize output for BCUnibicD)

## Outputs

- index matrix Y: integer matrix of the same dimensions as the input, each row containing column indexes reordered by increasing row values
- CSV export of the index matrix (unisort_index_matrix.csv)

## How to apply

Apply the runibic::unisort function to the preprocessed input matrix A = {aij} to derive the index matrix Y = {yij}; in the R implementation, unisort 'returns matrix of indexes based on the increasing order in each row'. Verify that ties are resolved so the smaller column index receives the higher rank priority. The resulting index matrix is the input to Step 2 (Index Matrix Partition) and to the pairwise longest common subsequence (LCS) computations of Step 3, since UniBic discovers trend-preserving biclusters by finding a LCS between two rows of this index matrix. Write the matrix out (e.g. unisort_index_matrix.csv) so later seeding/expansion steps can be reproduced.

## Related tools

- **runibic** (provides the unisort function that produces the row-wise sorted column-index matrix (Step 1 of UniBic)) — github.com/athril/runibic
- **UniBic** (original algorithm whose Index Matrix Creation step this skill implements; index matrix feeds row-pair LCS seeding) — sourceforge.net/projects/unibic
- **biclust** (supplies the BicatYeast sample dataset and Biclust wrapper used to exercise unisort in the vignette)

## Examples

```
library(runibic); library(biclust); data(BicatYeast); Y <- unisort(BicatYeast)
```

## Evaluation signals

- Each row of the output must contain exactly the column indexes 1..ncol of the input matrix (a permutation), with dimensions preserved.
- For each row, reindexing the input values by the output indexes yields a non-decreasing sequence (row-wise increasing order).
- Ties are broken so the smaller column index receives the higher rank priority; check adjacent equal values in the input row.
- Output matches runibic::unisort on the vignette's worked example (BicatYeast) to within a tolerance-free integer comparison.

## Limitations

- unisort only encodes per-row ordering (trend) information; magnitudes are discarded, so it is unsuited as a general-purpose normalization.
- Identification of general trend-preserving biclusters remains a challenging problem; the index matrix alone does not guarantee bicluster discovery without the LCS and extension steps (error rate r ≤ 0.3, significance p < 0.05).

## Evidence

- [methods] Let Y = {yij} be the index matrix derived from input matrix A = {aij} by setting:: "Let Y = {yij} be the index matrix derived from input matrix A = {aij} by setting:"
- [readme] unisort - returns matrix of indexes based on the increasing order in each row: "unisort - returns matrix of indexes based on the increasing order in each row"
- [abstract] The basic idea is to apply the longest common subsequence (LCS) framework to selected pairs of rows in an index matrix derived from an input data matrix: "The basic idea is to apply the longest common subsequence (LCS) framework to selected pairs of rows in an index matrix derived from an input data matrix"
