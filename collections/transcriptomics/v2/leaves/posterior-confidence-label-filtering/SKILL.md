---
name: posterior-confidence-label-filtering
description: Use when FFC cluster labels and pval_list are available for single-cell
  data (PBMC CITE-seq, multiome, MCA) and you want to quantify ARI/purity gains from
  restricting evaluation to high-confidence (PEP < 0.1) labels.
when_to_use_negative:
- The data contains only one cluster — every point receives a single label type, so
  Monte Carlo validation cannot estimate meaningful uncertainty.
- Cluster labels come from a method other than FFC's validate()/pval() pipeline, so
  no per-cell posterior significance values exist to threshold at 0.1.
- The raw count matrix has not been embedded/dimension-reduced first — FFC is sensitive
  to high dimensionality and should be applied after nonlinear dimensionality reduction
  or PCA, not to raw features.
edam_operation: http://edamontology.org/operation_2995
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: forest-fire-clustering
  role: Provides the FFC object whose validate()/pval() calls produce pval_list (posterior
    significance values) used as the PEP < 0.1 filtering criterion
  repo: https://github.com/gersteinlab/forest-fire-clustering
- name: scanpy
  role: Preprocesses RNA count matrices with the zheng17 recipe, computes neighbors/PCA
    embeddings, and supplies reference label workflows against which filtered clusters
    are evaluated
- name: scikit-learn (sklearn)
  role: Computes adjusted Rand index and purity metrics for pre- and post-filtering
    cluster labels
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-31107-8
    title: Forest Fire Clustering for single-cell sequencing combines iterative label
      propagation with parallelized Monte Carlo simulations
  source_content_hash: sha256:d1510036601fadcc54769eaa9961d4d4b5b8ac209c2764cc920d6dee9e992ec3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gersteinlab/forest-fire-clustering
  tools:
  - Python
  - forest-fire-clustering
  - numba
  - numpy
  - pegasus
  - scanpy
  - scikit-learn (sklearn)
  - scipy
  edam_operation: http://edamontology.org/operation_2995
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# posterior-confidence-label-filtering

## Summary

Filters single-cell cluster assignments to high-confidence cells using Forest Fire Clustering's non-parametric posterior significance values (PEP < 0.1), which the paper shows improves clustering ARI (e.g., >20% gain on PBMC benchmarks; 0.38→0.72 on the Mouse Cell Atlas).

## When to use

Use when FFC cluster labels and pval_list are available for single-cell data (PBMC CITE-seq, multiome, MCA) and you want to quantify ARI/purity gains from restricting evaluation to high-confidence (PEP < 0.1) labels.

## When NOT to use

- The data contains only one cluster — every point receives a single label type, so Monte Carlo validation cannot estimate meaningful uncertainty.
- Cluster labels come from a method other than FFC's validate()/pval() pipeline, so no per-cell posterior significance values exist to threshold at 0.1.
- The raw count matrix has not been embedded/dimension-reduced first — FFC is sensitive to high dimensionality and should be applied after nonlinear dimensionality reduction or PCA, not to raw features.

## Inputs

- Fitted FFC cluster object after validate()/entropy()/pval(), exposing cluster_labels and pval_list
- [sample x feature] embedding matrix preprocessed with scanpy zheng17 recipe and first 30 principal components (top 10,000 HVP ATAC peaks for multiome; top 5000 HVGs for MCA)
- Reference cell-type labels (CITE-seq surface-protein marker labels, multiome marker-gene labels, or MCA tissue-of-origin labels)

## Outputs

- Subset of cluster_labels restricted to cells with pval_list (PEP) < 0.1
- Pre- vs post-filtering adjusted Rand index and purity scores per method/dataset
- Result tables: ari_purity_benchmark.csv (PBMC) or mca_benchmark_results.csv (MCA)

## How to apply

After fitting Forest Fire Clustering (FFC(), cluster_obj.preprocess(X) on a [sample x feature] matrix, cluster_obj.fit(fire_temp=100)), run the internal validation by calling cluster_obj.validate(), then cluster_obj.entropy() and cluster_obj.pval() to populate cluster_obj.pval_list with per-cell posterior significance values. Retain only cells whose pval_list entry is below the 0.1 cutoff, and recompute evaluation metrics (adjusted Rand index and purity via scikit-learn) against reference labels — CITE-seq cell types from 10x Genomics surface-protein markers, multiome marker-gene labels, or MCA tissue-of-origin labels — on this filtered subset. Compare pre- and post-filtering ARI to quantify the confidence-filtering gain; the paper's rationale is that the Monte Carlo internal validation (re-propagating seed labels across trials) yields a non-parametric posterior probability per cell, so labels with PEP >= 0.1 are less trustworthy. Write the unfiltered and PEP<0.1-filtered ARI/purity per method and dataset into ari_purity_benchmark.csv (PBMC) or mca_benchmark_results.csv (MCA). Success is reproducing the reported pattern: PBMC ARI improves >20% after filtering, and MCA ARI rises from ~0.38 to ~0.72.

## Related tools

- **forest-fire-clustering** (Provides the FFC object whose validate()/pval() calls produce pval_list (posterior significance values) used as the PEP < 0.1 filtering criterion) — https://github.com/gersteinlab/forest-fire-clustering
- **scanpy** (Preprocesses RNA count matrices with the zheng17 recipe, computes neighbors/PCA embeddings, and supplies reference label workflows against which filtered clusters are evaluated)
- **scikit-learn (sklearn)** (Computes adjusted Rand index and purity metrics for pre- and post-filtering cluster labels)

## Examples

```
cluster_obj.validate(); cluster_obj.pval(); import numpy as np; labels_hc = np.array(cluster_obj.cluster_labels)[np.array(cluster_obj.pval_list) < 0.1]
```

## Evaluation signals

- PBMC benchmark: post-filter ARI on the PEP<0.1 subset exceeds the unfiltered ARI by more than 20% relative to Louvain/Leiden/PARC baselines.
- MCA benchmark: FFC ARI improves from approximately 0.38 to approximately 0.72 after filtering to pval_list < 0.1.
- All retained cells in the filtered subset have pval_list entries strictly below 0.1; entropy_list/pval_list lengths equal the number of cells.
- Pre- and post-filtering ARI/purity are computed with the same reference labels (e.g., MCA tissue-of-origin) and same metric (adjusted Rand index) so gains are attributable to filtering alone.
- Result CSVs (ari_purity_benchmark.csv, mca_benchmark_results.csv) contain per-method, per-dataset unfiltered and filtered ARI/purity values.

## Limitations

- The number of Monte Carlo trials varies with the number of clusters, so posterior estimates are not fixed across runs with different cluster counts.
- A large number of Monte Carlo trials can be computationally intensive (though parallelizable).
- Forest Fire Clustering is limited by the curse of dimensionality and should be performed with nonlinear dimensionality reduction (e.g., UMAP, t-SNE, PHATE).
- When there is only one cluster, every point receives one label type, so Monte Carlo internal validation cannot accurately estimate uncertainty.

## Evidence

- [readme] README pval_list is the posterior significance source for the 0.1 cutoff: "cluster_obj.pval_list # list of posterior significance values"
- [methods] Validation API enabling the filtering step: "from forest_fire_clustering.forest_fire_clustering import FFC
cluster_obj = FFC()
cluster_obj.preprocess(X)"
- [abstract] Posterior probability per cell is the rationale for confidence filtering: "calculates a non-parametric posterior probability that each cell is assigned a cell-type label"
- [intro] Internal validation is a core motivation of the method: "A single-cell clustering method should have the ability to internally validate its clustering results."
- [discussion] Cost caveat for the Monte Carlo validation underlying pval_list: "A large number of Monte Carlo trials can be computationally intensive, but the computation can be efficiently parallelized"
