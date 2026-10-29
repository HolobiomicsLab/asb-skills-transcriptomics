---
name: minimum-spanning-tree-topology-estimation
description: Use when inferring tree-shaped trajectories from single-cell expression
  data where a good clustering must be found automatically and cell connectivity should
  guide selection of end points (leaves) and branch points.
when_to_use_negative:
- The expected trajectory is disconnected or contains cycles — Totem cannot generate
  disconnected trajectories or trajectories that have cycles
- The trajectory is multi-root/converging — dynplot will not generate the correct
  pseudotime for multi-root trajectories without a manual work-around
- The dataset mixes extremely rare and extremely prevalent cell types — k-medoids
  may not work properly, requiring manual cluster merging
edam_operation: http://edamontology.org/operation_3432
tools:
- name: Totem
  role: Generates many CLARA k-medoids clusterings, builds an MST per clustering,
    and measures cell connectivity for topology selection
  repo: https://github.com/elolab/Totem
- name: Slingshot
  role: Constructs the Minimum Spanning Tree backbone (and linear curves through lineages)
    from a given clustering
- name: dyndimred
  role: Provides the LMDS dimensionality reduction used to generate a 5-dimensional
    low-dimensional embedding before clustering
- name: SingleCellExperiment
  role: Container class for the prepared single-cell data fed to PrepareTotem
- name: BiocManager
  role: Installs Totem from Bioconductor
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:ce5cbcda91e61ada67ba1c8dc1cda2ab323fa88cf8522e022668e73a1fefdfc4
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# minimum-spanning-tree-topology-estimation

## Summary

Estimates a tree-shaped single-cell trajectory (linear, bifurcating, or multifurcating) by generating many k-medoids (CLARA) clusterings, constructing a Minimum Spanning Tree (MST) backbone for each via Slingshot, and using cell connectivity plus the Calinski-Harabasz score to select the clustering/topology that best matches the data.

## When to use

Use when inferring tree-shaped trajectories from single-cell expression data where a good clustering must be found automatically and cell connectivity should guide selection of end points (leaves) and branch points.

## When NOT to use

- The expected trajectory is disconnected or contains cycles — Totem cannot generate disconnected trajectories or trajectories that have cycles
- The trajectory is multi-root/converging — dynplot will not generate the correct pseudotime for multi-root trajectories without a manual work-around
- The dataset mixes extremely rare and extremely prevalent cell types — k-medoids may not work properly, requiring manual cluster merging

## Inputs

- Normalized single-cell expression matrix (e.g. LogNormalize-normalized counts) wrapped in a SingleCellExperiment object
- Simulated or real trajectory datasets from the dynbenchmark Zenodo deposit (doi:10.5281/zenodo.1443566)
- Optional: a user-provided prior clustering to direct the search toward similar clusterings

## Outputs

- A milestone network edge list encoding the inferred tree-shaped trajectory topology (exported as milestone_networks.csv)
- Cell-connectivity profiles identifying likely end points (leaves) and branch points (exported as connectivity_profiles.csv)
- Ranked clustering candidates scored by Calinski-Harabasz score and cell connectivity

## How to apply

Wrap the normalized expression data (e.g. LogNormalize from Seurat) in a SingleCellExperiment object and prepare it with PrepareTotem. Reduce dimensionality with landmark MDS (LMDS from dyndimred, 5 dimensions; use 2D MDS only for visualization), then run the k-medoids CLARA algorithm a large number of times (default 10k) over a range of k values, filtering out clusterings that contain clusters smaller than min.cluster.size (default 5 cells). Construct an MST for each surviving clustering with Slingshot, rank the results using the Calinski-Harabasz score and cell connectivity (optionally ARI to a user-provided prior clustering), and build the final topology from the selected clustering. Compute cell-connectivity profiles from the ensemble of MSTs to visually and quantitatively pinpoint likely leaves and the middle branching point; in benchmarks on multifurcating_4 and mouse thymus T cell data, this outperformed Slingshot (with ground-truth clustering) and TinGa.

## Related tools

- **Totem** (Generates many CLARA k-medoids clusterings, builds an MST per clustering, and measures cell connectivity for topology selection) — https://github.com/elolab/Totem
- **Slingshot** (Constructs the Minimum Spanning Tree backbone (and linear curves through lineages) from a given clustering)
- **dyndimred** (Provides the LMDS dimensionality reduction used to generate a 5-dimensional low-dimensional embedding before clustering)
- **SingleCellExperiment** (Container class for the prepared single-cell data fed to PrepareTotem)
- **BiocManager** (Installs Totem from Bioconductor)

## Examples

```
remotes::install_github("elolab/Totem"); sce <- PrepareTotem(sce); embeddings <- dyndimred::dimred_lmds(sce, ndim = 5)
```

## Evaluation signals

- Predicted milestone network correlates well with the true milestone network, as demonstrated on the multifurcating_4 simulation and mouse thymus T cell (Han et al. 2018) datasets
- Cell connectivity profiles correctly locate the end points (leaves) and the middle branching point of the trajectory
- All clusterings containing clusters smaller than min.cluster.size (default 5 cells) are filtered out before MST construction
- Selected clustering scores highest under the Calinski-Harabasz score and cell connectivity ranking criteria
- Final outputs serialize as milestone_networks.csv and connectivity_profiles.csv edge lists compatible with the compared topologies (Fig. 4)

## Limitations

- k-medoids may not work well with high-dimensional PCA input (e.g. 30 principal components), producing 1-2 cell clusters
- If the dataset includes extremely rare cell types and extremely prevalent cell types at the same time, k-medoids might not work properly and manual cluster merging is needed
- No correct pseudotime support for multi-root (converging) trajectories; requires a manual work-around
- No support for disconnected trajectories or trajectories with cycles

## Evidence

- [abstract] Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells.: "Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells."
- [intro] `Totem` generates a large number of clustering results with a *k*-medoids algorithm (CLARA) and constructs an MST for each clustering.: "`Totem` generates a large number of clustering results with a *k*-medoids algorithm (CLARA) and constructs an MST for each clustering."
- [intro] `Totem` uses the `Calinski-Harabasz score` and the cell connectivity to rank the clustering results.: "`Totem` uses the `Calinski-Harabasz score` and the cell connectivity to rank the clustering results."
- [intro] The second parameter (`min.cluster.size`) determines the clusterings that are filtered out because they include too small clusters.: "The second parameter (`min.cluster.size`) determines the clusterings that are filtered out because they include too small clusters."
- [readme] Totem uses Slingshot [2] to find linear curves through the lineages of the tree.: [evidence span withheld — anchor status: external_doc]
