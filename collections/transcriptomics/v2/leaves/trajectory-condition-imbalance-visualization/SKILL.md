---
name: trajectory-condition-imbalance-visualization
description: Use when a single-cell dataset with a low-dimensional embedding (e.g.
  Dim1/Dim2 coordinates) and binary condition labels needs a per-cell diagnostic of
  where condition imbalance concentrates along a trajectory, before formal differential
  topology or progression testing.
when_to_use_negative:
- The analysis goal is gene-level differential expression along lineages (use conditionTest
  / fitGAM instead)
- A formal statistical test of differential topology, progression, or fate selection
  is required rather than a visual imbalance diagnostic (use topologyTest, progressionTest,
  or fateSelectionTest)
- Input is steady-state data where cells should be clustered and differential abundance
  assessed per cluster, with no trajectory structure to leverage
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: condiments
  role: Provides the imbalance_score function, the toy_dataset, and visualization
    of local condition imbalance along trajectories
  repo: https://github.com/HectorRDB/condiments
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:9a0462c203d9d427d5bd4e67f04955ebe90659322e29acff57abef2f5ea49791
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HectorRDB/condiments
  tools:
  - condiments
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# trajectory-condition-imbalance-visualization

## Summary

Compute and visualize per-cell condition imbalance scores along a trajectory using the condiments `imbalance_score` function on the toy dataset, quantifying how local condition distributions deviate from global ones within a k-nearest-neighbor graph on the reduced-dimensional representation.

## When to use

Use when a single-cell dataset with a low-dimensional embedding (e.g. Dim1/Dim2 coordinates) and binary condition labels needs a per-cell diagnostic of where condition imbalance concentrates along a trajectory, before formal differential topology or progression testing.

## When NOT to use

- The analysis goal is gene-level differential expression along lineages (use conditionTest / fitGAM instead)
- A formal statistical test of differential topology, progression, or fate selection is required rather than a visual imbalance diagnostic (use topologyTest, progressionTest, or fateSelectionTest)
- Input is steady-state data where cells should be clustered and differential abundance assessed per cluster, with no trajectory structure to leverage

## Inputs

- Reduced-dimension cell coordinate matrix (e.g. Dim1/Dim2 columns from toy_dataset in the condiments package)
- Per-cell condition label vector (binary or categorical)
- Optional cell metadata data.frame to receive the score columns

## Outputs

- Per-cell raw imbalance scores ('scores' column) measuring local vs global condition-distribution z-scores
- Locally smoothed/scaled scores ('scaled_scores' column) via cubic-spline smoothing (s=10)
- Annotated score table (imbalance_scores.csv) with one row per cell
- Visualization of smoothed scores across the reduced dimensions

## How to apply

Load the bundled toy dataset with data('toy_dataset', package='condiments') and extract the reduced-dimension coordinates (Dim1, Dim2) and condition labels. Run the imbalance_score function on the coordinate matrix with the condition vector: it builds a k-nearest-neighbor graph (default neighborhood size k=10) and assigns each cell a z-score measuring the imbalance between local and global distributions of condition labels, then locally scales the raw scores using cubic-spline smoothing (smoothing parameter s=10). Attach the returned 'scores' and 'scaled_scores' columns to the cell metadata. Visualize the smoothed (scaled) scores across the reduced dimensions to confirm the expected imbalance pattern localized to one lineage, and write the annotated score table to imbalance_scores.csv.

## Related tools

- **condiments** (Provides the imbalance_score function, the toy_dataset, and visualization of local condition imbalance along trajectories) — https://github.com/HectorRDB/condiments

## Examples

```
data('toy_dataset', package='condiments'); scores <- imbalance_score(X = as.matrix(toy_dataset[, c('Dim1','Dim2')]), conditions = toy_dataset$conditions)
```

## Evaluation signals

- Output file imbalance_scores.csv contains one row per cell with finite raw ('scores') and smoothed ('scaled_scores') values
- High smoothed scores concentrate on the lineage known to be imbalanced in the toy dataset
- Default parameters verified: k-nearest-neighbor size k=10 and smoothing parameter s=10 applied to local condition z-scores
- Scaled scores vary smoothly along the reduced-dimensional embedding (cubic-spline smoothing), with no abrupt cell-level discontinuities

## Limitations

- Cluster-based imbalance assessment ignores the continuity of cell state transitions and is suboptimal; this score is a diagnostic, not an inferential test with calibrated p-values
- p-values are not given a probabilistic interpretation in the condiments framework; only numerical-summary status
- Dependence of trajectory/pseudotime estimates on the same data (double-dipping) is ignored rather than resolved

## Evidence

- [methods] using the *imbalance_score* function: "using the *imbalance_score* function"
- [methods] toy_dataset loading: "data("toy_dataset", package = "condiments")"
- [intro] clustering suboptimality: "clustering the cells and assessing condition imbalances for each cluster ignores the continuity of the cellular state transition process, making these approaches suboptimal."
- [readme] vignette availability: [evidence span withheld — anchor status: external_doc]
