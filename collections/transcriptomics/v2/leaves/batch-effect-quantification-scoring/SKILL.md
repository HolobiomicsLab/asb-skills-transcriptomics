---
name: batch-effect-quantification-scoring
description: Use when comparing imputed vs. raw scRNA-Seq matrices for batch correction
  on datasets with known batch and cell-type annotations (e.g., Villani PBMC GSE80171,
  Baron pancreatic islet) and numeric batch-merging scores are needed.
when_to_use_negative:
- Data lack both batch annotations and known cell-type labels, making ASW batch-merging
  and ARI/NMI/purity scoring undefined
- Goal is imputation itself (use AutoClassImpute/DCA/SAVER/scImpute/MAGIC directly)
  rather than scoring their batch-effect removal
- Evaluating on the simulated Splatter Dataset 1 ground-truth recovery task, where
  data-recovery correlation rather than batch metrics is the correct criterion
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: AutoClass
  role: Imputes the Villani and Baron matrices (AutoClassImpute, dropout_rate=0.5,
    pre-clustering K over [8,9,10]) to produce the embeddings being scored for batch
    mixing
  repo: https://github.com/datapplab/AutoClass
- name: DCA
  role: Imputation control run with default settings on the same input matrices to
    provide a comparison baseline in the batch-effect scoring
  repo: https://github.com/theislab/dca
- name: SAVER
  role: Imputation control; in the paper it failed to reduce batch effect, so its
    scores serve as the negative comparison
  repo: https://github.com/mohuangx/SAVER
- name: scImpute
  role: Imputation control run with defaults before embedding and metric computation
  repo: https://github.com/Vivianstats/scImpute
- name: MAGIC
  role: Imputation control run with defaults before embedding and metric computation
  repo: https://github.com/KrishnaswamyLab/MAGIC
- name: sklearn
  role: Provides PCA(n_components=50), TSNE(n_components=2), KMeans(random_state=1),
    silhouette_score (ASW), adjusted_rand_score, normalized_mutual_info_score, and
    the contingency matrix used by purity_score
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:e7c6d72d9087efa6c57f605097a19a213e4dc501174db071e7679eca555708c5
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# batch-effect-quantification-scoring

## Summary

Quantifies batch-effect removal and cell-type separation in scRNA-Seq embeddings by clustering t-SNE coordinates with KMeans (K=8,9,10, random_state=1) and scoring against known labels using average silhouette width (ASW), adjusted Rand index (ARI), normalized mutual information (NMI), and purity score. This skill operationalizes the Fig. 5 / Supplementary Figs. 4-5 evaluation from AutoClass (Li et al., Nat Commun 2022).

## When to use

Use when comparing imputed vs. raw scRNA-Seq matrices for batch correction on datasets with known batch and cell-type annotations (e.g., Villani PBMC GSE80171, Baron pancreatic islet) and numeric batch-merging scores are needed.

## When NOT to use

- Data lack both batch annotations and known cell-type labels, making ASW batch-merging and ARI/NMI/purity scoring undefined
- Goal is imputation itself (use AutoClassImpute/DCA/SAVER/scImpute/MAGIC directly) rather than scoring their batch-effect removal
- Evaluating on the simulated Splatter Dataset 1 ground-truth recovery task, where data-recovery correlation rather than batch metrics is the correct criterion

## Inputs

- Raw scRNA-Seq count matrix (genes x cells CSV, e.g., Villani PBMC GSE80171, Baron pancreatic islet), loaded with pd.read_csv(..., index_col=0)
- Imputed/denoised expression matrices from AutoClassImpute (dropout_rate=0.5) and controls DCA, SAVER, scImpute, MAGIC
- Per-cell batch labels and known cell-type labels for each dataset

## Outputs

- batch_effect_metrics.csv with ASW, ARI, NMI, and purity values per dataset, method, and K (8, 9, 10)
- batch_effect_tsne.png showing raw-vs-AutoClass-vs-control t-SNE embeddings colored by batch and cell type
- Per-K KMeans cluster labels (K=8, 9, 10, random_state=1) on each method's 2-D t-SNE embedding

## How to apply

Normalize each raw count matrix with take_norm, reduce to 50 PCA components, and embed to 2-D t-SNE for the raw baseline; repeat for each imputed matrix (AutoClassImpute with dropout_rate=0.5, plus DCA, SAVER, scImpute, MAGIC at defaults). Fit KMeans with random_state=1 at K=8, 9, 10 on each t-SNE embedding to get cluster labels. Restrict clustering evaluation to the top-200 highly variable genes found with find_hv_genes(X, top=200), then compute ASW to score batch merging within cell types (Villani) and batch-effect reduction (Baron), plus ARI, NMI, and purity score (via contingency matrix) against known cell-type labels. The decision point is comparative: AutoClass must show batch mixing within cell types (higher within-cell-type ASW than raw and controls) on Villani and reduced batch effect on Baron, per the paper's finding that SAVER failed to reduce batch effect and other methods aggravated it.

## Related tools

- **AutoClass** (Imputes the Villani and Baron matrices (AutoClassImpute, dropout_rate=0.5, pre-clustering K over [8,9,10]) to produce the embeddings being scored for batch mixing) — https://github.com/datapplab/AutoClass
- **DCA** (Imputation control run with default settings on the same input matrices to provide a comparison baseline in the batch-effect scoring) — https://github.com/theislab/dca
- **SAVER** (Imputation control; in the paper it failed to reduce batch effect, so its scores serve as the negative comparison) — https://github.com/mohuangx/SAVER
- **scImpute** (Imputation control run with defaults before embedding and metric computation) — https://github.com/Vivianstats/scImpute
- **MAGIC** (Imputation control run with defaults before embedding and metric computation) — https://github.com/KrishnaswamyLab/MAGIC
- **sklearn** (Provides PCA(n_components=50), TSNE(n_components=2), KMeans(random_state=1), silhouette_score (ASW), adjusted_rand_score, normalized_mutual_info_score, and the contingency matrix used by purity_score)

## Examples

```
kmeans = KMeans(n_clusters = K, random_state=1).fit(tsne); labels = kmeans.labels_; ari = adjusted_rand_score(y_true, labels)
```

## Evaluation signals

- On Villani, AutoClass's ASW for batch merged within cell types must exceed both the raw baseline and all controls (DCA, SAVER, scImpute, MAGIC), with cell types staying separated
- On Baron, AutoClass's batch-effect score must improve relative to the raw (pre-imputation) baseline
- KMeans is run identically across methods with K in {8, 9, 10} and random_state=1, so ARI/NMI/purity must be computed for every K and method combination in batch_effect_metrics.csv
- ARI, NMI, and purity are computed against known cell-type labels on the top-200 highly variable genes selected by find_hv_genes(X, top=200); verify the gene subset was applied before metric computation
- The t-SNE figure must show the two Villani batches evenly mixed within each cell type after AutoClass imputation, unlike SAVER's output

## Limitations

- The article reports no explicit limitations; the discussion is entirely affirmative, so failure modes must be inferred (e.g., ASW sign depends on whether batches or cell types are being merged, and mislabeled annotations corrupt ARI/NMI/purity)
- Supplementary Tables 1-3 (hyperparameter settings and simulation parameters) are referenced but not provided, so exact per-dataset hyperparameters beyond K=[8,9,10] and dropout_rate are unavailable
- No software version numbers are stated, so reproducibility of the scoring pipeline depends on pinning versions externally

## Evidence

- [abstract] Batch-effect scoring grounded in abstract claim: "AutoClass outperforms the state-of-art methods in multiple types of scRNA-Seq data analyses, including data recovery, differential expression analysis, clustering analysis, and batch effect removal"
- [results] t-SNE baseline pipeline: "PCA_raw = PCA(n_components=50).fit_transform(X_norm)
tsne_raw = TSNE(n_components=2).fit_transform(PCA_raw)"
- [results] KMeans clustering with fixed seed: "kmeans = KMeans(n_clusters = K,random_state=1).fit(tsne)"
- [results] Highly variable gene selection for evaluation: "highvar_genes = find_hv_genes(X,top=200)"
- [results] Purity score via contingency matrix: "def purity_score(y_true, y_pred):"
- [methods] Dependency stack for the scoring pipeline: "pip install numpy pandas sklearn matplotlib"
