---
name: trajectory-inference-parameter-sweep
description: Use when you need to assess how sensitive a clustering-based trajectory
  inference method (e.g. Totem) is to its clustering-count and k-range parameters,
  by running a grid of parameter combinations with multiple random seeds against a
  benchmark ground truth.
when_to_use_negative:
- Trajectories that are disconnected or contain cycles — Totem cannot generate disconnected
  trajectories or trajectories that have cycles
- Multi-root (converging) trajectories where correct pseudotime is needed — dynplot
  will not generate the correct pseudotime for multi-root trajectories
- Datasets with extremely rare and extremely prevalent cell types simultaneously,
  where k-medoids may fail without manual cluster merging
edam_operation: http://edamontology.org/operation_3899
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Totem
  role: Generate many CLARA k-medoids clusterings over a range of k values, rank them,
    and derive tree-shaped trajectories; parameters swept are clustering count and
    upper k limit
  repo: https://github.com/elolab/Totem
- name: Slingshot
  role: Constructs the Minimum Spanning Tree trajectory backbone from the selected
    clustering for each sweep run
- name: SingleCellExperiment
  role: Container format for the benchmark datasets prior to PrepareTotem preprocessing
- name: dyndimred
  role: Provides LMDS dimensionality reduction to a 5-dimensional embedding used before
    clustering
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:32b11aa5f34e7010840b6214299f218c412a3cd89816a98a86fb0ae036ba80fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elolab/Totem
  tools:
  - R
  - SingleCellExperiment
  - Slingshot
  - Totem
  - dyndimred
  edam_operation: http://edamontology.org/operation_3899
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# trajectory-inference-parameter-sweep

## Summary

Reproduce Totem's robustness sweep by varying the number of CLARA k-medoids clusterings (100–7500) and the upper limit of the k range (5–40) across 10 dynverse benchmark datasets, scoring each run with dynverse metrics to quantify parameter sensitivity of the trajectory inference.

## When to use

Use when you need to assess how sensitive a clustering-based trajectory inference method (e.g. Totem) is to its clustering-count and k-range parameters, by running a grid of parameter combinations with multiple random seeds against a benchmark ground truth.

## When NOT to use

- Trajectories that are disconnected or contain cycles — Totem cannot generate disconnected trajectories or trajectories that have cycles
- Multi-root (converging) trajectories where correct pseudotime is needed — dynplot will not generate the correct pseudotime for multi-root trajectories
- Datasets with extremely rare and extremely prevalent cell types simultaneously, where k-medoids may fail without manual cluster merging

## Inputs

- 10 single-cell benchmark datasets from the dynverse Zenodo deposit doi:10.5281/zenodo.1443566, loaded as SingleCellExperiment objects
- Parameter grid: number of clusterings {100, 500, 1000, 2500, 5000, 7500} x upper k-range limit {5, 10, 15, 20, 30, 40}
- 10 random seeds per configuration
- Benchmark ground-truth trajectories for dynverse metric evaluation

## Outputs

- benchmark_overall_scores.csv with one dynverse overall-score entry per dataset, clustering-count value, k-range upper limit, and seed
- Per-configuration aggregated dynverse overall scores (mean over 10 seeds) reproducing Supplementary Figures S2/S3

## How to apply

Load each of the 10 single-cell benchmark datasets from the dynverse Zenodo deposit (doi 10.5281/zenodo.1443566) as SingleCellExperiment objects and prepare them with Totem's PrepareTotem. For every combination of the number of CLARA k-medoids clusterings (100, 500, 1000, 2500, 5000, 7500) and the upper limit of the k range (5, 10, 15, 20, 30, 40), run the Totem clustering with 10 different random seeds, applying the min.cluster.size filter (default 5 cells per cluster) to discard clusterings with too-small clusters. For each run, construct the Minimum Spanning Tree trajectory backbone with Slingshot from the selected clustering and derive the inferred trajectory. Evaluate each inferred trajectory against the benchmark ground truth with the dynverse metrics and record the overall score, then aggregate overall scores across the 10 seeds for each configuration and dataset. Rationale: the sweep tabulates the per-configuration dynverse overall scores corresponding to Supplementary Figures S2/S3, showing that robustness improves as the number of clusterings increases with small variation above 2500 clusterings.

## Related tools

- **Totem** (Generate many CLARA k-medoids clusterings over a range of k values, rank them, and derive tree-shaped trajectories; parameters swept are clustering count and upper k limit) — https://github.com/elolab/Totem
- **Slingshot** (Constructs the Minimum Spanning Tree trajectory backbone from the selected clustering for each sweep run)
- **SingleCellExperiment** (Container format for the benchmark datasets prior to PrepareTotem preprocessing)
- **dyndimred** (Provides LMDS dimensionality reduction to a 5-dimensional embedding used before clustering)

## Examples

```
remotes::install_github("elolab/Totem"); sce <- prepare_totem(sce); run_clustering <- cluster_totem(sce, k.min=2, k.max=30, n.clusters=2500, min.cluster.size=5)
```

## Evaluation signals

- Output table contains exactly one overall-score entry per (dataset, clustering-count, k-range upper limit, seed) combination
- All 10 benchmark datasets from doi:10.5281/zenodo.1443566 are covered in the results table
- All dynverse overall scores fall within the valid score range [0, 1]
- Results reproduce the reported trend: small variation in overall score when the number of clusterings exceeds 2500 (5000 or 7500)
- Each configuration's aggregated score is computed over all 10 random seeds

## Limitations

- k-medoids instability with high-dimensional PCA input (e.g. 30 principal components) can produce clusters of 1-2 cells
- k-medoids may not work properly on extremely imbalanced datasets containing both rare and prevalent cell types, requiring manual cluster merging
- Totem cannot generate disconnected trajectories or trajectories that have cycles
- dynplot will not generate the correct pseudotime for multi-root (converging) trajectories, requiring a manual work-around

## Evidence

- [results] Totem's robustness improved as the number of clusterings increased, with small variation when the number of clusterings exceeded 2500 (5000 or 7500).: [evidence span withheld — anchor status: not_found]
- [intro] min.cluster.size filter: "The second parameter (`min.cluster.size`) determines the clusterings that are filtered out because they include too small clusters."
- [intro] CLARA clustering default 10k: "the next step is to perform clustering that involves running the k-medoids (CLARA) algorithm a large number of times (default 10k)."
- [readme] Totem repo: [evidence span withheld — anchor status: external_doc]
- [intro] MST via Slingshot: "In trajectory inference methods like `Slingshot`[2] that require a clustering to construct the Minimum Spanning Tree (`MST`) as the backbone of the trajectory"
