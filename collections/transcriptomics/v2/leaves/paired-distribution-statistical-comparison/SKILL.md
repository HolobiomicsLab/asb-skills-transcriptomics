---
name: paired-distribution-statistical-comparison
description: Use when two palettes or measurement sets yield paired samples (e.g.,
  pairwise CIEDE2000 distances for Spaco colors vs tab20 colors) and you must test
  whether their discernibility distributions differ, as in the Spaco axolotl benchmark.
when_to_use_negative:
- Samples are independent/unpaired (Wilcoxon signed-rank assumes pairing; an unpaired
  test such as Mann–Whitney U would be required instead).
- Palettes contain very few colors, so the pairwise difference vectors are too short
  for a meaningful rank-based comparison.
- The goal is spatial-aware colorization itself rather than statistical evaluation
  of palette discernibility.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0092
tools:
- name: Spaco
  role: Extracts the image-guided palette (CIELab binning, CIEDE2000 update, FPS)
    and colorizes axolotl cell-type clusters, whose assigned colors form the tested
    distribution
  repo: https://github.com/BrainStOrmics/Spaco
- name: matplotlib
  role: Provides the tab20 reference palette against which Spaco's palette discernibility
    is compared
- name: Squidpy
  role: Loads the Stereo-seq axolotl brain dataset (cell coordinates and cell-type
    labels)
- name: Python
  role: Implementation language for the CIEDE2000 pairwise computation and Wilcoxon
    signed-rank test
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:d59fac2aa8a6ad7ba655275abc3c4952507d8645b6537060f81dd0c5ee9f53c4
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# paired-distribution-statistical-comparison

## Summary

Compares two paired distributions of pairwise measurements — here, CIEDE2000 color differences among Spaco-extracted palette colors versus matplotlib's tab20 palette — using a Wilcoxon signed-rank test to establish statistical comparability (p = 1.20e−11) of palette discernibility.

## When to use

Use when two palettes or measurement sets yield paired samples (e.g., pairwise CIEDE2000 distances for Spaco colors vs tab20 colors) and you must test whether their discernibility distributions differ, as in the Spaco axolotl benchmark.

## When NOT to use

- Samples are independent/unpaired (Wilcoxon signed-rank assumes pairing; an unpaired test such as Mann–Whitney U would be required instead).
- Palettes contain very few colors, so the pairwise difference vectors are too short for a meaningful rank-based comparison.
- The goal is spatial-aware colorization itself rather than statistical evaluation of palette discernibility.

## Inputs

- Spaco-extracted palette (colors from journal cover image via CIELab binning, CIEDE2000 frequency updating, and FPS refinement)
- matplotlib tab20 palette colors (reference)
- Stereo-seq axolotl brain dataset with cell coordinates and cell-type labels (loaded via Squidpy)

## Outputs

- Wilcoxon signed-rank test statistic and p-value (expected p = 1.20e−11)
- Pairwise color-difference distribution plot saved as color_difference_distribution.png

## How to apply

First colorize the Stereo-seq axolotl cell-type clusters with spaco.colorize using the image-extracted palette (CIELab frequency binning, CIEDE2000-guided refinement, farthest point sampling), then compute all pairwise CIEDE2000 color differences among the assigned colors and, separately, among the 20 matplotlib tab20 colors. Treat the two difference distributions as paired samples and run a Wilcoxon signed-rank test; the paper's reference result is p = 1.20e−11 for Spaco versus tab20, indicating comparable discernibility while preserving the journal-cover image's thematic color consistency. Plot both pairwise difference distributions side by side and save the figure (color_difference_distribution.png) as the visual evidence. Judging correctness: the reproduced p-value must agree with the reported value and the metric must be CIEDE2000 (not RGB Euclidean distance), since CIEDE2000 is the perceptual color-difference metric Spaco's palette extraction is built on.

## Related tools

- **Spaco** (Extracts the image-guided palette (CIELab binning, CIEDE2000 update, FPS) and colorizes axolotl cell-type clusters, whose assigned colors form the tested distribution) — https://github.com/BrainStOrmics/Spaco
- **matplotlib** (Provides the tab20 reference palette against which Spaco's palette discernibility is compared)
- **Squidpy** (Loads the Stereo-seq axolotl brain dataset (cell coordinates and cell-type labels))
- **Python** (Implementation language for the CIEDE2000 pairwise computation and Wilcoxon signed-rank test)

## Examples

```
from scipy.stats import wilcoxon; stat, p = wilcoxon(spaco_ciede2000_dists, tab20_ciede2000_dists)  # expect p = 1.20e-11
```

## Evaluation signals

- Reproduced Wilcoxon signed-rank p-value matches the paper's reported 1.20e−11 for Spaco versus tab20.
- color_difference_distribution.png exists and shows both pairwise difference distributions overlaid.
- Pairwise differences are computed with CIEDE2000, not Euclidean RGB distance.
- Both distributions have equal numbers of pairwise comparisons (paired structure preserved) before testing.

## Limitations

- Visual contrast obtainable through color space is limited when performing CVD-supporting colorization or when coloring hundreds of clusters, capping meaningful discernibility differences.
- The numpy version (1.22.x or 1.23.x) can influence the graph-guided mode of Spaco, potentially changing the assigned colors and hence the tested distribution.

## Evidence

- [abstract] adaptive palette selection amplifying chromatic distinctions: "Spaco incorporates an adaptive palette selection approach to amplify chromatic distinctions"
- [methods] image palette extraction maximizing differentiation while keeping theme: "We provide a method for extracting color plates from images. While maintaining the theme color, the color differentiation is maximized."
- [other] task finding: Wilcoxon comparison of Spaco vs tab20 color differences: [evidence span withheld — anchor status: not_found]
- [readme] readme: Spaco generates discriminate color assignments: "Spaco (**spa**tial **co**lorization) provides a spatially constrained approach that generates discriminate color assignments for visualizing single-cell spatial data"
