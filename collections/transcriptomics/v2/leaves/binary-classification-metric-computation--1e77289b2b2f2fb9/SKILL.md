---
name: binary-classification-metric-computation
description: Use when comparing differential detection methods on simulated trajectory
  data with known ground truth, where each method's per-gene (or per-state) calls
  must be scored as binary classification metrics.
when_to_use_negative:
- Input is real experimental data without simulation ground truth — TPR/PPV/NPV cannot
  be computed without known differential labels
- Methods produce continuous scores without p-values, making BH correction at a common
  5% FDR threshold inapplicable
- Systems are steady-state without a trajectory; the article notes clustering-based
  differential abundance is the reasonable approach for steady-state systems
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0091
tools:
- name: condiments
  role: Provides progressionTest and fateSelectionTest, the trajectory-aware differential
    tests being benchmarked as binary classifiers
  repo: github.com/HectorRDB/condiments
- name: dyngen
  role: Simulates trajectory datasets (T1-T4 topologies) with ground truth condition
    labels against which binary calls are scored
- name: DAseq
  role: Neighborhood-based differential abundance method scored as a baseline binary
    classifier
- name: milo
  role: Neighborhood-based differential abundance method scored as a baseline binary
    classifier
- name: slingshot
  role: Trajectory inference upstream of progressionTest and fateSelectionTest on
    the shared low-dimensional representation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:d555949d24f94ac19b5bd550aa9401899d671f2fdbd8edb848d7e81f8661b971
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DAseq
  - condiments
  - dyngen
  - milo
  - slingshot
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# binary-classification-metric-computation

## Summary

Benchmark differential-state/differential-abundance callers (condiments progressionTest, fateSelectionTest, DAseq, milo) as binary classifiers against dyngen simulation ground truth, computing TNR, TPR, PPV, NPV, and F1 on Benjamini-Hochberg-adjusted p-values at a 5% FDR threshold.

## When to use

Use when comparing differential detection methods on simulated trajectory data with known ground truth, where each method's per-gene (or per-state) calls must be scored as binary classification metrics.

## When NOT to use

- Input is real experimental data without simulation ground truth — TPR/PPV/NPV cannot be computed without known differential labels
- Methods produce continuous scores without p-values, making BH correction at a common 5% FDR threshold inapplicable
- Systems are steady-state without a trajectory; the article notes clustering-based differential abundance is the reasonable approach for steady-state systems

## Inputs

- dyngen-simulated single-cell trajectory datasets (T1-T4 topologies: 2, 3, 2, and 5 lineages) under WT/KO and WT/KO/UP condition designs across an m grid, with simulation ground truth
- raw p-values per gene/state from progressionTest (slingshot- and monocle3-based), fateSelectionTest, DAseq, and milo
- adjusted p-values from Benjamini-Hochberg correction at 5% FDR

## Outputs

- Binary differential calls per method per gene/state (adjusted p <= 0.05)
- fig2e_performance_table.csv containing TNR, TPR, PPV, NPV, and F1 for each of the four methods across T1-T4

## How to apply

First convert each method's raw p-values into binary differential calls using Benjamini-Hochberg correction at a 5% FDR threshold, so all four methods (progressionTest downstream of slingshot or monocle3, fateSelectionTest downstream of slingshot, and neighborhood-based DAseq and milo) are scored on a common decision rule. Then, for each dyngen-simulated dataset (topologies T1-T4 with 2, 3, 2, and 5 lineages under WT/KO and WT/KO/UP designs), compare the binarized calls against the simulation ground truth to compute TNR, TPR, PPV, NPV, and F1. Write the resulting performance table (Fig. 2e-style) to fig2e_performance_table.csv. Rationale: trajectory-aware tests leverage the dynamic continuity of cell states, which the article argues improves detection of true between-condition changes compared with clustering- and neighborhood-based differential abundance approaches. Regenerate simulations with a fixed random seed to keep the benchmark reproducible.

## Related tools

- **condiments** (Provides progressionTest and fateSelectionTest, the trajectory-aware differential tests being benchmarked as binary classifiers) — github.com/HectorRDB/condiments
- **dyngen** (Simulates trajectory datasets (T1-T4 topologies) with ground truth condition labels against which binary calls are scored)
- **DAseq** (Neighborhood-based differential abundance method scored as a baseline binary classifier)
- **milo** (Neighborhood-based differential abundance method scored as a baseline binary classifier)
- **slingshot** (Trajectory inference upstream of progressionTest and fateSelectionTest on the shared low-dimensional representation)

## Examples

```
sce <- fitGAM(counts = counts, sds = sds, conditions = df$conditions)
```

## Evaluation signals

- fig2e_performance_table.csv contains all five metrics (TNR, TPR, PPV, NPV, F1) for each of the four methods across topologies T1-T4
- All methods were binarized with the same Benjamini-Hochberg 5% FDR correction before scoring, ensuring a common decision rule
- Trajectory-aware methods (progressionTest, fateSelectionTest) show higher TPR/F1 than neighborhood-only DAseq and milo, matching the paper's reported simulation findings
- Simulations were regenerated with a fixed random seed, so metrics reproduce across runs
- Per-lineage pseudotime and lineage-weight testing results are consistent between the slingshot-based and monocle3-based progressionTest runs

## Limitations

- Ground-truth-based scoring is only possible on dyngen simulations; the article notes independent benchmarking had not been performed and real-world benchmarking case studies were still lacking
- Dependence of pseudotime/weight estimates on trajectory inference (double-dipping) is ignored rather than resolved
- P-values are given a numerical-summary status only, without probabilistic interpretation; batch-condition confounding can be inherent and unresolvable in some settings

## Evidence

- [intro] Benchmark condiments against more general differential abundance methods: [evidence span withheld — anchor status: not_found]
- [discussion] Simulation rationale: "taking into account the dynamic nature of biological systems via the trajectory representation enables condiments to better detect true changes"
- [intro] Cluster-based approach limitation: "clustering the cells and assessing condition imbalances for each cluster ignores the continuity of the cellular state transition process, making these approaches suboptimal"
- [discussion] Benchmarking need acknowledged: "The two issues raised in the previous paragraphs highlight the need for independent benchmarking."
- [readme] Package installation: [evidence span withheld — anchor status: external_doc]
