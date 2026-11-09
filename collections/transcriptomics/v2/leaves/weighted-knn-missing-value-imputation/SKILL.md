---
name: weighted-knn-missing-value-imputation
description: Use when a single-cell AS probability matrix (rows = AS sites, columns
  = cells) contains NA entries from dropouts and the values must be recovered before
  spectral clustering, or when imputation fidelity under forced dropout must be quantified.
when_to_use_negative:
- Input is already a dense, complete gene-expression or AS feature table with no NA
  entries to impute
- The goal is differential splicing quantification with known cell labels (e.g. BRIE,
  rMATS-style analyses) rather than preparing a matrix for unsupervised clustering
- The matrix is an expression count matrix — canonical clustering pipelines on gene
  expression data do not use this AS-specific imputation scheme
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0114
- http://edamontology.org/topic_3170
tools:
- name: SCASL
  role: Implements the iterative weighted KNN imputation (num_iteration, knn parameters)
    and NA-indicator concatenation within its pipeline
  repo: github.com/xryanglab/SCASL
- name: Leafcutter
  role: 'Upstream: identifies AS events from junction reads and builds the junction
    matrix whose AS probability matrix is imputed'
  repo: github.com/davidaknowles/leafcutter
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_008
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-46480-9
    title: Interrogations of single-cell RNA splicing landscapes with SCASL define
      new cell identities with physiological relevance
  source_content_hash: sha256:25b36f8ac0fd2ad9d3a29e12610aeceec199b34e898438477e1748f8ad04e2bc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Leafcutter
  - Python
  - SCASL
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0114
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# weighted-knn-missing-value-imputation

## Summary

SCASL's iterative weighted KNN imputation fills missing AS probability values in sparse single-cell alternative-splicing matrices, then concatenates a binary NA-indicator matrix so clustering can leverage missingness structure itself.

## When to use

Use when a single-cell AS probability matrix (rows = AS sites, columns = cells) contains NA entries from dropouts and the values must be recovered before spectral clustering, or when imputation fidelity under forced dropout must be quantified.

## When NOT to use

- Input is already a dense, complete gene-expression or AS feature table with no NA entries to impute
- The goal is differential splicing quantification with known cell labels (e.g. BRIE, rMATS-style analyses) rather than preparing a matrix for unsupervised clustering
- The matrix is an expression count matrix — canonical clustering pipelines on gene expression data do not use this AS-specific imputation scheme

## Inputs

- Filtered AS probability matrix (rows = AS sites, columns = cells) with NA entries, e.g. SCASL's normalized_matrix
- NA position information / indicator structure produced upstream by SCASL
- Config values num_iteration and knn from configs/srr.yaml
- For dropout testing: TNBC-2 demo intermediate files in data/junction

## Outputs

- normalized_matrix.csv: imputed matrix with no NA entries, 2M rows (M AS sites + M NA-indicator rows) and N cell columns, serving as input to spectral clustering
- Recovery-rate vs. dropout-proportion plot and t-test comparing imputed vs. true values (dropout fidelity assessment)

## How to apply

Load the filtered AS probability matrix and read num_iteration and knn from configs/srr.yaml. First fill each NA entry with the mean AS probability of the corresponding site as initialization; then for t = 3 iterations find the k = 10 nearest cells per cell by Euclidean distance on the current matrix and update each missing entry as the distance-weighted average of neighbors' values using normalized exponential weights, refreshing inferred values each round until convergence. Build a binary indicator matrix recording originally NA positions and vertically concatenate it with the imputed matrix to form the final 2M x N matrix (M sites + M indicators, N cells), saved as normalized_matrix.csv. To validate fidelity, randomly mask 10%, 30%, or 90% of non-NA values, re-run imputation, and compare imputed vs. true values at masked positions (t-test plus recovery-rate vs. dropout plot); the paper reports most values are still correctly recovered at 90% dropout (Fig. S1A).

## Related tools

- **SCASL** (Implements the iterative weighted KNN imputation (num_iteration, knn parameters) and NA-indicator concatenation within its pipeline) — github.com/xryanglab/SCASL
- **Leafcutter** (Upstream: identifies AS events from junction reads and builds the junction matrix whose AS probability matrix is imputed) — github.com/davidaknowles/leafcutter

## Examples

```
python main.py -y configs/srr.yaml  # with num_iteration: 3, knn: 10 in configs/srr.yaml; or run the imputation step directly on normalized_matrix inputs via the scasl package
```

## Evaluation signals

- Output matrix contains no NA entries and has exactly 2M rows (M AS sites + M indicator rows) and N cell columns
- The imputed full matrix concatenated with the NA indicator matrix is the input to downstream spectral clustering, matching SCASL's reported design
- Under 90% forced dropout, the majority of masked AS probability values are still correctly recovered (Fig. S1A)
- t-test comparing imputed vs. true values at masked positions shows no significant difference
- For identical parameters, results are generally consistent across runs, with only minor run-to-run differences from the stochastic imputation

## Limitations

- Single-cell AS profiles suffer greatly from missing data points due to dropouts during scRNA-seq and AS event identification, so imputation quality depends on remaining coverage
- Imputation, dimensionality reduction, and clustering are stochastic; run-to-run differences occur even with identical parameters
- SCASL was designed for sparse, polarized AS profiles — the weighted KNN + NA-indicator scheme is tailored to that setting rather than general expression data

## Evidence

- [discussion] Iterative weighted KNN imputation of missing values: "SCASL employs an iterative, weighted KNN process for imputation of the missing values"
- [discussion] Clustering with NA-indicator concatenation: "SCASL performs cell clustering with the full AS probability matrix, concatenated with a NA indicator matrix"
- [readme] Imputation parameters: "**num_iteration**: the number of iterations during weighted KNN imputation. - **knn**: the number of adjacent cells used in weighted KNN imputation."
- [results] Dropout fidelity finding: [evidence span withheld — anchor status: not_found]
- [discussion] Missingness motivation: "single-cell AS profiles suffer greatly from missing data points due to dropouts during single-cell RNA sequencing analysis"
