---
name: condition-imbalance-scoring
description: Use when a scRNA-Seq dataset spans multiple conditions and you need a
  neighborhood-level diagnostic of where condition labels are locally over-represented
  on a low-dimensional embedding (e.g., Dim1/Dim2) before running trajectory tests.
when_to_use_negative:
- The dataset contains only a single condition (no condition labels to compare locally
  vs globally), so local/global z-scores are undefined.
- The goal is a formal hypothesis test rather than a diagnostic visualization — use
  condiments' topologyTest, progressionTest, or fateSelectionTest instead of the imbalance
  score.
- The system is truly steady-state with discrete, biologically relevant cell identities;
  the article notes clustering plus per-cluster differential abundance is a reasonable
  approach in that setting.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: condiments
  role: Provides the imbalance_score function that computes the per-cell raw and locally
    scaled condition-imbalance scores on the k-nearest-neighbor graph
  repo: github.com/HectorRDB/condiments
- name: milo
  role: Referenced neighborhood-based differential abundance method that, like the
    imbalance score, relies on a common low-dimensional representation and data-driven
    local neighborhoods; cited as the conceptual backdrop for condiments' diagnostic
- name: DAseq
  role: Referenced differential abundance alternative using data-driven local neighborhoods
    on a shared low-dimensional representation; contrasted with condiments' trajectory-aware
    scoring
provenance:
  source_task_ids:
  - task_008
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:c72bba52c53c64d725b36fe7e04309abb83fc7f84a86d6fc494ef08031b41f70
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DAseq
  - condiments
  - milo
  - tradeSeq
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# condition-imbalance-scoring

## Summary

Computes condiments' per-cell imbalance score, a visual diagnostic that quantifies whether condition labels are locally imbalanced across a low-dimensional representation using k-nearest-neighbor condition-distribution z-scores smoothed along the trajectory. It precedes and motivates differential topology, progression, and fate selection testing.

## When to use

Use when a scRNA-Seq dataset spans multiple conditions and you need a neighborhood-level diagnostic of where condition labels are locally over-represented on a low-dimensional embedding (e.g., Dim1/Dim2) before running trajectory tests.

## When NOT to use

- The dataset contains only a single condition (no condition labels to compare locally vs globally), so local/global z-scores are undefined.
- The goal is a formal hypothesis test rather than a diagnostic visualization — use condiments' topologyTest, progressionTest, or fateSelectionTest instead of the imbalance score.
- The system is truly steady-state with discrete, biologically relevant cell identities; the article notes clustering plus per-cluster differential abundance is a reasonable approach in that setting.

## Inputs

- Reduced-dimensional coordinate matrix X for all cells (e.g., toy_dataset Dim1/Dim2 columns)
- Per-cell condition label vector (e.g., control vs ILD)
- Cell metadata table to receive the score columns

## Outputs

- Per-cell raw imbalance scores ('scores' column)
- Per-cell locally scaled/smoothed scores ('scaled_scores' column)
- Annotated one-row-per-cell score table (e.g., imbalance_scores.csv)
- Visualization of smoothed scores across the reduced dimensions

## How to apply

Load the coordinate matrix and condition labels (e.g., data('toy_dataset', package='condiments') with its Dim1, Dim2 columns) and run the imbalance_score function on the coordinate matrix with the condition vector. The method defines cell neighborhoods via a k-nearest neighbor graph on the reduced-dimensional representation X (default neighborhood size k=10), computes for each cell a z-score comparing the local to the global condition distribution, and locally scales the raw scores with cubic-spline smoothing (smoothing parameter s=10). Attach the returned 'scores' and 'scaled_scores' columns to the cell metadata and visualize the smoothed scores across the reduced dimensions; high smoothed scores should concentrate on the lineage or region known to be imbalanced. The rationale is that clustering cells and assessing condition imbalances per cluster ignores the continuity of the cellular state transition process, so the score is evaluated on continuous neighborhoods instead of discrete clusters. Write the annotated score table (one row per cell with finite raw and smoothed values) to a file such as imbalance_scores.csv.

## Related tools

- **condiments** (Provides the imbalance_score function that computes the per-cell raw and locally scaled condition-imbalance scores on the k-nearest-neighbor graph) — github.com/HectorRDB/condiments
- **milo** (Referenced neighborhood-based differential abundance method that, like the imbalance score, relies on a common low-dimensional representation and data-driven local neighborhoods; cited as the conceptual backdrop for condiments' diagnostic)
- **DAseq** (Referenced differential abundance alternative using data-driven local neighborhoods on a shared low-dimensional representation; contrasted with condiments' trajectory-aware scoring)

## Examples

```
data("toy_dataset", package = "condiments"); sc <- imbalance_score(X = toy_dataset[, c("Dim1","Dim2")], conditions = toy_dataset$conditions)
```

## Evaluation signals

- Output table contains exactly one row per cell with finite (non-NA) raw 'scores' and smoothed 'scaled_scores' values.
- High smoothed scores spatially concentrate on the lineage/region known to be condition-imbalanced in the toy dataset, not spread uniformly.
- Parameters match the documented defaults: k-nearest-neighbor neighborhood size k=10 and cubic-spline smoothing parameter s=10.
- Scores are interpretable as local-vs-global condition-distribution z-scores: near-zero where conditions are evenly mixed, large magnitude where a condition dominates locally.
- Score columns correctly join back to cell metadata (no row misalignment after attaching 'scores' and 'scaled_scores').

## Limitations

- The imbalance score is a diagnostic/visualization tool, not a calibrated hypothesis test; condiments' p-value-based tests (topologyTest, progressionTest, fateSelectionTest) are the formal inference steps.
- Like other neighborhood-based differential abundance approaches, it relies on a common low-dimensional representation of all observations, so embedding quality drives the result.
- Some batch-condition confounding can be inherent to the study design and unresolvable, which can distort local condition distributions.

## Evidence

- [methods] imbalance_score is called directly on the coordinate matrix: "using the *imbalance_score* function"
- [methods] The bundled toy dataset supplies the coordinates and condition labels: "data("toy_dataset", package = "condiments")"
- [intro] Rationale for neighborhood-based scoring over cluster-based differential abundance: "clustering the cells and assessing condition imbalances for each cluster ignores the continuity of the cellular state transition process, making these approaches suboptimal."
- [intro] Neighborhood-based DA methods share the low-dimensional-representation premise of the imbalance score: "borrowing from the field of mass cytometry15, milo16 and DAseq17 rely on a common low-dimensional representation of all observations"
- [readme] Official documentation entry point for usage: [evidence span withheld — anchor status: external_doc]
