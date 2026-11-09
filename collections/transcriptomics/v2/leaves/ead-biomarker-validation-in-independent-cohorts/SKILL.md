---
name: ead-biomarker-validation-in-independent-cohorts
description: Use when a cell-type-based biomarker or immune niche identified in a
  discovery scRNA-seq cohort (e.g., EAD vs non-EAD liver transplant samples) must
  be verified for association with a clinical outcome in an independent cohort.
when_to_use_negative:
- Input is a cell-cell communication (ligand-receptor) analysis problem rather than
  biomarker reproducibility testing — use scCrossTalk's find_lrpairs/plot_cci_* workflow
  instead
- The candidate biomarker was not defined as specific cell populations with annotated
  markers, so no per-sample cell-abundance score can be computed
- Only the original discovery cohort is available and no independent samples exist,
  making validation circular
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: R
  role: Language in which the verification pipeline (Seurat preprocessing, niche scoring,
    Welch's test, CSV export) is implemented
- name: Seurat
  role: Normalization, dimensionality reduction, clustering, and cell-type annotation
    of the independent-cohort scRNA-seq data; scCrossTalk is compatible with Seurat
    5
- name: scCrossTalk
  role: Source of the marker definitions for the niche cell populations and of the
    one-sided Welch's test comparison strategy between non-EAD and EAD patients
  repo: https://github.com/multitalk/scCrossTalk
- name: ggplot2
  role: Visualization of niche scores and verification results
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.eng.2023.12.004
    title: A Single-Cell Landscape of Human Liver Transplantation Reveals a Pathogenic
      Immune Niche Associated with Early Allograft Dysfunction
  source_content_hash: sha256:afe6b4438db7d1230078a445ce0c8861a98d0e865ed546ba2b4e74b228e9b410
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/multitalk/scCrossTalk
  repo_url: https://github.com/multitalk/scCrossTalk
  tools:
  - R
  - Seurat
  - ggplot2
  - scCrossTalk
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# ead-biomarker-validation-in-independent-cohorts

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproducibility protocol for verifying that the EAD-associated immune niche (MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils) discovered in the scCrossTalk single-cell landscape of transplanted livers is associated with early allograft dysfunction in an independent human liver transplantation scRNA-seq cohort.

## When to use

Use when a cell-type-based biomarker or immune niche identified in a discovery scRNA-seq cohort (e.g., EAD vs non-EAD liver transplant samples) must be verified for association with a clinical outcome in an independent cohort.

## When NOT to use

- Input is a cell-cell communication (ligand-receptor) analysis problem rather than biomarker reproducibility testing — use scCrossTalk's find_lrpairs/plot_cci_* workflow instead
- The candidate biomarker was not defined as specific cell populations with annotated markers, so no per-sample cell-abundance score can be computed
- Only the original discovery cohort is available and no independent samples exist, making validation circular

## Inputs

- Raw human liver transplantation scRNA-seq data from an independent cohort (SRA accession SRP313633)
- Marker definitions for MAIT cells, GZMB+ GZMK+ NK cells, and S100A12+ neutrophils from the scCrossTalk manuscript
- Per-sample EAD vs non-EAD clinical group labels

## Outputs

- ead_niche_verification.csv containing per-sample niche scores, group labels, test statistic, and p-value
- Per-sample immune niche abundance scores for EAD and non-EAD samples

## How to apply

Download raw scRNA-seq data for the independent verification cohort from SRA (accession SRP313633) and load it into R with the Seurat package. Perform standard Seurat preprocessing (normalization, dimensionality reduction, clustering) and annotate cell populations using the marker definitions from the scCrossTalk manuscript, identifying MAIT cells, GZMB+ GZMK+ NK cells, and S100A12+ neutrophils. Compute a per-sample immune niche score reflecting the abundance of the three niche cell populations in each EAD and non-EAD sample. Test the association between the niche score and EAD occurrence with a one-sided Welch's test comparing non-EAD and EAD patients, mirroring the comparison strategy used in the scCrossTalk paper for LR-pair counts and scores. Write the niche scores, group labels, test statistic and p-value to ead_niche_verification.csv.

## Related tools

- **R** (Language in which the verification pipeline (Seurat preprocessing, niche scoring, Welch's test, CSV export) is implemented)
- **Seurat** (Normalization, dimensionality reduction, clustering, and cell-type annotation of the independent-cohort scRNA-seq data; scCrossTalk is compatible with Seurat 5)
- **scCrossTalk** (Source of the marker definitions for the niche cell populations and of the one-sided Welch's test comparison strategy between non-EAD and EAD patients) — https://github.com/multitalk/scCrossTalk
- **ggplot2** (Visualization of niche scores and verification results)

## Examples

```
library(Seurat); obj <- readRDS("srp313633_seurat.rds"); obj <- NormalizeData(obj); obj <- FindClusters(obj, dims = 1:30); t.test(niche_score ~ ead_status, data = niche_df, alternative = "greater")
```

## Evaluation signals

- The three niche populations (MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils) are each identified in the independent cohort using the manuscript's marker definitions
- A niche score is produced for every sample in both EAD and non-EAD groups with no missing labels
- The association test is one-sided Welch's test comparing non-EAD vs EAD patients, matching the comparison strategy in the scCrossTalk paper
- ead_niche_verification.csv contains the niche scores, group labels, test statistic, and p-value with a statistically significant association supporting reproducibility
- Direction of the niche-score difference (higher in EAD) is consistent with the discovery-cohort finding

## Limitations

- Validation depends on the independent cohort's data quality and preprocessing; differences in Seurat clustering or annotation parameters can change niche cell abundance estimates
- The one-sided Welch's test presupposes the direction of the effect (niche enriched in EAD), so it should not be applied blindly to biomarkers with unknown directionality
- The verification inherits the marker definitions from the discovery cohort; populations poorly resolved in the independent cohort may not be comparable

## Evidence

- [results] The EAD-associated immune niche was verified in an independent human LT scRNA-seq cohort retrieved from the SRA database under accession number SRP313633.: [evidence span withheld — anchor status: not_found]
- [abstract] Comparison of 75,231 cells from non-EAD and EAD patients revealed an EAD-associated immune niche comprising MAIT cells, GZMB+ GZMK+ NK cells, and S100A12+ neutrophils.: "Comparison of the 75 231 cells of non-EAD and EAD patients revealed an EAD-associated immune niche comprising mucosal-associated invariant T cells, granzyme B+ (GZMB+) granzyme K (GZMK+) natural"
- [abstract] The immune niche and its association with EAD occurrence were verified in two independent cohorts.: "we verified this immune niche and its association with EAD occurrence in two independent cohorts"
- [readme] One-sided Welch's test is the comparison strategy used in the scCrossTalk paper for non-EAD vs EAD comparisons.: "Comparison of the number and score of LR pairs between non-EAD and EAD patients with one-sided Welch's test"
- [readme] scCrossTalk was designed to dissect cell-cell communication mechanisms within transplanted livers of EAD and non-EAD patients, defining the niche populations this skill validates.: "scCrossTalk was designed in our manuscript to dissect the different mechanism underlying cell-cell communications within the transplanted livers of EAD and non-EAD patients."
