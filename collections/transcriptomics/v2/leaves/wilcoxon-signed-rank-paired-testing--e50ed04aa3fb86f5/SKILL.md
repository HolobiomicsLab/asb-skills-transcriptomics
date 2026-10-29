---
name: wilcoxon-signed-rank-paired-testing
description: Use when two paired measurements over the same items (e.g., CIEDE2000
  color differences per adjacent cell-type pair under two palettes) must be compared
  without assuming normality, and a one-sided ity claim is needed.
when_to_use_negative:
- The two samples are independent or unpaired (e.g., color differences computed over
  different cell-type-pair sets) — the signed-rank pairing is invalid
- The goal is testing mean differences of approximately normal paired data where a
  paired t-test is appropriate and more powerful
- The underlying palette colors are not derived from spatial relationships (no CI-graph),
  so no spatially aware comparison exists to test
edam_operation: http://edamontology.org/operation_2239
tools:
- name: Spaco
  role: Generates the adaptive, graph-guided palette whose pairwise CIEDE2000 color
    differences are compared against a pre-defined palette via the paired test
  repo: https://github.com/BrainStOrmics/Spaco
- name: Python
  role: Hosts the statistical test (one-sided Wilcoxon signed-rank) and result export
    for the reproduction scripts
- name: Scanpy
  role: Supplies the default palette used as the pre-defined baseline in the paired
    comparison
- name: matplotlib
  role: Renders the distribution comparison plot of pairwise color differences
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:9ffc2dbc7a32780018de11d7f5cdf0ac6113242add9fdcb7e28b362bcaee83fe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BrainStOrmics/Spaco
  tools:
  - Python
  - Scanpy
  - Spaco
  - Squidpy
  - matplotlib
  edam_operation: http://edamontology.org/operation_2239
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# Wilcoxon signed-rank paired testing

## Summary

Applies a one-sided Wilcoxon signed-rank test to paired, non-normal distributions, as used in the Spaco paper to show that the adaptive (graph-guided) palette has significantly higher pairwise CIEDE2000 color differences than a pre-defined (Scanpy default) palette (p = 1.27e-11) on the Stereo-seq axolotl brain dataset.

## When to use

Use when two paired measurements over the same items (e.g., CIEDE2000 color differences per adjacent cell-type pair under two palettes) must be compared without assuming normality, and a one-sided superiority claim is needed.

## When NOT to use

- The two samples are independent or unpaired (e.g., color differences computed over different cell-type-pair sets) — the signed-rank pairing is invalid
- The goal is testing mean differences of approximately normal paired data where a paired t-test is appropriate and more powerful
- The underlying palette colors are not derived from spatial relationships (no CI-graph), so no spatially aware comparison exists to test

## Inputs

- Paired vectors of CIEDE2000 color differences (CIELab) per spatially adjacent cell-type pair, one under the Spaco adaptive palette and one under the Scanpy default palette
- Stereo-seq axolotl brain AnnData with cell spatial coordinates and cell-type labels (16 cell types)
- CI-graph of cell-type spatial interlacement from Spaco's Degree of Interlacement metric

## Outputs

- One-sided Wilcoxon signed-rank p-value (paper value: 1.27e-11) and test statistic
- Pairwise color-difference table (axolotl_colordiff_results.csv)
- Distribution comparison plot (axolotl_colordiff_comparison.png)

## How to apply

First construct paired observations: for the axolotl Stereo-seq case, compute CIEDE2000 color differences in CIELab space between cell-type pairs for both the Spaco adaptive palette and the Scanpy default palette, restricted to spatially adjacent cell-type pairs identified from the CI-graph built with Spaco's Degree of Interlacement metric. Pairing is by cell-type pair, so the two difference vectors must have identical length and ordering. Run a one-sided Wilcoxon signed-rank test (alternative='greater' if testing that Spaco differences exceed defaults) via scipy.stats.wilcoxon, since pairwise color differences are non-normal and bounded. The paper's benchmark reported p = 1.27e-11 for 16 cell types; pin numpy to 1.22.x or 1.23.x because the numpy version influences the graph-guided mode of Spaco and thus the palette being tested. Write the pairwise difference table (e.g., axolotl_colordiff_results.csv) and render the distribution comparison plot (e.g., axolotl_colordiff_comparison.png) with matplotlib.

## Related tools

- **Spaco** (Generates the adaptive, graph-guided palette whose pairwise CIEDE2000 color differences are compared against a pre-defined palette via the paired test) — https://github.com/BrainStOrmics/Spaco
- **Python** (Hosts the statistical test (one-sided Wilcoxon signed-rank) and result export for the reproduction scripts)
- **Scanpy** (Supplies the default palette used as the pre-defined baseline in the paired comparison)
- **matplotlib** (Renders the distribution comparison plot of pairwise color differences)

## Examples

```
from scipy.stats import wilcoxon; stat, p = wilcoxon(spaco_diffs, default_diffs, alternative='greater')
```

## Evaluation signals

- Reproduced one-sided Wilcoxon p-value matches the paper's 1.27e-11 within numerical tolerance
- Spaco's color-difference distribution is shifted upward relative to the Scanpy default palette (positive median of paired differences)
- Both difference vectors are paired: identical length and ordering by spatially adjacent cell-type pair (16 cell types)
- Color differences are computed as CIEDE2000 in CIELab space, restricted to adjacent pairs, not Euclidean RGB distances
- numpy version is fixed to 1.22.x or 1.23.x, since the numpy version can influence the graph-guided mode of Spaco

## Limitations

- numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco, so the tested palette (and hence p-value) is environment-dependent unless the version is pinned
- Spaco enforces a correspondence between color differences and the spatial distribution of cells, which may not match user preferences or literature conventions
- Visual contrast obtainable through color space is limited when performing CVD-supporting colorization or in the case of coloring hundreds of clusters

## Evidence

- [results] significantly higher than that of the pre-defined palette (one-sided Wilcoxon test, p = 1.27e−11): [evidence span withheld — anchor status: not_found]
- [methods] numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco: "numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco"
- [abstract] Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments: "Spaco utilizes the Degree of Interlacement metric to construct a weighted graph that evaluates the spatial relationships among different cell types, refining color assignments"
- [methods] pip install git+https://github.com/BrainStOrmics/Spaco.git: "pip install git+https://github.com/BrainStOrmics/Spaco.git"
