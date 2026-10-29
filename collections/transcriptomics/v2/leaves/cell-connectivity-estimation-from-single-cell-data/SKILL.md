---
name: cell-connectivity-estimation-from-single-cell-data
description: Use when you have a set of minimum spanning trees (one per Totem/CLARA
  clustering, constructed with Slingshot) and need per-cell connectivity values in
  [0,1] to rank clusterings and identify likely trajectory leaves and branch points.
when_to_use_negative:
- The data contain disconnected trajectories or trajectories with cycles — Totem cannot
  generate these and the connectivity model assumes a tree backbone.
- The analysis needs correct pseudotime for multi-root (converging) trajectories —
  dynplot will not generate the correct pseudotime, requiring a manual work-around.
- There is only a single fixed clustering with no set of MSTs to average over, or
  the k-medoids clustering is unreliable (e.g. extremely rare plus extremely prevalent
  cell types at the same time).
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Totem
  role: R package that generates the many CLARA clusterings, estimates their topologies
    as MSTs, and computes the per-cell connectivity used to rank clusterings
  repo: https://github.com/elolab/Totem
- name: Slingshot
  role: Constructs the Minimum Spanning Tree for each clustering, providing the graph
    over which connectivity is computed
- name: R
  role: Implementation language for the connectivity estimation module and Totem itself
- name: SingleCellExperiment
  role: Input container format for the prepared single-cell data passed to Totem
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:9ba329699ea0c047c8fd5f934ef3b54520a5d82515f2f0341666cfefcd68e15f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elolab/Totem
  tools:
  - R
  - SingleCellExperiment
  - Slingshot
  - Totem
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cell-connectivity-estimation-from-single-cell-data

## Summary

Computes Totem's per-cell connectivity from the set of Minimum Spanning Trees built over multiple CLARA k-medoids clusterings: for each MST, per-vertex connectivity combines inverse edge weights with vertex degrees, is scaled to a maximum of 1, and the per-clustering vectors are averaged to rank clusterings and locate leaves and branch points.

## When to use

Use when you have a set of minimum spanning trees (one per Totem/CLARA clustering, constructed with Slingshot) and need per-cell connectivity values in [0,1] to rank clusterings and identify likely trajectory leaves and branch points.

## When NOT to use

- The data contain disconnected trajectories or trajectories with cycles — Totem cannot generate these and the connectivity model assumes a tree backbone.
- The analysis needs correct pseudotime for multi-root (converging) trajectories — dynplot will not generate the correct pseudotime, requiring a manual work-around.
- There is only a single fixed clustering with no set of MSTs to average over, or the k-medoids clustering is unreliable (e.g. extremely rare plus extremely prevalent cell types at the same time).

## Inputs

- Set of Minimum Spanning Trees, one per clustering (each with vertices, cell-to-cluster assignments, edge weights dij, and degrees kj)
- Prepared SingleCellExperiment object (created with PrepareTotem) on which the Totem analysis was run
- Optional: low-dimensional embedding (e.g. 5-dimension LMDS from dyndimred, 2-dimension MDS for visualization) for interpreting connectivity

## Outputs

- One per-clustering connectivity vector, scaled to maximum value 1
- Final per-cell connectivity values in [0,1] (arithmetic mean over the L' scaled vectors)
- Ranking of clusterings usable with the Calinski-Harabasz score to select an appropriate trajectory

## How to apply

Take as input the set of Minimum Spanning Trees generated for Totem's clusterings (MSTs are constructed with Slingshot in the upstream step). For each MST, compute for every ordered pair of vertices the connectivity contribution as the inverse edge weight dij divided by the degree kj of the starting vertex, and sum the per-vertex contributions to obtain one connectivity vector per clustering; equivalently, per-vertex connectivity is the vertex degree divided by the number of clusters, with weights from inverse edge lengths. Scale each per-clustering connectivity vector so its maximum equals 1. Average the L' scaled connectivity vectors with the arithmetic mean (equation 1) to obtain the final per-cell connectivity values. The result is used by Totem alongside the Calinski-Harabasz score to rank clustering results, and lets users visually see which parts of the embedding are likely end points (leaves) and branch points of the trajectory.

## Related tools

- **Totem** (R package that generates the many CLARA clusterings, estimates their topologies as MSTs, and computes the per-cell connectivity used to rank clusterings) — https://github.com/elolab/Totem
- **Slingshot** (Constructs the Minimum Spanning Tree for each clustering, providing the graph over which connectivity is computed)
- **R** (Implementation language for the connectivity estimation module and Totem itself)
- **SingleCellExperiment** (Input container format for the prepared single-cell data passed to Totem)

## Examples

```
suppressMessages(library(SingleCellExperiment)); sce <- PrepareTotem(sce)
```

## Evaluation signals

- The returned per-cell connectivity values are bounded in [0,1] for every cell.
- The function runs without error on the Totem example data (e.g. the simulated binary_tree_1 dataset).
- High-connectivity cells fall at expected trajectory leaves (end points) in the binary_tree_1 example.
- Each per-clustering connectivity vector is scaled so its maximum equals exactly 1 before averaging.
- The final output equals the arithmetic mean of the L' scaled connectivity vectors (equation 1), one value per cell.

## Limitations

- k-medoids may not work well with PCA if the number of principal components is high (e.g. 30), producing 1–2 cell clusters that distort the MSTs feeding connectivity.
- If the dataset includes extremely rare cell types and extremely prevalent cell types at the same time, k-medoids might not work properly and manual cluster merging is needed.
- Clusterings that include too small clusters (min.cluster.size, default 5 cells per cluster) are filtered out before MST construction.
- Totem cannot generate disconnected trajectories or trajectories that have cycles, bounding the scope of the connectivity measure.

## Evidence

- [abstract] Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells: "Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells"
- [intro] MSTs come from k-medoids clusterings: "`Totem` generates a large number of clustering results with a *k*-medoids algorithm (CLARA) and constructs an MST for each clustering."
- [intro] Connectivity identifies leaves and branch points: "The cell connectivity allows users to visually see which parts of the embedding are likely to be end points (leaves) and branch points of the trajectory"
- [readme] Slingshot builds the MST backbone: [evidence span withheld — anchor status: external_doc]
