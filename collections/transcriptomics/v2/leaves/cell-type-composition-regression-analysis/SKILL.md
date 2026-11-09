---
name: cell-type-composition-regression-analysis
description: Use when a single-cell/nucleus atlas has cell-type or cell-state annotations
  plus per-donor covariates, and the goal is to test whether cell-type fractions differ
  across a condition (e.g. obesity, depot) while controlling for covariates.
when_to_use_negative:
- Input is a raw count matrix without cell-type/state labels — label the cells first
  (e.g. with JOINTLY joint clustering or WATLAS transfer learning) before computing
  fractions
- The question is about gene expression programs within cell states rather than the
  fractional composition of cell types across donors
- Cells would be modeled as independent observations instead of being aggregated to
  donor-level fractions (pseudoreplication)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: JOINTLY
  role: Used (with scANVI integration) to process, label and build the WATLAS atlas
    whose cell-by-cell type/state annotations supply the cell counts aggregated into
    donor-level fractions for regression
  repo: https://github.com/madsen-lab/rJOINTLY
- name: Seurat
  role: Framework holding the single-cell objects (Seurat/SingleCellExperiment) from
    which per-donor cell-type counts are tabulated
  repo: https://github.com/satijalab/seurat
- name: R lm
  role: Fits the ordinary linear regression of donor-level fraction on weight status,
    depot and sex and yields coefficients and 95% CIs
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:2f1ca7c4dfb9a4e7c4f8c0e5fa3ef213814175e9a21e7276e0e5a663888696ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madsen-lab/rJOINTLY
  tools:
  - JOINTLY
  - R lm
  - Seurat
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cell-type-composition-regression-analysis

## Summary

Donor-level ordinary linear regression of cell-type/state fractions from the WATLAS white adipose tissue atlas on weight status (lean vs obese), depot and sex, to quantify compositional shifts (Fig. 5C/5G style contrasts).

## When to use

Use when a single-cell/nucleus atlas has cell-type or cell-state annotations plus per-donor covariates, and the goal is to test whether cell-type fractions differ across a condition (e.g. obesity, depot) while controlling for covariates.

## When NOT to use

- Input is a raw count matrix without cell-type/state labels — label the cells first (e.g. with JOINTLY joint clustering or WATLAS transfer learning) before computing fractions
- The question is about gene expression programs within cell states rather than the fractional composition of cell types across donors
- Cells would be modeled as independent observations instead of being aggregated to donor-level fractions (pseudoreplication)

## Inputs

- WATLAS cell-by-cell type/state annotation table (task_007 output or deposited SCP2289 annotations)
- Per-donor metadata for the 46 donors: donor id, sex, depot, lean/obese weight status

## Outputs

- fig5_composition_effects.csv: effect sizes (regression coefficients) and 95% confidence intervals for obesity, depot and sex contrasts per cell type/state

## How to apply

Load the WATLAS cell-by-cell type/state annotation table (task_007 output or the deposited SCP2289 annotations) together with per-donor metadata (donor id, sex, depot, lean/obese weight status) for the 46 donors. Aggregate cell counts per donor and cell type/state and convert to donor-level fractions per cell type, so that cells are never treated as independent observations. For each cell type/state, fit an ordinary linear regression in R (lm) of the donor-level fraction on weight status, depot and sex as covariates. Extract the marginal effect size (regression coefficient) and its 95% confidence interval for each contrast of interest (obesity, depot, sex). Assemble all effect sizes and confidence intervals into a single table and write it as fig5_composition_effects.csv; validate that the signs reproduce the reported lean-vs-obese and depot-driven compositional differences, including shifts among the four adipocyte subpopulations.

## Related tools

- **JOINTLY** (Used (with scANVI integration) to process, label and build the WATLAS atlas whose cell-by-cell type/state annotations supply the cell counts aggregated into donor-level fractions for regression) — https://github.com/madsen-lab/rJOINTLY
- **Seurat** (Framework holding the single-cell objects (Seurat/SingleCellExperiment) from which per-donor cell-type counts are tabulated) — https://github.com/satijalab/seurat
- **R lm** (Fits the ordinary linear regression of donor-level fraction on weight status, depot and sex and yields coefficients and 95% CIs)

## Examples

```
fit <- lm(fraction ~ weight_status + depot + sex, data = donor_fractions); confint(fit)
```

## Evaluation signals

- Sign (and reported trend) of the obesity coefficient reproduces the described decrease in smooth muscle cell and FAP fractions in obese donors, and depot/sex contrasts reproduce Fig. 5C and 5G, including shifts among the four adipocyte subpopulations
- Donor-level fractions per cell type sum to 1 within each of the 46 donors
- Output table contains one coefficient plus 95% CI per cell type/state for each contrast (obesity, depot, sex) and is written as fig5_composition_effects.csv
- All cells are assigned to the 43 WATLAS cell types/states before aggregation (no unlabeled cells silently dropped)

## Limitations

- Obesity-associated decreases in smooth muscle cells and FAPs were reported as a trend, not significant, under this ordinary linear regression on fractional data
- Only sex and depot were controlled as covariates; other confounders (age, dataset/batch) are not included in the lm model as specified
- If re-deriving WATLAS labels for multi-condition, multi-sample data, regular PCA (not consensus PCA) is advised in JOINTLY, and no single integration method is best across all tasks

## Evidence

- [intro] Analysis of WATLAS revealed compositional differences between lean and obese donors and between different white adipose tissue depots: "Analysis of WATLAS revealed compositional differences between lean and obese donors and between different white adipose tissue depots"
- [abstract] we describe four adipocyte subpopulations and map compositional changes in obesity and between depots.: "we describe four adipocyte subpopulations and map compositional changes in obesity and between depots."
- [other] For each cell type/state, fit an ordinary linear regression of the donor-level fraction on weight status, depot and sex as covariates in R (lm).: [evidence span withheld — anchor status: not_found]
- [readme] For the white adipose tissue atlas (WATLAS), the model weights are availiable [here](https://zenodo.org/deposit/8086433): "For the white adipose tissue atlas (WATLAS), the model weights are availiable [here](https://zenodo.org/deposit/8086433)"
