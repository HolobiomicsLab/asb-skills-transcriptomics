---
name: kmeans-clustering-of-single-cell-transcriptomes
description: Use when raw or imputed single-cell count matrices come with known cell-type
  labels and you must quantify whether imputation (e.g., AutoClass, DCA, SAVER, scImpute,
  MAGIC) improves cell-type recovery via clustering metrics.
when_to_use_negative:
- The dataset has no known cell-type labels, so supervised metrics (ARI, JI, NMI,
  purity) cannot be computed and only unsupervised quality checks apply
- The analysis goal is differential expression or batch-effect removal rather than
  cell-type recovery via clustering
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: sklearn
  role: Provides KMeans (n_clusters=K, random_state=1), ARI/NMI/Jaccard metrics, and
    the contingency matrix used by purity_score
- name: AutoClass
  role: Imputes the count matrices (AutoClassImpute, dropout_rate=0.3) whose clustering
    metrics are compared against raw data and control methods
  repo: https://github.com/datapplab/AutoClass
- name: pandas
  role: Loads the count matrices, e.g. Buettner.csv, with read_csv(...,index_col=0)
- name: DCA
  role: Control imputation method compared on the same clustering metrics
  repo: theislab/dca
- name: SAVER
  role: Control imputation method compared on the same clustering metrics
  repo: mohuangx/SAVER
- name: scImpute
  role: Control imputation method compared on the same clustering metrics
  repo: Vivianstats/scImpute
- name: MAGIC
  role: Control imputation method compared on the same clustering metrics
  repo: KrishnaswamyLab/MAGIC
- name: TensorFlow >=2.0
  role: Backend dependency for running the AutoClass module used to impute the data
    before clustering
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:cf68fb1d01d195f34c217816738ca1bbc89c0e965108ee71986b92018a7d2710
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# kmeans-clustering-of-single-cell-transcriptomes

## Summary

Evaluates scRNA-Seq data quality by K-means clustering (with the true cluster number K, random_state=1) on raw and imputed expression matrices and scoring agreement with known cell-type labels using ARI, JI, NMI, and a contingency-matrix purity score, as in the AutoClass Table 1 comparison.

## When to use

Use when raw or imputed single-cell count matrices come with known cell-type labels and you must quantify whether imputation (e.g., AutoClass, DCA, SAVER, scImpute, MAGIC) improves cell-type recovery via clustering metrics.

## When NOT to use

- The dataset has no known cell-type labels, so supervised metrics (ARI, JI, NMI, purity) cannot be computed and only unsupervised quality checks apply
- The analysis goal is differential expression or batch-effect removal rather than cell-type recovery via clustering

## Inputs

- Raw scRNA-Seq count matrices for Buettner, Usoskin, Lake, and Zeisel datasets (.csv, read with pandas, index_col=0)
- Known cell-type label vectors for each dataset
- Imputed matrices from AutoClass (dropout_rate=0.3) and control methods DCA, SAVER, scImpute, MAGIC

## Outputs

- KMeans cluster labels per dataset x method (n_clusters = true K, random_state=1)
- ARI, JI, NMI, and purity scores per dataset x method combination
- clustering_metrics_table.csv with 4 datasets x 4 metrics (16 rows), reproducing Table 1

## How to apply

Load each real scRNA-Seq count matrix (Buettner, Usoskin, Lake, Zeisel) with pandas, preprocess per the Methods by filtering low-quality genes/cells and selecting highly variable genes (the notebook uses find_hv_genes; Table 1 evaluation retains the top 1000 HVGs). Impute each matrix with AutoClassImpute(dropout_rate=0.3) built on TensorFlow >=2.0 and with each control method (DCA, SAVER, scImpute, MAGIC). Run KMeans with n_clusters equal to the true number of cell types K and random_state=1 on each raw and imputed matrix, following the notebook's scheme (clustering in the paper is run on a PCA/t-SNE embedding: PCA to 50 components then t-SNE to 2 dimensions). Compute ARI, JI, NMI, and purity (purity_score via the contingency matrix between y_true and y_pred) between KMeans labels and true cell-type labels for every dataset x method combination, and assemble the results into the 4-dataset x 4-metric table.

## Related tools

- **sklearn** (Provides KMeans (n_clusters=K, random_state=1), ARI/NMI/Jaccard metrics, and the contingency matrix used by purity_score)
- **AutoClass** (Imputes the count matrices (AutoClassImpute, dropout_rate=0.3) whose clustering metrics are compared against raw data and control methods) — https://github.com/datapplab/AutoClass
- **pandas** (Loads the count matrices, e.g. Buettner.csv, with read_csv(...,index_col=0))
- **DCA** (Control imputation method compared on the same clustering metrics) — theislab/dca
- **SAVER** (Control imputation method compared on the same clustering metrics) — mohuangx/SAVER
- **scImpute** (Control imputation method compared on the same clustering metrics) — Vivianstats/scImpute
- **MAGIC** (Control imputation method compared on the same clustering metrics) — KrishnaswamyLab/MAGIC
- **TensorFlow >=2.0** (Backend dependency for running the AutoClass module used to impute the data before clustering)

## Examples

```
from sklearn.cluster import KMeans; kmeans = KMeans(n_clusters=K, random_state=1).fit(tsne)
```

## Evaluation signals

- Table reproduces the reported Table 1 pattern: e.g., Usoskin ARI = 0.869 for AutoClass versus 0.221 for raw data
- AutoClass is the only method that improves all four metrics (ARI, JI, NMI, purity) over the raw data on the real datasets
- AutoClass achieves the best clustering results for 3 out of 4 real datasets and outperforms the control imputation methods
- Purity scores match manual recomputation from the sklearn contingency matrix between y_true and KMeans y_pred
- KMeans uses exactly the true cluster number K and random_state=1 for every dataset x method combination (reproducibility check)

## Limitations

- Results depend on the clustering evaluation scheme (HVG selection, PCA-50/t-SNE-2 embedding before KMeans); clustering directly on raw high-dimensional counts is not the paper's scheme
- Comparison requires known cell-type labels, limiting applicability to datasets with curated annotations
- AutoClass hyperparameters (dropout_rate=0.3, pre-clustering number, bottleneck size) can affect downstream clustering, although the paper reports robustness to these settings
- Control methods run in heterogeneous environments (R for SAVER/scImpute, Python for DCA/MAGIC), which adds reproduction complexity

## Evidence

- [results] KMeans with fixed seed and true K: "kmeans = KMeans(n_clusters = K,random_state=1).fit(tsne)"
- [results] Purity score via contingency matrix: "def purity_score(y_true, y_pred):"
- [abstract] AutoClass outperforms on clustering analysis: "AutoClass outperforms the state-of-art methods in multiple types of scRNA-Seq data analyses, including data recovery, differential expression analysis, clustering analysis"
- [methods] Dependency stack for the AutoClass module: "pip install numpy pandas sklearn matplotlib"
- [results] Real dataset loading pattern: "expr = pd.read_csv('C:/Users/hli45/Desktop/DATA_in_paper/Buettner.csv',index_col=0)"
