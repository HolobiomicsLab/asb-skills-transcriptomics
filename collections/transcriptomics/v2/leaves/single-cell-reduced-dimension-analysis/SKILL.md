---
name: single-cell-reduced-dimension-analysis
description: Use when you need to assess whether condition labels (e.g., treatment
  vs control) are locally imbalanced across a single-cell reduced-dimension representation
  before running trajectory-level differential topology or abundance tests.
when_to_use_negative:
- You already have an inferred common trajectory and want formal tests — use topologyTest,
  progressionTest, or fateSelectionTest instead of this diagnostic
- You need gene-level differential expression along lineages — use fitGAM plus conditionTest,
  not the imbalance score
- The data are steady-state cells with no dynamic/trajectory structure where plain
  cluster-based differential abundance is acceptable
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: condiments
  role: Provides the imbalance_score function and the bundled toy_dataset used to
    compute and visualize per-cell condition imbalance on the reduced-dimensional
    representation
  repo: github.com/HectorRDB/condiments
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:ac859ce67404c170db3a12149ddc43459b0fa70151fe047d2d61a46e8d037853
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - condiments
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# single-cell-reduced-dimension-analysis

## Summary

Computes condiments' per-cell imbalance score on a low-dimensional representation (e.g., Dim1/Dim2 from a reduced dimension embedding) to quantify local versus global condition imbalance, supporting the visual diagnostic for differential abundance along trajectories.

## When to use

Use when you need to assess whether condition labels (e.g., treatment vs control) are locally imbalanced across a single-cell reduced-dimension representation before running trajectory-level differential topology or abundance tests.

## When NOT to use

- You already have an inferred common trajectory and want formal tests — use topologyTest, progressionTest, or fateSelectionTest instead of this diagnostic
- You need gene-level differential expression along lineages — use fitGAM plus conditionTest, not the imbalance score
- The data are steady-state cells with no dynamic/trajectory structure where plain cluster-based differential abundance is acceptable

## Inputs

- Matrix of reduced-dimension cell coordinates (e.g., Dim1, Dim2 columns of toy_dataset)
- Per-cell condition label vector (e.g., treatment/control)
- Optional parameters: neighborhood size k (default 10) and smoothing parameter s (default 10)

## Outputs

- Per-cell raw imbalance scores ('scores' column)
- Locally smoothed/scaled imbalance scores ('scaled_scores' column)
- Annotated score table (e.g., imbalance_scores.csv) for visualization

## How to apply

Load the bundled toy dataset with data('toy_dataset', package='condiments') or supply your own matrix of reduced-dimension coordinates (Dim1, Dim2) plus a condition vector. Call imbalance_score on the coordinate matrix with the condition labels: for each cell it defines a neighborhood via a k-nearest-neighbor graph (default k=10), computes a z-score comparing the local condition distribution to the global one, and locally scales/smooths raw scores with cubic splines (smoothing parameter s=10). Attach the returned 'scores' and 'scaled_scores' columns to cell metadata and visualize the smoothed scores across the reduced dimensions to check that imbalance localizes to the expected lineage. Rationale: cluster-based differential abundance ignores the continuity of cell-state transitions, so the trajectory-aware imbalance score is the recommended diagnostic.

## Related tools

- **condiments** (Provides the imbalance_score function and the bundled toy_dataset used to compute and visualize per-cell condition imbalance on the reduced-dimensional representation) — github.com/HectorRDB/condiments

## Examples

```
data("toy_dataset", package = "condiments"); ib <- imbalance_score(X = as.matrix(toy_dataset[, c("Dim1", "Dim2")]), conditions = toy_dataset$conditions)
```

## Evaluation signals

- Output table contains exactly one row per input cell with finite values in both 'scores' and 'scaled_scores' columns
- High smoothed (scaled) scores concentrate on the lineage known to be condition-imbalanced in the toy dataset
- Re-running with default parameters (k=10 neighborhoods, smoothing s=10) reproduces the expected localization pattern in the Dim1/Dim2 plot
- Condition label vector matches the coordinate matrix row-for-row with no missing labels
- Visual diagnostic shows imbalance localized rather than diffuse, consistent with a genuine local condition shift

## Limitations

- The score is a diagnostic, not a statistical test — it does not provide p-values or a probabilistic interpretation of imbalance
- Results depend on the quality of the low-dimensional representation; no independent benchmarking of the full condiments workflow has yet been performed
- Pseudotime/neighborhood estimates share dependence with downstream analyses (double-dipping), which the method ignores rather than resolves

## Evidence

- [other] The imbalance score assigns each cell a score measuring the imbalance between the local and global distributions of condition labels, with cell neighborhoods defined via a k-nearest neighbor graph: [evidence span withheld — anchor status: not_found]
- [methods] Filter step naming the function and section: "using the *imbalance_score* function"
- [methods] Bundled toy dataset loading command: "data("toy_dataset", package = "condiments")"
- [intro] Cluster-based DA approaches are suboptimal for dynamic systems: "clustering the cells and assessing condition imbalances for each cluster ignores the continuity of the cellular state transition process, making these approaches suboptimal."
- [readme] README confirming package installation and usage entry points: [evidence span withheld — anchor status: external_doc]
