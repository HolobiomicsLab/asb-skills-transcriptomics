---
name: single-cell-trajectory-backbone-construction
description: Use when you have a clustering of single cells (from CLARA k-medoids)
  plus a low-dimensional cell embedding and need an acyclic tree-shaped trajectory
  topology connecting the clusters.
when_to_use_negative:
- The desired trajectory is disconnected or contains cycles — Totem cannot generate
  disconnected trajectories or trajectories that have cycles, since the MST backbone
  is a connected acyclic tree
- The trajectory has multiple roots (converging topology) and correct pseudotime is
  required — dynplot will not generate correct pseudotime for multi-root trajectories
  and a manual work-around is needed
- Clustering input comes from a high-dimensional PCA embedding (e.g. 30 principal
  components) — k-medoids may produce 1-2 cell clusters, destabilizing the cluster
  vertices before MST construction
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Totem
  role: R package whose GenerateMST function builds the per-clustering MST backbone
    over cluster vertices; open-source at elolab/Totem
  repo: https://github.com/elolab/Totem
- name: Slingshot
  role: Provides the covariance-based Mahalanobis-like between-cluster distance used
    to build the MST and the linear-curve construction along lineages of the tree
- name: ape
  role: Estimates the minimum spanning tree over the cluster-vertex distance matrix
    inside GenerateMST
- name: SingleCellExperiment
  role: Container holding the clustering assignments and low-dimensional cell embedding
    fed into MST construction
- name: dyndimred
  role: Generates the low-dimensional embedding (landmark MDS, 5 dimensions) on which
    clusters are defined and the MST is built
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:ed37c36b72d23aafda562d60aa1a6d448291d126c6e784fdad707e765a78585f
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
  - dyndimred
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# single-cell-trajectory-backbone-construction

## Summary

Constructs the per-clustering trajectory backbone as a minimum spanning tree over cluster vertices (placed at cluster centroids), using Slingshot's covariance-based Mahalanobis-like between-cluster distances and the ape R package, as Totem does for each of its many CLARA k-medoids clustering results.

## When to use

Use when you have a clustering of single cells (from CLARA k-medoids) plus a low-dimensional cell embedding and need an acyclic tree-shaped trajectory topology connecting the clusters.

## When NOT to use

- The desired trajectory is disconnected or contains cycles — Totem cannot generate disconnected trajectories or trajectories that have cycles, since the MST backbone is a connected acyclic tree
- The trajectory has multiple roots (converging topology) and correct pseudotime is required — dynplot will not generate correct pseudotime for multi-root trajectories and a manual work-around is needed
- Clustering input comes from a high-dimensional PCA embedding (e.g. 30 principal components) — k-medoids may produce 1-2 cell clusters, destabilizing the cluster vertices before MST construction

## Inputs

- Clustering assignments for one clustering (e.g. from CLARA k-medoids, filtered by min.cluster.size, default 5 cells per cluster)
- Low-dimensional cell embedding stored in a SingleCellExperiment object (e.g. 5-dimension LMDS embedding)
- Per-cluster cell covariance implied by the embedded cell coordinates

## Outputs

- MST topology: edges connecting cluster vertices, spanning all clusters
- Distance matrix of pairwise Slingshot covariance-based Mahalanobis-like between-cluster distances
- MST stored together with its clustering (e.g. mst_topology.rds)

## How to apply

Load the clustering assignments for one selected clustering and the low-dimensional cell embedding (e.g. the 5-dimensional LMDS embedding produced by dyndimred) from the SingleCellExperiment object prepared with PrepareTotem. Represent each cluster as a graph vertex at its centroid, then compute all pairwise between-cluster distances with Slingshot's covariance-based Mahalanobis-like distance, which accounts for the shape and spread of each cluster's cells. Estimate the minimum spanning tree over the resulting cluster-vertex distance matrix using the ape R package, as done inside Totem's GenerateMST function. Store the MST topology (edges between cluster vertices) together with its clustering, e.g. in mst_topology.rds. Totem repeats this for many clusterings (CLARA run up to ~10k times over a range of k) and later ranks them by the Calinski-Harabasz score and cell connectivity to select the trajectory backbone.

## Related tools

- **Totem** (R package whose GenerateMST function builds the per-clustering MST backbone over cluster vertices; open-source at elolab/Totem) — https://github.com/elolab/Totem
- **Slingshot** (Provides the covariance-based Mahalanobis-like between-cluster distance used to build the MST and the linear-curve construction along lineages of the tree)
- **ape** (Estimates the minimum spanning tree over the cluster-vertex distance matrix inside GenerateMST)
- **SingleCellExperiment** (Container holding the clustering assignments and low-dimensional cell embedding fed into MST construction)
- **dyndimred** (Generates the low-dimensional embedding (landmark MDS, 5 dimensions) on which clusters are defined and the MST is built)

## Examples

```
MST <- ape::mst(D)  # D = Slingshot covariance-based Mahalanobis-like distances between cluster centroids, as in Totem's GenerateMST
```

## Evaluation signals

- The output tree is connected and acyclic, spanning all clusters with exactly n_clusters - 1 edges
- Every tree vertex corresponds to one cluster of the selected clustering (vertex at the cluster centroid), with no isolated clusters
- Between-cluster distances match Slingshot's covariance-based Mahalanobis-like distance, i.e. they reflect the shape and spread of cells within each cluster rather than plain centroid Euclidean distances
- The MST topology is stored with the clustering (mst_topology.rds) so it can be paired with downstream cell-connectivity and Calinski-Harabasz ranking
- Repeating construction for multiple clusterings yields per-clustering trees that can be ranked by cell connectivity to pinpoint leaves (end points) and branch points

## Limitations

- Totem cannot generate disconnected trajectories or trajectories that have cycles
- dynplot will not generate the correct pseudotime for multi-root (converging) trajectories, requiring a manual work-around
- k-medoids may not work well with PCA if the number of principal components is high (e.g. 30), producing 1-2 cell clusters that distort the MST backbone
- With extremely rare and extremely prevalent cell types at the same time, k-medoids might not work properly and manual cluster merging is required

## Evidence

- [abstract] Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells.: "Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells."
- [intro] CLARA clustering and MST construction: "`Totem` generates a large number of clustering results with a *k*-medoids algorithm (CLARA) and constructs an MST for each clustering."
- [intro] MSTs built with Slingshot: "After clustering we need to select one or several clusterings that are used to construct the Minimum Spanning Trees (MSTs) with Slingshot."
- [readme] Slingshot curves through tree lineages: [evidence span withheld — anchor status: external_doc]
