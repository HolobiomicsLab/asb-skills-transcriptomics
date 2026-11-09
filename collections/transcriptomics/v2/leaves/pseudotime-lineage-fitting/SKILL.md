---
name: pseudotime-lineage-fitting
description: Use when single-cell data from multiple conditions has been embedded
  (e.g., Dim1/Dim2 reduced-dimension coordinates with cluster and condition labels)
  and you must decide whether to fit one common trajectory or test for differential
  topology and differential progression between conditions.
when_to_use_negative:
- Cells are in steady state with no dynamic/trajectory structure, where cluster-based
  differential abundance methods are more appropriate than trajectory fitting.
- The trajectory inference method does not allow specification of an underlying skeleton
  structure, since topologyTest requires a mapping between trajectories across condition-label
  permutations (only e.g. slingshot, TSCAN work).
- The goal is gene-level differential expression along lineages, which is handled
  downstream by fitGAM and conditionTest rather than trajectory-level topology/progression
  tests.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: condiments
  role: Provides topologyTest (differential topology via condition-label permutation)
    and progressionTest (Kolmogorov-Smirnov comparison of pseudotime distributions
    per lineage and globally), plus the toy_dataset used for demonstration
  repo: github.com/HectorRDB/condiments
- name: slingshot
  role: Infers the common trajectory on the 2D embedding from cluster labels and supplies
    per-lineage pseudotimes (slingPseudotime) consumed by the condiments tests
- name: ggplot2
  role: Visualizes pseudotime density distributions per lineage colored by condition
    as a sanity check before running progressionTest
- name: knitr
  role: Renders the progressionTest p-value table with knitr::kable
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:713ad5bc0060add4fed89bed1e9f317d2e4dcd314d24ed5c29a041ce27deedb0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - condiments
  - dplyr
  - ggplot2
  - knitr
  - slingshot
  - tidyr
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# pseudotime-lineage-fitting

## Summary

Fit a common trajectory on a low-dimensional embedding with slingshot using cluster labels, extract per-lineage pseudotimes, and use condiments' topologyTest and progressionTest to decide whether one common trajectory suffices or whether conditions differ in topology or progression.

## When to use

Use when single-cell data from multiple conditions has been embedded (e.g., Dim1/Dim2 reduced-dimension coordinates with cluster and condition labels) and you must decide whether to fit one common trajectory or test for differential topology and differential progression between conditions.

## When NOT to use

- Cells are in steady state with no dynamic/trajectory structure, where cluster-based differential abundance methods are more appropriate than trajectory fitting.
- The trajectory inference method does not allow specification of an underlying skeleton structure, since topologyTest requires a mapping between trajectories across condition-label permutations (only e.g. slingshot, TSCAN work).
- The goal is gene-level differential expression along lineages, which is handled downstream by fitGAM and conditionTest rather than trajectory-level topology/progression tests.

## Inputs

- Reduced-dimension coordinate matrix (e.g., Dim1, Dim2 columns from the condiments toy_dataset)
- Per-cell condition labels (vector)
- Per-cell cluster labels (vector)
- Slingshot slingPseudotime lineage assignments (fitted common trajectory object)

## Outputs

- topology_test_results.csv: topologyTest statistics and permutation-based p-value table
- progression_test_pvalues.csv: global and per-lineage progressionTest statistics and p-values
- Optional ggplot2 density plots of pseudotime per lineage colored by condition

## How to apply

Load the multi-condition dataset (e.g., the toy_dataset bundled with condiments) and extract the reduced-dimension coordinates, condition assignments, and cluster labels. Infer a common trajectory with slingshot on the 2D embedding using the cluster labels (sds <- slingshot(rd, df$cl)), since the condiments tests require trajectory inference methods that allow specification of an underlying skeleton structure (e.g., slingshot, TSCAN) to maintain a mapping between trajectories across permutations. Run topologyTest on the fitted slingshot object with the condition vector, permuting condition labels (default 100 repeats), to test whether the null hypothesis of identical trajectories across all conditions can be rejected; the test includes a minimum magnitude requirement so only biologically relevant topological differences are called significant. Extract per-lineage pseudotimes with slingPseudotime, optionally visualize pseudotime density distributions per lineage colored by condition with ggplot2, then run progressionTest with conditions = df$conditions, global = TRUE and lineages = TRUE using the default custom Kolmogorov-Smirnov test to compare pseudotime distributions between conditions. Write the resulting test-statistic and p-value tables (topology and progression) to CSV outputs.

## Related tools

- **condiments** (Provides topologyTest (differential topology via condition-label permutation) and progressionTest (Kolmogorov-Smirnov comparison of pseudotime distributions per lineage and globally), plus the toy_dataset used for demonstration) — github.com/HectorRDB/condiments
- **slingshot** (Infers the common trajectory on the 2D embedding from cluster labels and supplies per-lineage pseudotimes (slingPseudotime) consumed by the condiments tests)
- **ggplot2** (Visualizes pseudotime density distributions per lineage colored by condition as a sanity check before running progressionTest)
- **knitr** (Renders the progressionTest p-value table with knitr::kable)

## Examples

```
library(condiments); sds <- slingshot(rd, df$cl); topologyTest(sds, df$conditions); progressionTest(sds, conditions = df$conditions, global = TRUE, lineages = TRUE)
```

## Evaluation signals

- topologyTest returns a permutation-based p-value table (default 100 repeats); identical trajectories across conditions should fail to reject the null of a common trajectory.
- On the toy_dataset validation, the progressionTest global test and the lineage-2 test yield small p-values (differential progression on the bottom lineage) while lineage 1 shows no significant difference, matching the expectation that 'the bottom one' drives the global difference.
- Pseudotime density plots per lineage colored by condition should visually show condition-specific shifts only on lineages flagged significant by progressionTest.
- Output CSVs contain both test statistics and p-values (topology_test_results.csv, progression_test_pvalues.csv) with global and per-lineage columns.
- Only biologically relevant topological differences should be called significant, per the topologyTest's minimum magnitude requirement.

## Limitations

- topologyTest only works with trajectory inference methods that allow specification of an underlying skeleton structure (e.g., slingshot, TSCAN) because a mapping between trajectories must be maintained across permutations.
- The topologyTest includes a minimum magnitude requirement so that only biologically relevant topological differences are called significant.
- Dependence of pseudotime/weight estimates (double-dipping) is ignored rather than resolved, as in other differential abundance methods.
- P-values are not given a strong probabilistic interpretation; they are treated as numerical summaries, and batch-condition confounding can be inherent and unresolvable in some settings.
- No independent benchmarking of the workflow has been performed, and real-world benchmarking case studies are still lacking.

## Evidence

- [intro] Step 1: assess whether the dynamic process (trajectory topology) is fundamentally different between conditions (differential topology): "In Step 1, condiments considers the trajectory inference question, assessing whether the dynamic process is fundamentally different between conditions, which we call differential topology."
- [intro] Step 2: differential progression along lineages: "In Step 2, it tests for global differences between conditions, both along lineages – differential progression – and between lineages – differential fate selection."
- [methods] topologyTest assesses the quality of the common trajectory inference: "The __topologyTest__ assess the quality of the common trajectory inference"
- [methods] progressionTest used for differential progression: "To test for differential progression, we use the __progressionTest__."
- [methods] toy_dataset loading from condiments package: "data("toy_dataset", package = "condiments")"
- [discussion] trajectory structure leveraged for detecting between-condition changes: "taking into account the dynamic nature of biological systems via the trajectory representation enables condiments to better detect true changes"
