---
name: euclidean-distance-neighbor-weighting
description: Use when a cell-by-feature matrix has sparse NA entries (e.g., single-cell
  AS probability dropouts) and missing values must be imputed before downstream clustering
  such as spectral clustering.
when_to_use_negative:
- The matrix has no NA entries — imputation would only distort observed AS probabilities.
- The downstream analysis requires raw unimputed AS probabilities (e.g., differential
  PSI testing with BRIE or Expedition rather than SCASL clustering).
- Features are cells rather than sites, i.e., the neighbor search orientation differs
  from SCASL's cell-to-cell Euclidean KNN scheme.
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3170
tools:
- name: SCASL
  role: Implements the iterative weighted KNN imputation of missing values in the
    AS probability matrix prior to spectral clustering
  repo: github.com/xryanglab/SCASL
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-46480-9
    title: Interrogations of single-cell RNA splicing landscapes with SCASL define
      new cell identities with physiological relevance
  source_content_hash: sha256:c1005dfc880a53ff54d0ee2e88883d1d02598a8b956a8f2ebec49c3843a6de1d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCASL
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# euclidean-distance-neighbor-weighting

## Summary

Iterative distance-weighted KNN imputation of missing values in the single-cell AS probability matrix, as implemented in SCASL: NA entries are first filled with site means, then refreshed over iterations using exponentially distance-weighted averages of the k nearest cells.

## When to use

Use when a cell-by-feature matrix has sparse NA entries (e.g., single-cell AS probability dropouts) and missing values must be imputed before downstream clustering such as spectral clustering.

## When NOT to use

- The matrix has no NA entries — imputation would only distort observed AS probabilities.
- The downstream analysis requires raw unimputed AS probabilities (e.g., differential PSI testing with BRIE or Expedition rather than SCASL clustering).
- Features are cells rather than sites, i.e., the neighbor search orientation differs from SCASL's cell-to-cell Euclidean KNN scheme.

## Inputs

- filtered AS probability matrix with NA entries (rows = AS sites, columns = cells)
- NA position information from upstream SCASL steps
- configs/srr.yaml values for num_iteration and knn

## Outputs

- fully imputed AS probability matrix with no NA entries (normalized_matrix.csv)
- binary missing-value indicator matrix of originally NA entries
- final 2M x N data matrix (imputed sites + indicator rows) feeding spectral clustering

## How to apply

Load the filtered AS probability matrix and its NA position information, reading num_iteration and knn from configs/srr.yaml. First fill each NA entry with the mean AS probability of the corresponding site as the initial guess. Then run the iterative imputation (num_iteration = 3 by default): each round, compute Euclidean distances between cells on the current matrix, find the k = 10 nearest neighbors per cell, and update each missing entry as the weighted average of neighbors' values using normalized exponential weights on the distances. Because the matrix is refreshed each round, the neighbor set and weights improve as the imputation converges. After convergence, build a binary missing-value indicator matrix for the originally NA entries and vertically concatenate it with the imputed matrix (2M x N: M AS site rows plus M indicator rows, N cell columns) so that SCASL's spectral clustering sees both the imputed values and the missingness pattern.

## Related tools

- **SCASL** (Implements the iterative weighted KNN imputation of missing values in the AS probability matrix prior to spectral clustering) — github.com/xryanglab/SCASL

## Examples

```
python main.py -y configs/srr.yaml  # with num_iteration: 3 and knn: 10 in configs/srr.yaml
```

## Evaluation signals

- Output matrix contains no NA entries after imputation.
- Output matrix has exactly 2M rows (M AS sites + M indicator rows) and N cell columns.
- num_iteration and knn match the values in configs/srr.yaml (default 3 iterations, 10 neighbors).
- Each imputed entry equals the normalized-exponential-weighted average of the k nearest cells' values under Euclidean distance on the current matrix.
- The imputed matrix concatenated with the NA indicator matrix reproduces SCASL's reported behavior as input for downstream cell clustering.

## Limitations

- Single-cell AS profiles suffer greatly from missing data points due to dropouts during scRNA-seq analysis and AS event identification, so imputation quality depends heavily on the sparsity level of the input matrix.
- Coverage of genes and AS events can be very unbalanced across different single cells, which can bias neighbor-distance estimates.
- Due to randomness of the interpolation, dimensionality reduction and unsupervised clustering processes, results may vary between runs (per README).

## Evidence

- [discussion] SCASL employs an iterative, weighted KNN process for imputation of the missing values: "SCASL employs an iterative, weighted KNN process for imputation of the missing values"
- [discussion] Clustering with concatenated NA indicator matrix: "SCASL performs cell clustering with the full AS probability matrix, concatenated with a NA indicator matrix"
- [discussion] Missing data due to dropouts: "single-cell AS profiles suffer greatly from missing data points due to dropouts during single-cell RNA sequencing analysis and the identification of AS events"
- [readme] Config parameter definitions: "**num_iteration**: the number of iterations during weighted KNN imputation.
- **knn**: the number of adjacent cells used in weighted KNN imputation."
- [readme] normalized_matrix output definition: "**normalized_matrix**: The final AS probability matrix, the rows correspond to sites, while the columns represent cells."
