---
name: condition-label-permutation-hypothesis-testing
description: 'Decision support for Step 1 of the condiments workflow: deciding whether
  a single common trajectory suffices for all cells or whether developmental trajectories
  differ between conditions (differential topology).'
when_to_use_negative:
- The trajectory inference method does not allow specification of an underlying skeleton
  structure, since permutations cannot maintain a mapping between trajectories
- The question is differential progression or differential fate selection on a shared
  trajectory (use progressionTest / fateSelectionTest, Step 2), not whether the topology
  itself differs
- The system is steady-state with no dynamic process, where cluster-based differential
  abundance assessment is the reasonable approach
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: condiments
  role: Provides the topologyTest permutation framework for assessing differential
    topology between conditions
  repo: github.com/HectorRDB/condiments
- name: slingshot
  role: Skeleton-based trajectory inference method that supplies the fitted common
    trajectory input to topologyTest
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:99dde2939e9beee409285adb950253979ab3a0ff0b8d38f3f7c969d35c69bf28
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - condiments
  - slingshot
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Condition-label permutation hypothesis testing

## Summary

A permutation-based hypothesis-testing strategy from the condiments workflow: assesses whether trajectories differ fundamentally between conditions (differential topology) by comparing condition-specific pseudotime distributions to their distribution under permuted condition labels.

## When to use

Decision support for Step 1 of the condiments workflow: deciding whether a single common trajectory suffices for all cells or whether developmental trajectories differ between conditions (differential topology).

## When NOT to use

- The trajectory inference method does not allow specification of an underlying skeleton structure, since permutations cannot maintain a mapping between trajectories
- The question is differential progression or differential fate selection on a shared trajectory (use progressionTest / fateSelectionTest, Step 2), not whether the topology itself differs
- The system is steady-state with no dynamic process, where cluster-based differential abundance assessment is the reasonable approach

## Inputs

- Reduced-dimension cell coordinates (e.g., Dim1/Dim2 from condiments toy_dataset)
- Per-cell condition assignments (treatment/control or equivalent)
- Per-cell cluster labels for trajectory skeleton construction
- A fitted trajectory object from a skeleton-based TI method (slingshot or TSCAN)

## Outputs

- Test statistic table for differential topology
- Permutation-based p-value (default over 100 label permutations)
- Binary decision: common trajectory acceptable vs. fundamentally different trajectories between conditions

## How to apply

Load condition assignments, cluster labels, and low-dimensional coordinates (e.g., the toy_dataset with Dim1/Dim2 from the condiments package), then infer a common trajectory with slingshot on the 2D embedding using cluster labels (sds <- slingshot(rd, df$cl)). Run condiments::topologyTest on the fitted slingshot object with the condition vector, permuting condition labels (default 100 repeats) to build the null distribution of identical trajectories across conditions. A mapping between trajectories must be maintained across permutations, so the test requires trajectory inference methods that accept an underlying skeleton structure, such as slingshot or TSCAN. Include a minimum magnitude requirement so only biologically relevant topological differences are called significant, and write the resulting statistics and permutation-based p-value table (e.g., topology_test_results.csv) for downstream decision-making.

## Related tools

- **condiments** (Provides the topologyTest permutation framework for assessing differential topology between conditions) — github.com/HectorRDB/condiments
- **slingshot** (Skeleton-based trajectory inference method that supplies the fitted common trajectory input to topologyTest)

## Examples

```
data("toy_dataset", package = "condiments"); sds <- slingshot(rd, df$cl); topologyTest(sds, conditions = df$conditions)
```

## Evaluation signals

- The p-value derives from condition-label permutations (default 100 repeats), so rerunning with a fixed seed should reproduce the permutation distribution and statistic table
- A significant call must coincide with a topological difference exceeding the minimum magnitude threshold — small but significant deviations should not be flagged
- The test only runs on skeleton-specified TI fits (e.g., slingshot sds object); attempting it on a non-skeleton method should fail
- Sanity check on toy_dataset: cells' condition-specific pseudotimes should map consistently onto the shared cluster-based lineage structure before permutation

## Limitations

- No independent benchmarking of the workflow has been performed; the authors call for it
- Dependence of pseudotime/weight estimates on the shared trajectory fit (double-dipping) is ignored rather than resolved
- p-values are not given a probabilistic interpretation; they carry only numerical-summary status

## Evidence

- [intro] In Step 1, condiments considers the trajectory inference question, assessing whether the dynamic process is fundamentally different between conditions, which we call differential topology.: "In Step 1, condiments considers the trajectory inference question, assessing whether the dynamic process is fundamentally different between conditions, which we call differential topology."
- [methods] The __topologyTest__ assess the quality of the common trajectory inference: "The __topologyTest__ assess the quality of the common trajectory inference"
- [readme] Start with the vignette [online](https://hectorrdb.github.io/condiments/articles/condiments.html) or explore on of our case studies in the paper: [evidence span withheld — anchor status: external_doc]
