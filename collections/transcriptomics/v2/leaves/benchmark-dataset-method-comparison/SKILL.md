---
name: benchmark-dataset-method-comparison
description: Use when evaluating how a new trajectory-inference method's predicted
  topology compares to established methods (Slingshot, TinGa) on benchmark single-cell
  datasets with ground-truth clustering and known milestone networks.
when_to_use_negative:
- The dataset has no known or reference topology (no simulated ground truth and no
  validated biological trajectory), so method rankings cannot be anchored to a true
  milestone network
- The trajectories of interest are disconnected, contain cycles, or require multi-root
  (converging) pseudotime — Totem cannot generate such topologies
- The dataset mixes extremely rare and extremely prevalent cell types, where k-medoids
  (CLARA) clustering inside Totem fails and manual cluster merging would be required
  instead
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: Totem
  role: 'Cell-connectivity-guided trajectory inference: generates many CLARA k-medoids
    clusterings, builds MSTs, and selects topology via Calinski-Harabasz score and
    cell connectivity'
  repo: https://github.com/elolab/Totem
- name: Slingshot
  role: 'Baseline method: constructs the trajectory MST backbone from the ground-truth
    cluster labels on each benchmark dataset'
- name: TinGa
  role: 'Baseline method: grows the trajectory on a k-nearest-neighbor graph; its
    inferred topology is extracted for comparison'
- name: dyndimred
  role: Provides landmark Multi-Dimensional Scaling (LMDS) to generate the 5-dimensional
    embedding used before clustering
- name: SingleCellExperiment
  role: Container format for wrapping the dynbenchmark datasets before running each
    method
- name: BiocManager
  role: Installs Totem from Bioconductor
- name: cowplot
  role: R plotting package used to assemble comparison figures
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:dfafae8b9f3a766ef11ba5c95f0e393540a659e226332962a41e640df8f49223
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# benchmark-dataset-method-comparison

## Summary

Benchmarking skill for comparing trajectory-inference methods (Totem, Slingshot, TinGa) on datasets with known topologies — a simulated multifurcating_4 tree and the bifurcating mouse thymus T cell dataset (Han et al. 2018) from the dynbenchmark Zenodo deposit (doi:10.5281/zenodo.1443566) — converting all outputs to milestone network edge lists for direct topology comparison.

## When to use

Use when evaluating how a new trajectory-inference method's predicted topology compares to established methods (Slingshot, TinGa) on benchmark single-cell datasets with ground-truth clustering and known milestone networks.

## When NOT to use

- The dataset has no known or reference topology (no simulated ground truth and no validated biological trajectory), so method rankings cannot be anchored to a true milestone network
- The trajectories of interest are disconnected, contain cycles, or require multi-root (converging) pseudotime — Totem cannot generate such topologies
- The dataset mixes extremely rare and extremely prevalent cell types, where k-medoids (CLARA) clustering inside Totem fails and manual cluster merging would be required instead

## Inputs

- SingleCellExperiment objects wrapping the multifurcating_4 simulated dataset and the mouse thymus T cell dataset (Han et al. 2018) from dynbenchmark Zenodo doi:10.5281/zenodo.1443566
- Ground-truth cluster labels for Slingshot's MST backbone construction
- True milestone network of each benchmark dataset for topology comparison

## Outputs

- milestone_networks.csv — method-by-dataset trajectory topologies as milestone network edge lists
- connectivity_profiles.csv — Totem cell-connectivity profiles identifying likely leaves and branch points
- Per-dataset inferred topologies from Totem, Slingshot, and TinGa for side-by-side comparison (cf. Fig. 4)

## How to apply

Load the multifurcating_4 and mouse thymus T cell (Han et al. 2018) datasets from the dynbenchmark Zenodo deposit (doi:10.5281/zenodo.1443566) and wrap them in SingleCellExperiment objects. Run Totem on both: prepare data with PrepareTotem, embed with 5-dimensional landmark MDS (LMDS) via dyndimred, run k-medoids (CLARA) clustering many times (default 10k) over a range of k, filter out clusterings with clusters smaller than min.cluster.size (default 5 cells), and select the top clustering using the Calinski-Harabasz score plus cell connectivity before building the MST topology. Run Slingshot using ground-truth cluster labels as the clustering from which the MST backbone is constructed, and run TinGa to extract its inferred topology. Compute cell-connectivity profiles for the Totem clusterings to pinpoint likely end points (leaves) and the middle branching point. Finally convert every method-by-dataset trajectory into a milestone network edge list (milestone_networks.csv) with connectivity_profiles.csv, so topologies can be compared quantitatively against the true milestone network as in Fig. 4.

## Related tools

- **Totem** (Cell-connectivity-guided trajectory inference: generates many CLARA k-medoids clusterings, builds MSTs, and selects topology via Calinski-Harabasz score and cell connectivity) — https://github.com/elolab/Totem
- **Slingshot** (Baseline method: constructs the trajectory MST backbone from the ground-truth cluster labels on each benchmark dataset)
- **TinGa** (Baseline method: grows the trajectory on a k-nearest-neighbor graph; its inferred topology is extracted for comparison)
- **dyndimred** (Provides landmark Multi-Dimensional Scaling (LMDS) to generate the 5-dimensional embedding used before clustering)
- **SingleCellExperiment** (Container format for wrapping the dynbenchmark datasets before running each method)
- **BiocManager** (Installs Totem from Bioconductor)
- **cowplot** (R plotting package used to assemble comparison figures)

## Examples

```
remotes::install_github("elolab/Totem"); sce <- SingleCellExperiment::SingleCellExperiment(assays=list(counts=mat)); totem <- PrepareTotem(sce)
```

## Evaluation signals

- Totem's inferred milestone network should correlate well with the true milestone network on both multifurcating_4 and mouse thymus, whereas Slingshot and TinGa diverge
- Totem's cell-connectivity profile should correctly localize the end points (leaves) and the middle branching point of the trajectory
- All methods' outputs must convert losslessly to milestone network edge lists in milestone_networks.csv, one topology per method-dataset pair
- Clusterings containing clusters smaller than min.cluster.size (default 5 cells) must have been filtered out before topology selection
- Selection of the top Totem clustering must combine the Calinski-Harabasz score with cell connectivity, not clustering score alone

## Limitations

- dynplot will not generate the correct pseudotime for multi-root (converging) trajectories; a manual work-around is required
- Totem cannot generate disconnected trajectories or trajectories that have cycles
- k-medoids may not work well with PCA input when the number of principal components is high (e.g. 30), producing 1–2 cell clusters
- If the dataset includes extremely rare and extremely prevalent cell types at the same time, k-medoids might not work properly and manual cluster merging is needed

## Evidence

- [abstract] Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells: "Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells."
- [intro] CLARA clustering over range of k with min.cluster.size filter: "the next step is to perform clustering that involves running the k-medoids (CLARA) algorithm a large number of times (default 10k)."
- [intro] min.cluster.size determines which clusterings are filtered out: "The second parameter (`min.cluster.size`) determines the clusterings that are filtered out because they include too small clusters."
- [intro] Ranking clusterings by Calinski-Harabasz score and connectivity: "`Totem` uses the `Calinski-Harabasz score` and the cell connectivity to rank the clustering results."
- [intro] LMDS embedding with 5 dimensions via dyndimred: "we use the landmark Multi-Dimensional Scaling (LMDS) from `dyndimred` package [4] to generate a low-dimensional embedding with 5 dimensions."
- [intro] Cell connectivity visually pinpoints end points and branch points: "The cell connectivity allows users to visually see which parts of the embedding are likely to be end points (leaves) and branch points of the trajectory"
- [readme] Totem install path from README: [evidence span withheld — anchor status: external_doc]
- [intro] Slingshot required clustering for MST construction: "In trajectory inference methods like `Slingshot`[2] that require a clustering to construct the Minimum Spanning Tree (`MST`) as the backbone of the trajectory"
