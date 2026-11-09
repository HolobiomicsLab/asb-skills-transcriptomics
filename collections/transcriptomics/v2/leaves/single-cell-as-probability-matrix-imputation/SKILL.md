---
name: single-cell-as-probability-matrix-imputation
description: Use when a single-cell AS probability matrix (rows = AS sites, columns
  = cells) produced from junction reads contains NA entries from dropouts and an imputed,
  NA-indicator-annotated matrix is needed before unsupervised clustering.
when_to_use_negative:
- Input is already a complete junction-read or AS probability matrix with no NA entries
  (imputation and indicator concatenation are unnecessary)
- The goal is gene-expression-based clustering, which bypasses the splicing-landscape
  matrix entirely
- 'Using bulk RNA-seq AS quantification: this imputation is designed for single-cell
  SMART-seq-derived AS profiles with sparse coverage'
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_3170
tools:
- name: SCASL
  role: Provides the filtered AS probability matrix, NA position information, and
    the imputation procedure itself (scasl package py files); imputation parameters
    set via configs/srr.yaml
  repo: https://github.com/xryanglab/SCASL
- name: Leafcutter
  role: 'Upstream: defines AS events from junction reads to build the junction matrix
    whose AS probability matrix is imputed'
  repo: https://github.com/davidaknowles/leafcutter
- name: scikit-learn
  role: Supplies KNN/neighbor machinery underlying the weighted KNN imputation within
    the scasl environment (python 3.9)
provenance:
  source_task_ids:
  - task_006
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-46480-9
    title: Interrogations of single-cell RNA splicing landscapes with SCASL define
      new cell identities with physiological relevance
  source_content_hash: sha256:0e8e8b5d8e3e3f6f7caa618c83bf2463ec9e1abfc14e0ef20f8ff8f72441f679
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xryanglab/SCASL
  tools:
  - Leafcutter
  - Python
  - SCASL
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# single-cell-as-probability-matrix-imputation

## Summary

Iterative weighted KNN imputation of missing values in the SCASL single-cell AS probability matrix, followed by vertical concatenation with a binary NA indicator matrix so that downstream spectral clustering accounts for sparsity. This resolves the biased and sparse single-cell RNA splicing coverage that blocks conventional clustering.

## When to use

Use when a single-cell AS probability matrix (rows = AS sites, columns = cells) produced from junction reads contains NA entries from dropouts and an imputed, NA-indicator-annotated matrix is needed before unsupervised clustering.

## When NOT to use

- Input is already a complete junction-read or AS probability matrix with no NA entries (imputation and indicator concatenation are unnecessary)
- The goal is gene-expression-based clustering, which bypasses the splicing-landscape matrix entirely
- Using bulk RNA-seq AS quantification: this imputation is designed for single-cell SMART-seq-derived AS profiles with sparse coverage

## Inputs

- Filtered, normalized AS probability matrix (rows = AS sites, columns = cells) from SCASL (e.g., normalized_matrix)
- NA position information recorded by the SCASL pipeline
- configs/srr.yaml with num_iteration (number of imputation iterations) and knn (number of adjacent cells)

## Outputs

- normalized_matrix.csv: imputed AS probability matrix vertically concatenated with the NA indicator matrix (2M rows: M sites + M indicator rows; N cell columns)
- Imputed full AS probability matrix with no NA entries, used as input for downstream spectral clustering

## How to apply

Load the filtered AS probability matrix and its NA position information from the SCASL pipeline, reading num_iteration and knn from configs/srr.yaml. Initialize each NA entry with the mean AS probability of the corresponding site. Run iterative distance-weighted KNN imputation (e.g., t = 3 iterations, k = 10 nearest cells) using Euclidean distances on the current matrix, updating each missing entry as the weighted average of neighbors' values with normalized exponential weights on distances; each round refreshes inferred values from the updated matrix until the iteration count is exhausted. Build a binary NA indicator matrix marking originally missing entries, then vertically concatenate it with the imputed AS probability matrix to form the final 2M x N matrix (M sites plus M indicator rows, N cells) and save as normalized_matrix.csv. The rationale is that single-cell AS profiles suffer greatly from dropouts, and the indicator matrix lets clustering distinguish signal from missingness.

## Related tools

- **SCASL** (Provides the filtered AS probability matrix, NA position information, and the imputation procedure itself (scasl package py files); imputation parameters set via configs/srr.yaml) — https://github.com/xryanglab/SCASL
- **Leafcutter** (Upstream: defines AS events from junction reads to build the junction matrix whose AS probability matrix is imputed) — https://github.com/davidaknowles/leafcutter
- **scikit-learn** (Supplies KNN/neighbor machinery underlying the weighted KNN imputation within the scasl environment (python 3.9))

## Examples

```
python -c "import pandas as pd, yaml; cfg = yaml.safe_load(open('configs/srr.yaml')); from scasl.imputation import knn_impute; M = knn_impute(pd.read_csv('normalized_matrix.csv'), num_iteration=cfg['num_iteration'], knn=cfg['knn']); M.to_csv('normalized_matrix.csv')"
```

## Evaluation signals

- Output matrix contains no NA entries after imputation
- Output matrix has exactly 2M rows (M AS sites plus M indicator rows) and N cell columns
- Forced-dropout fidelity check: imputed values at artificially masked non-NA positions (10%, 30%, 90% dropout) largely recover true values, consistent with Fig. S1A where even 90% dropout yields majority correct recovery
- t-test between imputed and true values across dropout levels flags no significant difference
- Reruns with identical num_iteration/knn parameters yield generally consistent downstream clustering despite randomness in imputation, dimensionality reduction, and unsupervised clustering

## Limitations

- Single-cell AS profiles suffer greatly from missing data points due to dropouts during single-cell RNA sequencing analysis and AS event identification, so imputation quality depends on remaining coverage
- Coverage of genes and AS events can be very unbalanced across different single cells
- Imputation introduces randomness; running results may differ between runs with identical parameters
- Many canonical clustering methods are unsuitable for the largely polarized AS profiles, which is why the imputed matrix is concatenated with an NA indicator matrix before spectral clustering

## Evidence

- [discussion] SCASL employs an iterative, weighted KNN process for imputation of the missing values: "SCASL employs an iterative, weighted KNN process for imputation of the missing values"
- [discussion] SCASL performs cell clustering with the full AS probability matrix, concatenated with a NA indicator matrix: "SCASL performs cell clustering with the full AS probability matrix, concatenated with a NA indicator matrix"
- [readme] num_iteration: the number of iterations during weighted KNN imputation; knn: the number of adjacent cells used in weighted KNN imputation: "**num_iteration**: the number of iterations during weighted KNN imputation.
- **knn**: the number of adjacent cells used in weighted KNN imputation."
- [readme] normalized_matrix: The final AS probability matrix, the rows correspond to sites, while the columns represent cells.: "**normalized_matrix**: The final AS probability matrix, the rows correspond to sites, while the columns represent cells."
- [results] even when 90% of AS probability values were dropped out, the majority of these data points were still correctly recovered: [evidence span withheld — anchor status: not_found]
