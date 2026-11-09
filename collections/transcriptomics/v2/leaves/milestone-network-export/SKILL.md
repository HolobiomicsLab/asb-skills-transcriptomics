---
name: milestone-network-export
description: Use when trajectory topologies inferred by multiple methods (e.g. Totem,
  Slingshot with ground-truth clustering, TinGa) must be compared or deposited as
  milestone network edge lists on datasets like multifurcating_4 or mouse thymus T
  cells.
when_to_use_negative:
- The trajectory of interest is disconnected or contains cycles — Totem explicitly
  cannot generate these, so milestone networks from it are not appropriate
- The target is a multi-root (converging) trajectory requiring correct pseudotime
  — dynplot will not generate correct pseudotime for multi-root trajectories
- The k-medoids backbone is unreliable, e.g. extremely imbalanced rare plus prevalent
  cell types or high-dimensional PCA input (e.g. 30 principal components) yielding
  1–2 cell clusters, requiring manual cluster merging before export
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Totem
  role: Generates k-medoids (CLARA) clusterings, ranks them with Calinski-Harabasz
    score and cell connectivity, constructs MST topologies, and yields the cell-connectivity
    profiles exported for each dataset
  repo: github.com/elolab/Totem
- name: Slingshot
  role: Constructs the trajectory MST backbone from a supplied clustering (here ground-truth
    cluster labels) and finds linear curves through the lineages; its topology is
    exported alongside Totem's
- name: TinGa
  role: Third trajectory-inference method whose inferred topology is extracted and
    converted to a milestone network edge list for the Fig. 4 comparison
- name: dyndimred
  role: Provides landmark Multi-Dimensional Scaling (LMDS) to generate the 5-dimensional
    low-dimensional embedding Totem clusters on
- name: dynplot
  role: R package (installed from GitHub) used downstream to visualize milestone network
    topologies; not on CRAN
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:0b7793cc5b89864a5fbb94efb4919bfc63511f314cb7b02db3dd0fbc3cca55d8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - R
  - SingleCellExperiment
  - Slingshot
  - TinGa
  - Totem
  - cowplot
  - dyndimred
  - dynplot
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# milestone-network-export

## Summary

Converts inferred tree-shaped trajectories from Totem, Slingshot, and TinGa into milestone network edge lists and exports them (milestone_networks.csv) together with Totem cell-connectivity profiles (connectivity_profiles.csv), enabling direct topology comparison across methods and datasets as in the Bioinformatics btad515 benchmark (Fig. 4).

## When to use

Use when trajectory topologies inferred by multiple methods (e.g. Totem, Slingshot with ground-truth clustering, TinGa) must be compared or deposited as milestone network edge lists on datasets like multifurcating_4 or mouse thymus T cells.

## When NOT to use

- The trajectory of interest is disconnected or contains cycles — Totem explicitly cannot generate these, so milestone networks from it are not appropriate
- The target is a multi-root (converging) trajectory requiring correct pseudotime — dynplot will not generate correct pseudotime for multi-root trajectories
- The k-medoids backbone is unreliable, e.g. extremely imbalanced rare plus prevalent cell types or high-dimensional PCA input (e.g. 30 principal components) yielding 1–2 cell clusters, requiring manual cluster merging before export

## Inputs

- SingleCellExperiment object wrapping a dynbenchmark dataset (e.g. simulated multifurcating_4 or mouse thymus T cells from Han et al. 2018, doi:10.5281/zenodo.1443566)
- 5-dimensional LMDS embedding generated via dyndimred
- Ground-truth cluster labels (for Slingshot MST backbone construction)
- Totem cell-connectivity profiles of the selected clustering
- TinGa inferred trajectory topology

## Outputs

- milestone_networks.csv — milestone network edge lists for each method-by-dataset combination (Totem, Slingshot, TinGa)
- connectivity_profiles.csv — per-cell connectivity profiles for Totem clusterings, indicating likely leaves and branch points

## How to apply

First run Totem on SingleCellExperiment-wrapped datasets: embed with 5-dimensional landmark MDS (LMDS) via dyndimred, run k-medoids (CLARA) clustering many times (default 10k) over a range of k, filter out clusterings with clusters smaller than min.cluster.size (default 5 cells), and select the top clustering using the Calinski-Harabasz score and cell connectivity before constructing the MST topology. Compute cell-connectivity profiles to pinpoint likely end points (leaves) and branch points. Run Slingshot using ground-truth cluster labels as the clustering from which the MST backbone is constructed, and extract TinGa's inferred topology. Finally convert every method-by-dataset trajectory to a milestone network edge list and export milestone_networks.csv with connectivity_profiles.csv, matching the topologies compared in Fig. 4.

## Related tools

- **Totem** (Generates k-medoids (CLARA) clusterings, ranks them with Calinski-Harabasz score and cell connectivity, constructs MST topologies, and yields the cell-connectivity profiles exported for each dataset) — github.com/elolab/Totem
- **Slingshot** (Constructs the trajectory MST backbone from a supplied clustering (here ground-truth cluster labels) and finds linear curves through the lineages; its topology is exported alongside Totem's)
- **TinGa** (Third trajectory-inference method whose inferred topology is extracted and converted to a milestone network edge list for the Fig. 4 comparison)
- **dyndimred** (Provides landmark Multi-Dimensional Scaling (LMDS) to generate the 5-dimensional low-dimensional embedding Totem clusters on)
- **dynplot** (R package (installed from GitHub) used downstream to visualize milestone network topologies; not on CRAN)

## Examples

```
remotes::install_github("dynverse/dynfeature"); remotes::install_github("dynverse/dynplot"); remotes::install_github("elolab/Totem")
```

## Evaluation signals

- milestone_networks.csv parses as method-by-dataset rows of milestone network edge lists covering all three methods (Totem, Slingshot, TinGa) on both multifurcating_4 and mouse thymus T cell datasets
- connectivity_profiles.csv contains per-cell connectivity values that mark the true end points (leaves) and the middle branching point of the trajectory
- Totem's exported topology correlates well with the true milestone network for multifurcating_4 and the bifurcating thymus data, unlike Slingshot and TinGa
- Clusterings with clusters smaller than min.cluster.size (default 5 cells) are absent from the exported topologies
- The selected clustering maximizes the Calinski-Harabasz score combined with cell connectivity before the MST is constructed

## Limitations

- Totem cannot generate disconnected trajectories or trajectories that have cycles
- dynplot will not generate the correct pseudotime for multi-root trajectories
- k-medoids may not work well with PCA if the number of principal components is high (e.g. 30), producing 1–2 cell clusters
- If the dataset includes extremely rare cell types and extremely prevalent cell types at the same time, k-medoids might not work properly and manual cluster merging is required

## Evidence

- [abstract] Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells.: "Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells."
- [intro] `Totem` uses the `Calinski-Harabasz score` and the cell connectivity to rank the clustering results.: "`Totem` uses the `Calinski-Harabasz score` and the cell connectivity to rank the clustering results."
- [intro] we use the landmark Multi-Dimensional Scaling (LMDS) from `dyndimred` package [4] to generate a low-dimensional embedding with 5 dimensions.: "Here, we use the landmark Multi-Dimensional Scaling (LMDS) from `dyndimred` package [4] to generate a low-dimensional embedding with 5 dimensions."
- [discussion] `Totem` cannot generate disconnected trajectories or trajectories that have cycles.: "`Totem` cannot generate disconnected trajectories or trajectories that have cycles."
- [readme] The `dynplot` and `dynfeature` R packages are not available on CRAN, and they must be installed from GitHub.: [evidence span withheld — anchor status: external_doc]
