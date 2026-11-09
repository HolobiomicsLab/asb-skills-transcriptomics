---
name: mean-initialization-of-missing-entries
description: Use when a sparse single-cell AS probability matrix contains NA entries
  from dropouts and a complete matrix is needed as the starting point for downstream
  distance-based imputation or clustering.
when_to_use_negative:
- The matrix contains no NA entries — mean-filling would be a no-op and any transformation
  could distort observed AS probabilities
- The goal is to feed raw, unimputed AS profiles to methods that tolerate missing
  data rather than to prepare SCASL's clustering input
- Missingness is so dense that site means are computed from very few observed cells,
  making the mean an unreliable initialization for those sites
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0114
- http://edamontology.org/topic_3170
tools:
- name: SCASL
  role: Pipeline whose imputation stage performs the mean initialization of NA entries
    in the AS probability matrix before iterative weighted KNN
  repo: github.com/xryanglab/SCASL
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-46480-9
    title: Interrogations of single-cell RNA splicing landscapes with SCASL define
      new cell identities with physiological relevance
  source_content_hash: sha256:5cd75c0ef3bcfb4698245064556e31bf4930a96e3498c0bda6ffeea693f3cfe8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCASL
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0114
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mean-initialization-of-missing-entries

## Summary

Initializes NA entries in the SCASL single-cell AS probability matrix by filling each missing value with the mean AS probability of the corresponding site, providing a complete starting point before the iterative weighted KNN imputation refines the estimates.

## When to use

Use when a sparse single-cell AS probability matrix contains NA entries from dropouts and a complete matrix is needed as the starting point for downstream distance-based imputation or clustering.

## When NOT to use

- The matrix contains no NA entries — mean-filling would be a no-op and any transformation could distort observed AS probabilities
- The goal is to feed raw, unimputed AS profiles to methods that tolerate missing data rather than to prepare SCASL's clustering input
- Missingness is so dense that site means are computed from very few observed cells, making the mean an unreliable initialization for those sites

## Inputs

- Filtered single-cell AS probability matrix (rows = AS sites, columns = cells) with NA entries
- NA position information recording which matrix entries were originally missing
- YAML config (configs/srr.yaml) with num_iteration and knn settings for the downstream imputation

## Outputs

- Mean-initialized complete AS probability matrix (seed matrix for iterative weighted KNN imputation)
- Downstream: imputed_matrix input for KNN rounds and the final normalized_matrix.csv (2M x N, AS probabilities plus NA indicator rows)

## How to apply

Load the filtered AS probability matrix produced upstream by SCASL along with its NA position information. For every NA entry, compute the mean of the non-NA AS probabilities within the same site (row) and assign that value to the missing cell. This mean-fill produces a fully populated matrix whose Euclidean distances are meaningful enough for the subsequent iterative weighted KNN step, where the updated matrix refreshes inferred missing values each round until convergence. After initialization and KNN refinement, the imputed matrix is vertically concatenated with a binary NA indicator matrix to form the final 2M x N input for spectral clustering. Parameters such as num_iteration and knn are read from configs/srr.yaml.

## Related tools

- **SCASL** (Pipeline whose imputation stage performs the mean initialization of NA entries in the AS probability matrix before iterative weighted KNN) — github.com/xryanglab/SCASL

## Examples

```
python main.py -y configs/srr.yaml
```

## Evaluation signals

- After mean initialization (and full imputation), the matrix contains no NA entries
- The final concatenated matrix has exactly 2M rows (M AS sites plus M indicator rows) and N cell columns
- The initialized values equal the per-site mean of non-NA AS probabilities for each originally missing entry
- The binary indicator matrix correctly marks exactly the entries that were originally NA in the filtered AS probability matrix
- The completed imputed-plus-indicator matrix, saved as normalized_matrix.csv, serves as valid input for downstream spectral clustering

## Limitations

- Single-cell AS profiles suffer greatly from missing data points due to dropouts during single-cell RNA sequencing analysis and AS event identification, so initialization quality depends on how much of a site's column is observed
- Coverage of genes and AS events can be very unbalanced across different single cells, so per-site means may be skewed toward well-covered cells
- Due to the randomness of the imputation process, there may be some differences between running results

## Evidence

- [intro] Imputation of missing values in the AS probability matrix: "The profiles of AS probabilities are then used for imputations of the missing"
- [discussion] Iterative weighted KNN imputation: "SCASL employs an iterative, weighted KNN process for imputation of the missing values"
- [discussion] Clustering with indicator concatenation: "SCASL performs cell clustering with the full AS probability matrix, concatenated with a NA indicator matrix"
- [readme] Config parameters: "**num_iteration**: the number of iterations during weighted KNN imputation."
- [discussion] Dropout motivation: "single-cell AS profiles suffer greatly from missing data points due to dropouts during single-cell RNA sequencing analysis and the identification of AS events"
