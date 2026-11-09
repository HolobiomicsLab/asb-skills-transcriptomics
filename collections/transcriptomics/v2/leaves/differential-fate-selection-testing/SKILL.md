---
name: differential-fate-selection-testing
description: Use when a single common trajectory (e.g., slingshot sds) spans multiple
  conditions and the goal is to test whether condition-level cell abundance differs
  between lineages (differential fate selection), after topologyTest has not rejected
  a common topology.
when_to_use_negative:
- topologyTest rejects the common-trajectory null and conditions require separate
  trajectory inferences — fate comparison on a single sds is then inappropriate
- The expected difference is in pseudotime distributions along a lineage rather than
  allocation between lineages — progressionTest is the correct test
- Cells are in a steady-state system with no trajectory structure — differential abundance
  methods such as DAseq or milo are the appropriate choice
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: condiments
  role: Provides fateSelectionTest, topologyTest, imbalance_score, and the overall
    three-step multi-condition trajectory workflow
  repo: github.com/HectorRDB/condiments
- name: slingshot
  role: Infers the common trajectory and lineage weights that fateSelectionTest operates
    on downstream
- name: monocle3
  role: Alternative trajectory inference on the shared low-dimensional representation
    used in the dyngen benchmark
- name: dyngen
  role: Generates simulated multi-lineage trajectories with ground truth for benchmarking
    fateSelectionTest against DAseq and milo
provenance:
  source_task_ids:
  - task_005
  - task_007
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:ccebcb355bcadf8fe3f3fd7f3d5ed7ae09cc4a0da5712bc6cf195b0ce2eee922
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DAseq
  - condiments
  - dyngen
  - ggplot2
  - milo
  - monocle3
  - slingshot
  - tradeSeq
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# Differential fate selection testing

## Summary

The condiments fateSelectionTest evaluates whether cells from different conditions are differentially allocated to lineages of a common inferred trajectory, using a classifier-based test on lineage weights. It isolates fate-choice differences (between-lineage abundance) from along-lineage progression differences.

## When to use

Use when a single common trajectory (e.g., slingshot sds) spans multiple conditions and the goal is to test whether condition-level cell abundance differs between lineages (differential fate selection), after topologyTest has not rejected a common topology.

## When NOT to use

- topologyTest rejects the common-trajectory null and conditions require separate trajectory inferences — fate comparison on a single sds is then inappropriate
- The expected difference is in pseudotime distributions along a lineage rather than allocation between lineages — progressionTest is the correct test
- Cells are in a steady-state system with no trajectory structure — differential abundance methods such as DAseq or milo are the appropriate choice

## Inputs

- A slingshot SLINGSEQ / slingPseudotime object (sds) fitted on integrated, normalized scRNA-seq counts with cluster labels and a shared reduced-dimension representation (e.g., Dim1/Dim2 coordinates)
- Per-cell condition labels (e.g., mock vs TGF-beta, control vs ILD, WT vs KO)
- Lineage weights assigned to each cell along each trajectory path

## Outputs

- Global and (with pairwise = TRUE) pairwise lineage-contrast p-values for differential fate selection (e.g., reported p ≤ 2.2e-16 in the Fibrosis ILD case study)
- BH-adjusted binary differential-fate calls at 5% FDR for inclusion in results tables (e.g., condiments_results_table.csv, condiments_emt_results.csv, fig2e_performance_table.csv)

## How to apply

First infer a common trajectory with slingshot (or monocle3) on a shared low-dimensional representation, and confirm with the permutation-based condiments topologyTest that a common (not condition-separate) topology is adequate. Then run fateSelectionTest on the sds object with per-cell condition labels and lineage weights; the default is the classifier test of Lopez-Paz and Oquab (2016), with global = TRUE for an overall test and pairwise = TRUE for pairwise lineage comparisons. Adjust the resulting p-values with Benjamini-Hochberg at a 5% FDR threshold before declaring differential fate selection. This test targets between-lineage differences; pair it with progressionTest when the question is differential pseudotime distributions along a lineage. In benchmarks (dyngen topologies T1–T4 with 2, 3, 2, and 5 lineages under WT/KO and WT/KO/UP designs), fateSelectionTest performed second-best after progressionTest when calls were scored against simulation ground truth (TNR, TPR, PPV, NPV, F1), outperforming DAseq and milo which rely only on data-driven local neighborhoods.

## Related tools

- **condiments** (Provides fateSelectionTest, topologyTest, imbalance_score, and the overall three-step multi-condition trajectory workflow) — github.com/HectorRDB/condiments
- **slingshot** (Infers the common trajectory and lineage weights that fateSelectionTest operates on downstream)
- **monocle3** (Alternative trajectory inference on the shared low-dimensional representation used in the dyngen benchmark)
- **dyngen** (Generates simulated multi-lineage trajectories with ground truth for benchmarking fateSelectionTest against DAseq and milo)

## Examples

```
fateTest <- condiments::fateSelectionTest(sds, conditions = df$conditions, global = TRUE, pairwise = TRUE)
```

## Evaluation signals

- topologyTest has been run first and fails to reject the common-trajectory null (e.g., reported p = 0.38 in the EMT case study)
- Output contains both global and pairwise lineage p-values when pairwise = TRUE, computed with the default classifier test (Lopez-Paz and Oquab, 2016)
- In the fibrosis case study, a significant fateSelectionTest (p ≤ 2.2e-16) should coincide with imbalance_score visualizing local condition imbalance and control cells almost exclusively in Lineage 2
- When benchmarked on dyngen T1–T4 simulations, BH-adjusted calls at 5% FDR must yield all five metrics (TNR, TPR, PPV, NPV, F1) per method, with fateSelectionTest ranking second behind progressionTest
- Results table (e.g., condiments_results_table.csv) must confirm the progressionTest-driven lineage-level pattern is separable from the fate-selection signal

## Limitations

- Dependence of pseudotime/weight estimates on trajectory inference ('double-dipping') is ignored rather than resolved
- P-values are given a numerical-summary status, not a strong probabilistic interpretation
- Batch-condition confounding can be inherent and unresolvable in some settings
- No independent benchmarking of the workflow has been performed; real-world benchmarking case studies are still lacking

## Evidence

- [methods] To test for differential fate selection, we use the __fateSelectionTest__.: "To test for differential fate selection, we use the __fateSelectionTest__."
- [intro] Step 2 fate selection definition: "In Step 2, it tests for global differences between conditions, both along lineages – differential progression – and between lineages – differential fate selection."
- [discussion] Trajectory-level abundance tests: "then differences in abundance at the trajectory level with the progressionTest and fateSelectionTest"
- [intro] Benchmark rationale vs neighborhood methods: [evidence span withheld — anchor status: not_found]
- [readme] Package availability: [evidence span withheld — anchor status: external_doc]
