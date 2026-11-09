---
name: drug-response-ic50-analysis
description: Use when CMS labels from CMScaller for pre-clinical models must be linked
  to drug response (e.g. 5-FU IC50) to test whether subtypes predict chemotherapy
  sensitivity in MSS CRC cell lines.
when_to_use_negative:
- The goal is CMS classification of primary colorectal tumors rather than linking
  pre-clinical model subtypes to drug response
- Only MSI-high cell lines are available — the reported finding concerns MSS models
  specifically, so the MSS-restricted analysis is not applicable
- Drug response data lacks quantitative IC50 (e.g. only categorical responder labels),
  which the median and Mann-Whitney design requires
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: CMScaller
  role: Provides the CMS class labels for the cell line panel that are merged with
    IC50 values and grouped by subtype
  repo: https://github.com/Lothelab/CMScaller
- name: R
  role: Environment in which the merge, per-CMS median computation, and one-sided
    Mann-Whitney (Wilcoxon rank-sum) test are executed
- name: Biobase
  role: Bioconductor base loaded alongside CMScaller to handle expression/CMS label
    objects in R
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:d5fdea6ee349bb06ffa788baf51488a4e4b9a37e37eb994c65db3d2189c359d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lothelab/CMScaller
  tools:
  - Biobase
  - CMScaller
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# drug-response-ic50-analysis

## Summary

Associates CMScaller-assigned CMS subtypes of MSS colorectal cancer cell lines with 5-FU IC50 values, using per-subtype median IC50 summaries and a one-sided Mann-Whitney test to demonstrate poorer 5-FU response in CMS4.

## When to use

Use when CMS labels from CMScaller for pre-clinical models must be linked to drug response (e.g. 5-FU IC50) to test whether subtypes predict chemotherapy sensitivity in MSS CRC cell lines.

## When NOT to use

- The goal is CMS classification of primary colorectal tumors rather than linking pre-clinical model subtypes to drug response
- Only MSI-high cell lines are available — the reported finding concerns MSS models specifically, so the MSS-restricted analysis is not applicable
- Drug response data lacks quantitative IC50 (e.g. only categorical responder labels), which the median and Mann-Whitney design requires

## Inputs

- CMScaller CMS class labels for MSS CRC cell line panel (from CMScaller R package)
- Published 5-FU IC50 (growth inhibition 50%) values for the same cell lines
- Cell line microsatellite instability (MSS/MSI) status annotations

## Outputs

- Table of per-CMS median 5-FU IC50 values (expected ~1.1, 1.1, 4.1, 6.8 uM for CMS1-4)
- Mann-Whitney p-value for CMS4 vs pooled CMS1/2/3 comparison
- cms_5fu_ic50_association.csv containing medians and p-value

## How to apply

Load the CMScaller CMS class labels for the MSS colorectal cancer cell line panel and the published 5-FU IC50 values, then merge the two tables on cell line identifier keeping only MSS models. Group the merged data by CMS class and compute the median IC50 per subtype, expecting medians of approximately 1.1, 1.1, 4.1 and 6.8 uM for CMS1-CMS4. Run a one-sided Mann-Whitney (Wilcoxon rank-sum) test in R comparing CMS4 IC50 values against the pooled CMS1/2/3 values to assess poorer 5-FU response (reported p = 7.2e-3). Write the per-CMS median IC50 values together with the p-value to an output CSV (e.g. cms_5fu_ic50_association.csv). The rationale is that CMScaller uses cancer cell-intrinsic markers, so CMS-drug associations in cell lines reflect cell-intrinsic chemosensitivity rather than stromal confounding.

## Related tools

- **CMScaller** (Provides the CMS class labels for the cell line panel that are merged with IC50 values and grouped by subtype) — https://github.com/Lothelab/CMScaller
- **R** (Environment in which the merge, per-CMS median computation, and one-sided Mann-Whitney (Wilcoxon rank-sum) test are executed)
- **Biobase** (Bioconductor base loaded alongside CMScaller to handle expression/CMS label objects in R)

## Examples

```
library(CMScaller); cms <- CMScaller(emat, RNAseq=TRUE); med <- tapply(ic50$ic50, cms$prediction[cms$prediction!='CMS4' | isMSS], median); wilcox.test(ic50[cms=='CMS4'], ic50[cms%in%c('CMS1','CMS2','CMS3')], alternative='greater')
```

## Evaluation signals

- Per-CMS median IC50 medians reproduce the reported pattern of higher 5-FU IC50 in CMS4 (~1.1, 1.1, 4.1, 6.8 uM for CMS1-4)
- One-sided Mann-Whitney CMS4-vs-CMS1/2/3 p-value is computed and reported (reference value p = 7.2e-3)
- Only MSS cell lines are retained in the merged table — MSI models excluded before grouping
- Every cell line in the merged table has both a CMS class label and a 5-FU IC50 value
- Output file cms_5fu_ic50_association.csv contains both median IC50 per CMS and the test p-value

## Limitations

- CMScaller predictions can be uncertain for small datasets (fewer than approximately forty samples) because input gene expression data must be centered and scaled
- CMScaller is not recommended for samples with a different human stromal component than primary CRCs, e.g. metastatic CRC biopsies
- Analysis of CMS-drug associations from pre-clinical drug screen data is hindered by imprecise subtyping of the models if CMScaller is not used

## Evidence

- [discussion] PDX and cell line drug response data suggest CMS groups add predictive information for 5-FU-based chemotherapy response, with poor response indicated in CMS4: "The PDX and cell line drug response data presented here suggest that the CMS-groups add additional predictive information, indicating a poor response in CMS4."
- [abstract] CMS subtypes predictive of response from public drug response data: "by analyzing public drug response data from patient-derived xenografts and cell lines, we show that the subtypes are predictive of response to standard CRC drugs"
- [intro] CMS classification as basis for stratified treatment: [evidence span withheld — anchor status: not_found]
- [readme] CMScaller package usage from README: "The `CMScaller` package provides Consensus Molecular Subtype (CMS) classification of colorectal cancer pre-clinical models"
- [discussion] CMS4 cell-intrinsic poor chemotherapy response: "these pre-clinical data indicate that the CMS4 group has cancer cell-intrinsic characteristics conferring poor response also in the absence of the tumor microenvironment."
