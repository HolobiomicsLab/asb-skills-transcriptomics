---
name: single-cell-trajectory-fate-selection-testing
description: Use when a slingshot SingleCellExperiment/trajectory object exists with
  per-cell lineage curve weights across ≥2 conditions and the goal is to test whether
  condition shifts the proportions of cells committed to each lineage.
when_to_use_negative:
- The question is differential progression (pseudotime shifts along the same lineage)
  — use progressionTest instead of fateSelectionTest.
- Trajectory topology itself may differ between conditions — run topologyTest (Step
  1) first, since a common trajectory is a prerequisite for Step 2 tests.
- The goal is gene-level differential expression along lineages — use fitGAM plus
  conditionTest (Step 3).
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: condiments
  role: Provides fateSelectionTest to compare lineage-weight distributions between
    conditions (Step 2b of the workflow)
  repo: github.com/HectorRDB/condiments
- name: slingshot
  role: Supplies the trajectory object and per-cell lineage curve weights (slingCurveWeights,
    as.probs = TRUE) that fateSelectionTest consumes
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:391e7bd9db48b338bbe22be35f583ecb7a88e022ece9900867b067e7e0c75239
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - condiments
  - slingshot
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# single-cell-trajectory-fate-selection-testing

## Summary

Assess differential fate selection — whether cells in different conditions differentiate in varying proportions between lineages — by testing the null hypothesis that lineage-weight (lineage-membership probability) distributions are equal between conditions, using the fateSelectionTest from the condiments workflow (Step 2b).

## When to use

Use when a slingshot SingleCellExperiment/trajectory object exists with per-cell lineage curve weights across ≥2 conditions and the goal is to test whether condition shifts the proportions of cells committed to each lineage.

## When NOT to use

- The question is differential progression (pseudotime shifts along the same lineage) — use progressionTest instead of fateSelectionTest.
- Trajectory topology itself may differ between conditions — run topologyTest (Step 1) first, since a common trajectory is a prerequisite for Step 2 tests.
- The goal is gene-level differential expression along lineages — use fitGAM plus conditionTest (Step 3).

## Inputs

- slingshot trajectory object (sds) built from a multi-condition dataset (e.g. condiments toy_dataset with Dim1/Dim2 coordinates)
- per-cell condition labels (e.g. df$conditions with levels A and B)
- per-cell lineage membership probabilities from slingCurveWeights(sds, as.probs = TRUE)

## Outputs

- Global test statistic and p-value for equality of lineage-weight distributions across conditions
- Pairwise test statistics and p-values per lineage pair, formatted as a table (e.g. fate_selection_test_results.csv)

## How to apply

Integrate the conditions into a single trajectory and extract lineage membership probabilities with slingCurveWeights(sds, as.probs = TRUE); these weights sum to 1 per cell. Run fateSelectionTest(sds, conditions = df$conditions, global = FALSE, pairwise = TRUE) in condiments, which by default uses the classifier-based test (Lopez-Paz & Oquab) to compare lineage-weight distributions between conditions. Set a seed (e.g. set.seed(12)) before running for reproducibility. Request global = TRUE when a single omnibus verdict is needed and pairwise = TRUE for per-lineage-pair statistics. The test is Step 2b of the condiments three-step workflow, applied after topology has been checked with topologyTest and alongside progressionTest for differential progression; format the resulting statistics and p-values as a table (e.g. fate_selection_test_results.csv).

## Related tools

- **condiments** (Provides fateSelectionTest to compare lineage-weight distributions between conditions (Step 2b of the workflow)) — github.com/HectorRDB/condiments
- **slingshot** (Supplies the trajectory object and per-cell lineage curve weights (slingCurveWeights, as.probs = TRUE) that fateSelectionTest consumes)

## Examples

```
set.seed(12); res <- condiments::fateSelectionTest(sds, conditions = df$conditions, global = FALSE, pairwise = TRUE)
```

## Evaluation signals

- The result table contains a test statistic and p-value per lineage pair (pairwise = TRUE) plus a global row.
- Sling curve weights are valid probabilities: each cell's weights across lineages sum to 1.
- A dataset with known differential fate selection (e.g. the condiments toy_dataset, conditions A vs B) yields a small/significant p-value.
- Output is reproducible when the same seed (e.g. set.seed(12)) is reused.
- Results are written in a well-formed CSV (fate_selection_test_results.csv) with statistic and p-value columns.

## Limitations

- The dependence of pseudotime/weight estimates on the same data is ignored ('double-dipping'), as in other differential abundance methods.
- P-values are treated as numerical summaries rather than given strong probabilistic interpretations.
- No independent benchmarking of the workflow has been performed; the authors call for it and note real-world benchmarking case studies are still lacking.
- Batch-condition confounding can be inherent and unresolvable in some settings.

## Evidence

- [methods] fateSelectionTest tests classifier separability of lineage weights: "To test for differential fate selection, we use the __fateSelectionTest__."
- [intro] Step 2 covers differential progression and fate selection: "In Step 2, it tests for global differences between conditions, both along lineages – differential progression – and between lineages – differential fate selection."
- [discussion] Workflow positions fateSelectionTest after topologyTest and with progressionTest: "then differences in abundance at the trajectory level with the progressionTest and fateSelectionTest"
- [readme] Tool availability and installation via Bioconductor: [evidence span withheld — anchor status: external_doc]
