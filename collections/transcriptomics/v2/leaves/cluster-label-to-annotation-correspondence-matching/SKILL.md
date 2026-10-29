---
name: cluster-label-to-annotation-correspondence-matching
description: Use when you have cells×genes log-transformed scRNA-seq matrices plus
  original cell-type annotations and need a numeric measure of how well clusters match
  the annotations (ARI) to benchmark methods.
when_to_use_negative:
- Input matrices are raw counts, not log2-transformed pre-processed matrices — the
  published comparison was performed on log2-transformed data, so ARI values would
  not be comparable
- The task is expression recovery (masked-matrix recovery) or variance stabilization
  rather than cell-clustering accuracy — other evaluation protocols in the AutoImpute
  paper apply
- No ground-truth cell-type annotations exist for the dataset — ARI cannot be computed
  against original annotations
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: AutoImpute
  role: Produces the imputed matrices whose K-means cluster labels are compared to
    cell annotations via ARI; one candidate imputation method in the benchmark
  repo: github.com/divyanshu-talwar/AutoImpute
- name: scikit-learn
  role: Provides the K-means implementation (K = number of unique cell-types) and
    the metric used to score cluster-label correspondence
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:3c0472f809e9f5afb3c778dd180c4b0274f1a82bbaed1887e3bacd5d402169c8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoImpute
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cluster-label-to-annotation-correspondence-matching

## Summary

Evaluates cell-clustering accuracy by clustering log2-transformed scRNA-seq expression matrices with scikit-learn K-means and scoring the correspondence between assigned cluster labels and original cell-type annotations via Adjusted Rand Index (ARI), as done in the AutoImpute paper's Table 1 comparison of imputation methods.

## When to use

Use when you have cells×genes log-transformed scRNA-seq matrices plus original cell-type annotations and need a numeric measure of how well clusters match the annotations (ARI) to benchmark methods.

## When NOT to use

- Input matrices are raw counts, not log2-transformed pre-processed matrices — the published comparison was performed on log2-transformed data, so ARI values would not be comparable
- The task is expression recovery (masked-matrix recovery) or variance stabilization rather than cell-clustering accuracy — other evaluation protocols in the AutoImpute paper apply
- No ground-truth cell-type annotations exist for the dataset — ARI cannot be computed against original annotations

## Inputs

- Pre-processed, log2-transformed scRNA-seq expression matrices (cells as rows, genes as columns; .csv or .mat), unimputed and/or imputed (e.g., blakeley.csv)
- Original cell-type annotations (per-cell labels) for each dataset
- Number of unique cell-types per dataset (used as K for K-means)

## Outputs

- ARI value per dataset–method pair quantifying cluster-label vs. annotation agreement
- Comparison table (e.g., ari_table.csv) with rows = datasets, columns = no-imputation / scImpute / drImpute / MAGIC / AutoImpute

## How to apply

Load the pre-processed, log2-transformed expression matrices (rows = cells, columns = genes) for each dataset, and run scikit-learn K-means with K initialized to the number of unique cell-types in that dataset. Run the same clustering both on the unimputed baseline matrix and on the matrices imputed by the methods being compared (e.g., scImpute, drImpute, MAGIC, AutoImpute). For each dataset–method pair, compute the Adjusted Rand Index between the K-means cluster labels and the original cell-type annotations, which corrects for chance agreement so values are comparable across Ks. Assemble the ARI values into a comparison table (rows = datasets, columns = methods plus no-imputation) and judge correctness by checking ARIs fall in [0, 1], the method ordering reproduces the published table, and the improved method beats the no-imputation baseline. Note that on very large datasets (e.g., PBMC with ~68,000 cells) some imputation methods may exhaust system memory, so restrict the comparison to the methods that scale.

## Related tools

- **AutoImpute** (Produces the imputed matrices whose K-means cluster labels are compared to cell annotations via ARI; one candidate imputation method in the benchmark) — github.com/divyanshu-talwar/AutoImpute
- **scikit-learn** (Provides the K-means implementation (K = number of unique cell-types) and the metric used to score cluster-label correspondence)

## Examples

```
from sklearn.cluster import KMeans; from sklearn.metrics import adjusted_rand_score; labels = KMeans(n_clusters=n_celltypes, random_state=0).fit_predict(X); ari = adjusted_rand_score(cell_annotations, labels)
```

## Evaluation signals

- All computed ARI values fall within [0, 1]
- Table ordering reproduces Table 1 of the AutoImpute paper across the eight datasets (Blakeley, Jurkat-293T, Kolodziejczyk, PBMC, Preimplantation, Quake, Usoskin, Zeisel)
- AutoImpute attains the best ARI on five of the eight datasets and its ARI exceeds the no-imputation baseline on every dataset
- K for each dataset's K-means equals the number of unique cell-types in that dataset's annotations
- Output table schema has exactly the expected columns (no-imputation, scImpute, drImpute, MAGIC, AutoImpute) and rows for all evaluated datasets

## Limitations

- On PBMC (~68,000 cells), scImpute, drImpute, and MAGIC exhaust system memory, so only AutoImpute can be evaluated for that dataset
- The article's source document lacks a discussion section ('No discussion section content available'), so caveats about the ARI protocol are not elaborated by the authors
- Results depend on the pre-processing (log2-transform, gene/cell filtering) done upstream in R; different pre-processing may change ARI ordering

## Evidence

- [abstract] clustering accuracy evaluated by ARI vs original annotations: "AutoImpute performed competitively wrt., the existing single-cell imputation methods, on the grounds of expression recovery from subsampled data, cell-clustering accuracy, variance stabilization and"
- [readme] input matrix orientation: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells."
- [readme] pre-processing scripts define the upstream matrix: "To pre-process any dataset, place the raw data (in `.csv` format) into `AutoImpute\ Model/data/raw/`"
