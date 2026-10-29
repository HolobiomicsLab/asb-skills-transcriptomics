---
name: tsne-embedding-of-single-cell-data
description: Use when single-cell expression matrices (e.g., Villani PBMC, Baron pancreatic
  islet) need a 2-D visualization of cell-type separation or batch mixing, or a t-SNE
  embedding is required before KMeans clustering for ASW/ARI/NMI/purity evaluation.
when_to_use_negative:
- Input is already a low-dimensional embedding or 2-D coordinates — re-embedding would
  discard signal
- The analysis goal is a quantitative reconstruction-accuracy check against true counts
  rather than visual/batch-effect assessment, where metric computation on highly variable
  genes is preferred
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: AutoClass
  role: Provides AutoClassImpute-imputed matrices whose t-SNE embeddings are compared
    against raw and control-method embeddings
  repo: https://github.com/datapplab/AutoClass
- name: sklearn
  role: Supplies PCA(n_components=50), TSNE(n_components=2), and KMeans(random_state=1)
    used to build and cluster the embeddings
- name: pandas
  role: Loads the scRNA-Seq count matrices with index_col=0 prior to normalization
    and embedding
- name: matplotlib
  role: Renders the t-SNE scatter plots colored by batch and cell type
- name: numpy
  role: Converts the DataFrame to an array (X = expr.values) for the normalization/PCA/TSNE
    pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:3754d2c414e89e408e41ed15a27a2a53d81dd6896c5acc470f1dd560354f7c42
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
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# tsne-embedding-of-single-cell-data

## Summary

Generate 2-D t-SNE embeddings of scRNA-Seq matrices (raw or imputed) by first reducing library-normalized counts to 50 PCA components, then embedding to 2-D with TSNE, for visual assessment of cell-type separation and batch mixing and as input to KMeans-based clustering metrics.

## When to use

Use when single-cell expression matrices (e.g., Villani PBMC, Baron pancreatic islet) need a 2-D visualization of cell-type separation or batch mixing, or a t-SNE embedding is required before KMeans clustering for ASW/ARI/NMI/purity evaluation.

## When NOT to use

- Input is already a low-dimensional embedding or 2-D coordinates — re-embedding would discard signal
- The analysis goal is a quantitative reconstruction-accuracy check against true counts rather than visual/batch-effect assessment, where metric computation on highly variable genes is preferred

## Inputs

- Raw scRNA-Seq count matrix (CSV, gene x cell, loaded with pd.read_csv(..., index_col=0), e.g., Baron.csv, Villani GSE80171)
- Imputed/denoised expression matrix from AutoClassImpute or control methods (DCA, SAVER, scImpute, MAGIC)
- Per-cell labels: batch and cell-type annotations

## Outputs

- 2-D t-SNE embedding matrix (n_cells x 2) per dataset and method
- KMeans cluster labels (K=8,9,10, random_state=1) computed on each t-SNE embedding
- t-SNE scatter plots colored by batch and cell type (batch_effect_tsne.png)

## How to apply

Load the raw count matrix with pandas (index_col=0), normalize it with take_norm, then reduce to 50 principal components with sklearn PCA(n_components=50) before running TSNE(n_components=2) — the PCA pre-step follows the article's stated pipeline (PCA_raw = PCA(n_components=50).fit_transform(X_norm); tsne_raw = TSNE(n_components=2).fit_transform(PCA_raw)). Apply the identical procedure to imputed matrices (e.g., AutoClassImpute output with dropout_rate=0.5) so raw-versus-imputed embeddings are directly comparable. On each 2-D embedding, run KMeans(n_clusters=K, random_state=1) for K=8,9,10 to obtain cluster labels for downstream metrics. Use the resulting embeddings to build plots colored by batch and cell type (batch_effect_tsne.png) and to judge whether cell types remain separated while batches merge within types.

## Related tools

- **AutoClass** (Provides AutoClassImpute-imputed matrices whose t-SNE embeddings are compared against raw and control-method embeddings) — https://github.com/datapplab/AutoClass
- **sklearn** (Supplies PCA(n_components=50), TSNE(n_components=2), and KMeans(random_state=1) used to build and cluster the embeddings)
- **pandas** (Loads the scRNA-Seq count matrices with index_col=0 prior to normalization and embedding)
- **matplotlib** (Renders the t-SNE scatter plots colored by batch and cell type)
- **numpy** (Converts the DataFrame to an array (X = expr.values) for the normalization/PCA/TSNE pipeline)

## Examples

```
X_norm = take_norm(X); PCA_raw = PCA(n_components=50).fit_transform(X_norm); tsne_raw = TSNE(n_components=2).fit_transform(PCA_raw)
```

## Evaluation signals

- Embedding pipeline follows the paper order: take_norm -> PCA(n_components=50) -> TSNE(n_components=2) on every method's matrix, ensuring raw-vs-imputed comparability
- KMeans on each embedding uses random_state=1 with K=8,9,10 so cluster labels are reproducible
- Villani check: cell types remain well separated while the two batches are evenly mixed within each cell type after AutoClass imputation
- Baron check: batch effect is visibly reduced relative to the raw-data t-SNE baseline
- Cluster labels from the embedding yield higher ASW/ARI/NMI/purity on top-200 find_hv_genes genes than raw and control methods

## Limitations

- t-SNE embeddings are stochastic; the workflow mitigates this only for KMeans (random_state=1), not for the TSNE step itself
- The article's discussion contains no limitation statements regarding embedding or visualization choices
- Interpretation of batch mixing is visual on the embedding and should be backed by ASW metrics per the validation criteria

## Evidence

- [results] PCA to 50 components followed by t-SNE to 2 dimensions: "PCA_raw = PCA(n_components=50).fit_transform(X_norm)
tsne_raw = TSNE(n_components=2).fit_transform(PCA_raw)"
- [results] KMeans clustering on t-SNE embedding for cluster quality metrics: "kmeans = KMeans(n_clusters = K,random_state=1).fit(tsne)"
- [results] Villani batch-mixing finding: [evidence span withheld — anchor status: not_found]
- [readme] Installation and dependency context: "AutoClass runs with Python 3, and you need to have TensorFlow (>=2.0) and a few other pacakges installed"
