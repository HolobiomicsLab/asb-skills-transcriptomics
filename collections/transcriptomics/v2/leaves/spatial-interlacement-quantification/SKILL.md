---
name: spatial-interlacement-quantification
description: Use when analyzing cellular-resolution SRT data (e.g., STARmap, seqFISH,
  Stereo-seq, IMC) to quantify how interlaced two cell-type clusters are, or to verify
  that DOI-guided colorization yields color differences proportional to spatial interlacement.
when_to_use_negative:
- Data lack single-cell spatial coordinates (e.g., bulk or non-spatial scRNA-seq)
  — the CI-graph and DOI require spatial interlacement between clusters
- Goal is to score batch correction or label agreement (e.g., ARI/NMI/LISI via scPOP)
  rather than spatial relationships between cell types
- A fixed color mapping is desired that respects literature conventions or user preferences
  rather than spatially driven color differences (a stated limitation of Spaco)
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Spaco
  role: Constructs the CI-graph, computes the Degree of Interlacement, and performs
    spatially aware cluster-color assignment
  repo: https://github.com/BrainStOrmics/Spaco
- name: Spaco_scripts
  role: Scripts to reproduce the benchmarking and analytic results of the Spaco paper,
    including the DOI–color-difference regression
  repo: https://github.com/BrainStOrmics/Spaco_scripts
- name: Python
  role: Runs the DOI computation, CIEDE2000 color-difference calculation, and scipy.stats
    OLS regression
- name: matplotlib
  role: Draws the scatter plot of DOI vs. realized color difference with the fitted
    regression line
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:281f5eb2173f35271916d593e545170af1321fe1dea6949e3257a298753694be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BrainStOrmics/Spaco
  tools:
  - Python
  - Spaco
  - Spaco_scripts
  - matplotlib
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# spatial-interlacement-quantification

## Summary

Quantifies the spatial relationship between cell-type pairs in single-cell spatial transcriptomics (SRT) data using Spaco's Degree of Interlacement (DOI) metric and cluster interlacement graph (CI-graph), and validates its contribution to color assignment by regressing realized pairwise color differences (CIEDE2000) on DOI scores.

## When to use

Use when analyzing cellular-resolution SRT data (e.g., STARmap, seqFISH, Stereo-seq, IMC) to quantify how interlaced two cell-type clusters are, or to verify that DOI-guided colorization yields color differences proportional to spatial interlacement.

## When NOT to use

- Data lack single-cell spatial coordinates (e.g., bulk or non-spatial scRNA-seq) — the CI-graph and DOI require spatial interlacement between clusters
- Goal is to score batch correction or label agreement (e.g., ARI/NMI/LISI via scPOP) rather than spatial relationships between cell types
- A fixed color mapping is desired that respects literature conventions or user preferences rather than spatially driven color differences (a stated limitation of Spaco)

## Inputs

- Cell-type/cluster annotation table or AnnData object from a cellular-resolution SRT dataset (e.g., STARmap mouse brain dataset)
- Spaco-assigned colors per cell type from a graph-guided color-assignment run
- DOI scores for cell-type pairs derived from the CI-graph
- Python environment with Spaco, scipy>=1.10.0, pyciede2000==0.0.21, matplotlib

## Outputs

- doi_color_regression.csv containing regression R-square and p-value of color difference vs. DOI
- Pairwise table of DOI scores and realized CIEDE2000 color differences per cell-type pair
- Optional scatter plot of DOI vs. color difference with fitted OLS line (matplotlib)

## How to apply

Load the cell-type segmentation/cluster labels of the SRT sample and build the weighted cluster interlacement graph (CI-graph) with Spaco's first colorization module, computing the Degree of Interlacement for every cell-type pair to capture spatial relationships ranging from isolated to fully intertwined. From a Spaco color-assignment run (e.g., on the STARmap mouse brain dataset), compute the realized pairwise color differences between assigned colors in a perceptual color space using CIEDE2000 (pyciede2000==0.0.21). Pair each realized color difference with the DOI score of the same cell-type pair, then fit an ordinary least-squares linear regression of color difference on DOI with scipy.stats (scipy>=1.10.0). Record R-square and p-value (benchmark: R-square = 0.45, p = 4e-61; acceptance window R-square 0.40–0.50 and p of the same order of magnitude) into doi_color_regression.csv, optionally drawing a scatter plot with the fitted line via matplotlib. A significant positive fit indicates that Spaco's optimization effectively used the DOI in its cluster-color assignment.

## Related tools

- **Spaco** (Constructs the CI-graph, computes the Degree of Interlacement, and performs spatially aware cluster-color assignment) — https://github.com/BrainStOrmics/Spaco
- **Spaco_scripts** (Scripts to reproduce the benchmarking and analytic results of the Spaco paper, including the DOI–color-difference regression) — https://github.com/BrainStOrmics/Spaco_scripts
- **Python** (Runs the DOI computation, CIEDE2000 color-difference calculation, and scipy.stats OLS regression)
- **matplotlib** (Draws the scatter plot of DOI vs. realized color difference with the fitted regression line)

## Examples

```
from scipy.stats import linregress; res = linregress(doi_scores, ciede2000_color_diffs); print(res.rvalue**2, res.pvalue)
```

## Evaluation signals

- Reproduced regression statistics fall within R-square 0.40–0.50 (target 0.45) and p-value of the same order of magnitude as 4e-61
- Each realized CIEDE2000 color difference is correctly paired with the DOI score of the identical cell-type pair from the CI-graph
- doi_color_regression.csv contains both R-square and p-value fields populated from scipy.stats output
- CIEDE2000 computation uses pyciede2000==0.0.21, matching Spaco's dependency, applied in a perceptual color space
- Regression is OLS of realized color difference on DOI (not the reverse) fitted with scipy.stats (scipy>=1.10.0)

## Limitations

- numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco, which feeds the DOI/color assignments
- Spaco enforces a correspondence between color differences and the spatial distribution of cells, which may not match user preferences or literature conventions
- Visual contrast obtainable through color space is limited when performing CVD-supporting colorization or when coloring hundreds of clusters, weakening the DOI–color-difference relationship in extreme cases

## Evidence

- [abstract] Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments: "Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments"
- [intro] a weighted graph, referred to as the cluster interlacement graph (CI-graph), is constructed to globally model these relationships: "a weighted graph, referred to as the cluster interlacement graph (CI-graph), is constructed to globally model these relationships."
- [methods] color_mapping = spaco.colorize(: "color_mapping = spaco.colorize("
- [readme] Scripts to reproduce benchmarking and analytic results in Spaco paper are in repository Spaco_scripts: "Scripts to reproduce benchmarking and analytic results in Spaco paper are in repository [Spaco_scripts](https://github.com/BrainStOrmics/Spaco_scripts)"
- [discussion] One major limitation of Spaco is its enforcement of a correspondence between color differences and the spatial distribution of cells.: "One major limitation of Spaco is its enforcement of a correspondence between color differences and the spatial distribution of cells."
