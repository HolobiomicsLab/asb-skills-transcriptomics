---
name: lineage-membership-weight-interpretation
description: Use when you have a common slingshot trajectory (sds) spanning multiple
  conditions and need to test whether lineage-membership weight distributions differ
  between conditions (differential fate selection).
when_to_use_negative:
- The question is whether cells progress along the same lineage at different rates
  (differential progression) — use progressionTest instead of fateSelectionTest
- The trajectory topology itself differs fundamentally between conditions — run the
  topologyTest (Step 1) first, since fate selection presupposes a common trajectory
- You need gene-level condition-specific expression changes — that is Step 3, handled
  by conditionTest on tradeSeq-style fits
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: condiments
  role: provides fateSelectionTest and imbalance tooling to test whether lineage-weight
    distributions are equal between conditions (Step 2b)
  repo: https://github.com/HectorRDB/condiments
- name: slingshot
  role: supplies the common trajectory object (sds) and slingCurveWeights lineage-membership
    probabilities that the test consumes
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:3207f89d9c0a620374e2341f69cec1c612b5178678c6aed1845ac094728f83b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HectorRDB/condiments
  tools:
  - R
  - condiments
  - slingshot
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# lineage-membership-weight-interpretation

## Summary

Interprets cell lineage-membership (curve) weights from a slingshot trajectory to test for differential fate selection in condiments: whether cells across conditions differentiate in different proportions between lineages. It applies the fateSelectionTest (Step 2b of the condiments workflow) to slingCurveWeights distributions.

## When to use

Use when you have a common slingshot trajectory (sds) spanning multiple conditions and need to test whether lineage-membership weight distributions differ between conditions (differential fate selection).

## When NOT to use

- The question is whether cells progress along the same lineage at different rates (differential progression) — use progressionTest instead of fateSelectionTest
- The trajectory topology itself differs fundamentally between conditions — run the topologyTest (Step 1) first, since fate selection presupposes a common trajectory
- You need gene-level condition-specific expression changes — that is Step 3, handled by conditionTest on tradeSeq-style fits

## Inputs

- slingshot trajectory object (sds) fitted on a common embedding (e.g., toy_dataset Dim1/Dim2) across all conditions
- per-cell condition labels (e.g., df$conditions with levels such as A and B)
- cluster labels / root state used for trajectory construction

## Outputs

- lineage-membership weight matrix from slingCurveWeights(sds, as.probs = TRUE) with rows summing to 1
- table of fateSelectionTest statistics and p-values per lineage pair (and global), e.g., fate_selection_test_results.csv

## How to apply

Build a single trajectory across conditions (e.g., slingshot on the condiments toy_dataset with Dim1, Dim2 coordinates, conditions, and cluster labels), then extract each cell's lineage-membership probabilities with slingCurveWeights(sds, as.probs = TRUE) so weights across lineages sum to 1. Set a seed (e.g., set.seed(12)) for reproducibility. Run fateSelectionTest(sds, conditions = df$conditions, global = FALSE, pairwise = TRUE), which uses the default classifier test (Lopez-Paz & Oquab) to test the null hypothesis that lineage-weight distributions are equal between conditions. Collect the per-pair (and optionally global) statistics and p-values into a table (e.g., fate_selection_test_results.csv); this is Step 2b of the condiments three-step workflow (differential topology → differential progression/fate selection → gene-level conditionTest). Interpret a small p-value as evidence that conditions A and B differentiate in different proportions between lineages.

## Related tools

- **condiments** (provides fateSelectionTest and imbalance tooling to test whether lineage-weight distributions are equal between conditions (Step 2b)) — https://github.com/HectorRDB/condiments
- **slingshot** (supplies the common trajectory object (sds) and slingCurveWeights lineage-membership probabilities that the test consumes)

## Examples

```
set.seed(12); res <- fateSelectionTest(sds, conditions = df$conditions, global = FALSE, pairwise = TRUE)
```

## Evaluation signals

- slingCurveWeights(sds, as.probs = TRUE) returns weights that sum to 1 across lineages for every cell
- The result table contains a test statistic and p-value for every lineage pair (and a global row when global = TRUE)
- On the condiments toy_dataset, conditions A vs B are expected to yield a significant (small) p-value indicating clear differential fate selection
- The p-value is reproducible when the seed (e.g., set.seed(12)) is fixed before running the classifier-based test
- The test was only applied after the topologyTest (Step 1) established a common trajectory across conditions

## Limitations

- Dependence of pseudotime/curve-weight estimates on the trajectory inference itself (double-dipping) is ignored rather than resolved
- P-values are treated as numerical summaries rather than probabilistic interpretations, which would require hard-to-verify assumptions
- No independent benchmarking of the workflow has been performed; the authors call for it, and real-world case studies are still becoming available

## Evidence

- [methods] To test for differential fate selection, we use the __fateSelectionTest__.: "To test for differential fate selection, we use the __fateSelectionTest__."
- [intro] In Step 2, it tests for global differences between conditions, both along lineages – differential progression – and between lineages – differential fate selection.: "In Step 2, it tests for global differences between conditions, both along lineages – differential progression – and between lineages – differential fate selection."
- [discussion] then differences in abundance at the trajectory level with the progressionTest and fateSelectionTest: "then differences in abundance at the trajectory level with the progressionTest and fateSelectionTest"
- [methods] data("toy_dataset", package = "condiments"): "data("toy_dataset", package = "condiments")"
- [readme] You can install the `condiments` from [bioconductor](https://bioconductor.org/packages/release/bioc/html/condiments.html) using: [evidence span withheld — anchor status: external_doc]
