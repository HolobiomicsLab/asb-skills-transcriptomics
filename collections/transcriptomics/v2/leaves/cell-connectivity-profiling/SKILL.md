---
name: cell-connectivity-profiling
description: Use when the input is a single-cell expression dataset (SingleCellExperiment)
  expected to follow a linear, bifurcating, or multifurcating tree trajectory, and
  you need to rank candidate clusterings and locate trajectory leaves and branch points
  without a prior clustering.
when_to_use_negative:
- The expected trajectory is disconnected or contains cycles — Totem cannot generate
  disconnected trajectories or trajectories that have cycles
- The trajectory has multiple roots (converging topology) — dynplot will not generate
  the correct pseudotime for multi-root trajectories without manual work-around
- The dataset mixes extremely rare and extremely prevalent cell types, where k-medoids
  may fail and manual cluster merging is required
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: Totem
  role: Implements the clustering generation, cell-connectivity measurement, and clustering-ranking
    (Calinski-Harabasz) workflow over SingleCellExperiment input
  repo: https://github.com/elolab/Totem
- name: Slingshot
  role: Constructs the Minimum Spanning Tree backbone and linear curves through the
    lineages of the tree for each candidate clustering
- name: dyndimred
  role: Provides landmark MDS (LMDS) embedding with 5 dimensions used as clustering
    input; faster than MDS for large datasets
- name: SingleCellExperiment
  role: Container format for the input expression data prepared via PrepareTotem
- name: BiocManager
  role: Installs Totem from Bioconductor
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:7a9923f98a022678e4e6614c274ce73c1522810eb9b1438329ffca412deb5087
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elolab/Totem
  tools:
  - BiocManager
  - R
  - SingleCellExperiment
  - Slingshot
  - TinGa
  - Totem
  - cowplot
  - dyndimred
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# cell-connectivity-profiling

## Summary

Profile single-cell trajectory structure by generating many k-medoids (CLARA) clusterings, building an MST for each with Slingshot, and aggregating them into per-cell connectivity scores that flag likely end points (leaves) and branch points of a tree-shaped trajectory.

## When to use

Use when the input is a single-cell expression dataset (SingleCellExperiment) expected to follow a linear, bifurcating, or multifurcating tree trajectory, and you need to rank candidate clusterings and locate trajectory leaves and branch points without a prior clustering.

## When NOT to use

- The expected trajectory is disconnected or contains cycles — Totem cannot generate disconnected trajectories or trajectories that have cycles
- The trajectory has multiple roots (converging topology) — dynplot will not generate the correct pseudotime for multi-root trajectories without manual work-around
- The dataset mixes extremely rare and extremely prevalent cell types, where k-medoids may fail and manual cluster merging is required

## Inputs

- Single-cell expression dataset as a SingleCellExperiment object (e.g. dynbenchmark datasets from Zenodo doi:10.5281/zenodo.1443566 such as multifurcating_4 or the Han et al. 2018 mouse thymus T cell data)
- Optional prior clustering (cell label vector) to direct clustering search via ARI similarity
- Parameters: number of LMDS dimensions (5), number of CLARA runs (default 10k), min.cluster.size (default 5)

## Outputs

- Per-cell connectivity profile identifying likely end points (leaves) and branching points of the trajectory
- Selected clustering and its trajectory topology as a milestone network edge list (e.g. milestone_networks.csv, connectivity_profiles.csv)
- Ranked clustering candidates scored by Calinski-Harabasz score and cell connectivity

## How to apply

Wrap the expression data in a SingleCellExperiment and prepare it with PrepareTotem, then embed it in 5 dimensions using landmark MDS (LMDS) from the dyndimred package (MDS only for the 2D visualization, since 3D is unsupported). Run the k-medoids (CLARA) algorithm a large number of times (default 10k) over a range of k values, filter out clusterings containing clusters smaller than min.cluster.size (default 5 cells), and rank the surviving clusterings with the Calinski-Harabasz score and the cell-connectivity measure (optionally ARI to a user-provided prior clustering). Construct the MST topology for the top clustering via Slingshot, which finds linear curves through the lineages of the tree. The resulting per-cell connectivity profile tells you which parts of the embedding are likely end points (leaves) and branch points, guiding both automatic trajectory selection and visual topology inspection.

## Related tools

- **Totem** (Implements the clustering generation, cell-connectivity measurement, and clustering-ranking (Calinski-Harabasz) workflow over SingleCellExperiment input) — https://github.com/elolab/Totem
- **Slingshot** (Constructs the Minimum Spanning Tree backbone and linear curves through the lineages of the tree for each candidate clustering)
- **dyndimred** (Provides landmark MDS (LMDS) embedding with 5 dimensions used as clustering input; faster than MDS for large datasets)
- **SingleCellExperiment** (Container format for the input expression data prepared via PrepareTotem)
- **BiocManager** (Installs Totem from Bioconductor)

## Examples

```
remotes::install_github("dynverse/dynfeature"); remotes::install_github("dynverse/dynplot"); remotes::install_github("elolab/Totem")
```

## Evaluation signals

- Predicted milestone network correlates well with the true (ground-truth) milestone network, as on the multifurcating_4 simulation and the mouse thymus T cell data where Totem outperformed Slingshot and TinGa (Fig. 4)
- Cell connectivity accurately locates the end points and the middle branching point of the trajectory
- Clusterings containing clusters smaller than min.cluster.size (default 5 cells) are removed before ranking
- Top clustering is selected by the Calinski-Harabasz score combined with cell connectivity, not by a single criterion
- MST topologies are reproducible when CLARA is re-run with the default 10k repetitions over the range of k

## Limitations

- Totem cannot generate disconnected trajectories or trajectories that have cycles
- dynplot will not generate the correct pseudotime for multi-root (converging) trajectories; a manual work-around is required
- k-medoids may not work well with high-dimensional PCA input (e.g. 30 principal components), producing 1-2 cell clusters; LMDS embedding is preferred
- If the dataset includes extremely rare cell types and extremely prevalent cell types at the same time, the k-medoids might not work properly and manual cluster merging may be needed

## Evidence

- [abstract] Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells: "Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells."
- [intro] The cell connectivity allows users to visually see which parts of the embedding are likely to be end points (leaves) and branch points of the trajectory: "The cell connectivity allows users to visually see which parts of the embedding are likely to be end points (leaves) and branch points of the trajectory"
- [intro] The second parameter (min.cluster.size) determines the clusterings that are filtered out because they include too small clusters.: "The second parameter (`min.cluster.size`) determines the clusterings that are filtered out because they include too small clusters."
- [results] In contrast to Slingshot and TinGa, Totem predicted trajectories that correlated well with the true milestone network, and its cell connectivity measure provided accurate information about end points and branching points.: [evidence span withheld — anchor status: not_found]
- [readme] Totem package is a tool designed to facilitate inference of tree-shaped trajectories from single-cell data.: "`Totem` package is a tool designed to facilitate inference of tree-shaped trajectories from single-cell
 data."
