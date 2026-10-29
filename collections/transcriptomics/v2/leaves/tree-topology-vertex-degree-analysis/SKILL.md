---
name: tree-topology-vertex-degree-analysis
description: Use when you have a set of clusterings from Totem's CLARA k-medoids step,
  each converted to an MST via Slingshot, and need per-cell connectivity values to
  rank clusterings or identify trajectory leaves and branch points.
when_to_use_negative:
- The data or goal involves trajectories with cycles or disconnected trajectory components
  — Totem cannot generate disconnected trajectories or trajectories that have cycles
- The goal is correct pseudotime for multi-root (converging) trajectories — dynplot
  will not generate the correct pseudotime for multi-root trajectories and a manual
  work-around is required
- You only have a single clustering and need trajectory construction, not connectivity
  estimation across multiple clusterings
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: Totem
  role: R package whose clustering-selection step computes cell connectivity from
    the set of MSTs generated for its clustering results
  repo: https://github.com/elolab/Totem
- name: Slingshot
  role: Constructs the Minimum Spanning Trees (MSTs) that serve as the input topologies
    for per-vertex connectivity computation
- name: SingleCellExperiment
  role: Container for the single-cell data prepared with PrepareTotem, providing the
    cells whose connectivity is estimated
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:f3ad2fe26c790b1da818177829b1c495591615770877b75ff9a9d9cc6392fb38
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
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# tree-topology-vertex-degree-analysis

## Summary

Quantifies cell connectivity from a set of Minimum Spanning Trees (MSTs), one per clustering, as implemented in the Totem R package: per-vertex connectivity is derived from inverse edge weights and vertex degrees, scaled per graph to a maximum of 1, and averaged over all L' clusterings.

## When to use

Use when you have a set of clusterings from Totem's CLARA k-medoids step, each converted to an MST via Slingshot, and need per-cell connectivity values to rank clusterings or identify trajectory leaves and branch points.

## When NOT to use

- The data or goal involves trajectories with cycles or disconnected trajectory components — Totem cannot generate disconnected trajectories or trajectories that have cycles
- The goal is correct pseudotime for multi-root (converging) trajectories — dynplot will not generate the correct pseudotime for multi-root trajectories and a manual work-around is required
- You only have a single clustering and need trajectory construction, not connectivity estimation across multiple clusterings

## Inputs

- Set of Minimum Spanning Trees (one per CLARA k-medoids clustering, constructed with Slingshot)
- Prepared SingleCellExperiment object with cells mapped to the MST vertices (e.g. via PrepareTotem on data like the simulated binary_tree_1 dataset)

## Outputs

- One per-cell connectivity vector per clustering (scaled to maximum 1)
- Final per-cell connectivity values (arithmetic mean over all L' scaled vectors, bounded in [0,1])

## How to apply

Take as input the set of Minimum Spanning Trees (one per clustering, constructed with Slingshot in the upstream Totem step). For each MST, compute for every ordered pair of vertices the connectivity contribution as the inverse edge weight dij divided by the degree kj of the starting vertex, and sum the per-vertex contributions to obtain each cell's connectivity within that MST, yielding one connectivity vector per clustering. Scale each per-clustering connectivity vector to a maximum value of 1. Average the L' scaled connectivity vectors over all clusterings with the arithmetic mean (equation 1) to obtain the final per-cell connectivity values. This connectivity feeds Totem's ranking of clusterings alongside the Calinski-Harabasz score and ARI-to-prior-clustering, and lets users visually see which parts of the embedding are likely end points (leaves) and branch points of the trajectory.

## Related tools

- **Totem** (R package whose clustering-selection step computes cell connectivity from the set of MSTs generated for its clustering results) — https://github.com/elolab/Totem
- **Slingshot** (Constructs the Minimum Spanning Trees (MSTs) that serve as the input topologies for per-vertex connectivity computation)
- **SingleCellExperiment** (Container for the single-cell data prepared with PrepareTotem, providing the cells whose connectivity is estimated)

## Examples

```
remotes::install_github("elolab/Totem")
```

## Evaluation signals

- The function must return per-cell connectivity values bounded in [0,1]
- Executes without error on the Totem example data (e.g. the simulated binary_tree_1 dataset)
- High-connectivity cells fall at expected trajectory leaves in the embedding visualization
- Each per-clustering connectivity vector, before averaging, attains a maximum value of exactly 1 after scaling
- The final values equal the arithmetic mean over all L' scaled per-clustering connectivity vectors (equation 1)

## Limitations

- Totem cannot generate disconnected trajectories or trajectories that have cycles
- dynplot will not generate the correct pseudotime for multi-root trajectories; a manual work-around is required
- The k-medoids algorithm may not work well with PCA if the number of principal components is high (e.g. 30), which can destabilize the clusterings feeding the MSTs
- If the dataset includes extremely rare cell types and extremely prevalent cell types at the same time, k-medoids might not work properly, degrading connectivity estimates

## Evidence

- [abstract] Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells.: "Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells."
- [intro] Totem generates a large number of clustering results with a k-medoids algorithm (CLARA) and constructs an MST for each clustering.: "`Totem` generates a large number of clustering results with a *k*-medoids algorithm (CLARA) and constructs an MST for each clustering."
- [intro] The cell connectivity allows users to visually see which parts of the embedding are likely to be end points (leaves) and branch points of the trajectory: "The cell connectivity allows users to visually see which parts of the embedding are likely to be end points (leaves) and branch points of the trajectory"
- [readme] Totem uses Slingshot to find linear curves through the lineages of the tree.: [evidence span withheld — anchor status: external_doc]
- [discussion] the k-medoids algorithm may not work well with PCA if the number principal components is high (e.g. 30): "the k-medoids algorithm may not work well with PCA if the number principal components is high (e.g. 30)"
