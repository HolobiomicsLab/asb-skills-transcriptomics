---
name: purity-score-cluster-contingency-evaluation
description: Use when comparing imputed or denoised scRNA-Seq matrices (e.g., AutoClass
  vs. DCA/SAVER/scImpute/MAGIC outputs) against a raw baseline and you have known
  cell-type labels to quantify cluster recovery.
when_to_use_negative:
- No ground-truth cell-type labels are available — purity, ARI, and NMI require y_true
  and cannot be computed
- Evaluating simulated data with known true counts where direct recovery error against
  truecounts (e.g., Dataset1 truecounts.csv) is the appropriate metric
- The goal is biological interpretation rather than method benchmarking; purity scores
  are for comparing imputation methods, not annotating novel cell types
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: sklearn
  role: KMeans clustering (random_state=1), silhouette/ARI/NMI metrics, contingency
    matrix construction
- name: AutoClass
  role: Provides the imputed matrices (AutoClassImpute, dropout_rate=0.5) and helper
    functions find_hv_genes/take_norm being benchmarked
  repo: https://github.com/datapplab/AutoClass
- name: DCA
  role: Imputation control whose clustering metrics are computed identically for comparison
  repo: theislab/dca
- name: SAVER
  role: Imputation control evaluated with the same purity/ARI/NMI/ASW scoring
  repo: mohuangx/SAVER
- name: scImpute
  role: Imputation control evaluated with the same purity/ARI/NMI/ASW scoring
  repo: Vivianstats/scImpute
- name: MAGIC
  role: Imputation control evaluated with the same purity/ARI/NMI/ASW scoring
  repo: KrishnaswamyLab/MAGIC
- name: numpy
  role: Array handling for contingency matrix and metric computation
- name: matplotlib
  role: Plotting t-SNE embeddings colored by cluster and cell-type labels
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:0ecf6bbeceaabc3b56bd6294e1c057c6045ae943dbe649f7fdf4817c64537eb3
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# purity-score-cluster-contingency-evaluation

## Summary

Evaluates clustering quality of scRNA-Seq imputation/denoising outputs by KMeans clustering a t-SNE embedding at K=8,9,10 (random_state=1) and scoring against known cell-type labels using purity via contingency matrix, adjusted Rand index, normalized mutual information, and average silhouette width on top-200 highly variable genes.

## When to use

Use when comparing imputed or denoised scRNA-Seq matrices (e.g., AutoClass vs. DCA/SAVER/scImpute/MAGIC outputs) against a raw baseline and you have known cell-type labels to quantify cluster recovery.

## When NOT to use

- No ground-truth cell-type labels are available — purity, ARI, and NMI require y_true and cannot be computed
- Evaluating simulated data with known true counts where direct recovery error against truecounts (e.g., Dataset1 truecounts.csv) is the appropriate metric
- The goal is biological interpretation rather than method benchmarking; purity scores are for comparing imputation methods, not annotating novel cell types

## Inputs

- Raw scRNA-Seq count matrix (CSV, genes x cells, loaded with pandas index_col=0)
- Imputed/denoised matrix from AutoClassImpute (dropout_rate=0.3 or 0.5) or controls (DCA, SAVER, scImpute, MAGIC)
- Known per-cell cell-type label vector
- Top-200 highly variable gene indices from find_hv_genes(X, top=200)

## Outputs

- Per-K KMeans cluster labels (K=8,9,10, random_state=1) on the 2-D t-SNE embedding
- Purity score computed via contingency matrix between true labels and KMeans labels
- Adjusted Rand index (ARI) and normalized mutual information (NMI) values
- Average silhouette width (ASW) values
- Aggregated metrics table per dataset, method, and K (e.g., batch_effect_metrics.csv)

## How to apply

Normalize the raw count matrix with take_norm or take the imputed output, then select the top-200 highly variable genes with find_hv_genes(X, top=200) as the evaluation feature set. Generate a 2-D t-SNE embedding (PCA to 50 components first for raw data), run KMeans with random_state=1 over K=8,9,10 to obtain cluster labels, and compute purity score via a contingency matrix (per-cluster majority label mass summed over clusters), plus ARI, NMI, and average silhouette width against known labels. Assemble all metric values per dataset, method, and K into a metrics table (e.g., batch_effect_metrics.csv) so methods and pre/post-imputation baselines are directly comparable. The rationale is that imputation should sharpen cell-type separation (higher purity, ARI, NMI) without aggravating batch structure.

## Related tools

- **sklearn** (KMeans clustering (random_state=1), silhouette/ARI/NMI metrics, contingency matrix construction)
- **AutoClass** (Provides the imputed matrices (AutoClassImpute, dropout_rate=0.5) and helper functions find_hv_genes/take_norm being benchmarked) — https://github.com/datapplab/AutoClass
- **DCA** (Imputation control whose clustering metrics are computed identically for comparison) — theislab/dca
- **SAVER** (Imputation control evaluated with the same purity/ARI/NMI/ASW scoring) — mohuangx/SAVER
- **scImpute** (Imputation control evaluated with the same purity/ARI/NMI/ASW scoring) — Vivianstats/scImpute
- **MAGIC** (Imputation control evaluated with the same purity/ARI/NMI/ASW scoring) — KrishnaswamyLab/MAGIC
- **numpy** (Array handling for contingency matrix and metric computation)
- **matplotlib** (Plotting t-SNE embeddings colored by cluster and cell-type labels)

## Examples

```
kmeans = KMeans(n_clusters=K, random_state=1).fit(tsne); ps = purity_score(y_true, kmeans.labels_)
```

## Evaluation signals

- Purity score recomputed from the contingency matrix should equal the fraction of cells in majority-matched clusters; verify it falls in [0, 1] and is highest for the best imputation method
- Metrics must be computed on the same top-200 find_hv_genes gene set for raw and imputed matrices so methods are comparable
- KMeans results must be stable with random_state=1 across K=8,9,10; a good method (AutoClass) should show higher purity/ARI/NMI than raw and controls
- On batch-effect datasets (Villani, Baron), AutoClass should give higher within-cell-type batch-mixing ASW than raw and controls, consistent with the paper's finding
- The output CSV should contain one row per (dataset, method, K) with non-null ASW, ARI, NMI, and PS columns

## Limitations

- Purity computed on KMeans labels of a 2-D t-SNE embedding is sensitive to the embedding and to K; the paper evaluates multiple K (8,9,10) to mitigate instability
- The paper states no explicit limitations section; supplementary Tables 1-3 (hyperparameter/simulation settings) are not in the available text
- No software version numbers are stated in the article, so exact reproducibility of metric values may vary across sklearn versions

## Evidence

- [results] KMeans on t-SNE with fixed seed: "kmeans = KMeans(n_clusters = K,random_state=1).fit(tsne)"
- [results] highly variable gene selection for clustering evaluation: "highvar_genes = find_hv_genes(X,top=200)"
- [results] purity score function via contingency matrix: "def purity_score(y_true, y_pred):"
- [abstract] AutoClass outperforms methods in clustering and batch effect removal: "AutoClass outperforms the state-of-art methods in multiple types of scRNA-Seq data analyses, including data recovery, differential expression analysis, clustering analysis, and batch effect removal"
- [results] PCA to 50 then t-SNE to 2D for raw baseline: "PCA_raw = PCA(n_components=50).fit_transform(X_norm)
tsne_raw = TSNE(n_components=2).fit_transform(PCA_raw)"
