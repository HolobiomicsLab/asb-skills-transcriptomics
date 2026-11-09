---
name: kde-jaccard-color-similarity-scoring
description: Use when benchmarking spatially aware colorization (e.g., Spaco vs. baseline
  palettes) on simulated 2D cluster pairs and you need a density-overlap color-difference
  score that is expected to be zero for spatially non-overlapping cluster pairs.
when_to_use_negative:
- When you need to distinguish isolated from distantly isolated cluster pairs or grade
  differences by spatial distance — KDE-Jaccard scored zero for all non-overlapping
  pairs, whereas DOI and LISI successfully distinguished the two isolated cases from
  the non-isolated ones
- When evaluating single-cell spatial colorization of 3D SRT data, where the paper
  notes fine-tuning of parameters is crucial to differentiate 'sparse' from 'dense'
  cell types
- When measuring batch correction or label mixing quality in scRNA-seq integration
  — use LISI via scPOP in its native context instead of this color-overlap scoring
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Spaco
  role: Spatially aware colorization whose output color assignments are scored with
    the KDE-Jaccard metric
  repo: github.com/BrainStOrmics/Spaco
- name: Spaco_scripts
  role: Deposited data-generation and benchmarking scripts used to simulate 2D cluster
    pairs and assemble the Figure 2B/2C score table
  repo: github.com/BrainStOrmics/Spaco_scripts
- name: scikit-image
  role: 2D density estimation of cluster point distributions prior to Jaccard overlap
    computation
- name: umap-learn
  role: Neighbor graph construction for the LISI comparison metric computed alongside
    KDE-Jaccard
- name: scPOP
  role: Reference implementation of the Local Inverse Simpson Index (LISI) metric
    family used for benchmark comparison
  repo: github.com/vinay-swamy/scPOP
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:c5408f9c2ba86238f41e22be1327cd4440b0a85e24f752b78fdb4accff167d44
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - Spaco
  - Spaco_scripts
  - scPOP
  - scikit-image
  - umap-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# KDE-Jaccard color similarity scoring

## Summary

A benchmark metric from the Spaco paper that scores color similarity between two spatial cluster pairs by estimating each cluster's 2D density (scikit-image density estimation) and computing the Jaccard overlap of the resulting density regions, used alongside DOI, Wasserstein distance, and LISI to evaluate colorization quality.

## When to use

Use when benchmarking spatially aware colorization (e.g., Spaco vs. baseline palettes) on simulated 2D cluster pairs and you need a density-overlap color-difference score that is expected to be zero for spatially non-overlapping cluster pairs.

## When NOT to use

- When you need to distinguish isolated from distantly isolated cluster pairs or grade differences by spatial distance — KDE-Jaccard scored zero for all non-overlapping pairs, whereas DOI and LISI successfully distinguished the two isolated cases from the non-isolated ones
- When evaluating single-cell spatial colorization of 3D SRT data, where the paper notes fine-tuning of parameters is crucial to differentiate 'sparse' from 'dense' cell types
- When measuring batch correction or label mixing quality in scRNA-seq integration — use LISI via scPOP in its native context instead of this color-overlap scoring

## Inputs

- Simulated or real 2D two-cluster point pairs (cell coordinates + cluster labels) under spatial relationships: mixed, partially interlaced, adjacent, isolated, distantly isolated (dense and sparse variants)
- Color assignments per cluster from spaco.colorize and baseline default palettes
- Spaco environment: python>=3.8, numpy pinned to 1.22.x or 1.23.x, scipy, scikit-learn, scikit-image, umap-learn

## Outputs

- Per-cluster-pair KDE-Jaccard color-difference scores for every colorization method
- Normalized score table figure_2BC_normalized_scores.csv with DOI/Wasserstein/LISI/KDE-Jaccard columns for all five relationships in dense and sparse variants (Figure 2B/2C layout)

## How to apply

Simulate or obtain 2D two-cluster point sets with labeled cell coordinates and colorizations from spaco.colorize (cell coordinates, cluster labels, palette, radius, n_neighbors). For each cluster pair, estimate each cluster's 2D density with scikit-image density estimation, binarize or compare the density regions, and compute the Jaccard overlap as the color-difference score. Normalize scores per metric together with the DOI, Wasserstein, and LISI (umap-learn neighbor graph) columns and assemble them in the Figure 2B/2C layout, writing figure_2BC_normalized_scores.csv. The rationale is that KDE-Jaccard captures whether color distributions overlap in dense regions, but as the Spaco authors found, it scores zero for every non-overlapping pair, so interpret it as a coarse overlap indicator rather than a graded spatial-distance measure like DOI or Wasserstein.

## Related tools

- **Spaco** (Spatially aware colorization whose output color assignments are scored with the KDE-Jaccard metric) — github.com/BrainStOrmics/Spaco
- **Spaco_scripts** (Deposited data-generation and benchmarking scripts used to simulate 2D cluster pairs and assemble the Figure 2B/2C score table) — github.com/BrainStOrmics/Spaco_scripts
- **scikit-image** (2D density estimation of cluster point distributions prior to Jaccard overlap computation)
- **umap-learn** (Neighbor graph construction for the LISI comparison metric computed alongside KDE-Jaccard)
- **scPOP** (Reference implementation of the Local Inverse Simpson Index (LISI) metric family used for benchmark comparison) — github.com/vinay-swamy/scPOP

## Evaluation signals

- KDE-Jaccard score is exactly zero for every non-overlapping pair (isolated, distantly isolated) and non-zero for overlapping pairs (mixed, partially interlaced, adjacent)
- figure_2BC_normalized_scores.csv exists, parses with pandas, and contains DOI/Wasserstein/LISI/KDE-Jaccard columns for all five spatial relationships in both dense and sparse variants
- Scores are normalized per metric before cross-metric comparison; the table reproduces the Figure 2B/2C trend in which spatially aware assignment yields higher normalized color difference for interlaced cluster pairs
- DOI and LISI columns distinguish the two isolated cases from the three non-isolated cases, providing the expected contrast with the flat KDE-Jaccard zeros
- Re-running with a different numpy version (not 1.22.x/1.23.x) is expected to alter graph-guided colorization results, so verify the environment pin before scoring

## Limitations

- The 2D KDE scored zero for all non-overlapping pairs, making it insensitive to how far apart two isolated clusters are — Wasserstein distance, by contrast, was overly sensitive to spatial distance
- Numpy version (1.22.x or 1.23.x) influences the result of graph-guided mode of Spaco, so downstream KDE-Jaccard scores on Spaco colorizations are environment-dependent
- Spaco's 3D colorization requires fine-tuning of parameters to differentiate 'sparse' from 'dense' cell types, so KDE-Jaccard evaluation on 3D data is not directly transferable

## Evidence

- [abstract] Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments: "Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments"
- [methods] pip install Spaco from GitHub: "pip install git+https://github.com/BrainStOrmics/Spaco.git"
- [methods] Scripts to reproduce benchmarking and analytic results in Spaco paper are in repository [Spaco_scripts](https://github.com/BrainStOrmics/Spaco_scripts): "Scripts to reproduce benchmarking and analytic results in Spaco paper are in repository [Spaco_scripts](https://github.com/BrainStOrmics/Spaco_scripts)"
- [methods] numpy version influences graph-guided mode: "numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco"
- [readme] README description of Spaco's spatially constrained colorization: "Spaco (spa)tial (co)lorization) provides a spatially constrained approach that generates discriminate color assignments for visualizing single-cell spatial data in various scenarios."
- [readme] scPOP README LISI definition: "Local Inverse Simpson Index (LISI): (avg) Number of labels present
    in the N nearest cells"
