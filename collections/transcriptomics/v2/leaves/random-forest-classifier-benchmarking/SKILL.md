---
name: random-forest-classifier-benchmarking
description: Use when gene-expression data from CRC pre-clinical models (cell lines,
  organoids, PDXs) needs CMS1-CMS4 subtype calls, and you must quantify how the original
  primary-CRC classifier's stromal/immune signal dependency leaves samples (notably
  CMS4) unclassified.
when_to_use_negative:
- Samples whose human stromal component differs from primary CRCs, e.g. metastatic
  CRC biopsies — CMScaller is not recommended there
- Datasets with fewer than approximately forty samples — centering/scaling inescapably
  introduces prediction uncertainty
- Single-cell RNA-seq data — it is exceedingly noisy and CMScaller is not optimized
  for it
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: CMScaller
  role: Cancer cell-intrinsic classifier (Nearest Template Prediction-based) that
    assigns CMS1-CMS4 to pre-clinical models lacking stromal/immune signal
  repo: https://github.com/Lothelab/CMScaller
- name: R
  role: Runtime environment; CMScaller is an R package built on Bioconductor
- name: Biobase
  role: Used to load expression matrices as ExpressionSet objects required for the
    workflow (library(Biobase))
- name: classifyCMS.RF
  role: Original RF-based CMS classifier for primary CRCs, run with default settings
    as the benchmarking comparator whose CMS4 failure is reproduced
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:0b001d7af8dae2eaf2a936e1698a8d28b9536dc4f64051daab92e75182e93485
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lothelab/CMScaller
  tools:
  - Biobase
  - CMScaller
  - R
  - classifyCMS.RF
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# random-forest-classifier-benchmarking

## Summary

Benchmark a tumor microenvironment-dependent CMS classifier (classifyCMS.RF) against the cancer cell-intrinsic CMScaller R package to reproduce its failure mode (219 unclassified, only 3 CMS4 of 551) and demonstrate recovery of all four CMS groups across CRC cell lines, organoids and PDXs.

## When to use

Use when gene-expression data from CRC pre-clinical models (cell lines, organoids, PDXs) needs CMS1-CMS4 subtype calls, and you must quantify how the original primary-CRC classifier's stromal/immune signal dependency leaves samples (notably CMS4) unclassified.

## When NOT to use

- Samples whose human stromal component differs from primary CRCs, e.g. metastatic CRC biopsies — CMScaller is not recommended there
- Datasets with fewer than approximately forty samples — centering/scaling inescapably introduces prediction uncertainty
- Single-cell RNA-seq data — it is exceedingly noisy and CMScaller is not optimized for it

## Inputs

- Pre-processed gene expression matrices for seven public pre-clinical CRC datasets (GSE59857, GSE76402, GSE35144, GSE64392, GSE74843, E-MTAB-991, Gao PDX)
- ExpressionSet object (via Biobase) of centered, scaled gene expression per dataset
- classifyCMS.RF classifier (Guinney 2015 RF implementation) for the failure-mode comparator run

## Outputs

- Per-sample CMS1-CMS4 predictions for 551 in vitro/in vivo models from CMScaller
- Per-dataset tabulation of CMS class fractions (CMS4 expected in 7.5-22% range per dataset)
- classifyCMS.RF comparator counts: 219 unclassified, 245 CMS2, 3 CMS4 of 551 samples

## How to apply

Retrieve the pre-processed expression matrices for the seven public pre-clinical datasets (GSE59857, GSE76402, GSE35144, GSE64392, GSE74843, E-MTAB-991, Gao PDX) from GEO/ArrayExpress. Center and scale each matrix within dataset — CMScaller requires scaled input — and load it as an ExpressionSet via Biobase. Run CMScaller on all 551 samples to assign CMS1-CMS4 and tabulate per-dataset class fractions; CMS4 should account for 7.5-22% of samples in each dataset. As comparator, run classifyCMS.RF with default settings on the same inputs and record unclassified/CMS2/CMS4 counts. Acceptance: classifyCMS.RF reproduces the reported 219 unclassified / 245 CMS2 / 3 CMS4 out of 551, while CMScaller yields all four groups present in every dataset.

## Related tools

- **CMScaller** (Cancer cell-intrinsic classifier (Nearest Template Prediction-based) that assigns CMS1-CMS4 to pre-clinical models lacking stromal/immune signal) — https://github.com/Lothelab/CMScaller
- **R** (Runtime environment; CMScaller is an R package built on Bioconductor)
- **Biobase** (Used to load expression matrices as ExpressionSet objects required for the workflow (library(Biobase)))
- **classifyCMS.RF** (Original RF-based CMS classifier for primary CRCs, run with default settings as the benchmarking comparator whose CMS4 failure is reproduced)

## Examples

```
library(Biobase); library(CMScaller); res <- CMScaller(emat, RNAseq=TRUE, doPlot=TRUE)
```

## Evaluation signals

- classifyCMS.RF comparator run reproduces exactly 219 unclassified / 245 CMS2 / 3 CMS4 calls out of 551 pre-clinical samples
- CMScaller classifies all 551 samples with no unassigned calls, and all four CMS groups are present in every one of the seven datasets
- Per-dataset CMS4 fractions from CMScaller fall within the reported 7.5-22% range
- Expression matrices were centered and scaled within each dataset before prediction, as CMScaller requires
- Install from GitHub (devtools::install_github("Lothelab/CMScaller")) and the quick-start call on crcTCGAsubset runs without error

## Limitations

- CMScaller in its current implementation is not recommended for samples with a different human stromal component than primary CRCs, including metastatic CRC biopsies
- Input must be centered and scaled; small datasets (fewer than ~40 samples) inescapably introduce prediction uncertainty
- Caution is warranted in highly selected datasets whose molecular distributions deviate from primary CRC cohorts, including metastatic samples
- Not optimized for single-cell RNA-sequencing data, which is exceedingly noisy
- Intra-tumor heterogeneity (immune/stromal infiltration and cancer-cell intrinsic) may explain prediction errors at class boundaries

## Evidence

- [abstract] The original classifier's stromal dependency causes the CMS4 failure mode being benchmarked: "The currently available classifier is dependent on gene expression signals from the immune and stromal compartments of tumors and fails to identify the poor-prognostic CMS4-mesenchymal group"
- [abstract] CMScaller recovers all subtypes across the 551 pre-clinical models: "This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total)."
- [discussion] Input scaling requirement for the classifier: "input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty"
- [readme] Package scope of the classifier being benchmarked: "The `CMScaller` package provides Consensus Molecular Subtype (CMS) classification of colorectal cancer pre-clinical models"
