---
name: ciede2000-color-difference-computation
description: Use when quantifying how visually distinguishable two colors (or two
  whole palettes) are — e.g. adjacent cell-type pairs in spatial-omics plots, image-extracted
  vs. default palettes, or DOI-driven color assignments.
when_to_use_negative:
- When the goal is spatially-aware color assignment itself rather than scoring already-assigned
  colors — use Spaco's graph-guided spaco.colorize instead
- When the palette needs to be extracted from an image while preserving its theme
  color — use Spaco's palette extraction (frequency binning in CIELab + farthest point
  sampling) rather than raw CIEDE2000 scoring
- When cell-type spatial relationships are irrelevant (e.g. purely non-spatial single-cell
  plots) — CIEDE2000 here is justified by adjacency/interlacement structure, not by
  label data alone
tools:
- name: Spaco
  role: Produces the color assignments whose quality is scored with CIEDE2000; pins
    pyciede2000==0.0.21 as its color-difference dependency
  repo: https://github.com/BrainStOrmics/Spaco
- name: scipy
  role: Runs the one-sided Wilcoxon rank-sum/signed-rank tests and OLS regression
    on the CIEDE2000-derived distributions (scipy>=1.10.0)
- name: matplotlib
  role: Renders the pairwise color-difference distributions for visualization (e.g.
    color_difference_distribution.png)
provenance:
  source_task_ids:
  - task_006
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:8a97e145c690c241bbaae2fa70b84b3c029741f3422c9a227baa7b707bcc92d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BrainStOrmics/Spaco
  tools:
  - Python
  - Spaco
  - Squidpy
  - matplotlib
  - numpy
  - pandas
  - scipy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# ciede2000-color-difference-computation

## Summary

Compute perceptual color differences between assigned colors using the CIEDE2000 metric in CIELab space, the standard quantification used by Spaco to benchmark palettes, compare spatially-aware color assignments against baselines (Scanpy default, random, Palo, matplotlib tab20), and regress realized color differences against the Degree of Interlacement (DOI) score.

## When to use

Use when quantifying how visually distinguishable two colors (or two whole palettes) are — e.g. adjacent cell-type pairs in spatial-omics plots, image-extracted vs. default palettes, or DOI-driven color assignments.

## When NOT to use

- When the goal is spatially-aware color assignment itself rather than scoring already-assigned colors — use Spaco's graph-guided spaco.colorize instead
- When the palette needs to be extracted from an image while preserving its theme color — use Spaco's palette extraction (frequency binning in CIELab + farthest point sampling) rather than raw CIEDE2000 scoring
- When cell-type spatial relationships are irrelevant (e.g. purely non-spatial single-cell plots) — CIEDE2000 here is justified by adjacency/interlacement structure, not by label data alone

## Inputs

- Assigned color palette (e.g. Spaco-assigned colors, matplotlib tab20, Scanpy default) as hex/RGB triplets
- List of cell-type pairs to evaluate (e.g. 17 anatomically adjacent cluster pairs from the Allen Brain Atlas annotation)
- DOI (Degree of Interlacement) scores per cell-type pair from the CI-graph (for mechanism regression)
- Field-of-view definitions (e.g. 500x500 um bins over STARmap coordinates)

## Outputs

- Pairwise CIEDE2000 color-difference matrix/distribution per palette
- Mean color difference per field of view per method
- Wilcoxon test p-values comparing color-difference distributions (e.g. written to wilcoxon_pvalues.csv)
- OLS regression R-square and p-value for DOI vs. realized color difference (doi_color_regression.csv)

## How to apply

Convert colors to the CIELab perceptual color space, then compute pairwise Delta-E values with the CIEDE2000 formula (Spaco pins the pyciede2000==0.0.21 dependency for this). For spatial-omics evaluation, restrict pairwise differences to the cell-type pairs of interest — e.g. the 17 anatomically adjacent cluster pairs annotated via the Allen Brain Atlas in the STARmap benchmark — or compute mean color difference per field of view (FOV, e.g. 500x500 um). Use the resulting distributions as input to one-sided Wilcoxon rank-sum or signed-rank tests (scipy.stats) to compare methods, as in Spaco vs Scanpy/random/Palo (p = 1.45e-4, 2.32e-3, 4.51e-3) or Spaco vs matplotlib tab20 (Wilcoxon signed-rank, p = 1.20e-11). For mechanism validation, pair each realized color difference with the corresponding Degree of Interlacement (DOI) score from the CI-graph and fit OLS regression (scipy.stats>=1.10.0), expecting R-square = 0.45, p = 4e-61 on the STARmap run.

## Related tools

- **Spaco** (Produces the color assignments whose quality is scored with CIEDE2000; pins pyciede2000==0.0.21 as its color-difference dependency) — https://github.com/BrainStOrmics/Spaco
- **scipy** (Runs the one-sided Wilcoxon rank-sum/signed-rank tests and OLS regression on the CIEDE2000-derived distributions (scipy>=1.10.0))
- **matplotlib** (Renders the pairwise color-difference distributions for visualization (e.g. color_difference_distribution.png))

## Examples

```
pip install pyciede2000==0.0.21
```

## Evaluation signals

- Reproduced Wilcoxon signed-rank p-value for Spaco vs. matplotlib tab20 matches the reported 1.20e-11
- Adjacent-pair Wilcoxon rank-sum p-values for Spaco vs. Scanpy/random/Palo on STARmap land at 1.45e-4, 2.32e-3, and 4.51e-3 (one-sided)
- DOI-vs-color-difference linear regression yields R-square within 0.40-0.50 and p-value of the order of 4e-61
- Output artifact color_difference_distribution.png exists and shows both compared distributions
- All color differences are computed in CIELab space with CIEDE2000 (pyciede2000==0.0.21), not Euclidean RGB distance

## Limitations

- numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco, so upstream assignments feeding CIEDE2000 evaluation must be pinned for reproducibility
- Visual contrast obtainable through color space is limited when performing CVD-supporting colorization or coloring hundreds of clusters, capping achievable CIEDE2000 gains
- Spaco's enforcement of a correspondence between color differences and spatial distribution may not match user preferences or literature conventions, so high CIEDE2000 separability is not automatically the 'correct' coloring

## Evidence

- [methods] Compute the pairwise CIEDE2000 color differences among the Spaco-assigned colors and, separately, among the matplotlib tab20 default palette colors.: [evidence span withheld — anchor status: not_found]
- [methods] For each method, compute the color difference (CIEDE2000) for each adjacent pair and the mean color difference per field of view (FOV).: [evidence span withheld — anchor status: not_found]
- [abstract] Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments: "Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments"
