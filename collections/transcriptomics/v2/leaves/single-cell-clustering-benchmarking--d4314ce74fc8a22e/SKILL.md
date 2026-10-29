---
name: single-cell-clustering-benchmarking
description: Use when clustering single-cell RNA-seq/multiome data (~10,000-cell PBMC
  or large-scale MCA) and you need to compare cluster quality (ARI, purity) against
  baselines or quantify label-confidence filtering gains.
when_to_use_negative:
- Data contain only a single cluster — Monte Carlo internal validation cannot accurately
  estimate uncertainty when every point receives one label
- Clustering very high-dimensional raw data without nonlinear dimensionality reduction
  (e.g., UMAP, t-SNE, PHATE), since the method is sensitive to high dimensionality
- Datasets where the goal is not comparative clustering evaluation — the skill targets
  benchmarking against state-of-the-art methods, not standalone one-off clustering
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: forest-fire-clustering
  role: 'Primary clustering method: FFC().preprocess(X), fit(fire_temp=100), validate(),
    pval() to produce labels and posterior significance values'
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: scanpy
  role: Preprocessing (zheng17 recipe), KNN affinity matrix via sc.pp.neighbors, and
    rank_genes_groups differential expression
- name: pegasus
  role: State-of-the-art baseline clustering via 'pg.infer_cell_types' with the human_immune
    marker gene set
- name: scikit-learn (sklearn)
  role: Adjusted Rand index and clustering baseline computation on shared embeddings
- name: numpy
  role: Linear algebra operations in the benchmark pipeline
- name: scipy
  role: Sparse matrix acceleration for affinity computations
- name: numba
  role: Numerical acceleration of the Monte Carlo simulations
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:ea6bf91d998789f9819eab79d5a9d81ab2d83a924177ec54912a7835060fbef3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gersteinlab/forest-fire-clustering
  tools:
  - FIt-SNE
  - Miscell
  - Python
  - SCANPY
  - Scanpy
  - Seurat
  - anndata
  - forest-fire-clustering
  - numba
  - numpy
  - pandas
  - pegasus
  - scMGCA
  - scVI
  - scanpy
  - scikit-learn
  - scikit-learn (sklearn)
  - scipy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# single-cell-clustering-benchmarking

## Summary

Benchmark Forest Fire Clustering (FFC) against state-of-the-art single-cell clustering baselines (Louvain, Leiden, PARC, pegasus) using ARI and purity scores on PBMC and Mouse Cell Atlas datasets, and quantify ARI gains from filtering to high-confidence labels (PEP < 0.1).

## When to use

Use when clustering single-cell RNA-seq/multiome data (~10,000-cell PBMC or large-scale MCA) and you need to compare cluster quality (ARI, purity) against baselines or quantify label-confidence filtering gains.

## When NOT to use

- Data contain only a single cluster — Monte Carlo internal validation cannot accurately estimate uncertainty when every point receives one label
- Clustering very high-dimensional raw data without nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE), since the method is sensitive to high dimensionality
- Datasets where the goal is not comparative clustering evaluation — the skill targets benchmarking against state-of-the-art methods, not standalone one-off clustering

## Inputs

- Gene expression count matrix (sample x feature) for PBMC CITE-seq or multiome datasets
- MCA gene expression count matrix
- Reference cell-type labels (CITE-seq surface-protein marker labels, MCA tissue-of-origin labels)
- PCA embeddings (first 30 PCs)

## Outputs

- Method x dataset ARI and purity benchmark table (ari_purity_benchmark.csv)
- Pre- vs post-PEP<0.1-filtering ARI comparison and runtime/memory metrics (mca_benchmark_results.csv)
- cluster_obj.cluster_labels, entropy_list, and pval_list per cell

## How to apply

Preprocess count matrices with scanpy's 'zheng17' recipe, keeping the first 30 principal components; for multiome, select the top 10,000 highly variable ATAC peaks, and for MCA the top 5000 highly variable genes. Run FFC via the forest-fire-clustering package (FFC().preprocess(X) on the [sample x feature] matrix, then fit(fire_temp=100)), then run Louvain, Leiden, PARC, or pegasus 'pg.infer_cell_types' baselines on the same embeddings. Derive reference labels (CITE-seq cell types from 10x Genomics marker genes, MCA tissue-of-origin labels) and compute adjusted Rand index and purity against them. Use cluster_obj.validate() and cluster_obj.pval() to obtain pval_list, retain cells with posterior significance PEP < 0.1, and recompute ARI on this subset. Write per-method and pre-/post-filter ARI, purity, runtime, and peak-memory metrics to a benchmark CSV (e.g., ari_purity_benchmark.csv, mca_benchmark_results.csv).

## Related tools

- **forest-fire-clustering** (Primary clustering method: FFC().preprocess(X), fit(fire_temp=100), validate(), pval() to produce labels and posterior significance values) — https://github.com/gersteinlab/forest-fire-clustering
- **scanpy** (Preprocessing (zheng17 recipe), KNN affinity matrix via sc.pp.neighbors, and rank_genes_groups differential expression)
- **pegasus** (State-of-the-art baseline clustering via 'pg.infer_cell_types' with the human_immune marker gene set)
- **scikit-learn (sklearn)** (Adjusted Rand index and clustering baseline computation on shared embeddings)
- **numpy** (Linear algebra operations in the benchmark pipeline)
- **scipy** (Sparse matrix acceleration for affinity computations)
- **numba** (Numerical acceleration of the Monte Carlo simulations)

## Examples

```
from forest_fire_clustering.forest_fire_clustering import FFC; cluster_obj = FFC(); cluster_obj.preprocess(X); cluster_obj.fit(fire_temp=100); cluster_obj.validate(); cluster_obj.pval()
```

## Evaluation signals

- PBMC PEP<0.1 filtering should improve cluster ARI by more than 20% relative to existing clustering methods
- On MCA, FFC's ARI should improve from approximately 0.38 to approximately 0.72 when restricted to PEP<0.1 high-confidence labels
- FFC runtime and peak memory should be competitive with (faster than) state-of-the-art baselines while producing similar-quality clusters
- Output CSV schema must include per-method ARI, purity, runtime, and memory, plus pre-/post-filtering ARI comparisons
- fire_temp=100 must be used consistently across FFC runs, with the same first-30-PC embeddings shared by all baselines

## Limitations

- The fire temperature c strongly influences the size and number of discovered clusters (resolution-like sensitivity)
- Sensitive to the curse of dimensionality; should be performed with nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE)
- Monte Carlo internal validation cannot accurately estimate uncertainty when there is only one cluster in the data
- The number of Monte Carlo trials varies with the number of clusters, and large trial counts are computationally intensive although parallelizable
- Distance matrices for datasets over 1 million points are computationally expensive and must be approximated via KNN

## Evidence

- [results] PEP filter ARI gain: [evidence span withheld — anchor status: not_found]
- [readme] FFC usage snippet: "cluster_obj = FFC()
cluster_obj.preprocess(X)
cluster_obj.fit(fire_temp=100)
cluster_obj.cluster_labels"
- [readme] pval validation API: "cluster_obj.validate()
cluster_obj.entropy()
cluster_obj.pval()"
- [discussion] PCA filter step: "After normalization and standardization, the first 30 principal components were used for clustering."
- [discussion] MCA HVG filter: "The gene expression count matrix for MCA was also preprocessed using the “zheng17” recipe in the scanpy package"
- [discussion] evaluation metrics: "There are three main ways to evaluate the performance of unsupervised methods. First, purity scores evaluate the cluster labels based on a set of reference labels."
- [discussion] fire temperature sensitivity: "The fire temperature c plays an important role in clustering and strongly influences the size and number of the clusters discovered"
