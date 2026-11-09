---
name: median-summary-statistics-by-subgroup
description: Use when drug response (e.g., 5-FU IC50) is available per sample and
  samples carry subgroup labels (CMS1-CMS4 from CMScaller), and you need per-group
  median summaries plus a CMS4-vs-CMS1/2/3 comparison.
when_to_use_negative:
- When subtyping of the pre-clinical models is imprecise or not done with CMScaller
  — the article notes CMS-drug analysis 'is hindered by imprecise subtyping of the
  models'
- When sample size per group is very small (the article warns that datasets with fewer
  than approximately forty samples introduce prediction uncertainty in CMS labels)
- When the question concerns MSI-dependent response rather than CMS effects in MSS
  models
edam_operation: http://edamontology.org/operation_2239
tools:
- name: CMScaller
  role: Provides the CMS class labels (CMS1-CMS4) used to subgroup the MSS cell line
    drug response data before computing per-group medians
  repo: https://github.com/Lothelab/CMScaller
- name: R
  role: Environment in which medians per CMS class and the Mann-Whitney (Wilcoxon
    rank-sum) test are computed
- name: Biobase
  role: Loaded in the CMScaller workflow to handle ExpressionSet-style inputs during
    CMS classification of the cell line panel
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:2d5c4c715119e820ec51218e46b57d6b8feafca9fa8e5d72b0fb448afe315328
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lothelab/CMScaller
  tools:
  - Biobase
  - CMScaller
  - R
  edam_operation: http://edamontology.org/operation_2239
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# median-summary-statistics-by-subgroup

## Summary

Computes per-subtype median summary statistics (e.g., 5-FU IC50 per CMS class) from subgrouped pre-clinical drug response data and tests one subgroup against the pooled remainder with a one-sided Mann-Whitney test, as used to demonstrate poor 5-FU response in CMS4 MSS colorectal cancer cell lines.

## When to use

Use when drug response (e.g., 5-FU IC50) is available per sample and samples carry subgroup labels (CMS1-CMS4 from CMScaller), and you need per-group median summaries plus a CMS4-vs-CMS1/2/3 comparison.

## When NOT to use

- When subtyping of the pre-clinical models is imprecise or not done with CMScaller — the article notes CMS-drug analysis 'is hindered by imprecise subtyping of the models'
- When sample size per group is very small (the article warns that datasets with fewer than approximately forty samples introduce prediction uncertainty in CMS labels)
- When the question concerns MSI-dependent response rather than CMS effects in MSS models

## Inputs

- CMScaller CMS class labels (R output of CMScaller) for MSS colorectal cancer cell lines
- Published 5-FU IC50 values per cell line
- Cell line identifier mapping to join CMS labels and drug response
- MSS/MSI status annotation per cell line

## Outputs

- Per-CMS median 5-FU IC50 values (uM) as a table
- One-sided Mann-Whitney p-value for CMS4 vs pooled CMS1/2/3
- CSV file (e.g., cms_5fu_ic50_association.csv) with medians and p-value

## How to apply

Merge the CMScaller CMS class labels for the MSS colorectal cancer cell line panel with the published 5-FU IC50 values on cell line identifier, keeping only MSS models so both labels and response exist per line. Restrict to MSS lines to assess response independent of microsatellite instability. Group the merged table by CMS class and compute the median IC50 per subtype (expected medians ~1.1, 1.1, 4.1, 6.8 uM for CMS1-4). Run a one-sided Mann-Whitney (Wilcoxon rank-sum) test comparing CMS4 IC50 values against the pooled CMS1/2/3 values, since the article reports poor response in CMS4 (p = 7.2e-3). Write per-CMS medians and the p-value to a CSV such as cms_5fu_ic50_association.csv.

## Related tools

- **CMScaller** (Provides the CMS class labels (CMS1-CMS4) used to subgroup the MSS cell line drug response data before computing per-group medians) — https://github.com/Lothelab/CMScaller
- **R** (Environment in which medians per CMS class and the Mann-Whitney (Wilcoxon rank-sum) test are computed)
- **Biobase** (Loaded in the CMScaller workflow to handle ExpressionSet-style inputs during CMS classification of the cell line panel)

## Examples

```
med <- tapply(ic50, cms, median); p <- wilcox.test(ic50[cms=='CMS4'], ic50[cms!='CMS4'], alternative='greater')
```

## Evaluation signals

- Per-CMS median 5-FU IC50 values reproduce the reported pattern of higher IC50 in CMS4, with medians approximately 1.1, 1.1, 4.1, and 6.8 uM for CMS1-CMS4
- The Mann-Whitney CMS4-vs-CMS1/2/3 comparison is computed and reported in the output table (reference p = 7.2e-3)
- All merged rows are MSS cell lines with both a CMS label and an IC50 value
- Output CSV contains both per-CMS medians and the p-value in cms_5fu_ic50_association.csv

## Limitations

- CMScaller labels rely on centered and scaled gene expression input; small datasets (fewer than ~40 samples) introduce prediction uncertainty in the subgroup labels
- Pre-clinical drug screen analysis is hindered by imprecise subtyping of models; labels must come from a classifier optimized for pre-clinical models
- The reported medians/p-value are specific to the MSS panel and 5-FU; other drugs or panels will yield different subgroup statistics

## Evidence

- [abstract] CMS subtypes are predictive of response to standard CRC drugs: "by analyzing public drug response data from patient-derived xenografts and cell lines, we show that the subtypes are predictive of response to standard CRC drugs"
- [discussion] CMS4 poor response rationale: "these pre-clinical data indicate that the CMS4 group has cancer cell-intrinsic characteristics conferring poor response also in the absence of the tumor microenvironment."
- [discussion] Small-dataset uncertainty caveat: "input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty"
- [readme] CMScaller scope: "The `CMScaller` package provides Consensus Molecular Subtype (CMS) classification of colorectal cancer pre-clinical models"
