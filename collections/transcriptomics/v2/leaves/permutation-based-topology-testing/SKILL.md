---
name: permutation-based-topology-testing
description: Use when single-cell data from multiple conditions must be fitted to
  trajectories and you must decide, as Step 1 of the condiments workflow, whether
  trajectories differ fundamentally between conditions (differential topology).
when_to_use_negative:
- The trajectory inference method does not allow specification of an underlying skeleton
  structure — the test requires a fixed mapping between trajectories to be maintained
  across label permutations
- The question is about differential progression or differential fate selection along
  a common trajectory (Step 2, progressionTest/fateSelectionTest) rather than whether
  the topology itself differs
- The system is steady-state with no dynamic process, where per-cluster differential
  abundance methods are the reasonable approach
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: condiments
  role: Provides the topologyTest function that performs the label-permutation test
    for differential topology in Step 1 of the workflow
  repo: github.com/HectorRDB/condiments
- name: slingshot
  role: Infers the common skeleton-based trajectory on the low-dimensional embedding
    that the topologyTest then evaluates
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:698350469253986d97a6a35128f28e635a30bfd88f77f68c3826e408038a660a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - condiments
  - slingshot
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# permutation-based-topology-testing

## Summary

A hypothesis-testing skill from the condiments workflow that decides whether a single common trajectory can be fitted across conditions or whether trajectory topology differs fundamentally between conditions. It works by permuting condition labels and comparing the resulting distribution of pseudotimes to the observed one.

## When to use

Use when single-cell data from multiple conditions must be fitted to trajectories and you must decide, as Step 1 of the condiments workflow, whether trajectories differ fundamentally between conditions (differential topology).

## When NOT to use

- The trajectory inference method does not allow specification of an underlying skeleton structure — the test requires a fixed mapping between trajectories to be maintained across label permutations
- The question is about differential progression or differential fate selection along a common trajectory (Step 2, progressionTest/fateSelectionTest) rather than whether the topology itself differs
- The system is steady-state with no dynamic process, where per-cluster differential abundance methods are the reasonable approach

## Inputs

- Low-dimensional embedding of cells (e.g., Dim1/Dim2 reduced-dimension coordinates)
- Per-cell condition assignments (condition vector)
- Per-cell cluster labels for trajectory inference
- A fitted trajectory object with an underlying skeleton structure (e.g., a slingshot object), or the toy_dataset bundled with condiments

## Outputs

- Test statistics from the topologyTest
- Permutation-based p-value table (e.g., written to topology_test_results.csv)
- Decision on whether a single common trajectory suffices or topology is differential between conditions

## How to apply

First infer a common trajectory with a TI method that allows specification of an underlying skeleton structure, such as slingshot on the reduced-dimension coordinates using cluster labels (e.g., on the toy_dataset's Dim1/Dim2 embedding: sds <- slingshot(rd, df$cl)). Then run condiments' topologyTest on the fitted slingshot object with the condition vector, permuting condition labels (default 100 repeats) to generate a null distribution. The test compares the actual distribution of pseudotimes for condition-specific trajectories to their distribution under permutation, and applies a minimum magnitude requirement so that only biologically relevant topological differences are called significant. Write the resulting test statistics and permutation-based p-value table to an output such as topology_test_results.csv; a small p-value indicates that a common trajectory is insufficient and condition-specific trajectories are fundamentally different.

## Related tools

- **condiments** (Provides the topologyTest function that performs the label-permutation test for differential topology in Step 1 of the workflow) — github.com/HectorRDB/condiments
- **slingshot** (Infers the common skeleton-based trajectory on the low-dimensional embedding that the topologyTest then evaluates)

## Examples

```
library(condiments); data("toy_dataset", package = "condiments"); sds <- slingshot(rd, df$cl); topo <- topologyTest(sds, conditions = df$conditions)
```

## Evaluation signals

- The output table contains both test statistics and a permutation-based p-value for the topology comparison
- The default permutation scheme uses 100 repeats of condition-label permutation; deviating from this should be documented
- Significant topological differences should only be called when the magnitude requirement is met, so tiny-but-geometrically-trivial differences must not dominate the p-values
- The trajectory object passed to topologyTest must come from a method that specifies an underlying skeleton (e.g., slingshot, TSCAN) so mappings persist across permutations

## Limitations

- The test only works with trajectory inference methods that allow specification of an underlying skeleton structure, because a mapping between trajectories must be maintained across permutations
- A minimum magnitude requirement is imposed so only biologically relevant topological differences are called significant
- No independent benchmarking of the workflow has been performed; the authors call for it, and real-world benchmarking case studies are still lacking
- Dependence of pseudotime/weight estimates (double-dipping) is ignored rather than resolved, and p-values are not given a strong probabilistic interpretation

## Evidence

- [methods] topologyTest assesses differential topology: "The __topologyTest__ assess the quality of the common trajectory inference"
- [intro] differential topology is Step 1: "In Step 1, condiments considers the trajectory inference question, assessing whether the dynamic process is fundamentally different between conditions, which we call differential topology."
- [methods] toy dataset is the packaged example input: "data("toy_dataset", package = "condiments")"
- [readme] condiments is available from Bioconductor: [evidence span withheld — anchor status: external_doc]
- [discussion] topologyTest precedes trajectory-level tests: "by first studying overall between-condition differences in trajectories with the topologyTest"
