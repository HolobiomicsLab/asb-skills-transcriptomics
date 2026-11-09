---
name: silhouette-width-batch-mixing-analysis
description: Use when comparing imputation/denoising methods on multi-batch scRNA-Seq
  data to quantify batch-effect removal while confirming cell-type separation, e.g.,
  merging batches of the same cell type.
when_to_use_negative:
- When the dataset has only one batch or no batch annotation — batch-mixing ASW is
  undefined without batches to compare
- When the analysis goal is data recovery or differential expression accuracy rather
  than batch-effect removal — use reconstruction/correlation metrics instead
- When cell-type labels are unavailable, since purity score, ARI, and NMI require
  ground-truth cell-type annotations
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: AutoClass
  role: Primary imputation method whose batch-effect removal is evaluated; run via
    AutoClassImpute with dropout_rate=0.5
  repo: https://github.com/datapplab/AutoClass
- name: DCA
  role: Imputation control run with default settings for ASW/ARI/NMI/purity comparison
  repo: https://github.com/theislab/dca
- name: SAVER
  role: Imputation control; in the Villani dataset SAVER failed to reduce batch effect
  repo: https://github.com/mohuangx/SAVER
- name: scImpute
  role: Imputation control run with default settings
  repo: https://github.com/Vivianstats/scImpute
- name: MAGIC
  role: Imputation control run with default settings
  repo: https://github.com/KrishnaswamyLab/MAGIC
- name: sklearn
  role: Provides KMeans (random_state=1), silhouette/ASW, ARI, NMI, and contingency-matrix
    purity score computation
- name: TensorFlow >=2.0
  role: Backend for running the AutoClass autoencoder-classifier network
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:1739f2f60fd01d3b7a238fb6b6cba4f9ffd3f24c630fe70d9aecc02872785086
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
  - matplotlib
  - numpy
  - pandas
  - scImpute
  - sklearn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7368421052631579
n_claims: 19
---

# silhouette-width-batch-mixing-analysis

## Summary

Evaluates whether an scRNA-Seq imputation/denoising method (e.g., AutoClass) corrects batch effects by computing average silhouette width (ASW) for batch merging within cell types, plus ARI, NMI, and purity score, on t-SNE embeddings of Villani PBMC and Baron pancreatic islet data.

## When to use

Use when comparing imputation/denoising methods on multi-batch scRNA-Seq data to quantify batch-effect removal while confirming cell-type separation, e.g., merging batches of the same cell type.

## When NOT to use

- When the dataset has only one batch or no batch annotation — batch-mixing ASW is undefined without batches to compare
- When the analysis goal is data recovery or differential expression accuracy rather than batch-effect removal — use reconstruction/correlation metrics instead
- When cell-type labels are unavailable, since purity score, ARI, and NMI require ground-truth cell-type annotations

## Inputs

- Villani PBMC scRNA-Seq raw count matrix (GEO accession GSE80171)
- Baron pancreatic islet scRNA-Seq count matrix (CSV, e.g. Baron.csv, loaded with pandas index_col=0)
- Known cell-type labels and batch labels for each cell
- Imputed expression matrices from AutoClassImpute and control methods (DCA, SAVER, scImpute, MAGIC)

## Outputs

- batch_effect_metrics.csv with ASW, ARI, NMI, and purity score per dataset, method, and K (8,9,10)
- batch_effect_tsne.png with raw-vs-AutoClass-vs-control t-SNE embeddings colored by batch and cell type
- 2-D t-SNE embeddings and KMeans cluster labels (random_state=1) per method

## How to apply

Download the Villani PBMC dataset (GEO GSE80171) and the Baron pancreatic islet count matrix, loading each with pandas (index_col=0). Normalize raw counts with take_norm, reduce to 50 PCA components, and embed to 2-D t-SNE as the pre-imputation baseline. Impute with AutoClassImpute (dropout_rate=0.5, TensorFlow >=2.0, pre-clustering K evaluated over K=[8,9,10]) and run controls (DCA, SAVER, scImpute, MAGIC) with default settings. For each method's t-SNE embedding, run KMeans (random_state=1) with K=8,9,10 and compute average silhouette width (ASW) to score batch mixing within cell types (Villani) and batch-effect reduction (Baron), plus ARI, NMI, and purity score against known labels on the top-200 highly variable genes selected with find_hv_genes. The rationale is that AutoClass is distribution agnostic, so batches of the same cell type merge (higher within-cell-type batch ASW) while distinct cell types stay separated.

## Related tools

- **AutoClass** (Primary imputation method whose batch-effect removal is evaluated; run via AutoClassImpute with dropout_rate=0.5) — https://github.com/datapplab/AutoClass
- **DCA** (Imputation control run with default settings for ASW/ARI/NMI/purity comparison) — https://github.com/theislab/dca
- **SAVER** (Imputation control; in the Villani dataset SAVER failed to reduce batch effect) — https://github.com/mohuangx/SAVER
- **scImpute** (Imputation control run with default settings) — https://github.com/Vivianstats/scImpute
- **MAGIC** (Imputation control run with default settings) — https://github.com/KrishnaswamyLab/MAGIC
- **sklearn** (Provides KMeans (random_state=1), silhouette/ASW, ARI, NMI, and contingency-matrix purity score computation)
- **TensorFlow >=2.0** (Backend for running the AutoClass autoencoder-classifier network)

## Examples

```
expr = pd.read_csv('Baron.csv',index_col=0); res = AutoClassImpute(expr.values, dropout_rate=0.5); kmeans = KMeans(n_clusters=9, random_state=1).fit(tsne)
```

## Evaluation signals

- On Villani, AutoClass shows higher ASW for batch merged within cell types than the raw data and all controls (SAVER, DCA, scImpute, MAGIC)
- On Baron, AutoClass reduces batch effect (ASW) relative to the raw pre-imputation baseline
- Cell types remain separated in the t-SNE embedding while the two batches are evenly mixed within each cell type
- ARI, NMI, and purity score are computed consistently on the top-200 highly variable genes (find_hv_genes, top=200) across all methods and K in [8,9,10]
- batch_effect_metrics.csv is well-formed with one row per dataset × method × K combination

## Limitations

- SAVER failed to reduce batch effect and other control methods aggravated it on the Villani dataset, so control-method results may worsen rather than improve embeddings
- Results are reported over multiple pre-clustering K values (K=8,9,10); a single K may not represent method behavior
- Evaluation depends on externally provided cell-type labels (e.g., from the GSE80171 metadata), which may not exist for all datasets

## Evidence

- [abstract] AutoClass outperforms the state-of-art methods in multiple types of scRNA-Seq data analyses, including data recovery, differential expression analysis, clustering analysis, and batch effect removal: "AutoClass outperforms the state-of-art methods in multiple types of scRNA-Seq data analyses, including data recovery, differential expression analysis, clustering analysis, and batch effect removal"
- [results] kmeans fitting on tsne: "kmeans = KMeans(n_clusters = K,random_state=1).fit(tsne)"
- [results] selecting top highly variable genes: "highvar_genes = find_hv_genes(X,top=200)"
- [results] AutoClassImpute call: "res = AutoClassImpute(X,dropout_rate=0.3)"
- [methods] AutoClass installation requirements: "AutoClass runs with Python 3, and you need to have TensorFlow (>=2.0) and a few other pacakges installed first"
