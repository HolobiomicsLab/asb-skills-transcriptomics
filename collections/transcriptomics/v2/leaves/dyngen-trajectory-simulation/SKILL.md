---
name: dyngen-trajectory-simulation
description: Use when you need synthetic dyngen-simulated trajectory datasets with
  known ground-truth condition differences to benchmark differential progression,
  fate selection, or differential abundance methods against simulated reality.
when_to_use_negative:
- The data are real-world, steady-state samples without a dynamic trajectory structure
  — the paper notes clustering-based differential abundance is the reasonable approach
  in steady-state systems
- You lack simulation ground truth and only need differential calls on one real dataset
  — this skill is a benchmarking harness, not a per-dataset analysis pipeline
- The system's trajectory topology itself differs fundamentally between conditions
  (differential topology), which is tested with topologyTest rather than progression/fate
  selection tests
tools:
- name: dyngen
  role: Simulation framework used to generate synthetic multi-lineage trajectory datasets
    with known ground-truth condition changes
- name: condiments
  role: Provides progressionTest and fateSelectionTest, the trajectory-aware methods
    being benchmarked downstream of slingshot
  repo: https://github.com/HectorRDB/condiments
- name: slingshot
  role: Trajectory inference supplying lineages and pseudotime for condiments progressionTest
    and fateSelectionTest
- name: monocle3
  role: Alternative trajectory inference used for monocle3-based progression testing
    on the shared low-dimensional representation
- name: DAseq
  role: Neighborhood-based differential abundance comparator relying on data-driven
    local neighborhoods
- name: milo
  role: Neighborhood-based differential abundance comparator borrowing from the mass
    cytometry field
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:ce48a909503dc4ea061cbe3d6f2512c8abd01810d6f4cb186102c0e903840f92
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HectorRDB/condiments
  tools:
  - DAseq
  - condiments
  - dyngen
  - milo
  - monocle3
  - slingshot
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# dyngen-trajectory-simulation

## Summary

Benchmark skill for simulating multi-lineage single-cell trajectory datasets with dyngen and using them as ground truth to compare trajectory-aware differential testing (condiments progressionTest/fateSelectionTest) against neighborhood-based differential abundance methods (DAseq, milo), with BH correction at 5% FDR and TNR/TPR/PPV/NPV/F1 scoring.

## When to use

Use when you need synthetic dyngen-simulated trajectory datasets with known ground-truth condition differences to benchmark differential progression, fate selection, or differential abundance methods against simulated reality.

## When NOT to use

- The data are real-world, steady-state samples without a dynamic trajectory structure — the paper notes clustering-based differential abundance is the reasonable approach in steady-state systems
- You lack simulation ground truth and only need differential calls on one real dataset — this skill is a benchmarking harness, not a per-dataset analysis pipeline
- The system's trajectory topology itself differs fundamentally between conditions (differential topology), which is tested with topologyTest rather than progression/fate selection tests

## Inputs

- dyngen simulation configuration (topology T1-T4: 2, 3, 2, 5 lineages) with WT/KO or WT/KO/UP condition design and an m parameter grid
- fixed random seed for reproducible simulation
- simulated count matrices with per-cell condition labels and per-gene ground-truth differential status
- shared low-dimensional representation of the simulated cells

## Outputs

- slingshot and monocle3 trajectory fits (lineages and pseudotime) per simulated dataset
- BH-adjusted p-values (5% FDR) and binary differential calls from progressionTest, fateSelectionTest, DAseq, and milo
- fig2e_performance_table.csv with TNR, TPR, PPV, NPV, and F1 for each of the four methods across T1-T4

## How to apply

Regenerate dyngen simulations for the target topologies — T1 (2 lineages), T2 (3 lineages), T3 (2 lineages), T4 (5 lineages) — under WT/KO and WT/KO/UP condition designs across the specified m grid with a fixed random seed for reproducibility. Infer trajectories on each simulated dataset using slingshot and monocle3 on a shared low-dimensional representation. Downstream of slingshot, run condiments progressionTest (global and per-lineage pseudotime distribution testing) and fateSelectionTest (pairwise lineage-weight testing), plus monocle3-based progression testing where applicable; in parallel run DAseq and milo, which rely on a common low-dimensional representation and data-driven local neighborhoods. Adjust all p-values with Benjamini-Hochberg at a 5% FDR threshold to obtain binary differential calls, then score each method against the dyngen simulation ground truth by computing TNR, TPR, PPV, NPV, and F1 and writing a Fig. 2e-style performance table. The rationale, per the paper, is that in dynamic systems clustering cells and assessing condition imbalances per cluster ignores the continuity of cell state transitions, so leveraging the trajectory representation should improve detection of true between-condition changes.

## Related tools

- **dyngen** (Simulation framework used to generate synthetic multi-lineage trajectory datasets with known ground-truth condition changes)
- **condiments** (Provides progressionTest and fateSelectionTest, the trajectory-aware methods being benchmarked downstream of slingshot) — https://github.com/HectorRDB/condiments
- **slingshot** (Trajectory inference supplying lineages and pseudotime for condiments progressionTest and fateSelectionTest)
- **monocle3** (Alternative trajectory inference used for monocle3-based progression testing on the shared low-dimensional representation)
- **DAseq** (Neighborhood-based differential abundance comparator relying on data-driven local neighborhoods)
- **milo** (Neighborhood-based differential abundance comparator borrowing from the mass cytometry field)

## Evaluation signals

- The compiled performance table contains all five metrics (TNR, TPR, PPV, NPV, F1) for each of the four methods (progressionTest, fateSelectionTest, DAseq, milo) across T1-T4
- All p-values are Benjamini-Hochberg adjusted and calls are thresholded at a 5% FDR level before scoring
- Fixed random seed makes dyngen simulation regeneration reproducible (identical datasets on re-run)
- Trajectory-aware methods (progressionTest, fateSelectionTest) show higher TPR/F1 than neighborhood-only DAseq and milo, matching the paper's reported simulation findings where progressionTest performed best
- Each simulated gene's calls are scored against the dyngen ground-truth differential status, so confusion-matrix counts reconcile with the known number of truly differential genes

## Limitations

- Simulation-grounded only: the paper states no independent benchmarking of the workflow has been performed and real-world benchmarking case studies are still lacking
- Dependence of pseudotime/weight estimates on the shared trajectory inference (double-dipping) is ignored rather than resolved
- p-values carry only numerical-summary status rather than strong probabilistic interpretation, and batch-condition confounding can be inherent and unresolvable in some settings

## Evidence

- [discussion] Simulation frameworks such as dyngen are crucial for benchmarking: "Simulation frameworks such as dyngen24 are crucial."
- [intro] Benchmark motivation: trajectory structure improves differential assessment: [evidence span withheld — anchor status: not_found]
- [discussion] Simulation finding: trajectory representation detects true changes better: "taking into account the dynamic nature of biological systems via the trajectory representation enables condiments to better detect true changes"
- [intro] Cluster-based approaches are suboptimal in dynamic systems: "clustering the cells and assessing condition imbalances for each cluster ignores the continuity of the cellular state transition process"
