---
name: local-score-smoothing-cubic-splines
description: Use when you need to visually assess whether condition labels (e.g.,
  treatment vs. control) are locally imbalanced across a low-dimensional cell embedding,
  as the first diagnostic step before differential topology testing with condiments.
when_to_use_negative:
- When you need to test differential topology, differential progression, or differential
  fate selection — use topologyTest, progressionTest, or fateSelectionTest instead,
  as imbalance_score is a diagnostic, not a statistical test
- When cells come from a steady-state system where clustering and per-cluster differential
  abundance is the reasonable approach, and trajectory continuity is not at issue
- When the input is raw count data rather than a reduced-dimensional representation
  — the score must be computed on embedded coordinates (e.g., Dim1/Dim2), not expression
  matrices
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: condiments
  role: Provides the imbalance_score function, the toy_dataset example data, and the
    kNN-based local condition-distribution scoring with cubic-spline smoothing
  repo: github.com/HectorRDB/condiments
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:cb99d26764ccc02d44081f3281c7604730b5f429f48e54692d27955c37945111
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - condiments
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Local score smoothing with cubic splines (condiments imbalance diagnostic)

## Summary

This skill reconstructs condiments' imbalance_score diagnostic, which quantifies per-cell imbalance between the local and global distributions of condition labels over a k-nearest-neighbor graph on a reduced-dimensional representation (X), then locally scales raw scores with cubic-spline smoothing to produce the visual diagnostic for condition imbalance.

## When to use

Use when you need to visually assess whether condition labels (e.g., treatment vs. control) are locally imbalanced across a low-dimensional cell embedding, as the first diagnostic step before differential topology testing with condiments.

## When NOT to use

- When you need to test differential topology, differential progression, or differential fate selection — use topologyTest, progressionTest, or fateSelectionTest instead, as imbalance_score is a diagnostic, not a statistical test
- When cells come from a steady-state system where clustering and per-cluster differential abundance is the reasonable approach, and trajectory continuity is not at issue
- When the input is raw count data rather than a reduced-dimensional representation — the score must be computed on embedded coordinates (e.g., Dim1/Dim2), not expression matrices

## Inputs

- Reduced-dimensional cell coordinate matrix (e.g., Dim1/Dim2 columns from the condiments toy_dataset)
- Per-cell condition label vector (e.g., treatment vs. control)

## Outputs

- Per-cell data frame with 'scores' (raw local imbalance z-scores) and 'scaled_scores' (cubic-spline smoothed scores) columns
- imbalance_scores.csv table with one row per cell and finite raw/smoothed score values
- A reduced-dimension plot of smoothed imbalance scores highlighting the imbalanced lineage

## How to apply

Load the bundled toy data with data('toy_dataset', package='condiments') and extract the reduced-dimension coordinates (Dim1, Dim2) and the condition labels. Call imbalance_score() on the coordinate matrix with the condition vector; it builds a k-nearest-neighbor graph (default neighborhood size k=10) and computes each cell's score as the z-score of the local condition-distribution deviation from the global distribution. The function returns both raw 'scores' and locally scaled 'scaled_scores', where smoothing uses cubic splines with smoothing parameter s=10 to enforce continuity, reflecting the paper's argument that cluster-based imbalance assessment ignores the continuity of cellular state transitions. Attach both columns to the cell metadata, plot the smoothed scores across Dim1/Dim2 to confirm imbalance localizes to one lineage, and write the annotated table to imbalance_scores.csv.

## Related tools

- **condiments** (Provides the imbalance_score function, the toy_dataset example data, and the kNN-based local condition-distribution scoring with cubic-spline smoothing) — github.com/HectorRDB/condiments

## Examples

```
data("toy_dataset", package = "condiments"); scores <- imbalance_score(X = as.matrix(toy_dataset[, c("Dim1", "Dim2")]), conditions = toy_dataset$conditions)
```

## Evaluation signals

- Output CSV contains exactly one row per input cell with both 'scores' and 'scaled_scores' columns present
- All raw and smoothed score values are finite (no NA/NaN/Inf)
- Smoothed scores vary continuously along the embedding; no blocky per-cluster discontinuities
- High smoothed scores concentrate on the lineage known to be condition-imbalanced in the toy dataset
- Results are reproducible with default parameters (neighborhood size k=10, smoothing parameter s=10) on re-run

## Limitations

- No independent benchmarking of the condiments workflow has been performed; the authors explicitly call for independent benchmarking
- Real-world benchmarking case studies are still lacking and only becoming available as more datasets are published
- The kNN-based local score depends on the chosen reduced-dimensional representation; batch-condition confounding can be inherent and unresolvable in some settings

## Evidence

- [methods] imbalance_score diagnostic: "using the *imbalance_score* function"
- [methods] toy dataset used as input: "data("toy_dataset", package = "condiments")"
- [intro] rationale for local smoothing over cluster-based imbalance: "clustering the cells and assessing condition imbalances for each cluster ignores the continuity of the cellular state transition process"
- [readme] tool installation from Bioconductor: [evidence span withheld — anchor status: external_doc]
