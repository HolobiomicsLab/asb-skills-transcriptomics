---
name: two-sample-distribution-testing-between-conditions
description: Use when cells from two conditions share a single slingshot trajectory
  and you must test whether lineage-membership weight distributions differ, i.e.,
  whether conditions allocate cells to lineages in different proportions.
when_to_use_negative:
- The research question is whether the trajectory topology itself differs between
  conditions (Step 1) — use topologyTest instead.
- The question is differential progression along a common lineage (pseudotime distributions)
  — use progressionTest, not fateSelectionTest.
- The goal is gene-level differential expression along lineages (Step 3) — use conditionTest
  / fitGAM instead.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: condiments
  role: Provides the fateSelectionTest function implementing the classifier-based
    two-sample test on lineage weights between conditions
  repo: https://github.com/HectorRDB/condiments
- name: slingshot
  role: Supplies the shared trajectory (sds) and per-cell lineage-membership curve
    weights that form the distributions being compared
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:230969de3436994527fdc733be69af3c681726c6b5bd518ac5fd5f12c687f9ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HectorRDB/condiments
  tools:
  - R
  - condiments
  - slingshot
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# two-sample-distribution-testing-between-conditions

## Summary

A two-sample distributional test (as instantiated by condiments' fateSelectionTest) that assesses whether two multivariate or scalar distributions differ, here applied to slingshot lineage-membership weights to detect differential fate selection between biological conditions.

## When to use

Use when cells from two conditions share a single slingshot trajectory and you must test whether lineage-membership weight distributions differ, i.e., whether conditions allocate cells to lineages in different proportions.

## When NOT to use

- The research question is whether the trajectory topology itself differs between conditions (Step 1) — use topologyTest instead.
- The question is differential progression along a common lineage (pseudotime distributions) — use progressionTest, not fateSelectionTest.
- The goal is gene-level differential expression along lineages (Step 3) — use conditionTest / fitGAM instead.

## Inputs

- slingshot trajectory object (sds) fit on data integrated across conditions (e.g., condiments toy_dataset with Dim1/Dim2 coordinates, conditions, cluster labels)
- per-cell condition labels (e.g., df$conditions with levels A and B)
- per-cell lineage-membership probability weights from slingCurveWeights(sds, as.probs = TRUE)

## Outputs

- table of test statistics and p-values per lineage pair (and globally if global = TRUE), e.g., written to fate_selection_test_results.csv
- a binary/global decision on differential fate selection based on p-value significance

## How to apply

Build a common slingshot trajectory (sds) from the integrated multi-condition data (e.g., the condiments toy_dataset with Dim1/Dim2 coordinates, condition labels, and cluster labels). Extract each cell's lineage-membership probabilities with slingCurveWeights(sds, as.probs = TRUE); these weights sum to 1 across lineages and are the sample-level observations for the test. Run fateSelectionTest(sds, conditions = df$conditions, global = FALSE, pairwise = TRUE) in condiments, which uses a classifier-based two-sample test (Lopez-Paz & Oquab default) with the null hypothesis that lineage-weight distributions are equal between conditions. Set a seed (e.g., set.seed(12)) before running for reproducibility. Format the returned per-pair (and global, if requested) statistics and p-values into a table (e.g., fate_selection_test_results.csv) and interpret small p-values as evidence of differential fate selection between conditions.

## Related tools

- **condiments** (Provides the fateSelectionTest function implementing the classifier-based two-sample test on lineage weights between conditions) — https://github.com/HectorRDB/condiments
- **slingshot** (Supplies the shared trajectory (sds) and per-cell lineage-membership curve weights that form the distributions being compared)

## Examples

```
set.seed(12); res <- fateSelectionTest(sds, conditions = df$conditions, global = FALSE, pairwise = TRUE)
```

## Evaluation signals

- The result table contains one test statistic and one p-value per lineage pair (pairwise = TRUE).
- The toy dataset is expected to yield a significant (small) p-value, reflecting clear differential fate selection between conditions A and B.
- Curve weights extracted via slingCurveWeights(sds, as.probs = TRUE) sum to 1 across lineages for each cell, confirming they are valid probability distributions.
- Setting the seed (e.g., set.seed(12)) before the test reproduces identical statistics and p-values on re-run.
- The test executes on a single shared sds object built from all conditions combined, not on separately inferred per-condition trajectories.

## Limitations

- The dependence of pseudotime/weight estimates on the trajectory inference (double-dipping) is ignored rather than resolved, as in other differential abundance methods.
- P-values are not given a strong probabilistic interpretation; they serve as numerical summaries, since strong interpretations would involve hard-to-verify assumptions.
- No independent benchmarking of the workflow has been performed; the authors call for independent benchmarking and real-world case studies.

## Evidence

- [methods] To test for differential fate selection, we use the __fateSelectionTest__.: "To test for differential fate selection, we use the __fateSelectionTest__."
- [intro] Step 2: test for global differences between conditions, both along lineages (differential progression) and between lineages (differential fate selection): "In Step 2, it tests for global differences between conditions, both along lineages – differential progression – and between lineages – differential fate selection."
- [results] Run fateSelectionTest(sds, conditions = df$conditions, global = FALSE, pairwise = TRUE) in condiments, using the default classifier test (Lopez-Paz & Oquab): [evidence span withheld — anchor status: not_found]
- [readme] Install from Bioconductor with BiocManager::install("condiments"): [evidence span withheld — anchor status: external_doc]
