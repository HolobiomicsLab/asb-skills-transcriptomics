---
name: dimensionality-reduction-visualization-of-single-cell-clusters
description: Use when SCASL (or similar splicing-landscape clustering) has produced
  an imputed AS probability matrix and cluster labels, and you need a 2D embedding
  to inspect cluster separation, coloring either by SCASL clustering results or user-supplied
  ground-truth labels.
when_to_use_negative:
- Clustering is based on overall gene expression levels rather than the AS probability
  matrix — this skill visualizes splicing-landscape clusters, and SCASL's findings
  (e.g., the TNBC C3 cluster) are 'not readily available via conventional cell clustering
  based on single-cell gene expression data'
- The junction matrix has not yet passed the sites_initial, runs_initial, sites_quality,
  and runs_quality QC filters and KNN imputation — the raw junction matrix (junc_matrix.csv)
  is not the input to clustering/visualization
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: SCASL
  role: Runs PCA, spectral clustering, and UMAP/t-SNE visualization of the imputed
    AS probability matrix, emitting cluster.png and reduce.png
  repo: https://github.com/xryanglab/SCASL
- name: umap-learn
  role: Provides the UMAP embedding when dimension_reduction_method is set to 'umap'
- name: scikit-learn
  role: Provides PCA, the default spectral clustering algorithm, and alternative agglomerative/kmeans
    clusterers
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-46480-9
    title: Interrogations of single-cell RNA splicing landscapes with SCASL define
      new cell identities with physiological relevance
  source_content_hash: sha256:31339ba014410bb83f2d36263e1c42d582b2c2f380c0145a512b21dd912db974
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xryanglab/SCASL
  tools:
  - Leafcutter
  - SCASL
  - scikit-learn
  - umap-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# dimensionality-reduction-visualization-of-single-cell-clusters

## Summary

Reduce the imputed single-cell alternative-splicing (AS) probability matrix produced by SCASL to a low-dimensional embedding and visualize SCASL-defined clusters (e.g., the precancerous C3 cluster in the 443-cell TNBC dataset) so cluster structure and cluster-to-cluster transitions can be inspected.

## When to use

Use when SCASL (or similar splicing-landscape clustering) has produced an imputed AS probability matrix and cluster labels, and you need a 2D embedding to inspect cluster separation, coloring either by SCASL clustering results or user-supplied ground-truth labels.

## When NOT to use

- Clustering is based on overall gene expression levels rather than the AS probability matrix — this skill visualizes splicing-landscape clusters, and SCASL's findings (e.g., the TNBC C3 cluster) are 'not readily available via conventional cell clustering based on single-cell gene expression data'
- The junction matrix has not yet passed the sites_initial, runs_initial, sites_quality, and runs_quality QC filters and KNN imputation — the raw junction matrix (junc_matrix.csv) is not the input to clustering/visualization

## Inputs

- Imputed AS probability matrix (normalized_matrix: rows = splice sites, columns = cells)
- SCASL cluster label file with preds (predicted AS cluster labels)
- Optional user label file (label_file with truth_column and run_column) for ground-truth coloring
- configs/srr.yaml settings: pca_dimension, num_cluster, cluster_algorithm, dimension_reduction_method ('umap' or 'tsne')

## Outputs

- cluster.png: dimensionality-reduction plot colored by SCASL clustering results
- reduce.png: dimensionality-reduction plot colored by user-provided ground-truth labels
- Final cluster label file containing preds (predicted AS cluster labels)

## How to apply

After SCASL imputes the missing values in the AS probability matrix with the iterative weighted KNN procedure (controlled by num_iteration and knn in configs/srr.yaml) and clusters cells with the default spectral algorithm (cluster_algorithm: 'spectral'; alternatives 'agglomerative' or 'kmeans') after PCA at pca_dimension with num_cluster set, generate the embedding with dimension_reduction_method ('umap' or 'tsne'). SCASL writes two visualization files: cluster.png, where cells are colored by the SCASL clustering results (preds, the predicted AS cluster labels), and reduce.png, where coloring is based on user-provided ground-truth labels. Because imputation, dimensionality reduction, and unsupervised clustering are stochastic, reruns with identical parameters may show some differences in the embedding, but cluster assignments should remain generally consistent. In the TNBC application this visualization surfaced the SCASL-defined C3 precancerous epithelial cluster that gene-expression-based clustering missed.

## Related tools

- **SCASL** (Runs PCA, spectral clustering, and UMAP/t-SNE visualization of the imputed AS probability matrix, emitting cluster.png and reduce.png) — https://github.com/xryanglab/SCASL
- **umap-learn** (Provides the UMAP embedding when dimension_reduction_method is set to 'umap')
- **scikit-learn** (Provides PCA, the default spectral clustering algorithm, and alternative agglomerative/kmeans clusterers)

## Examples

```
python main.py -y configs/srr.yaml  # with dimension_reduction_method: 'umap', cluster_algorithm: 'spectral', num_cluster and pca_dimension set in the YAML
```

## Evaluation signals

- cluster.png and reduce.png exist in the configured output_path
- The final cluster label file with preds (predicted AS cluster labels) exists and its label count matches num_cluster
- Reruns with identical parameters yield generally consistent clustering despite randomness in imputation, dimensionality reduction, and unsupervised clustering
- Visual coloring of reduce.png reproduces the user-supplied ground-truth labels while cluster.png reflects the SCASL cluster assignments
- In the 443-cell TNBC dataset the embedding separates the SCASL-defined precancerous epithelial C3 cluster (382 primary tumor + 61 normal epithelial cells) from other clusters

## Limitations

- Randomness in interpolation, dimensionality reduction, and unsupervised clustering means results may differ between runs
- The AS probability profiles are largely polarized with many NA values, so 'many of the canonical clustering methods are unsuitable' — spectral clustering after PCA is the default
- RNA splicing profiling via SMART-seq-style data is much more costly and harder to multiplex for large cell numbers than droplet-based scRNA-seq

## Evidence

- [intro] Spectral clustering of the cells based on the imputed AS probability matrix: "followed by spectral clustering of the cells"
- [discussion] Iterative weighted KNN imputation: "SCASL employs an iterative, weighted KNN process for imputation of the missing values"
- [discussion] Clustering with NA indicator matrix: "SCASL performs cell clustering with the full AS probability matrix, concatenated with a NA indicator matrix"
- [readme] cluster.png coloring by SCASL clustering: "cluster.png: This figure represents the outcome of the final dimensionality reduction visualization, where the coloring is based on the clustering results of SCASL."
- [readme] reduce.png coloring by ground truth: "reduce.png: This figure represents the outcome of the final dimensionality reduction visualization, where the coloring is based on the ground truth provided by the user."
- [methods] Run SCASL via main.py with YAML config: "python main.py -y configs/srr.yaml"
- [discussion] Canonical clustering unsuitable for polarized AS profiles: "many of the canonical clustering methods are unsuitable for processing the largely polarized AS profiles"
