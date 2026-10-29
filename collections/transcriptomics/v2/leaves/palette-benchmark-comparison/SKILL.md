---
name: palette-benchmark-comparison
description: 'Use when you need to quantitatively justify a spatial color-assignment
  method over defaults: you have cell coordinates plus cluster labels from spatial
  transcriptomics (e.g., STARmap) and a set of candidate colorization methods sharing
  one palette.'
when_to_use_negative:
- You only need to produce a good-looking color assignment and have no competing methods
  to benchmark — use Spaco's colorize directly rather than running the comparison
  protocol
- Your data lack spatial coordinates or spatially localized clusters, so adjacent-pair
  or FOV-based color-difference statistics cannot be defined
- You want to compare palettes rather than assignments — this protocol fixes one shared
  palette across all methods and evaluates assignments only
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Spaco
  role: Spatially aware colorization method under test, run in graph-guided mode via
    spaco.colorize on cell coordinates and labels
  repo: https://github.com/BrainStOrmics/Spaco
- name: Scanpy
  role: Handles the AnnData object and supplies the default palette as a baseline
    colorization
- name: Squidpy
  role: Dataset loader for the STARmap mouse brain spatial dataset (cell coordinates
    and cluster labels)
- name: scipy
  role: Provides scipy.stats one-sided Wilcoxon rank-sum tests comparing Spaco against
    each baseline
- name: pandas
  role: Writes labeled p-values to wilcoxon_pvalues.csv
- name: numpy
  role: Version pinned to 1.22.x or 1.23.x to keep the Spaco graph-guided result reproducible
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2023.100915
    title: 'Spaco: A comprehensive tool for coloring spatial data at single-cell resolution'
  source_content_hash: sha256:0b2cad6220e9022b9d7567efc62da4a0ffd5b8cb563f928b0879cb725ac13e7e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BrainStOrmics/Spaco
  tools:
  - Scanpy
  - Spaco
  - Squidpy
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# palette-benchmark-comparison

## Summary

A benchmarking protocol that compares spatially aware colorization (Spaco) against baseline palettes (Scanpy default, random permutation, Palo) on the same shared palette, quantifying CIEDE2000 color differences between anatomically adjacent cell-type pairs and per-field-of-view means, with one-sided Wilcoxon rank-sum tests.

## When to use

Use when you need to quantitatively justify a spatial color-assignment method over defaults: you have cell coordinates plus cluster labels from spatial transcriptomics (e.g., STARmap) and a set of candidate colorization methods sharing one palette.

## When NOT to use

- You only need to produce a good-looking color assignment and have no competing methods to benchmark — use Spaco's colorize directly rather than running the comparison protocol
- Your data lack spatial coordinates or spatially localized clusters, so adjacent-pair or FOV-based color-difference statistics cannot be defined
- You want to compare palettes rather than assignments — this protocol fixes one shared palette across all methods and evaluates assignments only

## Inputs

- Spatial transcriptomics data with cell coordinates and cluster labels (e.g., STARmap mouse brain dataset loaded via Squidpy's sq.datasets loader)
- A shared color palette used identically across all compared methods
- List of anatomically adjacent cell-type/cluster pairs derived from atlas annotation (e.g., 17 pairs among 10 Allen Brain Atlas-annotated STARmap clusters)

## Outputs

- wilcoxon_pvalues.csv with six one-sided Wilcoxon rank-sum p-values labeled by baseline method and test type (adjacent-pair color differences and FOV mean differences)
- Per-method CIEDE2000 color-difference values for each adjacent cluster pair
- Per-method mean color difference per field of view (FOV)

## How to apply

Load the spatial dataset (e.g., STARmap mouse brain cell coordinates and region-localized cluster labels) via Squidpy's dataset loader with Scanpy for handling. Run Spaco colorization with spaco.colorize in graph-guided mode (radius=0.05, n_neighbors=30), pinning numpy to 1.22.x or 1.23.x for reproducibility. Generate three baselines constrained to the same shared palette: the Scanpy default palette, a random permutation of it, and a Palo colorization. Annotate clusters with a reference atlas (Allen Brain Atlas) to define anatomically adjacent cluster pairs (17 pairs for 10 STARmap clusters). For each method compute CIEDE2000 color differences for each adjacent pair and the mean color difference per field of view, then run one-sided Wilcoxon rank-sum tests (scipy.stats) comparing Spaco against each baseline for both metrics, writing the six labeled p-values to CSV.

## Related tools

- **Spaco** (Spatially aware colorization method under test, run in graph-guided mode via spaco.colorize on cell coordinates and labels) — https://github.com/BrainStOrmics/Spaco
- **Scanpy** (Handles the AnnData object and supplies the default palette as a baseline colorization)
- **Squidpy** (Dataset loader for the STARmap mouse brain spatial dataset (cell coordinates and cluster labels))
- **scipy** (Provides scipy.stats one-sided Wilcoxon rank-sum tests comparing Spaco against each baseline)
- **pandas** (Writes labeled p-values to wilcoxon_pvalues.csv)
- **numpy** (Version pinned to 1.22.x or 1.23.x to keep the Spaco graph-guided result reproducible)

## Examples

```
import squidpy as sq, spaco; adata = sq.datasets.starmap(); color_mapping = spaco.colorize(adata, coordinates=adata.obsm['spatial'], labels=adata.obs['cluster'], radius=0.05, n_neighbors=30)
```

## Evaluation signals

- Reference adjacent-pair p-values reproduced: Spaco vs Scanpy p = 1.45e-4, vs random assignments p = 2.32e-3, vs Palo p = 4.51e-3 (one-sided Wilcoxon, no-CVD mode)
- Reference FOV-mean p-values reproduced: 8.42e-21, 9.16e-19, and 8.87e-21 for Spaco vs Scanpy, random, and Palo respectively
- All four methods are confirmed to share the identical palette so differences reflect assignment strategy only
- wilcoxon_pvalues.csv contains six rows, each labeled with baseline method and test type (adjacent pairs vs FOV means)
- CIEDE2000 is used consistently as the color-difference metric for both adjacent-pair and FOV-level comparisons

## Limitations

- numpy version (1.22.x or 1.23.x) influences the result of Spaco's graph-guided mode, so the benchmark must pin the numpy version to be reproducible
- Spaco enforces a correspondence between color differences and the spatial distribution of cells, which may not match user preferences or literature conventions
- Visual contrast obtainable through color space is limited when performing CVD-supporting colorization or when coloring hundreds of clusters, constraining benchmark headroom in those regimes

## Evidence

- [results] Spaco achieved significant visual differences between anatomically adjacent cluster pairs by one-sided Wilcoxon tests: [evidence span withheld — anchor status: not_found]
- [methods] graph-guided parameters and numpy pinning: "numpy version (1.22.x or 1.23.x) could influence the result of graph-guided mode of Spaco"
- [methods] Spaco colorize API: "color_mapping = spaco.colorize("
- [methods] benchmark scripts repository: "Scripts to reproduce benchmarking and analytic results in Spaco paper are in repository [Spaco_scripts](https://github.com/BrainStOrmics/Spaco_scripts)"
- [discussion] CVD and cluster-count contrast limitation: "the visual contrast that can be obtained through color space is limited when performing CVD-supporting colorization or in the case of coloring hundreds of clusters."
- [readme] Spaco spatial colorization purpose: "Spaco (**spa**tial **co**lorization) provides a spatially constrained approach that generates discriminate color assignments for visualizing single-cell spatial data"
