---
name: per-class-sensitivity-specificity-evaluation
description: Use when a CMS prediction (or any 4-class sample-subtype assignment)
  must be validated sample-by-sample against original reference labels on a held-out
  cohort and per-class sensitivity/specificity are required, not just a single accuracy
  figure.
when_to_use_negative:
- Input samples are metastatic CRC biopsies or otherwise have a different human stromal
  component than primary CRCs — CMScaller is not recommended for such samples in its
  current implementation
- Dataset contains fewer than approximately forty samples — the required centering/scaling
  inescapably introduces prediction uncertainty for small datasets
- Input is single-cell RNA-sequencing data, which is exceedingly noisy and not what
  the classifier was optimized for
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CMScaller
  role: Performs nearest template prediction (NTP) to assign CMS1–CMS4 labels whose
    agreement with original CMS labels is quantified by accuracy, sensitivity and
    specificity
  repo: https://github.com/Lothelab/CMScaller
- name: R
  role: Language in which CMScaller is distributed and in which accuracy, per-class
    sensitivity/specificity and the 95% confidence interval are computed
- name: Biobase
  role: Loaded to handle the expression data (ExpressionSet/matrix) fed to CMScaller
    for prediction
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:baa249fdda0d9d96fabe0230ce95c3e9975e2983818686a8066259f0a1376785
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lothelab/CMScaller
  tools:
  - Biobase
  - CMScaller
  - R
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# per-class-sensitivity-specificity-evaluation

## Summary

Evaluates a nearest-template-prediction CMS classifier against reference labels on a held-out test set (TCGA pCRC, n = 143) by computing overall accuracy with a 95% confidence interval plus class-wise sensitivity and specificity for CMS1–CMS4, mirroring the CMScaller benchmark (accuracy 0.83; CMS3 sensitivity 0.71; CMS4 specificity 0.91).

## When to use

Use when a CMS prediction (or any 4-class sample-subtype assignment) must be validated sample-by-sample against original reference labels on a held-out cohort and per-class sensitivity/specificity are required, not just a single accuracy figure.

## When NOT to use

- Input samples are metastatic CRC biopsies or otherwise have a different human stromal component than primary CRCs — CMScaller is not recommended for such samples in its current implementation
- Dataset contains fewer than approximately forty samples — the required centering/scaling inescapably introduces prediction uncertainty for small datasets
- Input is single-cell RNA-sequencing data, which is exceedingly noisy and not what the classifier was optimized for

## Inputs

- Held-out TCGA pCRC gene expression matrix (n = 143) with gene identifiers compatible with the CMScaller templates
- Original/reference CMS labels (CMS1–CMS4) for each test-set sample
- CMS template gene sets (templates.CMS shipped with CMScaller or output of the prior filtering task)

## Outputs

- Sample-wise table pairing NTP-predicted CMS class with the original CMS label
- Overall prediction accuracy with 95% confidence interval (expected ~0.83, CI 0.74–0.9 on TCGA pCRC)
- Class-wise sensitivity and specificity for CMS1–CMS4 (e.g. CMS3 sensitivity 0.71, CMS4 specificity 0.91)
- Metrics file (cms_ntp_validation_metrics.csv)

## How to apply

Load the held-out test-set gene expression matrix (e.g. TCGA primary colorectal cancers, n = 143) together with the original CMS labels. Center and scale the expression data, since CMScaller requires scaled input before nearest template prediction (NTP). Run NTP with CMScaller using either the filtered template gene sets from the prior task or the shipped templates.CMS as reference, passing RNAseq=TRUE for RNA-seq data. Compare the NTP-assigned CMS labels to the original labels sample by sample, then compute overall accuracy with a 95% confidence interval and class-wise sensitivity and specificity for CMS1–CMS4. Write the resulting metrics table (e.g. cms_ntp_validation_metrics.csv) so per-class values such as CMS3 sensitivity 0.71 and CMS4 specificity 0.91 can be compared to the published benchmark of 0.83 accuracy (95% CI 0.74–0.9).

## Related tools

- **CMScaller** (Performs nearest template prediction (NTP) to assign CMS1–CMS4 labels whose agreement with original CMS labels is quantified by accuracy, sensitivity and specificity) — https://github.com/Lothelab/CMScaller
- **R** (Language in which CMScaller is distributed and in which accuracy, per-class sensitivity/specificity and the 95% confidence interval are computed)
- **Biobase** (Loaded to handle the expression data (ExpressionSet/matrix) fed to CMScaller for prediction)

## Examples

```
library(Biobase); library(CMScaller); res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE); mean(res$prediction == original_cms_labels)
```

## Evaluation signals

- Overall accuracy on the TCGA pCRC test set reproduces the published value of 0.83 (95% CI 0.74–0.9) when original CMS labels are the reference
- Per-class metrics fall in the reported ranges, e.g. CMS3 sensitivity 0.71 and CMS4 specificity 0.91
- Every test sample has exactly one predicted class in {CMS1, CMS2, CMS3, CMS4} and is paired 1:1 with a reference label before metric computation
- Expression matrix was centered and scaled before prediction (prediction run without scaling is a protocol violation)
- Metrics table written as cms_ntp_validation_metrics.csv contains columns for overall accuracy, 95% CI, and per-class sensitivity and specificity

## Limitations

- CMScaller in its current implementation is not recommended for samples with a different human stromal component than primary CRCs, including metastatic CRC biopsies
- Required centering and scaling of input gene expression introduces prediction uncertainty for small datasets (fewer than approximately forty samples)
- Caution is warranted in highly selected datasets where molecular distributions deviate from pCRC cohorts
- Prediction errors at class boundaries may be explained by intra-tumor heterogeneity, from immune/stromal infiltration and intrinsic to the cancer cells
- Classifier is not optimized for single-cell RNA-sequencing data, which is exceedingly noisy

## Evidence

- [results] Using the original CMS classifications as reference for TCGA pCRCs, CMScaller achieved an overall prediction accuracy of 0.83 (95% confidence interval 0.74–0.9).: [evidence span withheld — anchor status: not_found]
- [discussion] input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty: "input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty"
- [readme] The core algorithm is *[Nearest Template Prediction (NTP)](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0015543)* algorithm proposed by Yujin Hoshida: "The core algorithm is *[Nearest Template Prediction (NTP)](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0015543)* algorithm proposed by Yujin Hoshida [Hoshida 2010]."
- [discussion] CMScaller in its current implementation is not recommended for use with samples with a different human stromal component than primary CRCs, including e.g. patient biopsies of metastatic CRC.: "CMScaller in its current implementation is not recommended for use with samples with a different human stromal component than primary CRCs, including e.g. patient biopsies of metastatic CRC."
