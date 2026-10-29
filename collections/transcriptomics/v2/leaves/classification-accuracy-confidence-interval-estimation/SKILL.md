---
name: classification-accuracy-confidence-interval-estimation
description: Use when a gene-expression matrix with pre-existing reference labels
  (e.g. original CMS classes for TCGA primary colorectal cancers, n = 143) must be
  re-classified and agreement quantified as accuracy with confidence intervals and
  per-class sensitivity/specificity.
when_to_use_negative:
- Samples with a different human stromal component than primary CRCs, e.g. patient
  biopsies of metastatic CRC — CMScaller is explicitly not recommended there
- Datasets with fewer than approximately forty samples — centering and scaling inescapably
  introduces prediction uncertainty
- Single-cell RNA-sequencing data, which is exceedingly noisy and not what the classifier
  is optimized for
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: CMScaller
  role: Performs the nearest template prediction that assigns CMS labels to be compared
    against reference classifications
  repo: github.com/Lothelab/CMScaller
- name: R
  role: Runtime environment in which CMScaller is executed and accuracy/CI metrics
    are computed
- name: Biobase
  role: Loaded to handle gene expression data structures in the CMScaller workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:a1cbba587fae5e2979024f391ab0f84449b657bfecd830f2bf6f2f14373a49c6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Biobase
  - CMScaller
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# classification-accuracy-confidence-interval-estimation

## Summary

Validate a nearest-template-prediction (NTP) subtype classifier such as CMScaller by comparing its assigned CMS labels against original reference labels on a held-out cohort, reporting overall accuracy with a 95% confidence interval plus class-wise sensitivity and specificity.

## When to use

Use when a gene-expression matrix with pre-existing reference labels (e.g. original CMS classes for TCGA primary colorectal cancers, n = 143) must be re-classified and agreement quantified as accuracy with confidence intervals and per-class sensitivity/specificity.

## When NOT to use

- Samples with a different human stromal component than primary CRCs, e.g. patient biopsies of metastatic CRC — CMScaller is explicitly not recommended there
- Datasets with fewer than approximately forty samples — centering and scaling inescapably introduces prediction uncertainty
- Single-cell RNA-sequencing data, which is exceedingly noisy and not what the classifier is optimized for

## Inputs

- Held-out TCGA primary colorectal cancer (pCRC) gene expression matrix (n = 143) with original CMS labels per sample
- CMScaller template gene sets from the prior filtering task, or the shipped templates.CMS reference
- R environment with the CMScaller and Biobase packages installed

## Outputs

- Per-sample NTP-assigned CMS1–CMS4 predictions
- cms_ntp_validation_metrics.csv containing overall accuracy with 95% CI and class-wise sensitivity and specificity for CMS1–CMS4

## How to apply

Load the held-out TCGA pCRC expression matrix together with the original CMS labels, then center and scale the expression data as required by CMScaller before prediction. Run CMScaller's nearest template prediction (NTP, Hoshida 2010) using the subtype-enriched template gene sets or the shipped templates.CMS reference to assign CMS1–CMS4 labels to each sample. Compare NTP-assigned labels to the original labels sample by sample, then compute overall accuracy with a 95% confidence interval and class-wise sensitivity and specificity for CMS1–CMS4, writing the metrics to cms_ntp_validation_metrics.csv. Correct application is judged by reproducing the reported benchmark of 0.83 overall accuracy (95% CI 0.74–0.9) on TCGA pCRCs, e.g. CMS3 sensitivity 0.71 and CMS4 specificity 0.91.

## Related tools

- **CMScaller** (Performs the nearest template prediction that assigns CMS labels to be compared against reference classifications) — github.com/Lothelab/CMScaller
- **R** (Runtime environment in which CMScaller is executed and accuracy/CI metrics are computed)
- **Biobase** (Loaded to handle gene expression data structures in the CMScaller workflow)

## Examples

```
library(Biobase); library(CMScaller); res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE)
```

## Evaluation signals

- Overall prediction accuracy on the TCGA pCRC test set reproduces 0.83 (95% confidence interval 0.74–0.9)
- Class-wise metrics fall within reported values, e.g. CMS3 sensitivity 0.71 and CMS4 specificity 0.91
- Every sample in the n = 143 test set receives exactly one CMS1–CMS4 prediction with no missing calls
- Expression matrix was centered and scaled before NTP prediction, as required by CMScaller
- Metrics file cms_ntp_validation_metrics.csv is produced with accuracy, CI, sensitivity and specificity columns

## Limitations

- Input gene expression data must be centered and scaled, which introduces prediction uncertainty for small datasets (fewer than approximately forty samples)
- Caution is warranted in highly selected datasets where molecular distributions deviate from pCRC cohorts, including metastatic samples
- Intra-tumor heterogeneity (immune/stromal infiltration and cancer-cell intrinsic) may explain prediction errors at class boundaries

## Evidence

- [abstract] CMScaller recapitulated subtypes: "This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total)."
- [readme] NTP core algorithm: [evidence span withheld — anchor status: not_found]
- [discussion] centering and scaling requirement: "input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty"
- [readme] quick-start TCGA prediction: "### CMS prediction of TCGA primary colorectal cancers
res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE)"
