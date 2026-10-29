---
name: dynverse-trajectory-scoring
description: Use when inferred trajectories (e.g. from Totem, Slingshot, or TinGa)
  must be scored against dynverse gold-standard trajectories with known milestone/branch
  structure, as in the 216 tree-shaped dynverse benchmark or the 10 dynverse Zenodo
  datasets (doi:10.5281/zenodo.1443566).
when_to_use_negative:
- The data have no gold-standard trajectory (no dynverse milestone/branch ground truth)
  — the dynverse metrics cannot be computed without a reference to score against
- The inferred trajectory contains cycles or disconnected components — Totem itself
  cannot generate such trajectories, so the comparison is out of scope for this pipeline
- The goal is pseudotime visualization of multi-root (converging) trajectories — dynplot
  will not generate correct pseudotime for multi-root trajectories, so downstream
  inspection via dynplot is unreliable
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Totem
  role: Generates the inferred tree-shaped trajectories (many CLARA k-medoids clusterings,
    MST backbones via Slingshot) whose overall scores are evaluated against the dynverse
    ground truth
  repo: https://github.com/elolab/Totem
- name: Slingshot
  role: Constructs the Minimum Spanning Tree trajectory backbone and linear curves
    through the lineages of the tree from the selected clustering; also the dynverse
    Slingshot baseline method
- name: dyndimred
  role: Provides LMDS dimensionality reduction to produce the standardized 5-dimensional
    embedding used as common input for all benchmarked methods
- name: TinGa
  role: Baseline trajectory inference method benchmarked against Totem and Slingshot
    on the same embeddings
- name: SingleCellExperiment
  role: Container format in which benchmark datasets are loaded and prepared with
    PrepareTotem before trajectory inference and scoring
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:533260efaac0bf462967b14ecd694f4f148203c9169bf3029e4fbd5f10614ab0
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# dynverse-trajectory-scoring

## Summary

Quantitative benchmarking of inferred single-cell trajectories against dynverse gold-standard trajectories, using the four dynverse metrics (correlation of geodesic distances, featureimp correlation, F1 branches, HIM) aggregated into a geometric-mean overall score in the [0, 1] range.

## When to use

Use when inferred trajectories (e.g. from Totem, Slingshot, or TinGa) must be scored against dynverse gold-standard trajectories with known milestone/branch structure, as in the 216 tree-shaped dynverse benchmark or the 10 dynverse Zenodo datasets (doi:10.5281/zenodo.1443566).

## When NOT to use

- The data have no gold-standard trajectory (no dynverse milestone/branch ground truth) — the dynverse metrics cannot be computed without a reference to score against
- The inferred trajectory contains cycles or disconnected components — Totem itself cannot generate such trajectories, so the comparison is out of scope for this pipeline
- The goal is pseudotime visualization of multi-root (converging) trajectories — dynplot will not generate correct pseudotime for multi-root trajectories, so downstream inspection via dynplot is unreliable

## Inputs

- SingleCellExperiment objects with dynverse gold-standard trajectory labels (linear, bifurcation, multifurcation, tree topologies) from Zenodo doi:10.5281/zenodo.1443566
- Inferred trajectory objects from Totem, the dynverse Slingshot implementation, or TinGa, computed on a common 5-dimensional LMDS embedding

## Outputs

- Per-method/per-topology table of the four dynverse metrics (geodesic-distance correlation, featureimp correlation, F1 branches, HIM) written to benchmark_metrics.csv
- Per-configuration overall scores (geometric mean, range [0, 1]) per dataset, parameter setting, and seed, e.g. benchmark_overall_scores.csv for Supplementary Figures S2/S3

## How to apply

First standardize the input so all methods are compared on equal footing: apply landmark Multi-Dimensional Scaling (LMDS from dyndimred) to obtain a 5-dimensional embedding, the common input used for Totem, the dynverse Slingshot implementation, and TinGa. Run each method (Totem: many CLARA k-medoids clusterings with min.cluster.size filtering, an MST per clustering via Slingshot, and clustering selection by Calinski-Harabasz score and cell connectivity). For each inferred trajectory, compute the four dynverse metrics against the gold standard: correlation of geodesic distances, feature-importance (featureimp) correlation, F1 branches, and HIM. Aggregate the four metrics per method and topology (linear vs nonlinear: bifurcation, multifurcation, tree) into a geometric-mean overall score. For method comparisons, apply Friedman tests and Wilcoxon signed-rank tests with Bonferroni adjustment; in the reference study Totem was superior for nonlinear trajectories (adjusted P <= 0.01) while Slingshot was superior for linear trajectories.

## Related tools

- **Totem** (Generates the inferred tree-shaped trajectories (many CLARA k-medoids clusterings, MST backbones via Slingshot) whose overall scores are evaluated against the dynverse ground truth) — https://github.com/elolab/Totem
- **Slingshot** (Constructs the Minimum Spanning Tree trajectory backbone and linear curves through the lineages of the tree from the selected clustering; also the dynverse Slingshot baseline method)
- **dyndimred** (Provides LMDS dimensionality reduction to produce the standardized 5-dimensional embedding used as common input for all benchmarked methods)
- **TinGa** (Baseline trajectory inference method benchmarked against Totem and Slingshot on the same embeddings)
- **SingleCellExperiment** (Container format in which benchmark datasets are loaded and prepared with PrepareTotem before trajectory inference and scoring)

## Evaluation signals

- Every overall score must lie within the dynverse score range [0, 1]
- The output table must contain one overall-score entry per dataset, parameter configuration, and seed, with all 10 dynverse benchmark datasets covered (robustness sweep) or all 216 tree-shaped trajectories (method comparison)
- The overall score must equal the geometric mean of the four dynverse metrics (correlation of geodesic distances, featureimp correlation, F1 branches, HIM), not an arithmetic mean
- Method ordering sanity check: for nonlinear topologies Totem should score statistically above TinGa and Slingshot (Friedman test; Wilcoxon signed-rank with Bonferroni adjustment, P <= 0.01), while Slingshot should lead on linear trajectories
- All methods must be scored on the same standardized LMDS 5-dimensional embedding to ensure a fair comparison

## Limitations

- Totem cannot generate disconnected trajectories or trajectories that have cycles, so the benchmark covers tree-shaped topologies only
- dynplot will not generate the correct pseudotime for multi-root (converging) trajectories, complicating visual verification of scored results
- k-medoids (CLARA) may not work well with high-dimensional PCA input (e.g. 30 principal components), producing 1-2 cell clusters, and can fail on extremely imbalanced datasets (rare plus prevalent cell types), affecting trajectory quality and hence scores

## Evidence

- [abstract] Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells.: "Totem generates a large number of clustering results, estimates their topologies as minimum spanning trees, and uses them to measure the connectivity of the cells."
- [intro] Totem uses the Calinski-Harabasz score and the cell connectivity to rank the clustering results.: "`Totem` uses the `Calinski-Harabasz score` and the cell connectivity to rank the clustering results."
- [intro] Here, we use the landmark Multi-Dimensional Scaling (LMDS) from dyndimred package to generate a low-dimensional embedding with 5 dimensions.: "Here, we use the landmark Multi-Dimensional Scaling (LMDS) from `dyndimred` package [4] to generate a low-dimensional embedding with 5 dimensions."
- [discussion] Totem cannot generate disconnected trajectories or trajectories that have cycles.: "`Totem` cannot generate disconnected trajectories or trajectories that have cycles."
- [discussion] dynplot will not generate the correct pseudotime for multi-root trajectories.: "`dynplot` will not generate the correct pseudotime for multi-root trajectories."
- [readme] Totem uses Slingshot to find linear curves through the lineages of the tree.: [evidence span withheld — anchor status: external_doc]
