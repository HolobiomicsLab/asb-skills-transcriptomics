---
name: kmeans-cell-type-clustering
description: Use when you have pre-processed, log2-transformed scRNA-seq expression
  matrices (cells as rows, genes as columns) with known cell-type labels and need
  to quantify whether imputation improves clustering accuracy.
when_to_use_negative:
- Input matrices are raw counts rather than log2-transformed expression, since the
  comparison protocol assumes log-transformed data
- Cell-type labels are unavailable, since the ARI cannot be computed against original
  annotations
- Datasets small enough that alternative clustering evaluation (e.g. silhouette) is
  desired and no ground-truth annotations exist to justify ARI
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: AutoImpute
  role: Autoencoder-based imputation whose output imputed count matrix is clustered
    with K-means and scored by ARI to assess cell-clustering accuracy
  repo: https://github.com/divyanshu-talwar/AutoImpute
- name: scikit-learn
  role: Provides the KMeans clustering implementation used on each unimputed and imputed
    matrix
- name: scImpute
  role: Baseline imputation method whose outputs are clustered and compared by ARI
- name: drImpute
  role: Baseline imputation method whose outputs are clustered and compared by ARI
- name: MAGIC
  role: Baseline imputation method whose outputs are clustered and compared by ARI
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:72e4a6587a67056b3eeaaeeb8ee9a411940421784f21bcb884447a71f2e6fef5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/divyanshu-talwar/AutoImpute
  tools:
  - AutoImpute
  - MAGIC
  - drImpute
  - scImpute
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# kmeans-cell-type-clustering

## Summary

Benchmark cell-type recovery on log2-transformed scRNA-seq matrices by running scikit-learn K-means (K = number of unique cell types per dataset) on raw and imputed data (scImpute, drImpute, MAGIC, AutoImpute) and scoring each clustering with the Adjusted Rand Index against original cell annotations.

## When to use

Use when you have pre-processed, log2-transformed scRNA-seq expression matrices (cells as rows, genes as columns) with known cell-type labels and need to quantify whether imputation improves clustering accuracy.

## When NOT to use

- Input matrices are raw counts rather than log2-transformed expression, since the comparison protocol assumes log-transformed data
- Cell-type labels are unavailable, since the ARI cannot be computed against original annotations
- Datasets small enough that alternative clustering evaluation (e.g. silhouette) is desired and no ground-truth annotations exist to justify ARI

## Inputs

- Pre-processed, log2-transformed scRNA-seq expression matrices (cells as rows, genes as columns; .csv or .mat)
- Original cell-type annotation vector per dataset
- Number of unique cell types per dataset (used as K-means K)
- Matrices imputed by scImpute, drImpute, MAGIC, and AutoImpute

## Outputs

- ARI value per dataset-method pair (Adjusted Rand Index in [0, 1])
- Comparison table (e.g. ari_table.csv) with eight dataset rows and columns for no-imputation, scImpute, drImpute, MAGIC, and AutoImpute
- K-means cluster label assignments per dataset and method

## How to apply

Load the pre-processed, log2-transformed matrices (genes as columns, cells as rows) for each dataset, e.g. Blakeley, Jurkat-293T, Kolodziejczyk, PBMC, Preimplantation, Quake, Usoskin, and Zeisel. Run sklearn KMeans on each unimputed matrix with K set to the number of unique cell types in that dataset, then repeat with identical K on matrices imputed by scImpute, drImpute, MAGIC, and AutoImpute. Compute the Adjusted Rand Index (ARI) between K-means labels and the original cell-type annotations for every dataset-method pair, and assemble the values into a table with datasets as rows and no-imputation/scImpute/drImpute/MAGIC/AutoImpute as columns. The rationale is that AutoImpute learns the input scRNA-seq distribution and should improve cell-clustering accuracy over the no-imputation baseline; expected behavior is AutoImpute attaining the best ARI on five of eight datasets while always exceeding the baseline. For very large datasets like PBMC (~68,000 cells), restrict evaluation to AutoImpute since other imputers exhaust system memory.

## Related tools

- **AutoImpute** (Autoencoder-based imputation whose output imputed count matrix is clustered with K-means and scored by ARI to assess cell-clustering accuracy) — https://github.com/divyanshu-talwar/AutoImpute
- **scikit-learn** (Provides the KMeans clustering implementation used on each unimputed and imputed matrix)
- **scImpute** (Baseline imputation method whose outputs are clustered and compared by ARI)
- **drImpute** (Baseline imputation method whose outputs are clustered and compared by ARI)
- **MAGIC** (Baseline imputation method whose outputs are clustered and compared by ARI)

## Examples

```
from sklearn.cluster import KMeans; from sklearn.metrics import adjusted_rand_score; labels = KMeans(n_clusters=k, random_state=0).fit_predict(X); ari = adjusted_rand_score(cell_types, labels)
```

## Evaluation signals

- All ARI values fall within [0, 1]
- Result ordering reproduces Table 1 of the AutoImpute paper
- AutoImpute attains the best ARI among evaluated methods on five of the eight tested datasets
- AutoImpute's ARI is always higher than the no-imputation K-means baseline for every dataset
- K per dataset equals the number of unique cell types in that dataset's annotations

## Limitations

- On PBMC (~68,000 cells), scImpute, drImpute, and MAGIC exhaust system memory, so only AutoImpute can be evaluated there
- AutoImpute.py expects a pre-processed count matrix with columns as genes and rows as cells; other datasets require R pre-processing via pre-process.R before imputation and clustering
- ARI depends on using the number of unique cell types as K; other choices of K change the comparison

## Evidence

- [abstract] AutoImpute performed competitively against existing single-cell imputation methods on expression recovery from subsampled data, cell-clustering accuracy, variance stabilization and cell-type separability: "AutoImpute performed competitively wrt., the existing single-cell imputation methods, on the grounds of expression recovery from subsampled data, cell-clustering accuracy, variance stabilization and"
- [intro] Comparison of AutoImpute against existing imputation methods on independent datasets: "We compare the performance of AutoImpute with the existing imputation methods on nine independent datasets."
- [readme] AutoImpute.py input format for pre-processed count single cell matrix: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells."
- [readme] Running the AutoImpute model on a dataset from the command line: "`AutoImpute` model can be run using the following command and command line arguments :
bash
python AutoImpute.py"
