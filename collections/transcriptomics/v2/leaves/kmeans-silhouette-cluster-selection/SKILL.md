---
name: kmeans-silhouette-cluster-selection
description: Use when you have a coefficient matrix of HVGs from CPPLS regression
  on spatial transcriptomics data and need to group genes with similar neighboring-cell-type
  regulation patterns prior to building a MIMO communication graph.
when_to_use_negative:
- The coefficient matrix still contains HVGs with coefficients equal to 0 — clustering
  before zero filtering will distort cluster structure
- Your goal is clustering expression profiles or cell types, not grouping HVGs by
  their neighboring-cell-type regulation coefficients from CPPLS modeling
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: cluster
  role: R package providing k-means clustering and silhouette computation used to
    cluster filtered HVG coefficients
- name: CPPLS-MLP
  role: Pipeline stage producing the filtered HVG coefficient matrix that is clustered;
    clustering step is invoked after coefficient filtering
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: networkx
  role: 'Downstream consumption: clustered gene groups are aggregated into the MIMO
    directed graph'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae198
    title: 'CPPLS-MLP: a method for constructing cell–cell communication networks
      and identifying related highly variable genes based on single-cell sequencing
      and spatial transcriptomics data'
  source_content_hash: sha256:ef0b527c41de892e24fe45e772a36ca435b96fc6c0c293ebe19b105c8037ab59
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wuzhenao/CPPLS-MLP
  tools:
  - CPPLS-MLP
  - R
  - Seurat
  - circlize
  - cluster
  - networkx
  - pls
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# kmeans-silhouette-cluster-selection

## Summary

Clusters filtered HVG regression coefficients (after zero-coefficient removal) with k-means, selecting the number of clusters k by the Silhouette method over a 2–15 range, as done in CPPLS-MLP to group genes by cell-communication coefficient patterns before MIMO graph aggregation.

## When to use

Use when you have a coefficient matrix of HVGs from CPPLS regression on spatial transcriptomics data and need to group genes with similar neighboring-cell-type regulation patterns prior to building a MIMO communication graph.

## When NOT to use

- The coefficient matrix still contains HVGs with coefficients equal to 0 — clustering before zero filtering will distort cluster structure
- Your goal is clustering expression profiles or cell types, not grouping HVGs by their neighboring-cell-type regulation coefficients from CPPLS modeling

## Inputs

- Filtered HVG coefficient matrix from CPPLS regression modeling (after two-step t-test/BH filtering at delta = 0.05)
- Vector of HVG coefficients with zero values removed

## Outputs

- Cluster assignment of each retained HVG for the optimal k (k in 2–15)
- Per-k silhouette scores used to select the optimal cluster count
- Gene-cluster groupings used to standardize counts across cell types in the MIMO graph

## How to apply

First filter out all HVGs whose CPPLS regression coefficients equal 0, since zero-coefficient genes carry no communication signal. Then apply k-means to the remaining filtered HVG coefficients, evaluating candidate k from 2 to 15 with the Silhouette method and retaining the k with the best average silhouette width. The resulting gene clusters are used to aggregate bipartite graphs into the MIMO graph with path attenuation (10^(1-i) decay per intermediate cell type), so cluster labels feed directly into network construction rather than being a terminal output.

## Related tools

- **cluster** (R package providing k-means clustering and silhouette computation used to cluster filtered HVG coefficients)
- **CPPLS-MLP** (Pipeline stage producing the filtered HVG coefficient matrix that is clustered; clustering step is invoked after coefficient filtering) — https://github.com/wuzhenao/CPPLS-MLP
- **networkx** (Downstream consumption: clustered gene groups are aggregated into the MIMO directed graph)

## Examples

```
library(cluster); km <- kmeans(filtered_hvg_coefs, centers = k); sil <- silhouette(km$cluster, dist(filtered_hvg_coefs))
```

## Evaluation signals

- Zero-coefficient HVGs are absent from the clustering input matrix (all coefficients equal to 0 filtered out first)
- Candidate k values tested span exactly 2 to 15 and the selected k maximizes average silhouette width
- Every retained HVG receives exactly one cluster label (partition is complete and non-overlapping)
- Clustered groups can be propagated into the MIMO graph so that gene-cluster counts are standardized across cell types
- In the Seq-Scope pipeline, the downstream edge-list contains edges connecting B cell (immature/IgA) and fibroblast cell types, indicating clustering did not discard the relevant gene groups

## Limitations

- The Silhouette method is only searched over k = 2 to 15; a natural cluster count outside this range would be missed
- Clustering operates on CPPLS-filtered coefficients, so genes removed by the two-step t-test/BH filtering (delta = 0.05) never reach this stage
- No automated justification is given in the paper for the 2–15 range or for silhouette over alternative indices (e.g. gap statistic)

## Evidence

- [methods] CPPLS-MLP employs the k-means method to cluster the filtered coefficients of HVGs [38]. The optimal number of clusters k is determined using the Silhouette method: "CPPLS-MLP employs the k-means method to cluster the filtered coefficients of HVGs [38]. The optimal number of clusters k is determined using the Silhouette method"
- [methods] Before clustering, all the HVGs with coefficients equal to 0 are filtered out.: "Before clustering, all the HVGs with coefficients equal to 0 are filtered out."
- [other] Filter out HVGs with zero coefficients, cluster the remaining HVG coefficients with k-means (k selected from 2 to 15 via the Silhouette method), aggregate the bipartite graphs into the MIMO graph with path attenuation (10^(1-i) decay per intermediate cell), and standardize gene cluster counts across cell types.: [evidence span withheld — anchor status: not_found]
- [readme] install.packages(c("cluster", "circlize", "dplyr", "pls", "purrr", "Seurat", "stringr"), dependencies = TRUE): "install.packages(c("cluster", "circlize", "dplyr", "pls", "purrr", "Seurat", "stringr"), dependencies = TRUE)"
