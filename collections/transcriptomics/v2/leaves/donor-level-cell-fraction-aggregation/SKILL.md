---
name: donor-level-cell-fraction-aggregation
description: Use when you have per-cell cell type/state labels plus per-donor metadata
  (donor id, sex, depot, lean/obese status) and need to test how tissue cellular composition
  differs between donor groups while controlling for covariates such as depot and
  sex.
when_to_use_negative:
- The goal is per-cell or within-donor differential expression rather than donor-level
  cellular composition changes
- No donor-level metadata (sex, depot, weight status) is available to serve as regression
  covariates, making the marginal-effect design unidentifiable
- Cell labels are raw JOINTLY cluster identities not yet transferred to the WATLAS
  43 cell types/states (e.g. clusters not filtered to named types with more than 10
  cells), so fractions would be defined over unstable cluster sets
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R lm (stats)
  role: Fits the per-cell-type ordinary linear regression of donor-level fraction
    on weight status, depot and sex and supplies coefficients with 95% confidence
    intervals
- name: JOINTLY / WATLAS
  role: Produces and hosts the cell type/state labels over the 46 WAT donors whose
    counts are aggregated to donor-level fractions
  repo: https://singlecell.broadinstitute.org/single_cell/study/SCP2289/an-integrated-single-cell-and-single-nucleus-rna-seq-white-adipose-tissue-atlas-watlas
- name: Seurat
  role: Framework holding the annotated single-cell object from which the cell-by-cell
    type table for aggregation is derived
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:5297a1625166b6a2ecaa5c757aebb56fbad4ca10de2ee7c572c81211b59820c3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - JOINTLY / WATLAS
  - R lm (stats)
  - Seurat
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# donor-level-cell-fraction-aggregation

## Summary

A composition-analysis method that aggregates single-cell cell type/state annotations (from WATLAS/SCP2289) into donor-level cell-type fractions for 46 donors and fits per-cell-type ordinary linear regressions on weight status (lean vs obese), depot and sex to estimate marginal compositional effect sizes with 95% confidence intervals.

## When to use

Use when you have per-cell cell type/state labels plus per-donor metadata (donor id, sex, depot, lean/obese status) and need to test how tissue cellular composition differs between donor groups while controlling for covariates such as depot and sex.

## When NOT to use

- The goal is per-cell or within-donor differential expression rather than donor-level cellular composition changes
- No donor-level metadata (sex, depot, weight status) is available to serve as regression covariates, making the marginal-effect design unidentifiable
- Cell labels are raw JOINTLY cluster identities not yet transferred to the WATLAS 43 cell types/states (e.g. clusters not filtered to named types with more than 10 cells), so fractions would be defined over unstable cluster sets

## Inputs

- WATLAS cell-by-cell type/state annotation table (task_007 output or deposited SCP2289 annotations)
- Per-donor metadata table for 46 donors (donor id, sex, depot, lean/obese weight status)

## Outputs

- Donor-level per-cell-type fractional abundance matrix (46 donors x cell types/states)
- fig5_composition_effects.csv with marginal effect sizes (regression coefficients) and 95% confidence intervals per cell type/state for obesity, depot and sex contrasts

## How to apply

Load the WATLAS cell-by-cell type/state annotation table (task_007 output or the deposited SCP2289 annotations) together with per-donor metadata (donor id, sex, depot, lean/obese weight status) for the 46 donors. Aggregate cell counts per donor and cell type/state and convert them to donor-level fractions per cell type, including the four adipocyte subpopulations described in WATLAS. For each cell type/state, fit an ordinary linear regression of the donor-level fraction on weight status, depot and sex as covariates using R's lm. Extract the marginal regression coefficient (effect size) and its 95% confidence interval for each contrast of interest (obesity, depot, sex). Assemble all effect sizes and confidence intervals into a single table written as fig5_composition_effects.csv; controlling for sex and depot, obesity is expected to show a trend towards decreased fractional abundance of smooth muscle cells and fibro-adipogenic progenitors (FAPs).

## Related tools

- **R lm (stats)** (Fits the per-cell-type ordinary linear regression of donor-level fraction on weight status, depot and sex and supplies coefficients with 95% confidence intervals)
- **JOINTLY / WATLAS** (Produces and hosts the cell type/state labels over the 46 WAT donors whose counts are aggregated to donor-level fractions) — https://singlecell.broadinstitute.org/single_cell/study/SCP2289/an-integrated-single-cell-and-single-nucleus-rna-seq-white-adipose-tissue-atlas-watlas
- **Seurat** (Framework holding the annotated single-cell object from which the cell-by-cell type table for aggregation is derived) — https://github.com/satijalab/seurat

## Examples

```
fit <- lm(fraction ~ weight_status + depot + sex, data = donor_fractions); data.frame(coef = coef(fit), confint(fit))
```

## Evaluation signals

- Sign and significance of the computed effect sizes must reproduce the reported compositional differences between lean and obese donors and between WAT depots (Fig. 5C and 5G)
- Obesity/depot-driven shifts among the four adipocyte subpopulations must be captured in the coefficient table
- Controlling for sex and depot, the obesity coefficient should show a trend towards a decrease in fractional smooth muscle cells and FAPs
- For each donor, per-cell-type fractions must sum to 1 and every donor must be represented in the aggregated table (46 donors)
- Each cell type/state contrast must carry both a regression coefficient and a 95% confidence interval in fig5_composition_effects.csv

## Limitations

- The regression is at the donor level with only 46 donors, so effects are reported as trends (e.g. smooth muscle cells and FAPs decrease with obesity) and power for small subpopulations is limited
- Result validity depends on prior batch integration/annotation quality; JOINTLY performance varies task-to-task and 'no method is always the best across all tasks', so labels feeding the aggregation should be verified for the specific integration task

## Evidence

- [methods] linear regression workflow on donor fractions: [evidence span withheld — anchor status: not_found]
- [methods] effect size extraction and output assembly: [evidence span withheld — anchor status: not_found]
- [methods] aggregate counts to donor-level fractions: [evidence span withheld — anchor status: not_found]
- [abstract] biological finding being reproduced: "we describe four adipocyte subpopulations and map compositional changes in obesity and between depots."
- [intro] composition differences motivation: "Analysis of WATLAS revealed compositional differences between lean and obese donors and between different white adipose tissue depots"
