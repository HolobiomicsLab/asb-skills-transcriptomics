---
name: single-cell-clustering-benchmark-against-known-cell-types
description: Use when a clustering (or cell-state) assignment on scRNA-seq data must
  be validated against an external reference cell-type labeling, e.g. marker-derived
  labels for PBMCs, using ARI/NMI/purity metrics.
when_to_use_negative:
- No external reference cell-type labels exist for the dataset, so ARI/NMI/purity
  cannot be computed against known types
- The clustering and the reference labels are defined over different sets of cells
  and cannot be aligned cell-by-cell
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: ACTIONet (Python package)
  role: Provides the network-based clusters being validated and the marker-loading/annotation
    functions (an.tl.load_markers, an.tl.annotate_cells_using_markers) that produce
    reference labels
  repo: https://github.com/shmohammadi86/ACTIONet
- name: scanpy
  role: Reads the 10x count matrix (sc.read_10x_h5('pbmc_10k_v3.h5')) and performs
    normalize_total/log1p preprocessing upstream of clustering
- name: Python
  role: Language in which the ARI/NMI/purity computations and clustering_metrics.csv
    export are implemented
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-18416-6
    title: A multiresolution framework to characterize single-cell state landscapes
  source_content_hash: sha256:ccf699b92da1153a687f31950f2b5b64303f2b7a74e3d6c815858dac8154482d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shmohammadi86/ACTIONet
  tools:
  - ACTIONet
  - ACTIONet (Python package)
  - Harmony
  - Python
  - Scanorama
  - Seurat
  - numpy
  - scanpy
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- single-cell-clustering-benchmarking
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# single-cell-clustering-benchmark-against-known-cell-types

## Summary

External validation of a single-cell clustering result by scoring cluster assignments against known cell-type labels with adjusted Rand index (ARI), normalized mutual information (NMI), and cluster purity, as done when comparing ACTIONet's network-based PBMC clusters to Seurat's HVG+PCA pipeline.

## When to use

Use when a clustering (or cell-state) assignment on scRNA-seq data must be validated against an external reference cell-type labeling, e.g. marker-derived labels for PBMCs, using ARI/NMI/purity metrics.

## When NOT to use

- No external reference cell-type labels exist for the dataset, so ARI/NMI/purity cannot be computed against known types
- The clustering and the reference labels are defined over different sets of cells and cannot be aligned cell-by-cell

## Inputs

- Per-cell cluster labels from the method under test (e.g. ACTIONet network-based clusters on pbmc_10k_v3.h5 data)
- Reference per-cell cell-type labels, e.g. marker-derived annotations from PBMC_Monaco2019_12celltypes
- Optionally, a second clustering result (e.g. Seurat HVG+PCA clusters) for comparison

## Outputs

- clustering_metrics.csv containing per-cell labels and ARI, NMI, and cluster purity scores
- Cluster-vs-cell-type contingency/purity table (visualized as a purity plot)

## How to apply

Obtain per-cell cluster labels from the clustering method under test (e.g. ACTIONet's network-based clusters from an.run_ACTIONet) and an independent reference labeling of the same cells (e.g. marker-derived labels from an.tl.load_markers('PBMC_Monaco2019_12celltypes') followed by an.tl.annotate_cells_using_markers). Compute ARI and NMI between the two label vectors and, for each cluster, its purity as the fraction of cells sharing the dominant reference cell type (cluster purity), aggregating over clusters. In the ACTIONet PBMC benchmark, correct recovery appears as a near-perfect diagonal in the cluster-vs-cell-type purity plot, with ARI and NMI close to 1 and purity approaching 1, whereas the Seurat HVG+PCA baseline merges cell types. Write the per-cell labels plus the three metric scores to clustering_metrics.csv so the result can be checked by file-existence and script-execution tests.

## Related tools

- **ACTIONet (Python package)** (Provides the network-based clusters being validated and the marker-loading/annotation functions (an.tl.load_markers, an.tl.annotate_cells_using_markers) that produce reference labels) — https://github.com/shmohammadi86/ACTIONet
- **scanpy** (Reads the 10x count matrix (sc.read_10x_h5('pbmc_10k_v3.h5')) and performs normalize_total/log1p preprocessing upstream of clustering)
- **Python** (Language in which the ARI/NMI/purity computations and clustering_metrics.csv export are implemented)

## Examples

```
metrics = compute_clustering_metrics(actionet_clusters, marker_labels); pd.DataFrame({'cell': adata.obs_names, 'cluster': actionet_clusters, 'cell_type': marker_labels, 'ARI': [metrics['ari']]*adata.n_obs, 'NMI': [metrics['nmi']]*adata.n_obs, 'purity': [metrics['purity']]*adata.n_obs}).to_csv('clustering_metrics.csv')
```

## Evaluation signals

- ARI and NMI between clusters and marker-derived cell-type labels are close to their maximum value of 1
- Cluster purity approaches 1, with a near-perfect diagonal in the cluster-vs-cell-type purity plot (reproducing Fig. 4b-d behavior)
- clustering_metrics.csv exists and the script that generates it runs to completion (file_exists and script_runs checks)
- The competing Seurat HVG+PCA baseline shows cell-type merging, i.e. markedly lower ARI/NMI/purity than ACTIONet clusters

## Limitations

- The article's PBMC dataset (14,311 cells, ref. 26) and the cell-sorted bulk PBMC validation set (ref. 27) are cited only by reference numbers without accessions or download links, so reproducing the exact reference labels depends on external resources
- No software/tool versions are stated for ACTIONet, R, Python, UMAP, or Leiden implementations, which can affect reproducibility of the benchmark
- Metrics assume a one-to-one-ish mapping between clusters and reference types; methods capturing continuous cell states may not map cleanly onto discrete reference labels

## Evidence

- [methods] Markers, directions, names = an.tl.load_markers('PBMC_Monaco2019_12celltypes'): "markers, directions, names = an.tl.load_markers('PBMC_Monaco2019_12celltypes')"
- [methods] adata = sc.read_10x_h5('pbmc_10k_v3.h5'): "adata = sc.read_10x_h5('pbmc_10k_v3.h5')
adata.var_names_make_unique(join='.')"
- [methods] filter_adata parameters: "an.pp.filter_adata(adata, min_cells_per_feature=0.01, min_features_per_cell=1000)"
- [abstract] Superior performance claim: "Using multiple synthetic and real data sets, we demonstrate ACTIONet's superior performance relative to existing alternatives"
- [readme] Multiresolution network description: "The result is a network, the ACTIONet, in which cells are nodes positioned by their archetypal composition, with a matching low-dimensional embedding"
