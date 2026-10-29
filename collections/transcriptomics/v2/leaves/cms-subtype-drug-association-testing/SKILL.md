---
name: cms-subtype-drug-association-testing
description: Use when cell line, PDX, or organoid models already have CMScaller CMS
  labels and per-model drug response values (e.g. 5-FU IC50), and you want to test
  whether subtype predicts drug sensitivity independently of MSI status.
when_to_use_negative:
- The samples have a different human stromal component than primary CRCs, e.g. patient
  biopsies of metastatic CRC — CMScaller is not recommended for such samples
- The input is single-cell RNA-sequencing data, which is exceedingly noisy and for
  which the classifier is not optimized
- The dataset is very small (fewer than approximately forty samples), where centering
  and scaling input expression data introduces prediction uncertainty at class boundaries
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0208
tools:
- name: CMScaller
  role: Assigns CMS class labels (CMS1–CMS4) to pre-clinical models from gene expression
    via Nearest Template Prediction; these labels are the grouping variable for the
    drug-response association test
  repo: https://github.com/Lothelab/CMScaller
- name: R
  role: Runtime environment for CMScaller and for the Mann-Whitney (Wilcoxon rank-sum)
    test and per-CMS median IC50 aggregation
- name: Biobase
  role: Bioconductor base package loaded alongside CMScaller to handle expression
    data objects
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:965f5b43621f797bb1d63a20ac1037485a4fb77de292bff52ba4e6fa20ff5619
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lothelab/CMScaller
  tools:
  - Biobase
  - CMScaller
  - R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0208
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cms-subtype-drug-association-testing

## Summary

Tests whether Consensus Molecular Subtype (CMS) classes, assigned to pre-clinical colorectal cancer models with the cancer cell-intrinsic CMScaller, predict response to standard CRC drugs such as 5-fluorouracil by comparing drug IC50 distributions across subtypes with Mann-Whitney testing.

## When to use

Use when cell line, PDX, or organoid models already have CMScaller CMS labels and per-model drug response values (e.g. 5-FU IC50), and you want to test whether subtype predicts drug sensitivity independently of MSI status.

## When NOT to use

- The samples have a different human stromal component than primary CRCs, e.g. patient biopsies of metastatic CRC — CMScaller is not recommended for such samples
- The input is single-cell RNA-sequencing data, which is exceedingly noisy and for which the classifier is not optimized
- The dataset is very small (fewer than approximately forty samples), where centering and scaling input expression data introduces prediction uncertainty at class boundaries

## Inputs

- CMScaller CMS class labels for a pre-clinical CRC model panel (R data frame with cell line identifier and prediction)
- Published drug response values (e.g. 5-FU IC50 in µM) for the same models
- MSS/MSI status annotations per cell line for subsetting

## Outputs

- Per-CMS median IC50 table (expected 5-FU medians ~1.1, 1.1, 4.1, 6.8 µM for CMS1–4)
- Mann-Whitney p-value for CMS4 vs. pooled CMS1/2/3 IC50 comparison
- Combined results file, e.g. cms_5fu_ic50_association.csv

## How to apply

Load the CMScaller CMS class labels for the pre-clinical model panel (produced by the CMScaller R package, which uses Nearest Template Prediction on cancer cell-intrinsic subtype-enriched markers) together with the published drug IC50 values for the same models. Merge the two tables on the model identifier, keeping only microsatellite-stable (MSS) models so both a CMS label and a drug response measurement exist per line — for the MSS CRC cell line panel this isolates subtype effects from microsatellite instability. Group by CMS class and compute the median IC50 per subtype; for 5-FU the expected medians are approximately 1.1, 1.1, 4.1, and 6.8 µM for CMS1–CMS4. Run a one-sided Mann-Whitney (Wilcoxon rank-sum) test comparing CMS4 IC50 values against pooled CMS1/2/3 values to test for poorer 5-FU response (reported p = 7.2 × 10−3). Write the per-CMS median IC50 values and the Mann-Whitney p-value to an output table such as cms_5fu_ic50_association.csv.

## Related tools

- **CMScaller** (Assigns CMS class labels (CMS1–CMS4) to pre-clinical models from gene expression via Nearest Template Prediction; these labels are the grouping variable for the drug-response association test) — https://github.com/Lothelab/CMScaller
- **R** (Runtime environment for CMScaller and for the Mann-Whitney (Wilcoxon rank-sum) test and per-CMS median IC50 aggregation)
- **Biobase** (Bioconductor base package loaded alongside CMScaller to handle expression data objects)

## Examples

```
library(CMScaller); res <- CMScaller(emat, RNAseq=TRUE, doPlot=TRUE); wilcox.test(ic50[class=='CMS4'], ic50[class %in% c('CMS1','CMS2','CMS3')], alternative='greater')
```

## Evaluation signals

- Per-CMS median 5-FU IC50 values reproduce the reported pattern of ~1.1, 1.1, 4.1, and 6.8 µM for CMS1–CMS4 in the MSS cell line panel
- CMS4 shows the highest median IC50 relative to CMS1/2/3, indicating poorer 5-FU response
- A one-sided Mann-Whitney p-value for CMS4 vs. pooled CMS1/2/3 is computed and reported (reference value p = 7.2 × 10−3)
- Only MSS models with both a CMS label and an IC50 measurement are retained after the merge on cell line identifier
- Output table (cms_5fu_ic50_association.csv) contains both the per-CMS medians and the p-value

## Limitations

- The original CMS classifier depends on immune and stromal gene expression signals and fails to identify CMS1 and CMS4 in immortalized cell lines, patient-derived organoids and xenografts — hence CMScaller's cancer cell-intrinsic markers are required for this workflow
- Input gene expression data must be centered and scaled, which introduces prediction uncertainty for small datasets (fewer than approximately forty samples)
- Caution is warranted for highly selected datasets where molecular distributions deviate from primary CRC cohorts, including metastatic samples
- CMScaller is not optimized for single-cell RNA-sequencing data and is not recommended for samples with a different human stromal component than primary CRCs

## Evidence

- [abstract] CMS subtypes predictive of 5-FU response: "by analyzing public drug response data from patient-derived xenografts and cell lines, we show that the subtypes are predictive of response to standard CRC drugs"
- [discussion] CMScaller availability: "CMScaller is platform independent and available as an R package."
- [discussion] CMS4 poor chemotherapy response rationale: "these pre-clinical data indicate that the CMS4 group has cancer cell-intrinsic characteristics conferring poor response also in the absence of the tumor microenvironment."
- [abstract] Failure of original classifier in pre-clinical models: "The currently available classifier is dependent on gene expression signals from the immune and stromal compartments of tumors and fails to identify the poor-prognostic CMS4-mesenchymal group"
- [readme] Installation from GitHub: "devtools::install_github("Lothelab/CMScaller")"
- [readme] CMScaller prediction call: "res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE)"
