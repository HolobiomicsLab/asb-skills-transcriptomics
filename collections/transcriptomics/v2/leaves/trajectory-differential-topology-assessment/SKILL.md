---
name: trajectory-differential-topology-assessment
description: Use when single-cell data from multiple conditions has been embedded
  in low-dimensional space and clustered, and you must decide whether trajectories
  can be jointly inferred or are differentially organized across conditions.
when_to_use_negative:
- When the trajectory inference method does not allow specification of an underlying
  skeleton structure, since a mapping between trajectories must be maintained across
  permutations
- When the question is about differences along or between lineages (differential progression
  or differential fate selection) rather than whether the trajectory structure itself
  differs — use progressionTest/fateSelectionTest instead
- When you only need gene-level differential expression along lineages — use tradeSeq
  fitGAM and conditionTest (Step 3)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: condiments
  role: Provides the topologyTest function and toy_dataset; implements the permutation-based
    differential topology assessment
  repo: github.com/HectorRDB/condiments
- name: slingshot
  role: Infers the common skeleton trajectory on the low-dimensional embedding using
    cluster labels, supplying the fitted object required by topologyTest
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:bdfa90004f2deb5d908962b8407585a9ab0322e793ca85fc742955b8d285e194
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - condiments
  - slingshot
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# trajectory-differential-topology-assessment

## Summary

Step 1 of the condiments workflow: use the topologyTest to decide whether a single common trajectory can be fitted to all cells or whether trajectory topology differs fundamentally between conditions, by permuting condition labels and comparing pseudotime distributions of condition-specific trajectories.

## When to use

Use when single-cell data from multiple conditions has been embedded in low-dimensional space and clustered, and you must decide whether trajectories can be jointly inferred or are differentially organized across conditions.

## When NOT to use

- When the trajectory inference method does not allow specification of an underlying skeleton structure, since a mapping between trajectories must be maintained across permutations
- When the question is about differences along or between lineages (differential progression or differential fate selection) rather than whether the trajectory structure itself differs — use progressionTest/fateSelectionTest instead
- When you only need gene-level differential expression along lineages — use tradeSeq fitGAM and conditionTest (Step 3)

## Inputs

- Low-dimensional cell embedding coordinates (e.g., Dim1, Dim2 from reducedDim or PCA/UMAP output)
- Per-cell condition assignments (treatment/control, disease/normal, etc.)
- Per-cell cluster labels used for skeleton-based trajectory inference
- A fitted trajectory object from a skeleton-based method (e.g., slingshot SlingshotDataSet, TSCAN)

## Outputs

- Test statistics for the topologyTest
- Permutation-based p-value table assessing rejection of the common-trajectory null (e.g., topology_test_results.csv)
- Decision: common trajectory suffices vs. differential topology between conditions

## How to apply

Load the reduced-dimension coordinates (e.g., Dim1/Dim2 from the condiments toy_dataset), condition assignments, and cluster labels. First infer a common trajectory with slingshot on the 2D embedding using the cluster labels, since the topologyTest requires a trajectory inference method that permits specification of an underlying skeleton structure (e.g., slingshot, TSCAN) so that a mapping between trajectories is maintained across permutations. Then run the condiments topologyTest on the fitted slingshot object with the condition vector, permuting condition labels (default 100 repeats) to obtain an empirical, permutation-based p-value comparing the actual distribution of pseudotimes for condition-specific trajectories against the permuted null. The test includes a minimum magnitude requirement so that only biologically relevant topological differences are called significant. If the null of identical trajectories across conditions cannot be rejected, proceed with a single common trajectory in later condiments steps; if it is rejected, treat the trajectories as having differential topology. Write the test statistics and permutation-based p-value table (e.g., to topology_test_results.csv) for downstream reporting.

## Related tools

- **condiments** (Provides the topologyTest function and toy_dataset; implements the permutation-based differential topology assessment) — github.com/HectorRDB/condiments
- **slingshot** (Infers the common skeleton trajectory on the low-dimensional embedding using cluster labels, supplying the fitted object required by topologyTest)

## Examples

```
library(condiments); data("toy_dataset", package = "condiments"); sds <- slingshot(rd, df$cl); topologyTest(sds, conditions = df$conditions)
```

## Evaluation signals

- topologyTest returns a table of test statistics together with a permutation-based p-value (default 100 repeats of condition-label permutation)
- A non-significant result implies a single common trajectory can be fitted across all conditions; a significant result implies differential topology — check the decision is consistent with the p-value threshold chosen
- Only biologically relevant topological differences should be flagged, due to the test's minimum magnitude requirement — near-null p-values with trivially small effect sizes warrant scrutiny
- The slingshot fit must be built on the cluster labels and reduced-dimension coordinates of the combined multi-condition dataset before running topologyTest
- The workflow's expected intermediate output (e.g., topology_test_results.csv) should contain the statistics and p-value per test, matching the condiments vignette structure

## Limitations

- Only works with trajectory inference methods that allow specification of an underlying skeleton structure (e.g., slingshot, TSCAN), because a mapping between trajectories must be maintained across condition-label permutations
- A minimum magnitude requirement is imposed so only biologically relevant topological differences are called significant; smaller real differences may go undetected
- No independent benchmarking of the workflow has been performed (authors call for it), and real-world benchmarking case studies are still lacking
- Dependence of pseudotime/weight estimates on the trajectory fit (double-dipping) is ignored rather than resolved

## Evidence

- [methods] The topologyTest assess the quality of the common trajectory inference: "The __topologyTest__ assess the quality of the common trajectory inference"
- [intro] In Step 1, condiments considers the trajectory inference question, assessing whether the dynamic process is fundamentally different between conditions, which we call differential topology.: "In Step 1, condiments considers the trajectory inference question, assessing whether the dynamic process is fundamentally different between conditions, which we call differential topology."
- [discussion] by first studying overall between-condition differences in trajectories with the topologyTest: "by first studying overall between-condition differences in trajectories with the topologyTest"
- [methods] data("toy_dataset", package = "condiments"): "data("toy_dataset", package = "condiments")"
- [readme] You can install the `condiments` from [bioconductor](https://bioconductor.org/packages/release/bioc/html/condiments.html): [evidence span withheld — anchor status: external_doc]
