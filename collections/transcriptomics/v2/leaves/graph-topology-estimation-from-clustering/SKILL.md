---
name: graph-topology-estimation-from-clustering
description: Use when you have a set of cluster assignments for single cells plus
  a low-dimensional embedding (e.g. LMDS with 5 dimensions) and need a tree-shaped
  trajectory backbone connecting the clusters.
when_to_use_negative:
- The desired trajectory is disconnected or contains cycles — Totem cannot generate
  disconnected trajectories or trajectories that have cycles
- You only have raw expression data with no clustering and no embedding; a clustering
  must exist before an MST can be estimated per clustering
- You need correct pseudotime for multi-root (converging) trajectories — dynplot will
  not generate the correct pseudotime for multi-root trajectories
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Totem
  role: Provides the GenerateMST function that builds the MST over cluster vertices
    for each of its many CLARA clusterings
  repo: https://github.com/elolab/Totem
- name: Slingshot
  role: Supplies the covariance-based, Mahalanobis-like between-cluster distance used
    as MST edge weights, and constructs linear curves through the tree lineages
- name: ape
  role: R package used to estimate the minimum spanning tree from the cluster-vertex
    distance matrix inside GenerateMST
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:57d4eaa604cb5c91708095b87b71ac4ef31ab203069164207db6f5a938732c47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elolab/Totem
  tools:
  - R
  - Slingshot
  - Totem
  - ape
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# graph-topology-estimation-from-clustering

## Summary

Reconstructs a minimum spanning tree (MST) over cluster vertices — as done in Totem's GenerateMST — where each cluster from a k-medoids (CLARA) clustering becomes a vertex at its centroid and edge weights are Slingshot's covariance-based, Mahalanobis-like between-cluster distances. This topology backbone underlies Totem's cell-connectivity-guided trajectory inference.

## When to use

Use when you have a set of cluster assignments for single cells plus a low-dimensional embedding (e.g. LMDS with 5 dimensions) and need a tree-shaped trajectory backbone connecting the clusters.

## When NOT to use

- The desired trajectory is disconnected or contains cycles — Totem cannot generate disconnected trajectories or trajectories that have cycles
- You only have raw expression data with no clustering and no embedding; a clustering must exist before an MST can be estimated per clustering
- You need correct pseudotime for multi-root (converging) trajectories — dynplot will not generate the correct pseudotime for multi-root trajectories

## Inputs

- SingleCellExperiment object containing clustering assignments (e.g. from CLARA k-medoids) and a low-dimensional cell embedding (LMDS, 5 dimensions)
- Pairwise between-cluster distance matrix computed with Slingshot's covariance-based Mahalanobis-like distance

## Outputs

- MST topology (edges connecting cluster vertices) stored with the clustering, e.g. mst_topology.rds
- Connected acyclic tree with n_clusters - 1 edges spanning all clusters

## How to apply

Load the clustering assignments for one selected clustering and the low-dimensional cell embedding from the SingleCellExperiment object prepared with PrepareTotem. Represent each cluster as a graph vertex placed at the cluster centroid, and compute all pairwise between-cluster distances using Slingshot's covariance-based, Mahalanobis-like distance, which accounts for the covariance (shape and spread) of the cells within each cluster. Estimate the minimum spanning tree over this cluster-vertex distance matrix using the ape R package, as done inside Totem's GenerateMST function. Store the resulting MST topology (edges connecting cluster vertices) together with the clustering, e.g. as mst_topology.rds. Totem repeats this per clustering across its many CLARA clusterings (default 10k runs) to estimate topologies for ranking by Calinski-Harabasz score and cell connectivity.

## Related tools

- **Totem** (Provides the GenerateMST function that builds the MST over cluster vertices for each of its many CLARA clusterings) — https://github.com/elolab/Totem
- **Slingshot** (Supplies the covariance-based, Mahalanobis-like between-cluster distance used as MST edge weights, and constructs linear curves through the tree lineages)
- **ape** (R package used to estimate the minimum spanning tree from the cluster-vertex distance matrix inside GenerateMST)

## Examples

```
MST_edges <- ape::mst(D)  # D = pairwise between-cluster Mahalanobis-like distance matrix from Slingshot, as in Totem::GenerateMST
```

## Evaluation signals

- The output is a connected acyclic tree spanning all clusters
- The tree has exactly n_clusters - 1 edges
- Vertices correspond one-to-one to clusters, positioned at cluster centroids
- Edge weights are covariance-based Mahalanobis-like distances (Slingshot-style), not plain Euclidean centroid distances
- The topology matches Slingshot's MST-backbone construction used to estimate clustering topologies in Totem

## Limitations

- Totem cannot generate disconnected trajectories or trajectories that have cycles
- dynplot will not generate the correct pseudotime for multi-root (converging) trajectories
- The k-medoids algorithm may not work well with PCA if the number of principal components is high (e.g. 30), which can destabilize the clusterings feeding the MST
- If the dataset includes extremely rare and extremely prevalent cell types at the same time, k-medoids might not work properly, requiring manual cluster merging

## Evidence

- [abstract] Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells.: "Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells."
- [intro] MST backbone from clustering: "In trajectory inference methods like `Slingshot`[2] that require a clustering to construct the Minimum Spanning Tree (`MST`) as the backbone of the trajectory"
- [intro] MST per clustering from CLARA: "`Totem` generates a large number of clustering results with a *k*-medoids algorithm (CLARA) and constructs an MST for each clustering."
- [readme] Slingshot finds curves through tree lineages: [evidence span withheld — anchor status: external_doc]
- [discussion] No cycles or disconnected trajectories: "`Totem` cannot generate disconnected trajectories or trajectories that have cycles."
