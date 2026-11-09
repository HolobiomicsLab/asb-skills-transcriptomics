---
name: mahalanobis-like-between-cluster-distance-computation
description: Use when you have cell-to-cluster assignments plus a low-dimensional
  SingleCellExperiment embedding and need between-cluster distances that account for
  within-cluster covariance, to build an MST cluster topology.
when_to_use_negative:
- The clustering contains clusters smaller than the min.cluster.size threshold (default
  5 cells) — such clusterings are filtered out before MST construction because covariance
  estimates on tiny clusters are unreliable
- You need trajectories with cycles or disconnected components — Totem's MST backbone
  cannot generate disconnected trajectories or trajectories that have cycles
- The embedding is high-dimensional PCA (e.g. 30 principal components) where k-medoids
  clustering itself is unstable, producing 1-2 cell clusters that make between-cluster
  distance/MST construction meaningless
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: Totem
  role: Encapsulates the MST generation per clustering via its GenerateMST function
    and stores the topology with the clustering
  repo: https://github.com/elolab/Totem
- name: Slingshot
  role: Provides the covariance-based Mahalanobis-like between-cluster distance used
    in the MST-backbone construction
- name: ape
  role: R package used to estimate the minimum spanning tree over the cluster-vertex
    distance matrix
- name: SingleCellExperiment
  role: Container for the clustering assignments and low-dimensional cell embedding
    fed into MST construction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:95816252144d8d05e9ba5b888e3bfbbc266965e31a33c61764176d4ab150fd20
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elolab/Totem
  tools:
  - R
  - SingleCellExperiment
  - Slingshot
  - Totem
  - ape
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# mahalanobis-like-between-cluster-distance-computation

## Summary

Computes pairwise distances between cluster centroids in a low-dimensional cell embedding using Slingshot's covariance-based Mahalanobis-like distance, which accounts for the shape and spread of cells within each cluster, and uses these distances to construct a minimum spanning tree (MST) over cluster vertices — the backbone of Totem's topology estimation for each clustering.

## When to use

Use when you have cell-to-cluster assignments plus a low-dimensional SingleCellExperiment embedding and need between-cluster distances that account for within-cluster covariance, to build an MST cluster topology.

## When NOT to use

- The clustering contains clusters smaller than the min.cluster.size threshold (default 5 cells) — such clusterings are filtered out before MST construction because covariance estimates on tiny clusters are unreliable
- You need trajectories with cycles or disconnected components — Totem's MST backbone cannot generate disconnected trajectories or trajectories that have cycles
- The embedding is high-dimensional PCA (e.g. 30 principal components) where k-medoids clustering itself is unstable, producing 1-2 cell clusters that make between-cluster distance/MST construction meaningless

## Inputs

- Clustering assignments for one clustering (e.g. from CLARA k-medoids over a range of k values, run ~10k times by default in Totem)
- Low-dimensional cell embedding stored in a SingleCellExperiment object (e.g. 5-dimensional LMDS from dyndimred)
- Cluster labels with a minimum cluster size threshold applied (min.cluster.size, default 5 cells per cluster)

## Outputs

- Pairwise between-cluster distance matrix (covariance-based Mahalanobis-like distances)
- MST topology: a connected acyclic tree over cluster vertices with n_clusters - 1 edges, stored with the clustering (e.g. mst_topology.rds)

## How to apply

Load the clustering assignments for one selected clustering and the low-dimensional cell embedding from the SingleCellExperiment object. Represent each cluster as a graph vertex placed at the cluster centroid. Compute all pairwise between-cluster distances using Slingshot's covariance-based Mahalanobis-like distance, which accounts for the covariance (shape and spread) of cells within each cluster rather than using plain Euclidean centroid distances. Estimate the minimum spanning tree over the cluster-vertex distance matrix with the ape R package, as done inside Totem's GenerateMST function. Store the resulting MST topology (edges connecting cluster vertices) together with the clustering, e.g. as mst_topology.rds; Totem repeats this for each of its many CLARA k-medoids clusterings before ranking them by Calinski-Harabasz score and cell connectivity.

## Related tools

- **Totem** (Encapsulates the MST generation per clustering via its GenerateMST function and stores the topology with the clustering) — https://github.com/elolab/Totem
- **Slingshot** (Provides the covariance-based Mahalanobis-like between-cluster distance used in the MST-backbone construction)
- **ape** (R package used to estimate the minimum spanning tree over the cluster-vertex distance matrix)
- **SingleCellExperiment** (Container for the clustering assignments and low-dimensional cell embedding fed into MST construction)

## Examples

```
R: mst <- ape::mst(D); # D = pairwise Slingshot covariance-based Mahalanobis-like distances between cluster centroids, via Totem::GenerateMST
```

## Evaluation signals

- The output MST is connected and acyclic, spanning all clusters with exactly n_clusters - 1 edges
- Distances are covariance-based (Mahalanobis-like, Slingshot style), not plain Euclidean centroid distances — check that within-cluster covariance influences edge selection for elongated/spread clusters
- The tree spans all clusters in the clustering; no cluster vertex is left unconnected
- The MST topology stored with the clustering (mst_topology.rds) matches the Slingshot MST-backbone construction used to estimate clustering topologies in Totem
- Per-cluster covariance matrices are estimable from the cells in each cluster (clusters exceed the min.cluster.size default of 5 cells)

## Limitations

- Totem cannot generate disconnected trajectories or trajectories that have cycles, since the MST backbone is a single connected acyclic tree
- k-medoids may not work well with high-dimensional PCA input (e.g. 30 principal components), producing degenerate 1-2 cell clusters that degrade the cluster graph
- On extremely imbalanced datasets (extremely rare plus extremely prevalent cell types), k-medoids may not work properly, requiring manual cluster merging before topology estimation
- MST construction depends on finding a good clustering that produces a biologically meaningful trajectory with the correct milestone connections

## Evidence

- [other] Mahalanobis-like covariance-based between-cluster distance underlies MST construction: [evidence span withheld — anchor status: not_found]
- [intro] MST per clustering in Totem pipeline: "`Totem` generates a large number of clustering results with a *k*-medoids algorithm (CLARA) and constructs an MST for each clustering."
- [intro] Slingshot MST backbone for clusterings: "In trajectory inference methods like `Slingshot`[2] that require a clustering to construct the Minimum Spanning Tree (`MST`) as the backbone of the trajectory"
- [intro] Clusterings filtered by minimum cluster size before MST: "The second parameter (`min.cluster.size`) determines the clusterings that are filtered out because they include too small clusters."
- [readme] README: Totem uses Slingshot for tree lineages: [evidence span withheld — anchor status: external_doc]
