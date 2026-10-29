---
name: degree-of-interlacement-quantification
description: Use when you need a scalar score describing how interlaced, adjacent,
  or isolated two cell-type clusters are in single-cell spatial coordinates (e.g.,
  SRT data), or when choosing which spatial-relationship metric to drive color assignment
  or further spatial analysis.
when_to_use_negative:
- When clusters are non-overlapping and you only need distributional shape overlap
  — 2D KDE-Jaccard scores zero for all non-overlapping pairs, so it carries no signal
  there
- When absolute spatial separation magnitude matters — Wasserstein distance is overly
  sensitive to spatial distance and can misrepresent relational interlacement
- For coloring hundreds of clusters or CVD-supporting colorization, where visual contrast
  obtainable through color space is limited regardless of the DOI-driven assignment
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Spaco
  role: Computes the Degree of Interlacement to build the CI-graph and performs spatially
    aware cluster-color assignment via spaco.colorize
  repo: https://github.com/BrainStOrmics/Spaco
- name: Spaco_scripts
  role: Deposited data-generation and benchmarking scripts to simulate the five spatial
    relationships and reproduce the DOI comparison (Figure 2B/2C)
  repo: https://github.com/BrainStOrmics/Spaco_scripts
- name: scPOP
  role: Reference implementation context for the LISI metric used as a comparison
    spatial-mingling score
  repo: https://github.com/vinay-swamy/scPOP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:702931bb4c3cb9ad3683369ac50bb69ac381c968d1a3e28417823a92b064dd8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BrainStOrmics/Spaco
  tools:
  - Python
  - Spaco
  - Spaco_scripts
  - scPOP
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Degree-of-Interlacement quantification

## Summary

Compute the Degree of Interlacement (DOI) metric from Spaco to quantify the overall spatial proximity/interlacement between cell-type (cluster) pairs in spatially resolved data, and benchmark it against alternative pairwise spatial metrics such as Wasserstein distance, LISI, and 2D KDE-Jaccard.

## When to use

Use when you need a scalar score describing how interlaced, adjacent, or isolated two cell-type clusters are in single-cell spatial coordinates (e.g., SRT data), or when choosing which spatial-relationship metric to drive color assignment or further spatial analysis.

## When NOT to use

- When clusters are non-overlapping and you only need distributional shape overlap — 2D KDE-Jaccard scores zero for all non-overlapping pairs, so it carries no signal there
- When absolute spatial separation magnitude matters — Wasserstein distance is overly sensitive to spatial distance and can misrepresent relational interlacement
- For coloring hundreds of clusters or CVD-supporting colorization, where visual contrast obtainable through color space is limited regardless of the DOI-driven assignment

## Inputs

- 2D (or 3D) cell/point coordinates with cluster labels (e.g., from SRT datasets such as STARmap mouse brain, Stereo-seq axolotl brain, seqFISH mouse embryo, or IMC human breast cancer data)
- Simulated two-cluster point pairs under the five spatial relationships (mixed, overlapped, adjacent, isolated, distantly isolated) in dense and sparse variants
- Spaco colorization outputs (color_mapping from spaco.colorize) when DOI is used for palette-driven evaluation

## Outputs

- Per-cluster-pair DOI scores quantifying degree of interlacement
- A weighted cluster interlacement (CI-) graph used for spatially aware color assignment
- Normalized comparison table (e.g., figure_2BC_normalized_scores.csv) with DOI, Wasserstein, LISI, and KDE-Jaccard columns across all five relationships in dense and sparse variants

## How to apply

Compute the DOI for every cluster pair directly from the simulated or measured point coordinates (cell centroid positions plus cluster labels), so that pairs spanning the spectrum of spatial relationships — mixed, partially interlaced, adjacent, isolated, and distantly isolated — receive graded scores reflecting their interlacement. Use the DOI to construct the weighted cluster interlacement (CI-) graph that Spaco uses to globally model spatial relationships before color assignment. When benchmarking DOI against alternatives, normalize scores per metric and compare across dense and sparse variants of the same five spatial relationships: DOI and LISI distinguish the two isolated cases from the non-isolated ones, 2D KDE-Jaccard scores zero for all non-overlapping pairs, and Wasserstein distance is overly sensitive to raw spatial distance. Reproducible execution uses Spaco installed from GitHub source (pip install git+https://github.com/BrainStOrmics/Spaco.git) with numpy pinned to 1.22.x or 1.23.x, since the numpy version influences graph-guided colorization results.

## Related tools

- **Spaco** (Computes the Degree of Interlacement to build the CI-graph and performs spatially aware cluster-color assignment via spaco.colorize) — https://github.com/BrainStOrmics/Spaco
- **Spaco_scripts** (Deposited data-generation and benchmarking scripts to simulate the five spatial relationships and reproduce the DOI comparison (Figure 2B/2C)) — https://github.com/BrainStOrmics/Spaco_scripts
- **scPOP** (Reference implementation context for the LISI metric used as a comparison spatial-mingling score) — https://github.com/vinay-swamy/scPOP

## Examples

```
pip install git+https://github.com/BrainStOrmics/Spaco.git; color_mapping = spaco.colorize(adata_cell, radius, n_neighbors)
```

## Evaluation signals

- figure_2BC_normalized_scores.csv exists, parses with pandas, and contains DOI/Wasserstein/LISI/KDE-Jaccard columns for all five relationships in both dense and sparse variants
- DOI (like LISI) separates the two isolated cases (isolated and distantly isolated) from the non-isolated ones (mixed, partially interlaced, adjacent)
- 2D KDE-Jaccard scores zero for all non-overlapping pairs — used as a negative control for metric sensitivity
- Normalized color difference is higher for interlaced cluster pairs under spatially aware assignment, reproducing the Figure 2B/2C trend
- Pinning numpy to 1.22.x or 1.23.x yields reproducible graph-guided colorization results

## Limitations

- Spaco enforces a correspondence between color differences and the spatial distribution of cells, which may not match user preferences or literature conventions
- numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco
- For 3D data, fine-tuning of parameters is crucial to differentiate 'sparse' from 'dense' cell types for efficient 3D colorization
- Visual contrast obtainable through color space is limited when performing CVD-supporting colorization or in the case of coloring hundreds of clusters

## Evidence

- [abstract] Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments: "Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments"
- [intro] a weighted graph, referred to as the cluster interlacement graph (CI-graph), is constructed to globally model these relationships.: "a weighted graph, referred to as the cluster interlacement graph (CI-graph), is constructed to globally model these relationships."
- [methods] numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco: "numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco"
- [discussion] one major limitation of Spaco is its enforcement of a correspondence between color differences and the spatial distribution of cells.: "One major limitation of Spaco is its enforcement of a correspondence between color differences and the spatial distribution of cells."
- [readme] pip install git+https://github.com/BrainStOrmics/Spaco.git: "pip install git+https://github.com/BrainStOrmics/Spaco.git"
