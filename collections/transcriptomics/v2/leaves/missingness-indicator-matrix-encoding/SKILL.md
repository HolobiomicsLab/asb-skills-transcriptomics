---
name: missingness-indicator-matrix-encoding
description: Use when a sparse single-cell AS probability (PSI) matrix contains NA
  entries from dropouts and the downstream step (e.g. spectral clustering in SCASL)
  requires a complete feature matrix that also preserves original missingness information.
when_to_use_negative:
- The AS probability matrix is already complete (no NA entries) — the imputation loop
  and indicator rows add no information.
- Downstream clustering operates directly on the raw junction-read count matrix (junc_matrix.csv)
  rather than the AS probability matrix.
- The goal is differential splicing analysis with known cell labels (e.g. BRIE18 or
  Expedition19 style), for which unsupervised imputation-then-clustering is not the
  intended workflow.
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0114
- http://edamontology.org/topic_3170
tools:
- name: SCASL
  role: Provides the iterative weighted KNN imputation and NA-indicator concatenation
    that produce the final matrix input for spectral clustering
  repo: https://github.com/xryanglab/SCASL
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-46480-9
    title: Interrogations of single-cell RNA splicing landscapes with SCASL define
      new cell identities with physiological relevance
  source_content_hash: sha256:94ff5ba5f49ba3f627da9313eae820c4e4570c4ceaaadc472b47c97a5df4da2a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xryanglab/SCASL
  tools:
  - SCASL
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0114
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# missingness-indicator-matrix-encoding

## Summary

Impute missing AS probabilities in a single-cell alternative splicing matrix with an iterative distance-weighted KNN procedure, then vertically concatenate the imputed matrix with a binary NA-indicator matrix so that downstream clustering sees both imputed values and original missingness patterns.

## When to use

Use when a sparse single-cell AS probability (PSI) matrix contains NA entries from dropouts and the downstream step (e.g. spectral clustering in SCASL) requires a complete feature matrix that also preserves original missingness information.

## When NOT to use

- The AS probability matrix is already complete (no NA entries) — the imputation loop and indicator rows add no information.
- Downstream clustering operates directly on the raw junction-read count matrix (junc_matrix.csv) rather than the AS probability matrix.
- The goal is differential splicing analysis with known cell labels (e.g. BRIE18 or Expedition19 style), for which unsupervised imputation-then-clustering is not the intended workflow.

## Inputs

- Filtered AS probability matrix (rows = AS sites, columns = cells) with NA entries
- NA position information recording which matrix entries were originally missing
- YAML config (configs/srr.yaml) providing num_iteration and knn parameters

## Outputs

- Imputed, NA-free AS probability matrix
- Binary missing-value (NA) indicator matrix
- Concatenated 2M x N final data matrix saved as normalized_matrix.csv (rows = sites plus indicators, columns = cells)

## How to apply

Load the filtered AS probability matrix and its NA position information, reading num_iteration and knn from configs/srr.yaml. First fill each NA entry with the mean AS probability of the corresponding site as an initial imputation. Then run the iterative weighted KNN imputation: for t iterations (e.g. 3), find the k nearest cells (e.g. k=10) per cell using Euclidean distances on the current matrix and update each missing entry as the weighted average of neighbors' values with normalized exponential weights on the distances; the updated matrix refreshes the inferred values each round until convergence. Build a binary missing-value indicator matrix recording which entries were originally NA, vertically concatenate it with the imputed matrix to form a 2M x N data matrix (M AS sites plus M indicator rows, N cells), and save as normalized_matrix.csv with rows as sites (sites plus indicators) and columns as cells. The rationale is that SCASL clusters cells with the full AS probability matrix concatenated with an NA indicator matrix, so the clustering input encodes both imputed splicing levels and original dropout patterns.

## Related tools

- **SCASL** (Provides the iterative weighted KNN imputation and NA-indicator concatenation that produce the final matrix input for spectral clustering) — https://github.com/xryanglab/SCASL

## Examples

```
python main.py -y configs/srr.yaml  # set num_iteration and knn in configs/srr.yaml before running
```

## Evaluation signals

- Output matrix (normalized_matrix.csv) contains no NA entries after imputation.
- Output matrix has exactly 2M rows (M AS sites plus M indicator rows) and N cell columns.
- Indicator rows correctly mark exactly the entries that were originally NA in the upstream filtered matrix.
- Imputation uses the configured number of iterations and neighbors from configs/srr.yaml (e.g. num_iteration=3, knn=10) and iterates until convergence, refreshing missing values from the updated matrix each round.
- The concatenated matrix reproduces SCASL's reported behavior of clustering cells with the full AS probability matrix concatenated with a NA indicator matrix.

## Limitations

- Single-cell AS profiles suffer greatly from missing data points due to dropouts, so imputation quality depends on coverage; coverage of genes and AS events can be very unbalanced across different single cells.
- Largely polarized AS profiles make many canonical clustering methods unsuitable, which is why SCASL concatenates the NA indicator matrix rather than relying on the imputed values alone.
- Due to the randomness of the interpolation, dimensionality reduction and unsupervised clustering processes, there may be some differences between running results.

## Evidence

- [discussion] SCASL employs an iterative, weighted KNN process for imputation of the missing values: "SCASL employs an iterative, weighted KNN process for imputation of the missing values"
- [discussion] Clustering input concatenation: "SCASL performs cell clustering with the full AS probability matrix, concatenated with a NA indicator matrix"
- [discussion] Missing data motivation: "single-cell AS profiles suffer greatly from missing data points due to dropouts during single-cell RNA sequencing analysis and the identification of AS events"
- [readme] Config parameters: "num_iteration**: the number of iterations during weighted KNN imputation. - **knn**: the number of adjacent cells used in weighted KNN imputation."
- [readme] Output file description: "normalized_matrix**: The final AS probability matrix, the rows correspond to sites, while the columns represent cells."
