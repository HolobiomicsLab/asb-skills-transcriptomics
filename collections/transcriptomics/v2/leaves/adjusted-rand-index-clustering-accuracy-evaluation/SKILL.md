---
name: adjusted-rand-index-clustering-accuracy-evaluation
description: Use when a task requires quantifying how well K-means clusters of imputed
  (or unimputed) log-transformed scRNA-seq expression matrices agree with known cell-type
  annotations via Adjusted Rand Index.
when_to_use_negative:
- The goal is expression-value recovery or masked-matrix reconstruction rather than
  clustering accuracy — use an expression recovery test (e.g., AutoImpute's --masked_matrix_test)
  instead
- Cell-type annotations are unavailable, so ARI against original labels cannot be
  computed
- The data are not log-transformed count matrices (e.g., raw unnormalized counts or
  other omics), which would break comparability with the paper's Table 1 protocol
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: AutoImpute
  role: Autoencoder-based imputation method whose output matrices are clustered and
    compared against the no-imputation baseline and other imputers via ARI
  repo: https://github.com/divyanshu-talwar/AutoImpute
- name: scikit-learn
  role: Provides the K-means implementation used to assign cluster labels (K = number
    of unique cell-types per dataset) on unimputed and imputed matrices
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:1f3baeba781bc494cfe6ebafe5fa5001816be06e260303f5b3664f613d9746e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/divyanshu-talwar/AutoImpute
  tools:
  - AutoImpute
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# adjusted-rand-index-clustering-accuracy-evaluation

## Summary

Evaluates whether imputation of log2-transformed scRNA-seq dropout improves K-means cell-clustering accuracy by computing the Adjusted Rand Index (ARI) between K-means labels and original cell-type annotations, as done in the AutoImpute benchmark (Scientific Reports, 2018).

## When to use

Use when a task requires quantifying how well K-means clusters of imputed (or unimputed) log-transformed scRNA-seq expression matrices agree with known cell-type annotations via Adjusted Rand Index.

## When NOT to use

- The goal is expression-value recovery or masked-matrix reconstruction rather than clustering accuracy — use an expression recovery test (e.g., AutoImpute's --masked_matrix_test) instead
- Cell-type annotations are unavailable, so ARI against original labels cannot be computed
- The data are not log-transformed count matrices (e.g., raw unnormalized counts or other omics), which would break comparability with the paper's Table 1 protocol

## Inputs

- Pre-processed, log2-transformed scRNA-seq expression matrices (.csv or .mat, cells as rows, genes as columns) for each dataset
- Imputed expression matrices from scImpute, drImpute, MAGIC, and AutoImpute for the same datasets
- Original cell-type annotation labels (one label per cell) for each dataset

## Outputs

- ARI table (e.g., ari_table.csv) with rows for the eight datasets and columns for no-imputation, scImpute, drImpute, MAGIC, and AutoImpute
- Per dataset-method pair ARI values in [0, 1]

## How to apply

Load the pre-processed, log2-transformed expression matrices (cells as rows, genes as columns) for each dataset (e.g., Blakeley, Jurkat-293T, Kolodziejczyk, PBMC, Preimplantation, Quake, Usoskin, Zeisel). Run scikit-learn K-means with K equal to the number of unique cell-types in each dataset, once on the unimputed matrix and once on each imputed matrix (scImpute, drImpute, MAGIC, AutoImpute); for PBMC (~68,000 cells) evaluate only AutoImpute because the other imputers exhaust system memory. Compute the ARI between the K-means-assigned cluster labels and the original cell-type annotations for every dataset-method pair, then assemble the values into a table (rows = datasets, columns = no-imputation, scImpute, drImpute, MAGIC, AutoImpute). The rationale is that a better imputation method should recover cluster structure closer to the curated annotations, so a higher ARI indicates better cell-clustering accuracy.

## Related tools

- **AutoImpute** (Autoencoder-based imputation method whose output matrices are clustered and compared against the no-imputation baseline and other imputers via ARI) — https://github.com/divyanshu-talwar/AutoImpute
- **scikit-learn** (Provides the K-means implementation used to assign cluster labels (K = number of unique cell-types per dataset) on unimputed and imputed matrices)

## Examples

```
cd AutoImpute\ Model && python AutoImpute.py --data blakeley.csv --iterations 7000 --imputed_save imputed_matrix
```

## Evaluation signals

- All reported ARI values fall within [0, 1]
- The assembled table reproduces the ordering of the paper's Table 1
- AutoImpute attains the best ARI on five of the eight datasets
- AutoImpute's ARI is always higher than the no-imputation K-means baseline
- K for K-means equals the number of unique cell-types in each dataset

## Limitations

- On the PBMC dataset (~68,000 cells), scImpute, drImpute, and MAGIC exhaust system memory, so only AutoImpute can be evaluated alongside the no-imputation baseline
- AutoImpute.py by default trains with Python 2.7 and TensorFlow (7000 iterations, learning rate 0.0001, hidden_units 2000, lambda_val 1, threshold 0.0001), which may need adapting on modern environments
- ARI depends on running K-means with the known number of cell-types; it cannot be applied when true cell-type labels or counts are unknown

## Evidence

- [abstract] Abstract: AutoImpute competitive performance scope: "AutoImpute performed competitively wrt., the existing single-cell imputation methods"
- [intro] Intro: proposal of AutoImpute: "In this work, we propose AutoImpute (Fig. 1), a novel method for sparse gene expression matrix imputation using overcomplete autoencoders."
- [readme] README: expected input format: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells."
