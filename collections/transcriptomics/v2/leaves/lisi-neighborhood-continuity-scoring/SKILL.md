---
name: lisi-neighborhood-continuity-scoring
description: Use when benchmarking spatially aware color assignments or spatial proximity
  metrics on 2D single-cell/cluster point data, and you need a neighborhood-mixing
  score that separates mixed/interlaced pairs from isolated ones.
when_to_use_negative:
- When the input is not spatially embedded point data with per-cell cluster labels
  — LISI requires coordinates plus labels to define neighborhoods
- When the goal is to measure absolute spatial separation regardless of interlacement
  — Wasserstein distance is the more appropriate (though distance-sensitive) metric
  in this benchmark
- When clusters overlap completely or not at all and a binary overlap measure suffices
  — 2D KDE-Jaccard scored zero for all non-overlapping pairs and is not informative
  there
edam_operation: http://edamontology.org/operation_2945
tools:
- name: Spaco
  role: Source of simulated cluster pairs and the DOI metric; colorization whose outputs
    LISI evaluates
  repo: github.com/BrainStOrmics/Spaco
- name: Python
  role: Implementation language; umap-learn neighbor graph and scikit-image/scipy
    used for comparison metrics
- name: scPOP
  role: Reference R implementation of the LISI metric (Local Inverse Simpson Index
    over N nearest cells)
  repo: vinay-swamy/scPOP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:c210bc26adfef1eeba65d17e98d4d99490a6bc4e328d1e7335194225f48550ac
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Spaco
  - scPOP
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# lisi-neighborhood-continuity-scoring

## Summary

Use the Local Inverse Simpson Index (LISI), computed from a umap-learn neighbor graph over spatial point coordinates, to score how continuously two cluster distributions interleave — here repurposed in the Spaco benchmark as a color-difference evaluation metric to distinguish isolated from interlaced 2D cluster pairs.

## When to use

Use when benchmarking spatially aware color assignments or spatial proximity metrics on 2D single-cell/cluster point data, and you need a neighborhood-mixing score that separates mixed/interlaced pairs from isolated ones.

## When NOT to use

- When the input is not spatially embedded point data with per-cell cluster labels — LISI requires coordinates plus labels to define neighborhoods
- When the goal is to measure absolute spatial separation regardless of interlacement — Wasserstein distance is the more appropriate (though distance-sensitive) metric in this benchmark
- When clusters overlap completely or not at all and a binary overlap measure suffices — 2D KDE-Jaccard scored zero for all non-overlapping pairs and is not informative there

## Inputs

- Simulated 2D two-cluster point pairs (dense and sparse variants) with cluster labels under five spatial relationships (mixed, overlapped, adjacent, isolated, distantly isolated)
- Colorized versions of each pair from spaco.colorize and baseline default palettes
- DOI scores computed from the same point coordinates (for per-metric normalization context)

## Outputs

- Per-pair LISI continuity scores for every cluster pair and colorization method
- figure_2BC_normalized_scores.csv with normalized DOI/Wasserstein/LISI/KDE-Jaccard columns for all five relationships in dense and sparse variants

## How to apply

For each simulated two-cluster pair (mixed, partially interlaced, adjacent, isolated, distantly isolated), build a neighbor graph over the point coordinates with umap-learn and compute LISI, which measures the average number of distinct labels present among the N nearest cells. Treat LISI as a color-difference/continuity score per cluster pair per colorization method (Spaco spatially aware assignment vs. baseline default palettes). Normalize LISI scores per metric alongside DOI, Wasserstein distance, and 2D KDE-Jaccard before assembling the Figure 2B/2C comparison table. Judge correctness by whether LISI, like DOI, successfully distinguishes the two isolated cases from the non-isolated ones, whereas 2D KDE scores zero for all non-overlapping pairs and Wasserstein is overly sensitive to spatial distance.

## Related tools

- **Spaco** (Source of simulated cluster pairs and the DOI metric; colorization whose outputs LISI evaluates) — github.com/BrainStOrmics/Spaco
- **Python** (Implementation language; umap-learn neighbor graph and scikit-image/scipy used for comparison metrics)
- **scPOP** (Reference R implementation of the LISI metric (Local Inverse Simpson Index over N nearest cells)) — vinay-swamy/scPOP

## Examples

```
pip install umap-learn; knn = umap.umap_.nearest_neighbors(coords, n_neighbors=15, metric='euclidean'); lisi = compute_lisi(knn, labels, k=15)
```

## Evaluation signals

- LISI scores separate the two isolated cases (isolated, distantly isolated) from the three non-isolated cases (mixed, partially interlaced, adjacent) in both dense and sparse variants
- figure_2BC_normalized_scores.csv exists, parses with pandas, and contains DOI/Wasserstein/LISI/KDE-Jaccard columns for all five spatial relationships
- Normalized LISI tracks the same trend as DOI: higher normalized color difference for interlaced cluster pairs
- Per-metric normalization applied before cross-metric comparison so scores are on a common scale

## Limitations

- LISI captures neighborhood label mixing but, unlike DOI, was not designed for spatial colorization evaluation; its interpretation depends on N-neighbor count
- In scPOP usage, the 'optimal' LISI score direction depends on the label type (high is better for batch mixing, low for cell-type separation), so score direction must be fixed deliberately
- LISI requires ~10 GB of memory for 100K cells and scales linearly with cell number

## Evidence

- [other] both DOI and LISI successfully distinguished the two isolated cases (isolated and distantly isolated) from the non-isolated ones (mixed, partially interlaced, adjacent): [evidence span withheld — anchor status: not_found]
- [other] the Wasserstein distance was overly sensitive to spatial distance: [evidence span withheld — anchor status: not_found]
- [readme] Local Inverse Simpson Index (LISI): (avg) Number of labels present in the N nearest cells: [evidence span withheld — anchor status: external_doc]
- [readme] Lisi requires about 10gb of memory for 100K cells and scales linearly with number of cells: [evidence span withheld — anchor status: external_doc]
- [abstract] Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types: "Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments"
