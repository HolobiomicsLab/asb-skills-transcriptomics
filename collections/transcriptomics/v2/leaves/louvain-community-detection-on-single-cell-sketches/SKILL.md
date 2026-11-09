---
name: louvain-community-detection-on-single-cell-sketches
description: Use when evaluating whether a cell-sampling/sketching strategy (e.g.,
  spherical thresholding vs. geometric/box sketching) preserves biological cell-type
  separability in large scRNA-seq datasets, by clustering sketched subsets and scoring
  against known cell-type labels.
when_to_use_negative:
- When the input is raw gene expression rather than a reduced feature space (the article's
  workflow operates on cells × PCs matrices such as zeisel_pca.csv)
- When datasets lack ground-truth cell-type annotations, since the ARI benchmark cannot
  be computed without labels
- When the goal is trajectory inference rather than cell-type clustering — the article
  treats Monocle 2 trajectory inference as a separate downstream use of sketches
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Sphetcher
  role: Generates the spherical sketch indicator vectors (Pearson-correlation metric)
    whose clusterability is benchmarked against geometric sketches
  repo: https://github.com/canzarlab/Sphetcher
- name: Louvain
  role: Community detection on the k-nearest-neighbor graph of sketched cells to obtain
    cluster assignments for the ARI evaluation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2020.101126
    title: 'Sphetcher: Spherical Thresholding Improves Sketching of Single-Cell Transcriptomic
      Heterogeneity'
  source_content_hash: sha256:b4e32d2ec55583d0c01f056fe88813fb442e0a705de51c8f0d849539cad741bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/Sphetcher
  tools:
  - Louvain
  - Sphetcher
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# louvain-community-detection-on-single-cell-sketches

## Summary

Benchmark skill for comparing how well spherical sketches (Sphetcher) versus geometric sketches preserve cell-type structure: run Louvain community detection on k-nearest-neighbor graphs built from sketched cells, propagate labels back to the full dataset with a kNN classifier, and score agreement with ground-truth annotations using Adjusted Rand Index.

## When to use

Use when evaluating whether a cell-sampling/sketching strategy (e.g., spherical thresholding vs. geometric/box sketching) preserves biological cell-type separability in large scRNA-seq datasets, by clustering sketched subsets and scoring against known cell-type labels.

## When NOT to use

- When the input is raw gene expression rather than a reduced feature space (the article's workflow operates on cells × PCs matrices such as zeisel_pca.csv)
- When datasets lack ground-truth cell-type annotations, since the ARI benchmark cannot be computed without labels
- When the goal is trajectory inference rather than cell-type clustering — the article treats Monocle 2 trajectory inference as a separate downstream use of sketches

## Inputs

- sketch_indicator_output.csv — indicator vector with one entry per cell (1 = cell in sketch, 0 otherwise), produced by Sphetcher per dataset and sketch size
- PCA feature matrices for each dataset (e.g., zeisel_pca.csv, rows = cells, columns = PCs)
- Ground-truth cell-type annotations per cell (class label column vectors, integer labels 1..K)
- Precomputed or downloaded sketch archives for Saunders (665,858 cells), MOCA/Cao (2,026,641 cells), and Zeisel (465,281 cells) at 1%/4%/10% sketch sizes

## Outputs

- ari_results.csv — table of (dataset, sketching method, sketch size, ARI) values
- figure3_s4_ari.png — grouped bar chart comparing spherical vs. geometric sketch ARI across datasets and sketch sizes
- Per-sketch Louvain cluster assignments and kNN-propagated labels for all cells

## How to apply

Load the sketch indicator vectors (sketch_indicator_output.csv) produced by Sphetcher at sketch sizes of 1%, 4%, and 10%, and extract the sketched cell subsets for both the spherical sketch (computed with Pearson correlation as distance metric) and the geometric (box-approximating) sketch. Build a k-nearest-neighbor graph on each sketch's PCA feature matrix and run Louvain community detection to obtain cluster assignments for the sketched cells. Propagate sketch-level cluster labels to all cells of the original dataset using a k-nearest-neighbor classifier trained on the sketch cells' features. Compute the Adjusted Rand Index (a chance-adjusted agreement metric) between propagated labels and ground-truth cell-type annotations for every dataset × method × sketch-size combination, tabulate into ari_results.csv, and render a grouped bar chart (figure3_s4_ari.png) mirroring the paper's Figure 3/S4 comparison. The rationale is that Sphetcher's spherical sketches sample the transcriptomic landscape more evenly than geometric sketches, so Louvain clustering on them should yield better or comparable cell-type separation; validation passes when relative ARI trends across sketch sizes and methods are consistent with the reported figure.

## Related tools

- **Sphetcher** (Generates the spherical sketch indicator vectors (Pearson-correlation metric) whose clusterability is benchmarked against geometric sketches) — https://github.com/canzarlab/Sphetcher
- **Louvain** (Community detection on the k-nearest-neighbor graph of sketched cells to obtain cluster assignments for the ARI evaluation)

## Examples

```
sphetcher zeisel_pca.csv 4643 sketch_indicator_output.csv
```

## Evaluation signals

- sketch_indicator_output.csv contains exactly sketch_size entries equal to 1 (README: 'there are sketch_size 1's in the vector')
- ari_results.csv covers all 36 dataset × method × sketch-size combinations (three datasets, two methods, sizes 1%/4%/10%)
- Louvain clustering of spherical sketches achieves better or comparable ARI than geometric sketches in the strong majority of instances; geometric sketching wins the median ARI in only three of thirty-six instances
- Relative ARI trends across sketch sizes and methods in figure3_s4_ari.png are consistent with the reported Figure 3/S4
- Propagated labels cover all cells of the original dataset, and ARI is a chance-adjusted metric relative to the ground-truth annotations

## Limitations

- The interplay of sketching with a given distance metric and subsequent algorithmic analysis was not systematically addressed in the study
- Sphetcher's underlying model might be less compatible with assumptions made by clustering algorithms other than Louvain and Monocle 2
- No software version numbers are stated for Sphetcher or the downstream tools used
- MOCA has only 1% and 4% sketch sizes precomputed (no 10% archive), so the full 3-size grid is only available for Saunders and Zeisel datasets

## Evidence

- [readme] sphetcher sketch indicator output format: "sketch_indicator_output.csv : an indicator vector of `n` samples where 1 indicates the sample is in the sketch, 0 otherwise (there are `sketch_size` 1's in the vector)."
- [discussion] Sphetcher distance metric: "In most of the experiments in this study, Sphetcher used Pearson correlation as distance metric and was combined with a specific algorithm for downstream analysis."
- [discussion] Louvain usage: "Even though Louvain community detection and Monocle 2 are widely used for scRNA-seq clustering"
- [abstract] spherical sketch representation quality: "We show that the spherical sketch computed by Sphetcher constitutes a more accurate representation of the original transcriptomic landscape"
- [readme] example invocation: "sphetcher zeisel_pca.csv 1000 sketch_indicator_output.csv"
