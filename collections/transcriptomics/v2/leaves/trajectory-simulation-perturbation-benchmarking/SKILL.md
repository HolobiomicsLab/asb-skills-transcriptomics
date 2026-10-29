---
name: trajectory-simulation-perturbation-benchmarking
description: Use when validating a trajectory-level differential test (e.g. progressionTest)
  against realistic trajectory-inference errors, or when reporting how much pseudotime
  noise and lineage misassignment a method tolerates.
when_to_use_negative:
- The question is whether trajectory topology differs between conditions — use topologyTest,
  not perturbation-robustness benchmarking of progressionTest
- The input is real single-cell data without ground-truth pseudotimes; swapping lineage
  assignments would not be verifiable against a known reference
- Gene-level differential expression is the goal — conditionTest along lineages is
  the appropriate skill instead
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: condiments
  role: Provides progressionTest (custom KS test, global=TRUE, lineages=TRUE) whose
    robustness is benchmarked
  repo: github.com/HectorRDB/condiments
- name: dyngen
  role: Generates the ground-truth trajectory simulation with known pseudotimes and
    lineage assignments (T1, m=0.85)
- name: slingshot
  role: Trajectory inference framework providing lineage/pseudotime structure perturbed
    in the benchmark
- name: ggplot2
  role: Plots median-p-value curves against noise and misassignment levels
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:9a49d63e6f6637f48a4d66ee96ac8c968813e9f496e2e949233444285b0754c8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - condiments
  - dyngen
  - ggplot2
  - slingshot
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# trajectory-simulation-perturbation-benchmarking

## Summary

A simulation-based benchmarking skill that perturbs a ground-truth dyngen trajectory's pseudotimes and lineage assignments to stress-test the robustness of condiments::progressionTest, showing the test still rejects the null at the 5% level under substantial trajectory-inference error.

## When to use

Use when validating a trajectory-level differential test (e.g. progressionTest) against realistic trajectory-inference errors, or when reporting how much pseudotime noise and lineage misassignment a method tolerates.

## When NOT to use

- The question is whether trajectory topology differs between conditions — use topologyTest, not perturbation-robustness benchmarking of progressionTest
- The input is real single-cell data without ground-truth pseudotimes; swapping lineage assignments would not be verifiable against a known reference
- Gene-level differential expression is the goal — conditionTest along lineages is the appropriate skill instead

## Inputs

- Ground-truth dyngen simulation with known pseudotimes and lineage assignments (e.g. T1 topology, noise m=0.85)
- Pseudotime noise levels (Gaussian standard deviation in [0, 4])
- Lineage misassignment proportions p in {0, 0.1, 0.2, 0.3, 0.4, 0.5}
- Condition labels per cell for the differential progression test

## Outputs

- Median p-values across ~100 replicates per noise/misassignment combination
- Median-p-value curves versus pseudotime noise and misassignment proportion (e.g. median_pvalue_curves.png via ggplot2)
- Robustness statement relative to the 5% nominal significance level

## How to apply

Generate the dyngen reference simulation (e.g. T1 with noise parameter m=0.85) as ground truth with known pseudotimes and lineage assignments. For each noise level, add Gaussian noise to the true pseudotimes with standard deviation swept over [0, 4] (up to 200% of pseudotime scale), and for each misassignment proportion p in {0, 0.1, 0.2, 0.3, 0.4, 0.5} randomly swap lineage assignments for that fraction of cells. Run condiments::progressionTest on each perturbed replicate using a custom KS test with global=TRUE and lineages=TRUE, across ~100 replicates per noise/misassignment combination. Compute the median p-value per setting, then plot median-p-value curves versus pseudotime noise level and misassignment proportion with ggplot2. The result validates the paper's finding that progressionTest remains significant up to ~60% misassigned cells and 200% pseudotime noise, so moderate TI errors have small impact on the test outcome.

## Related tools

- **condiments** (Provides progressionTest (custom KS test, global=TRUE, lineages=TRUE) whose robustness is benchmarked) — github.com/HectorRDB/condiments
- **dyngen** (Generates the ground-truth trajectory simulation with known pseudotimes and lineage assignments (T1, m=0.85))
- **slingshot** (Trajectory inference framework providing lineage/pseudotime structure perturbed in the benchmark)
- **ggplot2** (Plots median-p-value curves against noise and misassignment levels)

## Examples

```
Rscript -e 'progressionTest(perturbed_sds, conditions = cond, lineages = TRUE, global = TRUE)'
```

## Evaluation signals

- Median p-values across replicates stay below 0.05 (correct rejection of the no-differential-progression null) up to ~60% lineage misassignment
- Median p-values remain significant up to 200% pseudotime noise (Gaussian SD up to ~4 on the pseudotime scale)
- p=0 misassignment, 0 noise control replicates reproduce the unperturbed progressionTest result
- Each noise/misassignment setting is evaluated over 100 replicates before taking the median
- Reproduced curves qualitatively match the paper's Fig. S4 robustness behavior

## Limitations

- No independent benchmarking of the condiments workflow has been performed; the authors call for it
- Dependence of pseudotime/weight estimates (double-dipping) is ignored rather than resolved in the statistical treatment
- p-values are not given a probabilistic interpretation, only numerical-summary status, which limits how strongly simulation robustness can be interpreted

## Evidence

- [other] progressionTest robustness: [evidence span withheld — anchor status: not_found]
- [discussion] dyngen role: "Simulation frameworks such as dyngen24 are crucial."
- [methods] progressionTest method: "To test for differential progression, we use the __progressionTest__."
- [abstract] condiments scope: "we present condiments, a method for the inference and downstream interpretation of cell trajectories across multiple conditions"
- [readme] README install: [evidence span withheld — anchor status: external_doc]
