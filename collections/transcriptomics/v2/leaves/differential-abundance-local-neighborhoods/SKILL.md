---
name: differential-abundance-local-neighborhoods
description: Use when comparing cell condition compositions in single-cell data with
  a shared low-dimensional representation but no trajectory assumed, or when computing
  per-cell imbalance scores for a visual diagnostic.
when_to_use_negative:
- A trajectory (lineages with pseudotime) has been inferred and the goal is differential
  progression or fate selection — trajectory-aware condiments tests outperform neighborhood-only
  DA in the article's dyngen benchmark
- The system is a dynamic process where clustering or local neighborhoods ignore the
  continuity of cell state transitions
- No shared low-dimensional representation across conditions exists, since both DAseq
  and milo require one
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: DAseq
  role: Differential abundance testing via data-driven local neighborhoods on a common
    low-dimensional representation
- name: milo
  role: Differential abundance testing borrowing from mass cytometry, using local
    neighborhoods on a shared embedding
- name: condiments
  role: Provides imbalance_score for the per-cell local-vs-global condition distribution
    diagnostic and trajectory-aware comparators
  repo: github.com/HectorRDB/condiments
- name: dyngen
  role: Simulation framework generating dyngen trajectory datasets with ground truth
    to benchmark DA methods
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:fc0c326b3efa90fc332371326fa1805094c48d6531e1c49a0c3f11e497c1e241
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DAseq
  - condiments
  - dyngen
  - milo
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# differential-abundance-local-neighborhoods

## Summary

Detects condition imbalance in single-cell data using neighborhood-based differential abundance methods (DAseq, milo) and the condiments imbalance_score diagnostic, which scores cells by comparing local to global condition distributions on a k-nearest-neighbor graph. The article benchmarks these neighborhood-only approaches against trajectory-aware condiments tests.

## When to use

Use when comparing cell condition compositions in single-cell data with a shared low-dimensional representation but no trajectory assumed, or when computing per-cell imbalance scores for a visual diagnostic.

## When NOT to use

- A trajectory (lineages with pseudotime) has been inferred and the goal is differential progression or fate selection — trajectory-aware condiments tests outperform neighborhood-only DA in the article's dyngen benchmark
- The system is a dynamic process where clustering or local neighborhoods ignore the continuity of cell state transitions
- No shared low-dimensional representation across conditions exists, since both DAseq and milo require one

## Inputs

- Low-dimensional cell representation (e.g., Dim1/Dim2 coordinates or PCA/UMAP matrix) shared across conditions
- Per-cell condition label vector (e.g., WT/KO)
- Count matrix for differential abundance methods (DAseq, milo)
- Neighborhood size k (default k=10 for imbalance_score)

## Outputs

- Per-cell raw imbalance 'scores' and locally smoothed 'scaled_scores' columns (imbalance_score)
- Differential abundance calls or p-values per neighborhood/cluster (DAseq, milo)
- BH-adjusted (5% FDR) binary differential calls per cell or neighborhood
- Performance metrics (TNR, TPR, PPV, NPV, F1) against simulation ground truth

## How to apply

Both DAseq and milo rely on a common low-dimensional representation of all observations and define data-driven local neighborhoods, then test for differential abundance per neighborhood. For the condiments diagnostic, run imbalance_score on the reduced-dimension coordinate matrix (e.g., Dim1/Dim2 of the bundled toy_dataset) with the condition vector; it computes a k-nearest-neighbor (default k=10) z-score of the local condition distribution versus the global one, then locally smooths/scales scores with cubic-spline smoothing (default s=10), returning 'scores' and 'scaled_scores' columns. The article cautions that clustering cells and assessing condition imbalances per cluster ignores the continuity of cell state transitions; benchmarking on dyngen simulations (topologies T1-T4 with 2-5 lineages) with Benjamini-Hochberg correction at 5% FDR showed these neighborhood-only methods underperform trajectory-aware progressionTest and fateSelectionTest. Use this skill when no trajectory structure is assumed or as a baseline comparator, and prefer trajectory-aware tests when a dynamic process is present.

## Related tools

- **DAseq** (Differential abundance testing via data-driven local neighborhoods on a common low-dimensional representation)
- **milo** (Differential abundance testing borrowing from mass cytometry, using local neighborhoods on a shared embedding)
- **condiments** (Provides imbalance_score for the per-cell local-vs-global condition distribution diagnostic and trajectory-aware comparators) — github.com/HectorRDB/condiments
- **dyngen** (Simulation framework generating dyngen trajectory datasets with ground truth to benchmark DA methods)

## Examples

```
data('toy_dataset', package='condiments'); scores <- imbalance_score(X = toy[,c('Dim1','Dim2')], conditions = toy$conditions)
```

## Evaluation signals

- imbalance_scores.csv contains one row per cell with finite raw and smoothed score values
- High smoothed scores concentrate on the lineage known to be imbalanced in the toy dataset
- All neighborhoods/cells receive BH-adjusted p-values at a 5% FDR threshold yielding binary differential calls
- Benchmark table contains TNR, TPR, PPV, NPV, and F1 for each method across T1-T4 topologies
- Neighborhood-only methods (DAseq, milo) show lower F1/TPR than trajectory-aware progressionTest on dyngen simulations with two or three lineages

## Limitations

- Clustering the cells and assessing condition imbalances for each cluster ignores the continuity of the cellular state transition process, making these approaches suboptimal
- In simulation benchmarks, DAseq and milo performed worse than trajectory-aware condiments tests (progressionTest and fateSelectionTest)
- The imbalance score is a visual diagnostic, not a statistical test with p-value calibration

## Evidence

- [intro] DAseq and milo rely on common low-dimensional representation and data-driven local neighborhoods: "milo16 and DAseq17 rely on a common low-dimensional representation of all observations and define data-driven local neighborhoods"
- [intro] Clustering ignores continuity of cell state transitions: "clustering the cells and assessing condition imbalances for each cluster ignores the continuity of the cellular state transition process, making these approaches suboptimal."
- [results] Benchmark shows condiments tests outperform DA methods on dyngen simulations: [evidence span withheld — anchor status: not_found]
- [methods] imbalance_score function in methods: "using the *imbalance_score* function"
- [methods] toy_dataset bundled with condiments: "data("toy_dataset", package = "condiments")"
- [intro] Benchmark goal: leveraging trajectory improves DA assessment: [evidence span withheld — anchor status: not_found]
