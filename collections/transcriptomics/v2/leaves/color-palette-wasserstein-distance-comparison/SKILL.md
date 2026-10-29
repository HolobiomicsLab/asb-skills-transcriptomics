---
name: color-palette-wasserstein-distance-comparison
description: Use when you need to quantitatively evaluate whether a spatial colorization
  method (e.g., Spaco vs. default palettes) assigns more contrasting colors to spatially
  interlaced cell-type pairs, using distributional color metrics.
when_to_use_negative:
- When the goal is benchmarking batch correction of single-cell RNA-seq embeddings
  — the scPOP package's LISI/ARI/NMI implementation targets that setting, not color-difference
  scoring
- When evaluating colorization on real spatial transcriptomics data only — this skill's
  benchmark design depends on simulated cluster pairs with known ground-truth spatial
  relationships
- When clusters are non-overlapping and only 2D KDE-Jaccard is used — it scores zero
  for all non-overlapping pairs and cannot distinguish isolated from adjacent relationships
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Spaco
  role: Provides the spatially aware colorization (spaco.colorize) being benchmarked
    and the DOI metric used as the reference spatial-relationship measure
  repo: https://github.com/BrainStOrmics/Spaco
- name: Spaco_scripts
  role: Deposited scripts for generating the simulated 2D cluster pairs and reproducing
    the benchmarking results
  repo: https://github.com/BrainStOrmics/Spaco_scripts
- name: Python
  role: Implementation environment for DOI, Wasserstein distance, LISI (umap-learn
    neighbor graph), and 2D KDE-Jaccard (scikit-image) computations
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:2a46a09464ef3e7d45c21f659db15a3668eb57594d0288bfdea4371c4a9a6360
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BrainStOrmics/Spaco
  tools:
  - Python
  - Spaco
  - Spaco_scripts
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# color-palette-wasserstein-distance-comparison

## Summary

Benchmark colorization quality by computing Wasserstein distance, LISI, and 2D KDE-Jaccard color-difference scores for simulated 2D cluster pairs (mixed, partially interlaced, adjacent, isolated, distantly isolated) and comparing them against the Degree of Interlacement (DOI) metric from the Spaco paper.

## When to use

Use when you need to quantitatively evaluate whether a spatial colorization method (e.g., Spaco vs. default palettes) assigns more contrasting colors to spatially interlaced cell-type pairs, using distributional color metrics.

## When NOT to use

- When the goal is benchmarking batch correction of single-cell RNA-seq embeddings — the scPOP package's LISI/ARI/NMI implementation targets that setting, not color-difference scoring
- When evaluating colorization on real spatial transcriptomics data only — this skill's benchmark design depends on simulated cluster pairs with known ground-truth spatial relationships
- When clusters are non-overlapping and only 2D KDE-Jaccard is used — it scores zero for all non-overlapping pairs and cannot distinguish isolated from adjacent relationships

## Inputs

- Simulated 2D two-cluster point pairs with cluster labels (dense and sparse variants of mixed, partially interlaced, adjacent, isolated, distantly isolated relationships)
- Colorized outputs from spaco.colorize (cell coordinates, cluster labels, assigned palette colors, radius, n_neighbors)
- Baseline default-palette colorizations for comparison

## Outputs

- figure_2BC_normalized_scores.csv: per-metric normalized color-difference table with DOI/Wasserstein/LISI/KDE-Jaccard columns for all five relationships in dense and sparse variants
- Per-cluster-pair DOI values computed from simulated point coordinates
- Figure 2B/2C layout reproducing the normalized-score trend

## How to apply

Simulate 2D two-cluster point pairs under five spatial relationships (mixed, overlapped/partially interlaced, adjacent, isolated, distantly isolated) in dense and sparse variants using the Spaco_scripts data-generation scripts. Colorize each pair with spaco.colorize (cell coordinates, cluster labels, palette, radius, n_neighbors) and with baseline default palettes, pinning numpy to 1.22.x or 1.23.x since the numpy version influences graph-guided colorization results. For each cluster pair and method, compute pairwise color-difference scores: Wasserstein distance between per-cluster color distributions, LISI on a umap-learn neighbor graph, and 2D KDE-Jaccard via scikit-image density estimation. Compute the DOI for every pair from the point coordinates, normalize all scores per metric, and assemble them in a Figure 2B/2C-style table (figure_2BC_normalized_scores.csv). Interpretation follows the paper's finding: DOI and LISI distinguish the two isolated cases from non-isolated ones, 2D KDE scores zero for all non-overlapping pairs, and Wasserstein distance is overly sensitive to spatial distance.

## Related tools

- **Spaco** (Provides the spatially aware colorization (spaco.colorize) being benchmarked and the DOI metric used as the reference spatial-relationship measure) — https://github.com/BrainStOrmics/Spaco
- **Spaco_scripts** (Deposited scripts for generating the simulated 2D cluster pairs and reproducing the benchmarking results) — https://github.com/BrainStOrmics/Spaco_scripts
- **Python** (Implementation environment for DOI, Wasserstein distance, LISI (umap-learn neighbor graph), and 2D KDE-Jaccard (scikit-image) computations)

## Examples

```
pip install git+https://github.com/BrainStOrmics/Spaco.git; color_mapping = spaco.colorize(adata, radius=..., n_neighbors=...)
```

## Evaluation signals

- figure_2BC_normalized_scores.csv exists and parses with pandas
- The table contains DOI/Wasserstein/LISI/KDE-Jaccard columns for all five spatial relationships in both dense and sparse variants
- The table reproduces the Figure 2B/2C trend: spatially aware assignment yields higher normalized color difference for interlaced cluster pairs
- DOI and LISI both successfully distinguish the two isolated cases (isolated, distantly isolated) from non-isolated ones (mixed, partially interlaced, adjacent)
- 2D KDE-Jaccard scores are zero for all non-overlapping pairs, confirming the expected failure mode

## Limitations

- Wasserstein distance is overly sensitive to spatial distance, confounding spatial separation with color difference
- The 2D KDE-Jaccard metric scores zero for all non-overlapping pairs, so it cannot discriminate among non-interlaced relationships
- numpy version (1.22.x or 1.23.x) influences the result of the graph-guided mode of Spaco, so colorization results are environment-dependent

## Evidence

- [abstract] Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments: "Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types"
- [methods] numpy version note: "numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco"
- [methods] benchmark scripts repository: "Scripts to reproduce benchmarking and analytic results in Spaco paper are in repository [Spaco_scripts]"
- [readme] LISI description from scPOP README: [evidence span withheld — anchor status: external_doc]
- [results] benchmark finding on simulated pairs: [evidence span withheld — anchor status: not_found]
