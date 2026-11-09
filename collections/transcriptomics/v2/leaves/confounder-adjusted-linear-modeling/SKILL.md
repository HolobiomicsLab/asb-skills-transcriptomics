---
name: confounder-adjusted-linear-modeling
description: Use when you have donor-aggregated single-cell type/state counts (e.g.
  WATLAS annotations for 46 donors) and must test whether a condition such as obesity
  shifts cell-type composition while controlling for covariates like depot and sex.
when_to_use_negative:
- Input is raw per-cell data without donor-level aggregation or donor metadata (weight
  status, depot, sex) — donor-level fractions and covariates are required for the
  lm() design.
- The goal is joint clustering or batch integration of single-cell datasets — use
  JOINTLY for that; this skill only quantifies compositional differences after annotation
  exists.
- Multi-condition, multi-sample batch-effect correction is needed at the expression
  level — the JOINTLY article advises regular PCA in JOINTLY for such datasets, not
  composition regression.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R lm
  role: Fits the ordinary linear regression of donor-level cell-type fraction on weight
    status, depot and sex covariates
- name: manual expert review
  role: Validation that computed effect-size signs and significance reproduce the
    reported Fig. 5C/5G compositional differences
- name: WATLAS (SCP2289)
  role: Source of the cell type/state annotations and donor metadata whose composition
    is modeled
  repo: https://singlecell.broadinstitute.org/single_cell/study/SCP2289/an-integrated-single-cell-and-single-nucleus-rna-seq-white-adipose-tissue-atlas-watlas
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-44279-8
    title: 'JOINTLY: interpretable joint clustering of single-cell transcriptomes'
  source_content_hash: sha256:0ecf77be5ee130dacc5d8a70215f1d3269a55431de6f8715c5a5dcfe4469dcc5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R lm
  - WATLAS (SCP2289)
  - manual expert review
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# confounder-adjusted-linear-modeling

## Summary

Fits per-cell-type ordinary linear regressions of donor-level cell-type/state fractions on disease weight status, depot and sex to quantify compositional changes in the WATLAS white adipose tissue atlas, extracting marginal effect sizes (regression coefficients) with 95% confidence intervals per contrast.

## When to use

Use when you have donor-aggregated single-cell type/state counts (e.g. WATLAS annotations for 46 donors) and must test whether a condition such as obesity shifts cell-type composition while controlling for covariates like depot and sex.

## When NOT to use

- Input is raw per-cell data without donor-level aggregation or donor metadata (weight status, depot, sex) — donor-level fractions and covariates are required for the lm() design.
- The goal is joint clustering or batch integration of single-cell datasets — use JOINTLY for that; this skill only quantifies compositional differences after annotation exists.
- Multi-condition, multi-sample batch-effect correction is needed at the expression level — the JOINTLY article advises regular PCA in JOINTLY for such datasets, not composition regression.

## Inputs

- Cell-by-cell type/state annotation table from WATLAS (task_007 output or deposited SCP2289 annotations)
- Per-donor metadata for the 46 donors (donor id, sex, depot, lean/obese weight status)

## Outputs

- fig5_composition_effects.csv: table of regression effect sizes (coefficients) and 95% confidence intervals per cell type/state for each contrast (obesity, depot, sex)

## How to apply

Aggregate cell-by-cell-type/state annotation counts (from the WATLAS labels, e.g. task_007 output or deposited SCP2289 annotations) per donor and convert them to donor-level fractions for each of the 46 donors in the 43-cell-type/state atlas. For each cell type/state, fit an ordinary linear regression in R with lm(), regressing the donor-level fraction on weight status (lean vs obese), depot and sex as covariates. Extract the marginal regression coefficient and its 95% confidence interval for each contrast of interest (obesity, depot, sex). Assemble all effect sizes and confidence intervals into a single table written as fig5_composition_effects.csv. The expected result, per the source card, is a trend towards a decrease in the fractional number of smooth muscle cells and fibro-adipogenic progenitors (FAPs) with obesity after controlling for sex and depot, reproducing the obesity- and depot-driven shifts among the four adipocyte subpopulations shown in Fig. 5C and 5G.

## Related tools

- **R lm** (Fits the ordinary linear regression of donor-level cell-type fraction on weight status, depot and sex covariates)
- **manual expert review** (Validation that computed effect-size signs and significance reproduce the reported Fig. 5C/5G compositional differences)
- **WATLAS (SCP2289)** (Source of the cell type/state annotations and donor metadata whose composition is modeled) — https://singlecell.broadinstitute.org/single_cell/study/SCP2289/an-integrated-single-cell-and-single-nucleus-rna-seq-white-adipose-tissue-atlas-watlas

## Examples

```
fit <- lm(fraction ~ weight_status + depot + sex, data = donor_fractions); coef(fit); confint(fit)
```

## Evaluation signals

- Sign and significance of the obesity coefficients reproduce the reported decrease in fractional number of smooth muscle cells and FAPs in obesity after controlling for sex and depot.
- Computed obesity and depot effect sizes match the reported compositional differences between lean and obese donors and between WAT depots (Fig. 5C and 5G), including shifts among the four adipocyte subpopulations.
- Every cell type/state contrast carries both a marginal coefficient and a 95% confidence interval in fig5_composition_effects.csv.
- Donor-level fractions sum to 1 per donor across cell types/states and cover all 46 donors.
- The model design includes weight status, depot and sex as covariates for every per-cell-type regression.

## Limitations

- The obesity association with SMC and FAP fractions is reported as a trend, not a significant effect, so per-cell-type results may not reach significance.
- Fractions are compositional; ordinary lm() on donor-level fractions does not model count/binomial uncertainty or apply compositional-data corrections.
- Results depend on upstream WATLAS annotation quality and on the six-dataset integration performed with JOINTLY plus scANVI; errors there propagate to the composition effects.
- WATLAS analysis of composition is presented for hypothesis generation in the atlas context and should be confirmed on independent data.

## Evidence

- [other] workflow: ordinary linear regression with covariates: [evidence span withheld — anchor status: not_found]
- [other] task card finding: [evidence span withheld — anchor status: not_found]
- [intro] article intro research question: "Analysis of WATLAS revealed compositional differences between lean and obese donors and between different white adipose tissue depots"
- [abstract] article abstract on adipocyte subpopulations: "we describe four adipocyte subpopulations and map compositional changes in obesity and between depots."
- [readme] README WATLAS hosting location: "the atlas can be explored and downloaded [here](https://singlecell.broadinstitute.org/single_cell/study/SCP2289/an-integrated-single-cell-and-single-nucleus-rna-seq-white-adipose-tissue-atlas-watlas)"
