---
name: spatial-point-pattern-simulation
description: Use when you need synthetic ground-truth 2D point-cluster layouts with
  known spatial relationships to validate or compare spatial-awareness metrics (DOI,
  Wasserstein, LISI, KDE-Jaccard) or spatial colorization algorithms like Spaco.
when_to_use_negative:
- When the goal is comparing real SRT datasets (e.g. STARmap, Stereo-seq, seqFISH,
  IMC) rather than controlled metric validation — benchmark directly on the real data
  instead
- 'When 3D spatial colorization is the target: the paper notes fine-tuning of parameters
  is crucial to differentiate ''sparse'' from ''dense'' cell types for efficient 3D
  colorization, so 2D simulation results do not transfer directly'
- When evaluating batch-correction quality on scRNA-seq embeddings — tools like scPOP
  (ARI, NMI, LISI, silhouette width) are designed for that, not spatial point-pattern
  benchmarking
tools:
- name: Spaco
  role: Colorizes each simulated cluster pair via spaco.colorize (cell coordinates,
    cluster labels, palette, radius, n_neighbors); the DOI metric it uses is the reference
    spatial metric being benchmarked
  repo: https://github.com/BrainStOrmics/Spaco
- name: Spaco_scripts
  role: Deposited data-generation scripts used to simulate 2D two-cluster point pairs
    under the five spatial relationships and reproduce the paper's benchmarking figures
  repo: https://github.com/BrainStOrmics/Spaco_scripts
- name: scPOP
  role: Source of the LISI (Local Inverse Simpson Index) metric definition used as
    a comparison metric for cluster interlacement
  repo: https://github.com/vinay-swamy/scPOP
- name: rpy2
  role: Python-to-R bridge enabling Python-based Spaco colorization results to be
    evaluated alongside R packages (e.g. LISI from scPOP)
  repo: https://github.com/rpy2/rpy2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:d1884589edf58c1ce9416ee6ffaf5380e523456a4942e85fbd8b6f0c08f5bea6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BrainStOrmics/Spaco
  tools:
  - Python
  - Spaco
  - Spaco_scripts
  - rpy2
  - scPOP
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# spatial-point-pattern-simulation

## Summary

Simulates 2D two-cluster point pairs under controlled spatial relationships (mixed, partially interlaced, adjacent, isolated, distantly isolated) in dense and sparse variants, to benchmark spatial metrics such as the Degree of Interlacement (DOI), Wasserstein distance, LISI, and 2D KDE-Jaccard used by Spaco.

## When to use

Use when you need synthetic ground-truth 2D point-cluster layouts with known spatial relationships to validate or compare spatial-awareness metrics (DOI, Wasserstein, LISI, KDE-Jaccard) or spatial colorization algorithms like Spaco.

## When NOT to use

- When the goal is comparing real SRT datasets (e.g. STARmap, Stereo-seq, seqFISH, IMC) rather than controlled metric validation — benchmark directly on the real data instead
- When 3D spatial colorization is the target: the paper notes fine-tuning of parameters is crucial to differentiate 'sparse' from 'dense' cell types for efficient 3D colorization, so 2D simulation results do not transfer directly
- When evaluating batch-correction quality on scRNA-seq embeddings — tools like scPOP (ARI, NMI, LISI, silhouette width) are designed for that, not spatial point-pattern benchmarking

## Inputs

- Simulated 2D point coordinates for two clusters with controlled spatial relationship (mixed, partially interlaced, adjacent, isolated, distantly isolated)
- Cluster labels per simulated point (dense and sparse variants)
- Color palette specification for Spaco colorization
- Spaco_scripts data-generation scripts from github.com/BrainStOrmics/Spaco_scripts

## Outputs

- figure_2BC_normalized_scores.csv containing per-metric normalized DOI, Wasserstein, LISI, and KDE-Jaccard scores for all five relationships in dense and sparse variants
- Per-pair color-difference score tables for each colorization method (Spaco vs baseline palettes)
- Figure 2B/2C style normalized-score comparison layout

## How to apply

Install Spaco from GitHub source (pip install git+https://github.com/BrainStOrmics/Spaco.git) pinning numpy to 1.22.x or 1.23.x, since the numpy version influences graph-guided colorization results. Run the data-generation scripts from the deposited Spaco_scripts repository to simulate 2D two-cluster point pairs under the five spatial relationships (mixed, overlapped, adjacent, isolated, distantly isolated) in dense and sparse variants. Colorize each pair with spaco.colorize (cell coordinates, cluster labels, palette, radius, n_neighbors) and baseline default palettes, then compute DOI for every cluster pair from the simulated coordinates plus pairwise color-difference scores via Wasserstein distance, LISI (umap-learn neighbor graph), and 2D KDE-Jaccard (scikit-image density estimation plus Jaccard overlap). Normalize scores per metric and assemble them in the Figure 2B/2C layout as figure_2BC_normalized_scores.csv.

## Related tools

- **Spaco** (Colorizes each simulated cluster pair via spaco.colorize (cell coordinates, cluster labels, palette, radius, n_neighbors); the DOI metric it uses is the reference spatial metric being benchmarked) — https://github.com/BrainStOrmics/Spaco
- **Spaco_scripts** (Deposited data-generation scripts used to simulate 2D two-cluster point pairs under the five spatial relationships and reproduce the paper's benchmarking figures) — https://github.com/BrainStOrmics/Spaco_scripts
- **scPOP** (Source of the LISI (Local Inverse Simpson Index) metric definition used as a comparison metric for cluster interlacement) — https://github.com/vinay-swamy/scPOP
- **rpy2** (Python-to-R bridge enabling Python-based Spaco colorization results to be evaluated alongside R packages (e.g. LISI from scPOP)) — https://github.com/rpy2/rpy2

## Examples

```
pip install git+https://github.com/BrainStOrmics/Spaco.git; color_mapping = spaco.colorize(adata_cell, radius=..., n_neighbors=...)
```

## Evaluation signals

- figure_2BC_normalized_scores.csv exists, parses with pandas, and contains DOI/Wasserstein/LISI/KDE-Jaccard columns for all five spatial relationships in both dense and sparse variants
- The results reproduce the Figure 2B/2C trend: spatially aware assignment yields higher normalized color difference for interlaced cluster pairs
- DOI and LISI successfully distinguish the two isolated cases (isolated, distantly isolated) from the non-isolated ones (mixed, partially interlaced, adjacent)
- 2D KDE-Jaccard scores zero for all non-overlapping pairs, and Wasserstein distance is overly sensitive to spatial distance — deviations from these known behaviors indicate a simulation or scoring bug
- Re-running with a different numpy version (not 1.22.x/1.23.x) changes graph-guided colorization results, confirming the environment pin was needed

## Limitations

- numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco, so simulation-to-colorization results are environment-sensitive
- Wasserstein distance is overly sensitive to spatial distance and 2D KDE scores zero for all non-overlapping pairs, so these metrics alone cannot grade isolation gradations
- Spaco enforces a correspondence between color differences and the spatial distribution of cells, which may not match user preferences or literature conventions
- The visual contrast obtainable through color space is limited when performing CVD-supporting colorization or when coloring hundreds of clusters

## Evidence

- [abstract] Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types: "Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types"
- [methods] pip install from GitHub source: "pip install git+https://github.com/BrainStOrmics/Spaco.git"
- [methods] Spaco_scripts repository for benchmarking scripts: "Scripts to reproduce benchmarking and analytic results in Spaco paper are in repository"
- [methods] numpy version sensitivity: "numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco"
- [methods] spaco.colorize API call: "color_mapping = spaco.colorize("
- [readme] README description of spatially constrained colorization: "Spaco (spa tial co lorization) provides a spatially constrained approach that generates discriminate color assignments for visualizing single-cell spatial data"
