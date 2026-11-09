---
name: benchmark-score-normalization-tabulation
description: Use when comparing colorization or spatial-metric performance across
  cluster-pair configurations and you need per-metric normalized scores tabulated
  (e.g., a Figure 2B/2C-style benchmark of DOI vs Wasserstein vs LISI vs KDE-Jaccard).
when_to_use_negative:
- When the goal is visual quality assessment of a single colorized image rather than
  a quantitative multi-metric benchmark across cluster-pair configurations
- When 2D KDE-Jaccard is the sole metric of interest for non-overlapping cluster pairs
  — it scores zero for all non-overlapping pairs, so it cannot distinguish isolated
  from adjacent configurations
- When data are 3D spatial transcriptomics volumes requiring parameter fine-tuning
  to differentiate 'sparse' from 'dense' cell types, which the paper flags as an unresolved
  reproducibility gap
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Spaco
  role: Colorizes each simulated cluster pair via spaco.colorize (cell coordinates,
    cluster labels, palette, radius, n_neighbors) and supplies the DOI metric computed
    from point coordinates
  repo: https://github.com/BrainStOrmics/Spaco
- name: Python
  role: 'Runtime environment for the benchmark: pip install from GitHub source with
    numpy pinned to 1.22.x or 1.23.x; computes Wasserstein, LISI (umap-learn neighbor
    graph), and 2D KDE-Jaccard (scikit-image) scores and writes the CSV'
  repo: https://github.com/BrainStOrmics/Spaco_scripts
- name: scPOP
  role: R package providing the LISI metric implementation and metric-aggregation
    utilities (calc_sumZscore) referenced for normalization-style comparisons
  repo: https://github.com/vinay-swamy/scPOP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:714cdf302c07f004d678df52026d79eecd98c2d8c91bfd7240f45fad0d8495e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BrainStOrmics/Spaco
  tools:
  - Python
  - Spaco
  - scPOP
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# Benchmark score normalization and tabulation

## Summary

A workflow skill for computing multiple pairwise color-difference metrics (Wasserstein distance, LISI, 2D KDE-Jaccard, and the Degree of Interlacement DOI) on simulated 2D cluster pairs, normalizing each metric's scores, and assembling them into a per-relationship, per-variant CSV table used to reproduce the Spaco paper's Figure 2B/2C benchmark.

## When to use

Use when comparing colorization or spatial-metric performance across cluster-pair configurations and you need per-metric normalized scores tabulated (e.g., a Figure 2B/2C-style benchmark of DOI vs Wasserstein vs LISI vs KDE-Jaccard).

## When NOT to use

- When the goal is visual quality assessment of a single colorized image rather than a quantitative multi-metric benchmark across cluster-pair configurations
- When 2D KDE-Jaccard is the sole metric of interest for non-overlapping cluster pairs — it scores zero for all non-overlapping pairs, so it cannot distinguish isolated from adjacent configurations
- When data are 3D spatial transcriptomics volumes requiring parameter fine-tuning to differentiate 'sparse' from 'dense' cell types, which the paper flags as an unresolved reproducibility gap

## Inputs

- Simulated 2D two-cluster point-pair coordinates with cluster labels under five spatial relationships (mixed, overlapped/partially interlaced, adjacent, isolated, distantly isolated) in dense and sparse variants
- Color assignments from spaco.colorize and from baseline default palettes for each cluster pair
- Python environment: python>=3.8, numpy (pinned 1.22.x or 1.23.x), scipy, scikit-learn, scikit-image, umap-learn

## Outputs

- figure_2BC_normalized_scores.csv containing normalized DOI, Wasserstein, LISI, and KDE-Jaccard scores for all five relationships in both dense and sparse variants
- Figure 2B/2C-style normalized score layout showing metric behavior across spatial relationships

## How to apply

First colorize each simulated two-cluster pair with spaco.colorize (passing cell coordinates, cluster labels, palette, radius, and n_neighbors) and with baseline default palettes. For every cluster pair, compute the Degree of Interlacement (DOI) directly from the simulated point coordinates, then compute pairwise color-difference scores with three baseline metrics: Wasserstein distance, LISI (using the umap-learn neighbor graph), and 2D KDE-Jaccard (scikit-image density estimation plus Jaccard overlap). Normalize the scores per metric so metrics with different ranges are comparable, then assemble them in the Figure 2B/2C layout and write the table to figure_2BC_normalized_scores.csv, covering all five spatial relationships (mixed, partially interlaced, adjacent, isolated, distantly isolated) in both dense and sparse variants. Pin numpy to 1.22.x or 1.23.x because the numpy version influences graph-guided colorization results. Interpret the table by checking whether DOI and LISI separate the isolated cases from the non-isolated ones while noting that 2D KDE scores zero for all non-overlapping pairs and Wasserstein is overly sensitive to spatial distance.

## Related tools

- **Spaco** (Colorizes each simulated cluster pair via spaco.colorize (cell coordinates, cluster labels, palette, radius, n_neighbors) and supplies the DOI metric computed from point coordinates) — https://github.com/BrainStOrmics/Spaco
- **Python** (Runtime environment for the benchmark: pip install from GitHub source with numpy pinned to 1.22.x or 1.23.x; computes Wasserstein, LISI (umap-learn neighbor graph), and 2D KDE-Jaccard (scikit-image) scores and writes the CSV) — https://github.com/BrainStOrmics/Spaco_scripts
- **scPOP** (R package providing the LISI metric implementation and metric-aggregation utilities (calc_sumZscore) referenced for normalization-style comparisons) — https://github.com/vinay-swamy/scPOP

## Examples

```
pip install git+https://github.com/BrainStOrmics/Spaco.git "numpy==1.23.*" && python -c "import spaco; m = spaco.colorize(coordinates, labels, palette, radius=30, n_neighbors=30)"
```

## Evaluation signals

- figure_2BC_normalized_scores.csv exists and parses with pandas
- The CSV contains DOI, Wasserstein, LISI, and KDE-Jaccard columns for all five spatial relationships in both dense and sparse variants
- The table reproduces the Figure 2B/2C trend in which the spatially aware assignment yields higher normalized color difference for interlaced cluster pairs
- DOI and LISI successfully separate the two isolated cases (isolated and distantly isolated) from the non-isolated ones (mixed, partially interlaced, adjacent)
- 2D KDE-Jaccard scores equal zero for all non-overlapping pairs, and Wasserstein scores scale with spatial distance, matching the expected metric behaviors

## Limitations

- The numpy version (1.22.x or 1.23.x) influences the result of graph-guided mode of Spaco, so scores are environment-dependent
- Wasserstein distance is overly sensitive to spatial distance and 2D KDE scores zero for all non-overlapping pairs, limiting their discriminative value in this benchmark
- Spaco's enforcement of a correspondence between color differences and spatial distribution may not match user preferences or literature conventions

## Evidence

- [results] On simulated dense cluster pairs, both DOI and LISI successfully distinguished the two isolated cases from the non-isolated ones, whereas the 2D KDE scored zero for all non-overlapping pairs.: [evidence span withheld — anchor status: not_found]
- [methods] numpy version influences graph-guided colorization results.: "numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco"
- [abstract] Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types.: "Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments"
- [methods] Scripts to reproduce benchmarking and analytic results in Spaco paper are in the Spaco_scripts repository.: "Scripts to reproduce benchmarking and analytic results in Spaco paper are in repository [Spaco_scripts](https://github.com/BrainStOrmics/Spaco_scripts)"
- [readme] LISI: (avg) Number of labels present in the N nearest cells, from the scPOP README.: "Local Inverse Simpson Index (LISI): (avg) Number of labels present
    in the N nearest cells"
