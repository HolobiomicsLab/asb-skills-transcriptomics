---
name: r-function-implementation
description: 'Use when reimplementing or verifying Totem''s cell-connectivity calculation:
  you have multiple Slingshot-constructed MSTs (one per clustering) plus edge weights,
  and need per-cell connectivity values for trajectory ranking and leaf/branch-point
  identification.'
when_to_use_negative:
- The target trajectory is disconnected or contains cycles — Totem cannot generate
  disconnected trajectories or trajectories that have cycles
- The trajectory is multi-root (converging) and correct pseudotime is required — dynplot
  will not generate the correct pseudotime for multi-root trajectories
- The dataset pairs extremely rare with extremely prevalent cell types — k-medoids
  may fail and manual cluster merging is needed before connectivity is meaningful
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: Totem
  role: Source package whose clustering-to-MST pipeline supplies the MSTs and whose
    connectivity ranking consumes the function's output
  repo: https://github.com/elolab/Totem
- name: Slingshot
  role: Constructs the Minimum Spanning Tree for each clustering that the connectivity
    function ingests
- name: R
  role: Implementation language for the connectivity function
- name: SingleCellExperiment
  role: Container for the prepared expression data feeding the CLARA clustering /
    MST upstream step
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:6bf76a471a94fc6c850ddc8273dec1fda073608d68d3c08698422cc7bf3f0de2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elolab/Totem
  tools:
  - R
  - SingleCellExperiment
  - Slingshot
  - Totem
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# r-function-implementation

## Summary

Reconstructs Totem's cell-connectivity estimation module as an R function: it takes the set of minimum spanning trees (one per CLARA clustering, built with Slingshot) and returns a per-cell connectivity score in [0,1] used to rank clusterings and locate trajectory leaves and branch points.

## When to use

Use when reimplementing or verifying Totem's cell-connectivity calculation: you have multiple Slingshot-constructed MSTs (one per clustering) plus edge weights, and need per-cell connectivity values for trajectory ranking and leaf/branch-point identification.

## When NOT to use

- The target trajectory is disconnected or contains cycles — Totem cannot generate disconnected trajectories or trajectories that have cycles
- The trajectory is multi-root (converging) and correct pseudotime is required — dynplot will not generate the correct pseudotime for multi-root trajectories
- The dataset pairs extremely rare with extremely prevalent cell types — k-medoids may fail and manual cluster merging is needed before connectivity is meaningful

## Inputs

- Set of Minimum Spanning Trees (one per CLARA clustering, constructed with Slingshot)
- Per-edge weights d_ij of each MST
- Per-vertex degrees k_j of each MST
- SingleCellExperiment object prepared with PrepareTotem (e.g. the simulated binary_tree_1 dataset)
- L' selected clusterings from the CLARA clustering step

## Outputs

- One connectivity vector per clustering (L' vectors), each scaled to maximum 1
- Final per-cell connectivity values as the arithmetic mean over the L' scaled vectors, bounded in [0,1]

## How to apply

Take as input the set of Minimum Spanning Trees produced upstream (one per CLARA k-medoids clustering, constructed with Slingshot). For each MST, compute the per-vertex connectivity contribution for every ordered pair of vertices as the inverse edge weight d_ij divided by the degree k_j of the starting vertex, then sum contributions per vertex to get one connectivity vector per clustering. Scale each per-clustering vector so its maximum is exactly 1. Average the L' scaled vectors with the arithmetic mean (equation 1) to obtain the final per-cell connectivity values. This connectivity feeds Totem's ranking of clusterings (together with the Calinski-Harabasz score) and lets users visually identify likely end points (leaves) and branch points of the trajectory in the 2D embedding.

## Related tools

- **Totem** (Source package whose clustering-to-MST pipeline supplies the MSTs and whose connectivity ranking consumes the function's output) — https://github.com/elolab/Totem
- **Slingshot** (Constructs the Minimum Spanning Tree for each clustering that the connectivity function ingests)
- **R** (Implementation language for the connectivity function)
- **SingleCellExperiment** (Container for the prepared expression data feeding the CLARA clustering / MST upstream step)

## Examples

```
# per-MST inverse-weight/degree connectivity, then max-scale and average
scaled <- lapply(mst_conns, function(v) v / max(v)); conn <- rowMeans(do.call(cbind, scaled))
```

## Evaluation signals

- Function returns per-cell connectivity values bounded in [0,1]
- Each per-clustering connectivity vector is scaled so its maximum equals exactly 1
- Final values equal the arithmetic mean over the L' scaled per-clustering vectors (equation 1)
- Execution without error on the Totem example data (simulated binary_tree_1 dataset)
- High-connectivity cells fall at expected trajectory leaves in the 2D embedding

## Limitations

- k-medoids may not work well with PCA if the number of principal components is high (e.g. 30), producing 1-2 cell clusters and destabilizing the downstream MSTs
- k-medoids might not work properly when extremely rare and extremely prevalent cell types co-occur, requiring manual cluster merging
- Totem cannot generate disconnected trajectories or trajectories with cycles, bounding the applicability of connectivity-based ranking
- dynplot will not generate correct pseudotime for multi-root (converging) trajectories, requiring manual work-arounds

## Evidence

- [abstract] Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells.: "Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells."
- [intro] connectivity for visual interpretation: "The cell connectivity allows users to visually see which parts of the embedding are likely to be end points (leaves) and branch points of the trajectory"
- [intro] connectivity in clustering ranking: "`Totem` uses the `Calinski-Harabasz score` and the cell connectivity to rank the clustering results."
- [readme] Slingshot builds MSTs: [evidence span withheld — anchor status: external_doc]
