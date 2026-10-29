---
name: imputation-method-benchmarking-across-datasets
description: Use when raw scRNA-Seq count matrices with known cell-type labels are
  available and you need to quantify whether imputation improves downstream clustering
  versus raw data and control methods.
when_to_use_negative:
- Data are simulated with known true counts, where direct data-recovery metrics (e.g.,
  correlation to truecounts.csv) are more appropriate than clustering benchmarks
- The dataset lacks trusted cell-type labels, so ARI/JI/NMI/purity cannot be computed
  against ground truth
- The goal is differential expression analysis or batch effect removal rather than
  clustering-based imputation benchmarking
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_3170
tools:
- name: AutoClass
  role: Primary imputation method under evaluation via AutoClassImpute with dropout_rate=0.3;
    installed by cloning the GitHub repo and running with Python 3 and TensorFlow
    >=2.0
  repo: https://github.com/datapplab/AutoClass
- name: DCA
  role: Control imputation method (deep count autoencoder, ZINB loss) benchmarked
    against AutoClass on the same matrices
  repo: https://github.com/theislab/dca
- name: SAVER
  role: Control imputation method (regularized regression with empirical Bayes) benchmarked
    against AutoClass
  repo: https://github.com/mohuangx/SAVER
- name: scImpute
  role: Control imputation method (mixture of Gaussian and Gamma model) benchmarked
    against AutoClass
  repo: https://github.com/Vivianstats/scImpute
- name: MAGIC
  role: Control imputation method benchmarked against AutoClass
  repo: https://github.com/KrishnaswamyLab/MAGIC
- name: sklearn
  role: Provides KMeans (n_clusters=K, random_state=1), ARI, NMI, Jaccard index, and
    the contingency-matrix-based purity score
- name: TensorFlow >=2.0
  role: Required dependency for running the AutoClass module
- name: pandas
  role: Loads the CSV count matrices (e.g., pd.read_csv('Buettner.csv', index_col=0))
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:ed7998a90f64a0879e28633700a10c7e7f19846c9b66840778b54e79559f7f24
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/datapplab/AutoClass
  tools:
  - AutoClass
  - DCA
  - MAGIC
  - Python
  - SAVER
  - TensorFlow >=2.0
  - numpy
  - pandas
  - scImpute
  - sklearn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 18
---

# imputation-method-benchmarking-across-datasets

## Summary

Benchmark scRNA-Seq imputation methods (AutoClass vs DCA, SAVER, scImpute, MAGIC) by K-means clustering real count matrices and comparing ARI, JI, NMI, and purity against known cell-type labels, as in Table 1 of the AutoClass paper.

## When to use

Use when raw scRNA-Seq count matrices with known cell-type labels are available and you need to quantify whether imputation improves downstream clustering versus raw data and control methods.

## When NOT to use

- Data are simulated with known true counts, where direct data-recovery metrics (e.g., correlation to truecounts.csv) are more appropriate than clustering benchmarks
- The dataset lacks trusted cell-type labels, so ARI/JI/NMI/purity cannot be computed against ground truth
- The goal is differential expression analysis or batch effect removal rather than clustering-based imputation benchmarking

## Inputs

- Real scRNA-Seq count matrices as CSV (e.g., Buettner.csv, Usoskin, Lake, Zeisel) loadable with pandas
- Known cell-type label vectors for each dataset
- True cluster number K per dataset

## Outputs

- clustering_metrics_table.csv with one row per dataset x method combination and columns for ARI, JI, NMI, and purity score
- Per-dataset KMeans cluster label assignments on raw and imputed matrices

## How to apply

Load each real scRNA-Seq count matrix (e.g., Buettner, Usoskin, Lake, Zeisel) with pandas together with its known cell-type labels, then preprocess per the Methods by filtering low-quality genes/cells and retaining the top 1000 highly variable genes. Impute each matrix with AutoClassImpute (dropout_rate=0.3) built on TensorFlow >=2.0 and with each control method (DCA, SAVER, scImpute, MAGIC), keeping raw data as a baseline. Run KMeans with the true cluster number K and random_state=1 on each raw and imputed matrix following the notebook's evaluation scheme. Compute ARI, JI, NMI, and purity score (purity via the contingency-matrix function) between KMeans labels and true labels for every dataset x method combination. Assemble the results into a table (e.g., 4 datasets x 4 metrics) and validate that the reported pattern holds, e.g., Usoskin ARI = 0.869 for AutoClass vs 0.221 raw, with AutoClass the only method improving all four metrics over raw data.

## Related tools

- **AutoClass** (Primary imputation method under evaluation via AutoClassImpute with dropout_rate=0.3; installed by cloning the GitHub repo and running with Python 3 and TensorFlow >=2.0) — https://github.com/datapplab/AutoClass
- **DCA** (Control imputation method (deep count autoencoder, ZINB loss) benchmarked against AutoClass on the same matrices) — https://github.com/theislab/dca
- **SAVER** (Control imputation method (regularized regression with empirical Bayes) benchmarked against AutoClass) — https://github.com/mohuangx/SAVER
- **scImpute** (Control imputation method (mixture of Gaussian and Gamma model) benchmarked against AutoClass) — https://github.com/Vivianstats/scImpute
- **MAGIC** (Control imputation method benchmarked against AutoClass) — https://github.com/KrishnaswamyLab/MAGIC
- **sklearn** (Provides KMeans (n_clusters=K, random_state=1), ARI, NMI, Jaccard index, and the contingency-matrix-based purity score)
- **TensorFlow >=2.0** (Required dependency for running the AutoClass module)
- **pandas** (Loads the CSV count matrices (e.g., pd.read_csv('Buettner.csv', index_col=0)))

## Examples

```
expr = pd.read_csv('Buettner.csv', index_col=0); res = AutoClassImpute(expr.values, dropout_rate=0.3); kmeans = KMeans(n_clusters=K, random_state=1).fit_predict(res)
```

## Evaluation signals

- The 16-row table (4 datasets x 4 methods plus raw baseline) reproduces the paper's Table 1 pattern, e.g., Usoskin ARI = 0.869 for AutoClass vs 0.221 for raw data
- AutoClass is the only method that improves all four clustering metrics (ARI, JI, NMI, purity) over the raw data
- AutoClass achieves the best clustering results for 3 out of 4 real datasets, outperforming DCA, SAVER, scImpute, and MAGIC
- KMeans uses the true cluster number K with random_state=1 for every method x dataset cell, so differences reflect imputation quality not clustering stochasticity
- Each cell's metrics are computed against the same known cell-type labels using identical preprocessing (top 1000 highly variable genes)

## Limitations

- Conclusions rest on four real datasets; the article reports no explicit limitations section or caveat statements in the discussion
- Clustering metrics are computed on labels from KMeans with fixed random_state=1; sensitivity to the evaluation pipeline (e.g., t-SNE embedding in the notebook scheme) is not systematically quantified in the provided text
- Preprocessing choices (filtering low-quality genes/cells, top 1000 highly variable genes) affect results and must be applied identically across methods for a fair comparison

## Evidence

- [abstract] AutoClass outperforms the state-of-art methods in multiple types of scRNA-Seq data analyses, including data recovery, differential expression analysis, clustering analysis: "AutoClass outperforms the state-of-art methods in multiple types of scRNA-Seq data analyses, including data recovery, differential expression analysis, clustering analysis"
- [results] AutoClassImpute usage with dropout_rate: "res = AutoClassImpute(X,dropout_rate=0.3)"
- [results] KMeans clustering with fixed random state: "kmeans = KMeans(n_clusters = K,random_state=1).fit(tsne)"
- [results] Purity score computed via contingency matrix: "def purity_score(y_true, y_pred):"
- [methods] Runtime dependencies for AutoClass: "AutoClass runs with Python 3, and you need to have TensorFlow (>=2.0) and a few other pacakges installed"
- [results] KMeans assignment of cluster labels: "kmeans = KMeans(n_clusters = K,random_state=1).fit(tsne)"
