---
name: kolmogorov-smirnov-pseudotime-distribution-comparison
description: Use when a common trajectory (e.g. slingshot lineages with slingPseudotime)
  has been inferred across multiple experimental conditions and you need to test whether
  pseudotime distributions differ between conditions per lineage or globally.
when_to_use_negative:
- When the trajectory topology itself differs between conditions — test differential
  topology first with topologyTest, since progression testing assumes a common trajectory
- When you want to test whether cells commit to different lineages between conditions
  (differential fate selection) — use fateSelectionTest instead
- When the goal is gene-level differential expression along lineages — use conditionTest/fitGAM
  on count matrices rather than pseudotime distribution testing
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: condiments
  role: Provides the progressionTest function that runs the Kolmogorov-Smirnov (or
    classifier) test on pseudotime distributions and returns global/per-lineage p-values
  repo: https://github.com/HectorRDB/condiments
- name: slingshot
  role: Infers the common trajectory and per-lineage pseudotime values that progressionTest
    compares between conditions
- name: ggplot2
  role: Used to plot per-lineage pseudotime density distributions colored by condition
    as a pre-test diagnostic
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:17a26605cf7efa066e34e3c9befd48c32168f9e57d040ac0c562b862db0a8bee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HectorRDB/condiments
  tools:
  - condiments
  - dplyr
  - ggplot2
  - slingshot
  - tidyr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# kolmogorov-smirnov-pseudotime-distribution-comparison

## Summary

Statistical test for differential progression in the condiments framework: compares per-lineage pseudotime distributions between conditions using the non-parametric Kolmogorov-Smirnov (or classifier) test to detect whether cells from different conditions occupy different positions along a shared trajectory.

## When to use

Use when a common trajectory (e.g. slingshot lineages with slingPseudotime) has been inferred across multiple experimental conditions and you need to test whether pseudotime distributions differ between conditions per lineage or globally.

## When NOT to use

- When the trajectory topology itself differs between conditions — test differential topology first with topologyTest, since progression testing assumes a common trajectory
- When you want to test whether cells commit to different lineages between conditions (differential fate selection) — use fateSelectionTest instead
- When the goal is gene-level differential expression along lineages — use conditionTest/fitGAM on count matrices rather than pseudotime distribution testing

## Inputs

- Slingshot (or similar trajectory) object with per-lineage pseudotime values for all cells
- Condition labels vector for each cell (e.g. df$conditions)
- Reduced-dimension embedding and cluster labels used for trajectory inference (e.g. toy_dataset from the condiments package)

## Outputs

- Table of global and per-lineage test statistics and p-values for differential progression (e.g. progression_test_pvalues.csv)
- Optional density plots of pseudotime per lineage colored by condition for sanity checking

## How to apply

First infer a common trajectory integrating all conditions — e.g. run slingshot on the reduced-dimension embedding (Dim1, Dim2) using cluster labels, then extract per-lineage pseudotime with slingPseudotime. Visualize pseudotime density distributions per lineage colored by condition (ggplot2) to spot condition-specific shifts before testing. Then call progressionTest from condiments with conditions = df$conditions, global = TRUE and lineages = TRUE; by default it applies the custom Kolmogorov-Smirnov test to the null hypothesis of equal pseudotime distributions between conditions, with the classifier test as an alternative for two or more distributions. The output is a p-value table of global and per-lineage statistics (e.g. exportable via knitr::kable to progression_test_pvalues.csv). A small p-value for a lineage indicates differential progression on that lineage, which may drive a significant global test.

## Related tools

- **condiments** (Provides the progressionTest function that runs the Kolmogorov-Smirnov (or classifier) test on pseudotime distributions and returns global/per-lineage p-values) — https://github.com/HectorRDB/condiments
- **slingshot** (Infers the common trajectory and per-lineage pseudotime values that progressionTest compares between conditions)
- **ggplot2** (Used to plot per-lineage pseudotime density distributions colored by condition as a pre-test diagnostic)

## Examples

```
progressionTest(sce, conditions = df$conditions, global = TRUE, lineages = TRUE)
```

## Evaluation signals

- In the condiments toy_dataset example, the global test and the lineage-2 test yield small p-values (differential progression on the bottom lineage) while lineage 1 is non-significant
- The p-value table contains both global = TRUE and lineages = TRUE entries (one row per lineage plus a global row) with statistic and p-value columns
- Pre-test density plots of slingPseudotime per lineage show the condition-specific shifts that the significant lineages' p-values reflect
- Per-lineage conclusions are consistent with the default custom Kolmogorov-Smirnov test being applied (two or more condition distributions compared non-parametrically)

## Limitations

- p-values are not given a strong probabilistic interpretation; the authors treat them as numerical summaries rather than attaching strong probabilistic interpretations
- Dependence of pseudotime/weight estimates on the trajectory inference (double-dipping) is ignored rather than resolved
- The workflow has not been independently benchmarked, and real-world benchmarking case studies are still lacking
- Batch-condition confounding can be inherent and unresolvable in some settings

## Evidence

- [other] The progressionTest assesses the null hypothesis of equal pseudotime distributions between conditions using non-parametric tests, such as the Kolmogorov-Smirnov test or the classifier test: [evidence span withheld — anchor status: not_found]
- [methods] To test for differential progression, we use the __progressionTest__.: "To test for differential progression, we use the __progressionTest__."
- [methods] data("toy_dataset", package = "condiments"): "data("toy_dataset", package = "condiments")"
- [readme] Start with the vignette [online](https://hectorrdb.github.io/condiments/articles/condiments.html): [evidence span withheld — anchor status: external_doc]
