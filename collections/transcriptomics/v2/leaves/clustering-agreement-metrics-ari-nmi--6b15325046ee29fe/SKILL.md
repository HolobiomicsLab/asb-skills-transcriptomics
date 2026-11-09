---
name: clustering-agreement-metrics-ari-nmi
description: Use when comparing cluster assignments from KMeans (random_state=1) on
  raw or imputed scRNA-Seq data against known cell-type labels to quantify whether
  imputation (e.g., AutoClassImpute) improves clustering (ARI, JI, NMI, purity).
when_to_use_negative:
- No ground-truth cell-type labels are available for the dataset, since ARI/JI/NMI/purity
  all require a reference labeling
- The goal is unsupervised cluster-quality scoring only (e.g., silhouette width for
  batch mixing), not agreement with known labels
- Differential expression or data-recovery evaluation against true counts (e.g., Dataset1
  truecounts.csv) rather than label agreement is the target
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: AutoClass
  role: Imputes the scRNA-Seq count matrix (AutoClassImpute) whose downstream KMeans
    labels are scored against true labels
  repo: https://github.com/datapplab/AutoClass
- name: sklearn
  role: Provides KMeans clustering, ARI, NMI, and contingency-matrix functionality
    for computing purity score
- name: pandas
  role: Loads count matrices and label tables (index_col=0) and writes the metrics
    table CSV
- name: DCA
  role: Control imputation method whose clustering labels are scored in the same metric
    pipeline
  repo: https://github.com/theislab/dca
- name: SAVER
  role: Control imputation method whose clustering labels are scored in the same metric
    pipeline
  repo: https://github.com/mohuangx/SAVER
- name: scImpute
  role: Control imputation method whose clustering labels are scored in the same metric
    pipeline
  repo: https://github.com/Vivianstats/scImpute
- name: MAGIC
  role: Control imputation method whose clustering labels are scored in the same metric
    pipeline
  repo: https://github.com/KrishnaswamyLab/MAGIC
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:509ebea235af023afad1b6b7753f9720d3b56898caa5eedfeaf66084da34c186
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
  - matplotlib
  - numpy
  - pandas
  - scImpute
  - sklearn
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# clustering-agreement-metrics-ari-nmi

## Summary

Evaluate K-means clustering of scRNA-Seq matrices (raw or imputed, e.g., by AutoClass) against known cell-type labels using adjusted Rand index, Jaccard index, normalized mutual information, and purity score computed from a contingency matrix.

## When to use

Use when comparing cluster assignments from KMeans (random_state=1) on raw or imputed scRNA-Seq data against known cell-type labels to quantify whether imputation (e.g., AutoClassImpute) improves clustering (ARI, JI, NMI, purity).

## When NOT to use

- No ground-truth cell-type labels are available for the dataset, since ARI/JI/NMI/purity all require a reference labeling
- The goal is unsupervised cluster-quality scoring only (e.g., silhouette width for batch mixing), not agreement with known labels
- Differential expression or data-recovery evaluation against true counts (e.g., Dataset1 truecounts.csv) rather than label agreement is the target

## Inputs

- KMeans cluster labels per cell (e.g., from t-SNE embedding or imputed expression matrix, K = true cluster number, random_state=1)
- Known cell-type label vector per cell (e.g., Buettner, Usoskin, Lake, Zeisel datasets loaded with pandas index_col=0)
- Optional: batch_effect_metrics.csv-style per-method, per-K metric records

## Outputs

- ARI, JI, NMI, and purity score per dataset x imputation-method combination (e.g., clustering_metrics_table.csv with 16 rows)
- Comparison table showing whether imputed data improves all four metrics over raw data

## How to apply

Run KMeans (sklearn, random_state=1) with the true cluster number K on each raw and imputed matrix (e.g., AutoClassImpute output with dropout_rate=0.3, or DCA/SAVER/scImpute/MAGIC controls). For every dataset x method combination, compute adjusted Rand index (ARI), Jaccard index (JI), and normalized mutual information (NMI) between KMeans labels and true cell-type labels using sklearn. Compute the purity score by assigning each predicted cluster its majority true label via a contingency matrix and summing the majority counts divided by the number of cells. Assemble all dataset x metric values into a table (e.g., clustering_metrics_table.csv, one row per dataset x method, 4 datasets x 4 metrics = 16 rows) and verify the expected pattern, e.g., Usoskin ARI = 0.869 for AutoClass versus 0.221 for raw data.

## Related tools

- **AutoClass** (Imputes the scRNA-Seq count matrix (AutoClassImpute) whose downstream KMeans labels are scored against true labels) — https://github.com/datapplab/AutoClass
- **sklearn** (Provides KMeans clustering, ARI, NMI, and contingency-matrix functionality for computing purity score)
- **pandas** (Loads count matrices and label tables (index_col=0) and writes the metrics table CSV)
- **DCA** (Control imputation method whose clustering labels are scored in the same metric pipeline) — https://github.com/theislab/dca
- **SAVER** (Control imputation method whose clustering labels are scored in the same metric pipeline) — https://github.com/mohuangx/SAVER
- **scImpute** (Control imputation method whose clustering labels are scored in the same metric pipeline) — https://github.com/Vivianstats/scImpute
- **MAGIC** (Control imputation method whose clustering labels are scored in the same metric pipeline) — https://github.com/KrishnaswamyLab/MAGIC

## Examples

```
from sklearn.cluster import KMeans; from sklearn.metrics import adjusted_rand_score, normalized_mutual_info_score; kmeans = KMeans(n_clusters=K, random_state=1).fit(tsne); ari = adjusted_rand_score(y_true, kmeans.labels_)
```

## Evaluation signals

- Table reproduces the paper's Table 1 pattern: AutoClass is the only method improving all four metrics (ARI, JI, NMI, purity) over raw data on all four datasets (Buettner, Usoskin, Lake, Zeisel)
- Usoskin ARI reaches 0.869 for AutoClass versus 0.221 for raw data
- Purity score is computed via the contingency-matrix function (majority-label sum / number of cells) rather than a bespoke formula
- KMeans is run with random_state=1 and K set to the true cluster number for every dataset x method combination
- Metrics are computed on the top highly variable genes selected with find_hv_genes (e.g., top-200 for clustering evaluation) after filtering to top-1000 HVGs

## Limitations

- Metrics require known cell-type labels; on simulated data, use the true group labels, and results depend on pre-clustering K choices (AutoClass was evaluated over K=[8,9,10])
- No software version numbers are stated in the paper's results/discussion, so exact library versions for sklearn metric reproducibility are unspecified
- Label agreement scores can be sensitive to the KMeans random_state and the embedding (t-SNE vs. expression space) on which clustering is performed

## Evidence

- [results] Purity via contingency matrix: "def purity_score(y_true, y_pred):"
- [results] KMeans with fixed seed: "kmeans = KMeans(n_clusters = K,random_state=1).fit(tsne)"
- [results] HVG selection for evaluation: "highvar_genes = find_hv_genes(X,top=200)"
- [full_text] AutoClass is the only method that improves all four clustering metrics over the raw data: [evidence span withheld — anchor status: not_found]
- [results] Preprocessing pipeline on which metrics are computed: "X_norm = take_norm(X)"
