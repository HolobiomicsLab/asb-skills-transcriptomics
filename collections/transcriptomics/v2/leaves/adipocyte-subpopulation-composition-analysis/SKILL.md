---
name: adipocyte-subpopulation-composition-analysis
description: Use when a cell-by-cell type annotation table (e.g., WATLAS/SCP2289 labels)
  must be converted to per-donor cell type fractions and tested for obesity, depot,
  or sex effects via covariate-adjusted linear models.
when_to_use_negative:
- Input is raw single-cell expression data without cell type annotations — annotations
  must first be produced via JOINTLY/scANVI labelling (WATLAS) before composition
  testing
- The analysis goal is batch integration or clustering rather than compositional inference
  across donors — use the JOINTLY joint clustering workflow instead
- Donor-level metadata lacks weight status, depot, or sex covariates, making the covariate-adjusted
  lm design unidentifiable
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3360
tools:
- name: JOINTLY
  role: Produced the interpretable joint clustering and cell type/state labels for
    the WATLAS that feed the composition analysis
  repo: https://github.com/madsen-lab/rJOINTLY
- name: R lm
  role: Fits the ordinary linear regression of donor-level cell type fractions on
    weight status, depot, and sex to extract effect sizes and 95% confidence intervals
- name: WATLAS (SCP2289)
  role: Reference atlas with 43 characterised cell types and states providing the
    annotated cells whose composition is compared between lean and obese donors and
    depots
  repo: https://singlecell.broadinstitute.org/single_cell/study/SCP2289/an-integrated-single-cell-and-single-nucleus-rna-seq-white-adipose-tissue-atlas-watlas
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:80dcdda8a05cb94af2d4a6db7d512540342fafcd7cfba8dfb68bdef349d1b7fd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/madsen-lab/rJOINTLY
  tools:
  - JOINTLY
  - R lm
  - WATLAS (SCP2289)
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# adipocyte-subpopulation-composition-analysis

## Summary

Quantifies how white adipose tissue (WAT) cell type and cell state composition differs between lean and obese donors and between depots, using WATLAS cell annotations and donor-level fractional abundances fitted with ordinary linear regression.

## When to use

Use when a cell-by-cell type annotation table (e.g., WATLAS/SCP2289 labels) must be converted to per-donor cell type fractions and tested for obesity, depot, or sex effects via covariate-adjusted linear models.

## When NOT to use

- Input is raw single-cell expression data without cell type annotations — annotations must first be produced via JOINTLY/scANVI labelling (WATLAS) before composition testing
- The analysis goal is batch integration or clustering rather than compositional inference across donors — use the JOINTLY joint clustering workflow instead
- Donor-level metadata lacks weight status, depot, or sex covariates, making the covariate-adjusted lm design unidentifiable

## Inputs

- WATLAS cell-by-cell type/state annotation table (SCP2289 deposited annotations or task_007 output)
- Per-donor metadata for 46 donors: donor id, sex, depot (subcutaneous/omental), lean/obese weight status

## Outputs

- fig5_composition_effects.csv containing per-cell-type regression coefficients (effect sizes) and 95% confidence intervals for obesity, depot, and sex contrasts
- Donor-level fractional abundance matrix per cell type/state

## How to apply

Load the WATLAS cell-by-cell type/state annotation table (task_007 output or the deposited SCP2289 annotations) together with per-donor metadata (donor id, sex, depot, lean/obese weight status) for the 46 donors. Aggregate cell counts per donor and cell type/state, then convert them to donor-level fractions per cell type — this includes the four adipocyte subpopulations described in WATLAS. For each cell type/state, fit an ordinary linear regression (R lm) of the donor-level fraction on weight status, depot and sex as covariates, extracting the marginal regression coefficient and its 95% confidence interval for each contrast (obesity, depot, sex). Assemble all effect sizes and confidence intervals into a single table written as fig5_composition_effects.csv. The workflow assumes the underlying annotations came from JOINTLY-based labelling of six independent datasets integrated with scANVI, which retains subtle cell state differences between biological conditions.

## Related tools

- **JOINTLY** (Produced the interpretable joint clustering and cell type/state labels for the WATLAS that feed the composition analysis) — https://github.com/madsen-lab/rJOINTLY
- **R lm** (Fits the ordinary linear regression of donor-level cell type fractions on weight status, depot, and sex to extract effect sizes and 95% confidence intervals)
- **WATLAS (SCP2289)** (Reference atlas with 43 characterised cell types and states providing the annotated cells whose composition is compared between lean and obese donors and depots) — https://singlecell.broadinstitute.org/single_cell/study/SCP2289/an-integrated-single-cell-and-single-nucleus-rna-seq-white-adipose-tissue-atlas-watlas

## Examples

```
fit <- lm(fraction ~ weight_status + depot + sex, data = donor_fractions); coef(fit); confint(fit)
```

## Evaluation signals

- Signs and significance of computed obesity effect sizes must reproduce the reported trend towards a decrease in fractional smooth muscle cells and fibro-adipogenic progenitors (FAPs) in obese donors (Fig. 5C)
- Depot effect sizes must reproduce the reported depot-driven compositional shifts (Fig. 5G), including shifts among the four adipocyte subpopulations
- Donor-level fractions must sum to 1 per donor per annotation level, confirming correct aggregation of counts to fractions
- Output fig5_composition_effects.csv must contain a marginal coefficient and 95% CI for every cell type/state × contrast (obesity, depot, sex) combination
- Model design must include sex and depot as covariates when estimating the obesity effect (marginal effects, not univariate comparisons)

## Limitations

- The obesity association with reduced smooth muscle cell and FAP fractions is reported as a trend, not a definitive significant effect after controlling for sex and depot
- Linear regression on donor-level fractions does not model count-based uncertainty; results depend on the 46-donor sample size
- Composition results depend on the quality of upstream WATLAS annotations; JOINTLY performance varies task-to-task and no single integration method is always best

## Evidence

- [intro] Analysis of WATLAS revealed compositional differences between lean and obese donors and between different white adipose tissue depots: "Analysis of WATLAS revealed compositional differences between lean and obese donors and between different white adipose tissue depots"
- [abstract] we describe four adipocyte subpopulations and map compositional changes in obesity and between depots: "we describe four adipocyte subpopulations and map compositional changes in obesity and between depots."
- [full_text] Controlling for sex and depot, obesity is associated with a trend towards a decrease in the fractional number of smooth muscle cells and fibro-adipogenic progenitors (FAPs).: [evidence span withheld — anchor status: not_found]
- [abstract] JOINTLY is robust against over-correction while retaining subtle cell state differences between biological conditions: "JOINTLY is robust against over-correction while retaining subtle cell state differences between biological conditions"
- [readme] the atlas can be explored and downloaded at the SCP2289 study hosting WATLAS: "the atlas can be explored and downloaded [here](https://singlecell.broadinstitute.org/single_cell/study/SCP2289/an-integrated-single-cell-and-single-nucleus-rna-seq-white-adipose-tissue-atlas-watlas)"
