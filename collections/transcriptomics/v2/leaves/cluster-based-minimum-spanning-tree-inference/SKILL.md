---
name: cluster-based-minimum-spanning-tree-inference
description: Use when you have clustering assignments for single-cell data plus a
  low-dimensional cell embedding and need to build the tree-shaped trajectory backbone
  (MST over clusters) for topology estimation, as in Totem's cell-connectivity-guided
  trajectory inference.
when_to_use_negative:
- The dataset is expected to contain a disconnected trajectory or trajectories with
  cycles — Totem (and hence this MST-backbone construction) cannot generate those
- You need correct pseudotime for multi-root (converging) trajectories — dynplot will
  not generate the correct pseudotime and a manual work-around is required
- The clustering assignments contain extremely small clusters (below min.cluster.size,
  default 5 cells); such clusterings should be filtered out before MST construction
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Totem
  role: Hosts the GenerateMST function that builds an MST for each generated clustering
    and stores the topology with the clustering (mst_topology.rds)
  repo: https://github.com/elolab/Totem
- name: Slingshot
  role: Provides the covariance-based Mahalanobis-like between-cluster distance used
    to weight cluster vertices, and constructs the MST as the trajectory backbone
- name: ape
  role: R package used to estimate the minimum spanning tree over the cluster-vertex
    distance matrix inside GenerateMST
- name: SingleCellExperiment
  role: Container from which the clustering assignments and low-dimensional cell embedding
    are loaded before MST construction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:be03101a193a134410d0e26fbeea534fdb912def7e6682768da6198d89be27b5
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# cluster-based-minimum-spanning-tree-inference

## Summary

Reconstructs the per-clustering minimum spanning tree (MST) module of Totem: each cluster becomes a graph vertex at its centroid, and pairwise between-cluster distances are computed with Slingshot's covariance-based Mahalanobis-like distance before an MST is estimated over the cluster-vertex distance matrix.

## When to use

Use when you have clustering assignments for single-cell data plus a low-dimensional cell embedding and need to build the tree-shaped trajectory backbone (MST over clusters) for topology estimation, as in Totem's cell-connectivity-guided trajectory inference.

## When NOT to use

- The dataset is expected to contain a disconnected trajectory or trajectories with cycles — Totem (and hence this MST-backbone construction) cannot generate those
- You need correct pseudotime for multi-root (converging) trajectories — dynplot will not generate the correct pseudotime and a manual work-around is required
- The clustering assignments contain extremely small clusters (below min.cluster.size, default 5 cells); such clusterings should be filtered out before MST construction

## Inputs

- Clustering assignments for a selected clustering (e.g. from k-medoids/CLARA clustering over a range of k values)
- Low-dimensional cell embedding (e.g. 5-dimensional LMDS embedding from dyndimred) stored in a SingleCellExperiment object prepared with PrepareTotem
- Pairwise between-cluster distance matrix (covariance-based Mahalanobis-like, Slingshot style)

## Outputs

- mst_topology.rds — the MST topology (edges connecting cluster vertices) stored together with the clustering
- A connected acyclic tree spanning all clusters, used as the trajectory backbone for Slingshot lineage curves and cell-connectivity scoring

## How to apply

Load the clustering assignments for one selected clustering and the low-dimensional cell embedding from the SingleCellExperiment object, and represent each cluster as a graph vertex placed at the cluster centroid. Compute all pairwise between-cluster distances using Slingshot's covariance-based Mahalanobis-like distance, which accounts for the covariance (shape and spread) of the cells within each cluster rather than using Euclidean centroid distances. Estimate the minimum spanning tree over this cluster-vertex distance matrix using the ape R package, as done inside Totem's GenerateMST function. Store the resulting MST topology (edges connecting cluster vertices) together with the clustering in mst_topology.rds; Totem performs this step after selecting clusterings by Calinski-Harabasz score, cell connectivity, and ARI to a prior clustering, and Slingshot then uses the MST backbone to find linear curves through the tree lineages.

## Related tools

- **Totem** (Hosts the GenerateMST function that builds an MST for each generated clustering and stores the topology with the clustering (mst_topology.rds)) — https://github.com/elolab/Totem
- **Slingshot** (Provides the covariance-based Mahalanobis-like between-cluster distance used to weight cluster vertices, and constructs the MST as the trajectory backbone)
- **ape** (R package used to estimate the minimum spanning tree over the cluster-vertex distance matrix inside GenerateMST)
- **SingleCellExperiment** (Container from which the clustering assignments and low-dimensional cell embedding are loaded before MST construction)

## Examples

```
Rscript -e 'library(Totem); mst <- Totem::GenerateMST(sce, clustering_id="selected_clustering"); saveRDS(mst, "mst_topology.rds")'
```

## Evaluation signals

- Output is a connected acyclic tree spanning all clusters: it must have exactly n_clusters - 1 edges and no cycles
- MST topology is persisted together with its clustering in mst_topology.rds
- Between-cluster distances reflect Slingshot's covariance-based Mahalanobis-like distance (accounting for within-cluster covariance), not plain Euclidean centroid distances
- MST is computed with the ape R package, matching Totem's GenerateMST implementation
- Downstream, Slingshot can find linear curves through the lineages of the inferred tree, and cell connectivity can pinpoint leaves (end points) and branch points

## Limitations

- Totem cannot generate disconnected trajectories or trajectories that have cycles, so the per-clustering MST backbone presumes a connected tree topology
- k-medoids instability with high-dimensional PCA input (e.g. 30 principal components) can produce 1-2 cell clusters, distorting the cluster vertices feeding the MST
- On extremely imbalanced datasets (rare plus prevalent cell types), k-medoids may fail and require manual cluster merging before reliable MST construction
- dynplot will not generate correct pseudotime for multi-root trajectories produced from the MST backbone

## Evidence

- [abstract] Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells: "Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells"
- [intro] Slingshot MST backbone: "In trajectory inference methods like `Slingshot`[2] that require a clustering to construct the Minimum Spanning Tree (`MST`) as the backbone of the trajectory"
- [intro] Select clusterings then build MSTs with Slingshot: "After clustering we need to select one or several clusterings that are used to construct the Minimum Spanning Trees (MSTs) with Slingshot."
- [readme] Slingshot finds curves through tree lineages: [evidence span withheld — anchor status: external_doc]
