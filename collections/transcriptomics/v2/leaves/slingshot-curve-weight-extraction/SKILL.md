---
name: slingshot-curve-weight-extraction
description: Use when a slingshot trajectory object (sds) with multiple lineages is
  available and you need each cell's lineage-membership probabilities as input to
  condiments' fateSelectionTest for differential fate selection.
when_to_use_negative:
- The question is whether the trajectory topology itself differs between conditions
  — run the topologyTest (Step 1) instead.
- The question is whether cells progress at different rates along the same lineage
  (pseudotime distribution differences) — use progressionTest (Step 2a) rather than
  fate selection.
- No trajectory has been inferred yet (raw counts without a fitted slingshot object)
  — curve weights do not exist until slingshot lineages are fitted.
edam_operation: http://edamontology.org/operation_2264
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: slingshot
  role: Provides the trajectory object (sds) and the slingCurveWeights function (as.probs
    = TRUE) that yields per-cell lineage-membership probabilities consumed by fate
    selection testing
- name: condiments
  role: Consumes the slingshot object and condition labels in fateSelectionTest to
    test the null hypothesis of equal lineage-weight distributions between conditions
  repo: github.com/HectorRDB/condiments
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:17d0772d42d10e289eef42cfccb23f4cf78ede76438470f6c65ce261cd792167
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - condiments
  - slingshot
  edam_operation: http://edamontology.org/operation_2264
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# slingshot-curve-weight-extraction

## Summary

Extracts per-cell lineage-membership probabilities (curve weights) from a fitted slingshot trajectory so they can be fed to the condiments fateSelectionTest, which tests whether lineage weight distributions differ between conditions (Step 2b, differential fate selection).

## When to use

Use when a slingshot trajectory object (sds) with multiple lineages is available and you need each cell's lineage-membership probabilities as input to condiments' fateSelectionTest for differential fate selection.

## When NOT to use

- The question is whether the trajectory topology itself differs between conditions — run the topologyTest (Step 1) instead.
- The question is whether cells progress at different rates along the same lineage (pseudotime distribution differences) — use progressionTest (Step 2a) rather than fate selection.
- No trajectory has been inferred yet (raw counts without a fitted slingshot object) — curve weights do not exist until slingshot lineages are fitted.

## Inputs

- slingshot trajectory object (sds / SlingshotDataSet) fitted on the combined multi-condition low-dimensional embedding (e.g., toy_dataset with Dim1, Dim2 coordinates and cluster labels)
- Cell-level condition labels (e.g., df$conditions with conditions A and B) aligned to the cells in sds

## Outputs

- Matrix/data.frame of lineage-membership probabilities per cell (slingCurveWeights(sds, as.probs = TRUE); rows = cells, columns = lineages, each row sums to 1)
- fateSelectionTest result table with test statistic and p-value per global/pairwise lineage comparison, written e.g. to fate_selection_test_results.csv

## How to apply

Fit a slingshot trajectory on the combined multi-condition embedding (e.g., the condiments toy_dataset built from Dim1/Dim2 coordinates, cluster labels, and condition metadata) to obtain a slingshot object sds. Extract curve weights with slingCurveWeights(sds, as.probs = TRUE), which returns each cell's lineage-membership probabilities that sum to 1 across lineages. Set a seed (e.g., set.seed(12)) before downstream testing for reproducibility. Pass the sds together with the condition vector (df$conditions) to fateSelectionTest(sds, conditions = df$conditions, global = FALSE, pairwise = TRUE), which uses the default classifier test (Lopez-Paz & Oquab) to test the null hypothesis that lineage-weight distributions are equal between conditions. Format the returned global/pairwise statistics and p-values as a table (e.g., fate_selection_test_results.csv); the toy dataset is expected to yield a significant (small) p-value indicating clear differential fate selection between conditions A and B.

## Related tools

- **slingshot** (Provides the trajectory object (sds) and the slingCurveWeights function (as.probs = TRUE) that yields per-cell lineage-membership probabilities consumed by fate selection testing)
- **condiments** (Consumes the slingshot object and condition labels in fateSelectionTest to test the null hypothesis of equal lineage-weight distributions between conditions) — github.com/HectorRDB/condiments

## Examples

```
set.seed(12); wts <- slingCurveWeights(sds, as.probs = TRUE); fst <- fateSelectionTest(sds, conditions = df$conditions, global = FALSE, pairwise = TRUE)
```

## Evaluation signals

- Each cell's weights across lineages sum to 1 when slingCurveWeights is called with as.probs = TRUE (probabilities, not raw distances).
- Weight matrix has one row per cell and one column per lineage of the fitted sds object (toy_dataset yields a two-lineage structure).
- fateSelectionTest output contains a test statistic and p-value for every requested comparison (global and/or per lineage pair).
- The toy dataset is expected to yield a significant (small) p-value indicating clear differential fate selection between conditions A and B.
- Results are reproducible when the same seed (e.g., set.seed(12)) is set before running the test.

## Limitations

- Dependence of pseudotime/curve-weight estimates on the trajectory inference (double-dipping) is ignored rather than resolved.
- p-values are not given a strong probabilistic interpretation; they carry only numerical-summary status.
- Batch-condition confounding can be inherent and unresolvable in some settings, which distorts lineage-weight comparisons.
- The weight distribution test assumes a common trajectory is meaningful; a failed differential-topology assessment (Step 1) invalidates proceeding to fate selection.

## Evidence

- [methods] To test for differential fate selection, we use the __fateSelectionTest__.: "To test for differential fate selection, we use the __fateSelectionTest__."
- [intro] Step 2 tests global differences between conditions including fate selection: "In Step 2, it tests for global differences between conditions, both along lineages – differential progression – and between lineages – differential fate selection."
- [intro] Cells are projected onto lineages with pseudotime given a root state: "Individual cells are then projected onto these lineages and – given a root state (either user-provided or automatically detected) – their distance along each path is called pseudotime."
- [methods] toy_dataset is loaded from the condiments package: "data("toy_dataset", package = "condiments")"
- [readme] The package is installed from Bioconductor and a vignette is available online: [evidence span withheld — anchor status: external_doc]
