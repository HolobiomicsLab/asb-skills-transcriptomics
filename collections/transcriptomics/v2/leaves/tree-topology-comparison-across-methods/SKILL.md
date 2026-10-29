---
name: tree-topology-comparison-across-methods
description: Use when comparing trajectory inference methods for tree-shaped (linear,
  bifurcation, multifurcation) single-cell topologies on a common benchmark set, requiring
  standardized scoring with dynverse metrics and topology-stratified statistical testing.
when_to_use_negative:
- The trajectories of interest are disconnected or contain cycles — Totem cannot generate
  disconnected trajectories or trajectories that have cycles, so tree-only benchmarking
  methods are the wrong scope
- Trajectories have multiple roots (converging topologies) — dynplot will not generate
  the correct pseudotime for multi-root trajectories, corrupting the comparison
- The embedding is a high-dimensional PCA (e.g. 30 principal components) — k-medoids
  may perform poorly, so Totem-based benchmark results would be confounded by the
  embedding choice
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Totem
  role: Method under comparison; generates many CLARA k-medoids clusterings (default
    10k), builds per-clustering MSTs with Slingshot, and selects clusterings using
    the Calinski-Harabasz score and cell connectivity
  repo: https://github.com/elolab/Totem
- name: Slingshot
  role: dynverse Slingshot implementation run as a competing method on the same standardized
    5-dimensional LMDS embeddings; also used by Totem internally to construct MSTs
- name: TinGa
  role: Competing trajectory inference method run on the identical standardized embeddings
    for the benchmark comparison
- name: dyndimred
  role: Provides LMDS dimensionality reduction to create the common 5-dimensional
    embedding used as input by all compared methods
- name: dynfeature
  role: dynverse dependency installed from GitHub for feature-importance (featureimp)
    scoring of inferred trajectories
  repo: https://github.com/dynverse/dynfeature
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:bbed4b74b7c9b9d96591eb4dca187e2f09e26f6d6a86789ad5fafaf30f6e307a
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
  - dyndimred
  - dynfeature
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# tree-topology-comparison-across-methods

## Summary

Benchmarks multiple tree-shaped trajectory inference methods (Totem, dynverse Slingshot, TinGa) on the same standardized single-cell datasets, scoring each against gold standards with the four dynverse metrics and comparing methods statistically by topology type.

## When to use

Use when comparing trajectory inference methods for tree-shaped (linear, bifurcation, multifurcation) single-cell topologies on a common benchmark set, requiring standardized scoring with dynverse metrics and topology-stratified statistical testing.

## When NOT to use

- The trajectories of interest are disconnected or contain cycles — Totem cannot generate disconnected trajectories or trajectories that have cycles, so tree-only benchmarking methods are the wrong scope
- Trajectories have multiple roots (converging topologies) — dynplot will not generate the correct pseudotime for multi-root trajectories, corrupting the comparison
- The embedding is a high-dimensional PCA (e.g. 30 principal components) — k-medoids may perform poorly, so Totem-based benchmark results would be confounded by the embedding choice

## Inputs

- dynverse gold-standard synthetic trajectory datasets (216 tree-shaped trajectories, Zenodo doi:10.5281/zenodo.1443566)
- SingleCellExperiment objects per dataset
- 5-dimensional LMDS embeddings generated with dyndimred (common input for all methods)
- Gold-standard milestone/branch assignments per trajectory for scoring

## Outputs

- Per-method, per-topology table of the four dynverse metric scores (correlation of geodesic distances, featureimp correlation, F1 branches, HIM), written to benchmark_metrics.csv
- Geometric mean overall scores per method and topology
- Statistical comparison results (Friedman test; Bonferroni-adjusted Wilcoxon signed-rank tests)

## How to apply

Load the gold-standard dynverse synthetic trajectories (216 tree-shaped datasets spanning linear, bifurcation, multifurcation, and tree topologies from Zenodo doi:10.5281/zenodo.1443566) and standardize all methods on the same input by applying landmark MDS (LMDS from the dyndimred package) to produce a 5-dimensional embedding. Run each method on identical embeddings: Totem generates many CLARA k-medoids clusterings (default 10k runs, filtering clusterings with clusters below min.cluster.size), builds an MST per clustering with Slingshot, and ranks clusterings by the Calinski-Harabasz score and cell connectivity; run the dynverse Slingshot implementation and TinGa on the same embeddings. Score every inferred trajectory against its gold standard with the four dynverse metrics: correlation of geodesic distances, feature-importance (featureimp) correlation, F1 branches, and HIM. Aggregate the four metrics per method and topology into a geometric mean overall score, then compare methods with a Friedman test and pairwise Wilcoxon signed-rank tests using Bonferroni adjustment (per the source task, adjusted P ≤ 0.001 overall and P ≤ 0.01 for pairwise comparison were reported). Write per-method/per-topology results to benchmark_metrics.csv.

## Related tools

- **Totem** (Method under comparison; generates many CLARA k-medoids clusterings (default 10k), builds per-clustering MSTs with Slingshot, and selects clusterings using the Calinski-Harabasz score and cell connectivity) — https://github.com/elolab/Totem
- **Slingshot** (dynverse Slingshot implementation run as a competing method on the same standardized 5-dimensional LMDS embeddings; also used by Totem internally to construct MSTs)
- **TinGa** (Competing trajectory inference method run on the identical standardized embeddings for the benchmark comparison)
- **dyndimred** (Provides LMDS dimensionality reduction to create the common 5-dimensional embedding used as input by all compared methods)
- **dynfeature** (dynverse dependency installed from GitHub for feature-importance (featureimp) scoring of inferred trajectories) — https://github.com/dynverse/dynfeature

## Examples

```
remotes::install_github("dynverse/dynfeature"); remotes::install_github("dynverse/dynplot"); remotes::install_github("elolab/Totem")
```

## Evaluation signals

- All methods consume the same standardized input: a 5-dimensional LMDS embedding produced by dyndimred for each of the 216 trajectories — any method-specific preprocessing breaks comparability
- Each inferred trajectory is scored against its gold standard with all four dynverse metrics (geodesic-distance correlation, featureimp correlation, F1 branches, HIM); missing metrics indicate incomplete scoring
- Overall scores per method/topology equal the geometric mean of the four metric scores
- benchmark_metrics.csv contains one row per method per topology (linear, bifurcation, multifurcation, tree) across the 216 datasets
- Method differences are confirmed by Friedman test plus Bonferroni-adjusted Wilcoxon signed-rank tests; per the source task, Totem was statistically superior for nonlinear trajectories (adjusted P ≤ 0.001 / ≤ 0.01) while Slingshot was superior for linear trajectories

## Limitations

- Totem cannot generate disconnected trajectories or trajectories that have cycles, so the comparison is restricted to tree-shaped topologies
- dynplot will not generate the correct pseudotime for multi-root (converging) trajectories; correct pseudotime for such cases requires manual work-arounds
- k-medoids may not work well with PCA embeddings when the number of principal components is high (e.g. 30), producing 1-2 cell clusters; the benchmark therefore standardizes on 5-dimensional LMDS
- If a dataset includes extremely rare and extremely prevalent cell types simultaneously, k-medoids may fail and require manual cluster merging
- Method conclusions are topology-dependent: Totem was superior for nonlinear trajectories while Slingshot was superior for linear trajectories

## Evidence

- [abstract] Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells.: "Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells."
- [intro] `Totem` uses the `Calinski-Harabasz score` and the cell connectivity to rank the clustering results.: "`Totem` uses the `Calinski-Harabasz score` and the cell connectivity to rank the clustering results."
- [intro] Here, we use the landmark Multi-Dimensional Scaling (LMDS) from `dyndimred` package [4] to generate a low-dimensional embedding with 5 dimensions.: "Here, we use the landmark Multi-Dimensional Scaling (LMDS) from `dyndimred` package [4] to generate a low-dimensional embedding with 5 dimensions."
- [discussion] `Totem` cannot generate disconnected trajectories or trajectories that have cycles.: "`Totem` cannot generate disconnected trajectories or trajectories that have cycles."
- [readme] A user-friendly tool for inferring tree-shaped trajectories from single cell data.: "A user-friendly tool for inferring tree-shaped trajectories from single cell data."
