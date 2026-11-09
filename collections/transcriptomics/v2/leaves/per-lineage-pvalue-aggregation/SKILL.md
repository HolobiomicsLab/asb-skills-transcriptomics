---
name: per-lineage-pvalue-aggregation
description: Use when a common trajectory has been inferred across multiple conditions
  (e.g. with slingshot) and you need to test which lineages show differential progression
  — per-lineage pseudotime distribution shifts between conditions.
when_to_use_negative:
- When the trajectory topology itself may differ between conditions — use topologyTest
  first to assess differential topology before progression testing
- When the question is differences between lineages rather than along lineages — use
  fateSelectionTest for differential fate selection instead
- When testing gene-level expression differences along lineages — use conditionTest
  (via tradeSeq fitGAM) rather than the progressionTest
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: condiments
  role: Provides progressionTest for non-parametric (Kolmogorov-Smirnov) testing of
    differential progression per lineage and globally
  repo: github.com/HectorRDB/condiments
- name: slingshot
  role: Infers the common trajectory across conditions and provides per-lineage pseudotime
    via slingPseudotime as test input
- name: ggplot2
  role: Visualizes per-lineage pseudotime density distributions colored by condition
    as a pre-test sanity check
- name: knitr
  role: Renders the global and per-lineage p-value table with kable
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-44823-0
    title: Trajectory inference across multiple conditions with condiments
  source_content_hash: sha256:f9322011da2ae819aadaca7bec486256435d05b3f11da76d224a83dcb1cd7a01
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# per-lineage-pvalue-aggregation

## Summary

Apply the condiments progressionTest to slingshot-inferred pseudotime to obtain a p-value table with both a global between-condition statistic and per-lineage statistics, testing the null hypothesis of equal pseudotime distributions between conditions using non-parametric tests such as the Kolmogorov-Smirnov test.

## When to use

Use when a common trajectory has been inferred across multiple conditions (e.g. with slingshot) and you need to test which lineages show differential progression — per-lineage pseudotime distribution shifts between conditions.

## When NOT to use

- When the trajectory topology itself may differ between conditions — use topologyTest first to assess differential topology before progression testing
- When the question is differences between lineages rather than along lineages — use fateSelectionTest for differential fate selection instead
- When testing gene-level expression differences along lineages — use conditionTest (via tradeSeq fitGAM) rather than the progressionTest

## Inputs

- Single-cell dataset with condition labels (e.g. condiments toy_dataset: Dim1/Dim2 coordinates, cluster labels cl, condition assignments)
- Slingshot object with inferred common trajectory and per-lineage pseudotime (slingPseudotime values)

## Outputs

- P-value table with global and per-lineage statistics and p-values (e.g. progression_test_pvalues.csv)
- Rendered table via knitr::kable

## How to apply

Load the toy_dataset (or your own single-cell dataset) from the condiments package and infer a common trajectory with slingshot on the reduced-dimension matrix (e.g. Dim1, Dim2) using cluster labels, then extract per-lineage pseudotime values with slingPseudotime. Optionally first visualize pseudotime density distributions per lineage colored by condition with ggplot2 to check for condition-specific shifts before testing. Run progressionTest on the slingshot object with conditions = df$conditions, global = TRUE and lineages = TRUE, using the default custom Kolmogorov-Smirnov test, which assesses the null of equal pseudotime distributions between conditions non-parametrically. Render the resulting p-value table (global and per-lineage statistics and p-values), e.g. with knitr::kable, and export it (e.g. progression_test_pvalues.csv). Interpret per-lineage significance against the global result: in the toy dataset the global test and the lineage-2 test yield small p-values (differential progression on the bottom lineage) while lineage 1 shows no significant difference, indicating the bottom lineage drives the global difference.

## Related tools

- **condiments** (Provides progressionTest for non-parametric (Kolmogorov-Smirnov) testing of differential progression per lineage and globally) — github.com/HectorRDB/condiments
- **slingshot** (Infers the common trajectory across conditions and provides per-lineage pseudotime via slingPseudotime as test input)
- **ggplot2** (Visualizes per-lineage pseudotime density distributions colored by condition as a pre-test sanity check)
- **knitr** (Renders the global and per-lineage p-value table with kable)

## Examples

```
progressionTest(sds, conditions = df$conditions, global = TRUE, lineages = TRUE)
```

## Evaluation signals

- The output table contains both a global row and one row per lineage (global = TRUE, lineages = TRUE were set)
- In the toy dataset, the global test and lineage-2 test yield small p-values while lineage 1 is non-significant, matching the expectation that the bottom lineage drives the global difference
- The default test is the custom Kolmogorov-Smirnov test — confirm non-parametric two-or-more-sample comparison of pseudotime distributions was used
- Pseudotime densities per lineage colored by condition were inspected with ggplot2 before interpreting p-values
- The resulting p-value table is reproducibly saved (e.g. progression_test_pvalues.csv) and matches the knitr::kable rendering

## Limitations

- Dependence of pseudotime/weight estimates (double-dipping) is ignored rather than resolved, as in other differential abundance methods
- P-values are not given a probabilistic interpretation; only numerical-summary status
- Batch-condition confounding can be inherent and unresolvable in some settings
- No independent benchmarking of the workflow has been performed; the authors call for it

## Evidence

- [methods] progressionTest assesses the null hypothesis of equal pseudotime distributions between conditions using non-parametric tests: "To test for differential progression, we use the __progressionTest__."
- [intro] Step 2 tests for global differences between conditions along lineages, termed differential progression: "In Step 2, it tests for global differences between conditions, both along lineages – differential progression – and between lineages – differential fate selection."
- [readme] condiments is installed from Bioconductor via BiocManager and used through its vignette and case studies: [evidence span withheld — anchor status: external_doc]
- [discussion] The discussion distinguishes trajectory-level tests (progressionTest, fateSelectionTest) from gene-level tests: "then differences in abundance at the trajectory level with the progressionTest and fateSelectionTest"
