---
name: cell-line-pharmacogenomic-data-joining
description: Use when CMS class labels from CMScaller must be merged with published
  per-cell-line drug response data (e.g., 5-FU IC50) in MSS colorectal cancer models
  to test whether subtypes predict drug response.
when_to_use_negative:
- The sample set has a different human stromal component than primary colorectal cancers,
  e.g., patient biopsies of metastatic CRC — CMScaller in its current implementation
  is not recommended for such samples.
- The expression dataset is very small (fewer than about forty samples), where centering
  and scaling of input gene expression inescapably introduces prediction uncertainty.
- Input is single-cell RNA-sequencing data, which is exceedingly noisy and for which
  CMScaller is not optimized.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3053
tools:
- name: CMScaller
  role: Assigns CMS subtypes (CMS1–CMS4) to pre-clinical models from gene expression
    via Nearest Template Prediction; these labels are one side of the joined table
  repo: https://github.com/Lothelab/CMScaller
- name: R
  role: Merges CMS labels with 5-FU IC50 values, computes per-CMS median IC50, and
    runs the one-sided Mann-Whitney test
- name: Biobase
  role: Bioconductor dependency loaded alongside CMScaller to handle expression data
    structures used for subtype prediction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:5d7ef341ac3449ce2f3671d17e217a4ef0e2f9b514bee71adaf8c93d2598f713
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# cell-line-pharmacogenomic-data-joining

## Summary

Joins CMScaller-derived CMS subtype labels to published drug response values (e.g., 5-FU IC50) for MSS colorectal cancer cell lines and tests subtype-response associations, showing that CMS4 predicts poor 5-FU response independent of microsatellite instability.

## When to use

Use when CMS class labels from CMScaller must be merged with published per-cell-line drug response data (e.g., 5-FU IC50) in MSS colorectal cancer models to test whether subtypes predict drug response.

## When NOT to use

- The sample set has a different human stromal component than primary colorectal cancers, e.g., patient biopsies of metastatic CRC — CMScaller in its current implementation is not recommended for such samples.
- The expression dataset is very small (fewer than about forty samples), where centering and scaling of input gene expression inescapably introduces prediction uncertainty.
- Input is single-cell RNA-sequencing data, which is exceedingly noisy and for which CMScaller is not optimized.

## Inputs

- CMScaller CMS class labels (data frame from CMScaller(cemat, RNAseq=...)) for MSS colorectal cancer cell lines
- Published 5-FU IC50 (concentration for 50% growth inhibition) values for the same cell lines
- Cell line identifiers shared between the CMS label table and the drug response table

## Outputs

- Per-CMS median 5-FU IC50 values (µM) for CMS1–CMS4
- Mann-Whitney (Wilcoxon rank-sum) p-value for CMS4 vs pooled CMS1/2/3
- cms_5fu_ic50_association.csv containing the medians and the p-value

## How to apply

Load CMScaller CMS class labels for the MSS colorectal cancer cell line panel (predicted from a gene expression matrix with CMScaller, e.g., RNAseq=TRUE, using the Nearest Template Prediction core algorithm) and the published 5-FU IC50 values for the same cell lines. Merge the two tables on cell line identifier, keeping only microsatellite-stable (MSS) models so both a CMS label and a drug response exist per line. Group the merged data by CMS class and compute the median 5-FU IC50 per subtype, expecting approximately 1.1, 1.1, 4.1, and 6.8 µM for CMS1–CMS4. Run a one-sided Mann-Whitney (Wilcoxon rank-sum) test in R comparing CMS4 IC50 values against the pooled CMS1/2/3 IC50 values, which should yield roughly p = 7.2e-3, confirming poor 5-FU response in the mesenchymal CMS4 group. Write the per-CMS median IC50 values together with the Mann-Whitney p-value to cms_5fu_ic50_association.csv.

## Related tools

- **CMScaller** (Assigns CMS subtypes (CMS1–CMS4) to pre-clinical models from gene expression via Nearest Template Prediction; these labels are one side of the joined table) — https://github.com/Lothelab/CMScaller
- **R** (Merges CMS labels with 5-FU IC50 values, computes per-CMS median IC50, and runs the one-sided Mann-Whitney test)
- **Biobase** (Bioconductor dependency loaded alongside CMScaller to handle expression data structures used for subtype prediction)

## Examples

```
res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE)
```

## Evaluation signals

- Per-CMS median 5-FU IC50 values reproduce the reported pattern of approximately 1.1, 1.1, 4.1, and 6.8 µM for CMS1–CMS4 in MSS cell lines.
- Mann-Whitney CMS4-vs-CMS1/2/3 comparison is computed and reported, with the published result near p = 7.2e-3.
- Output file cms_5fu_ic50_association.csv contains both the per-CMS medians and the Mann-Whitney p-value.
- Merge retains only MSS cell lines that have both a CMS label and a 5-FU IC50 value (no NA responses in the grouped data).
- CMS4 medians are strictly higher than CMS1/2/3 medians, consistent with poorer 5-FU response in the mesenchymal subtype.

## Limitations

- CMScaller in its current implementation is not recommended for samples with a different human stromal component than primary CRCs, including patient biopsies of metastatic CRC.
- Input gene expression data must be centered and scaled, so small datasets (fewer than approximately forty samples) inescapably introduce prediction uncertainty.
- Caution is warranted in highly selected datasets where molecular distributions deviate from primary CRC cohorts, including metastatic samples.
- The poor 5-FU response signal is established in MSS models; microsatellite instability is a confounder that this workflow explicitly controls by keeping only MSS cell lines.

## Evidence

- [abstract] CMS subtypes are predictive of response to standard CRC drugs based on public drug response data from patient-derived xenografts and cell lines: "by analyzing public drug response data from patient-derived xenografts and cell lines, we show that the subtypes are predictive of response to standard CRC drugs"
- [discussion] CMS4 poor chemotherapy response is cancer cell-intrinsic: "these pre-clinical data indicate that the CMS4 group has cancer cell-intrinsic characteristics conferring poor response also in the absence of the tumor microenvironment"
- [discussion] Small-dataset prediction uncertainty from centering/scaling: "input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty"
- [readme] CMScaller package scope for pre-clinical models: "The `CMScaller` package provides Consensus Molecular Subtype (CMS) classification of colorectal cancer pre-clinical models"
