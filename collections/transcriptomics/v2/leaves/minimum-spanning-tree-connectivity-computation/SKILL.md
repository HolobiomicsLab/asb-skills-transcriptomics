---
name: minimum-spanning-tree-connectivity-computation
description: Use when a set of per-clustering MSTs (from Slingshot/Totem CLARA clustering)
  exists and you need a per-cell connectivity score in [0,1] to identify trajectory
  leaves and branch points.
when_to_use_negative:
- No per-clustering MSTs or clustering results are available (the computation operates
  on Totem/Slingshot clustering outputs, not raw counts or embeddings)
- The intended trajectory is disconnected or contains cycles — Totem cannot generate
  disconnected trajectories or trajectories that have cycles
- The dataset is extremely imbalanced (rare plus prevalent cell types) so k-medoids
  clustering fails upstream and produces unreliable MSTs requiring manual cluster
  merging
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Totem
  role: Hosts the connectivity estimation module; generates the CLARA clusterings
    whose MSTs feed the connectivity computation and uses the scores to rank clustering
    results
  repo: https://github.com/elolab/Totem
- name: Slingshot
  role: Constructs the Minimum Spanning Tree for each clustering, the direct input
    to the per-vertex connectivity computation
- name: SingleCellExperiment
  role: Container format for the prepared input data used in Totem analysis
- name: R
  role: Implementation language of the Totem package and the connectivity module
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:8063d99f52b98e9efddd222ea2a3db9ef78088227a732b42b37aaea03424e5ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elolab/Totem
  tools:
  - R
  - SingleCellExperiment
  - Slingshot
  - Totem
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Minimum spanning tree cell connectivity computation

## Summary

Computes per-cell connectivity in Totem by, for each clustering's Minimum Spanning Tree, aggregating inverse-edge-weight contributions normalized by vertex degree, scaling each per-clustering vector to max 1, and averaging over all L' clusterings to rank cells along a tree-shaped trajectory.

## When to use

Use when a set of per-clustering MSTs (from Slingshot/Totem CLARA clustering) exists and you need a per-cell connectivity score in [0,1] to identify trajectory leaves and branch points.

## When NOT to use

- No per-clustering MSTs or clustering results are available (the computation operates on Totem/Slingshot clustering outputs, not raw counts or embeddings)
- The intended trajectory is disconnected or contains cycles — Totem cannot generate disconnected trajectories or trajectories that have cycles
- The dataset is extremely imbalanced (rare plus prevalent cell types) so k-medoids clustering fails upstream and produces unreliable MSTs requiring manual cluster merging

## Inputs

- Set of Minimum Spanning Trees (one per CLARA clustering, constructed via Slingshot)
- Totem clustering results (k-medoids CLARA over a range of k values, filtered by min.cluster.size)
- SingleCellExperiment object prepared with PrepareTotem

## Outputs

- Per-cell connectivity vector with values bounded in [0,1] (arithmetic mean over L' scaled per-clustering connectivity vectors)

## How to apply

Take as input the set of Minimum Spanning Trees built with Slingshot, one per Totem k-medoids (CLARA) clustering. For each MST, compute for every ordered pair of vertices the connectivity contribution as the inverse edge weight dij divided by the degree kj of the starting vertex, then sum per-vertex contributions to obtain one connectivity vector per clustering. Scale each per-clustering connectivity vector to a maximum of 1 so values are bounded in [0,1]. Average the L' scaled vectors with the arithmetic mean (equation 1) to produce final per-cell connectivity. These scores feed Totem's ranking (alongside the Calinski-Harabasz score) and let users visually pinpoint likely end points (leaves) and branch points of the trajectory in the 2D MDS embedding.

## Related tools

- **Totem** (Hosts the connectivity estimation module; generates the CLARA clusterings whose MSTs feed the connectivity computation and uses the scores to rank clustering results) — https://github.com/elolab/Totem
- **Slingshot** (Constructs the Minimum Spanning Tree for each clustering, the direct input to the per-vertex connectivity computation)
- **SingleCellExperiment** (Container format for the prepared input data used in Totem analysis)
- **R** (Implementation language of the Totem package and the connectivity module)

## Examples

```
remotes::install_github("elolab/Totem")
```

## Evaluation signals

- Returned per-cell connectivity values are all bounded in [0,1], with each per-clustering vector's maximum scaled to exactly 1
- The function executes without error on the Totem example data (e.g. the simulated binary_tree_1 dataset)
- High-connectivity cells fall at expected trajectory leaves in the simulated binary_tree_1 dataset
- The final values equal the arithmetic mean over the L' scaled per-clustering connectivity vectors (equation 1)
- Connectivity scores, combined with the Calinski-Harabasz score, produce a sensible ranking of clustering results

## Limitations

- Connectivity depends on upstream clustering quality; k-medoids (CLARA) may not work well with high-dimensional PCA input (e.g. 30 principal components), producing 1-2 cell clusters
- k-medoids may fail on datasets with extremely rare and extremely prevalent cell types simultaneously, degrading MSTs and the connectivity scores
- Totem cannot generate disconnected trajectories or trajectories that have cycles, so connectivity is only meaningful for tree-shaped (linear, bifurcating, multifurcating) topologies
- dynplot will not generate correct pseudotime for multi-root (converging) trajectories, limiting downstream use of the connectivity-selected trajectory

## Evidence

- [abstract] Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells.: "Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells."
- [intro] Totem generates a large number of clustering results with a k-medoids algorithm (CLARA) and constructs an MST for each clustering.: "`Totem` generates a large number of clustering results with a *k*-medoids algorithm (CLARA) and constructs an MST for each clustering."
- [readme] Totem uses Slingshot to find linear curves through the lineages of the tree.: [evidence span withheld — anchor status: external_doc]
