---
name: pairwise-subtype-concordance-analysis
description: Use when you need to quantify how sensitive CMS subtype assignments (from
  CMScaller nearest-template prediction) are to template-development choices such
  as marker-selection p-value cutoffs, cell line panels, LFC thresholds, or gene-set
  composition.
when_to_use_negative:
- Samples with a different human stromal component than primary CRCs (e.g. metastatic
  CRC biopsies) — CMScaller is explicitly not recommended for these
- Single-cell RNA-sequencing data — the data are exceedingly noisy and CMScaller is
  not optimized for them
- Very small datasets (fewer than ~40 samples) where required centering and scaling
  inescapably introduces prediction uncertainty, if the goal is a definitive classification
  rather than a concordance sensitivity check
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: CMScaller
  role: Provides the nearest-template-prediction CMS classification applied with each
    of the five templates
  repo: github.com/Lothelab/CMScaller
- name: R
  role: Environment in which templates are built, samples classified, pairwise concordance
    computed, and the report written
- name: Biobase
  role: Used to load the target gene expression dataset into an ExpressionSet prior
    to classification
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:cf1e353f93b4f2d8b825ba261395851149e4cb971610cb5e525edf44fdbbf416
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Biobase
  - CMScaller
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# pairwise-subtype-concordance-analysis

## Summary

Sensitivity analysis of CMScaller CMS classification: build a reference nearest-template-prediction template plus four single-parameter-altered alternative templates, classify the same expression dataset with all five, and compute pairwise concordance and consistent-assignment counts across templates.

## When to use

Use when you need to quantify how sensitive CMS subtype assignments (from CMScaller nearest-template prediction) are to template-development choices such as marker-selection p-value cutoffs, cell line panels, LFC thresholds, or gene-set composition.

## When NOT to use

- Samples with a different human stromal component than primary CRCs (e.g. metastatic CRC biopsies) — CMScaller is explicitly not recommended for these
- Single-cell RNA-sequencing data — the data are exceedingly noisy and CMScaller is not optimized for them
- Very small datasets (fewer than ~40 samples) where required centering and scaling inescapably introduces prediction uncertainty, if the goal is a definitive classification rather than a concordance sensitivity check

## Inputs

- Gene expression matrix for the target cohort (e.g. Gao et al. PDX models, 50 samples) loadable into a Biobase ExpressionSet
- Reference CMScaller nearest-template-prediction template
- Four alternative CMS prediction templates (altered p-value cutoff 1e-4, alternative cell line panel, LFC threshold 1, genes lost between Isella et al. panel and primary template removed)

## Outputs

- cms_concordance_report.csv containing pairwise concordance values for the five templates (values in [0, 1]) and a non-negative count of samples consistently assigned to a single CMS
- Per-template CMS assignment vectors for all samples

## How to apply

In R with CMScaller, build the reference CMS prediction template and four alternative templates, varying exactly one parameter per alternative: a stricter adjusted p-value cutoff of 1e-4 for marker selection, an alternative cell line panel for template derivation, an LFC threshold of 1, and removal of genes lost between the Isella et al. panel and the primary template. Load the target dataset (e.g. the Gao et al. PDX gene expression matrix) into an ExpressionSet via Biobase, and center and scale the expression matrix as required for nearest-template prediction. Classify the samples with each of the five templates using CMScaller's NTP-based prediction, then compute pairwise classification concordance across the five sets of CMS assignments. Count samples assigned to the same CMS by all templates and write the pairwise concordance table and consistent-assignment counts to cms_concordance_report.csv. In the source study, the five classifications of 50 Gao et al. PDX models showed concordance of 0.95–0.98, with 47/50 samples (94%) consistently assigned.

## Related tools

- **CMScaller** (Provides the nearest-template-prediction CMS classification applied with each of the five templates) — github.com/Lothelab/CMScaller
- **R** (Environment in which templates are built, samples classified, pairwise concordance computed, and the report written)
- **Biobase** (Used to load the target gene expression dataset into an ExpressionSet prior to classification)

## Examples

```
library(Biobase); library(CMScaller); res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE)
```

## Evaluation signals

- Output file cms_concordance_report.csv exists and contains pairwise concordance entries for all five templates
- All pairwise concordance values lie in the interval [0, 1]; in the source study expected values were 0.95–0.98
- Consistent-assignment count is non-negative and, for the Gao et al. PDX benchmark, expected to be 47/50 samples (94%)
- Each alternative template differs from the reference by exactly one parameter (p-value cutoff 1e-4, cell line panel, LFC threshold 1, or removed genes)
- Expression matrix is centered and scaled before nearest-template prediction, as required by CMScaller

## Limitations

- Input gene expression data must be centered and scaled, which introduces prediction uncertainty for small datasets (fewer than approximately forty samples)
- CMScaller is not recommended for samples with a different human stromal component than primary CRCs, e.g. metastatic CRC biopsies
- Caution is warranted for highly selected datasets where molecular distributions deviate from pCRC cohorts, including metastatic samples
- CMScaller is not optimized for single-cell RNA-sequencing data, which is exceedingly noisy
- The prediction template gene set is planned to be further improved using larger, higher-quality datasets as they become available

## Evidence

- [methods] NTP-based classification of five templates: [evidence span withheld — anchor status: not_found]
- [readme] Package context and install/usage: "The `CMScaller` package provides Consensus Molecular Subtype (CMS) classification of colorectal cancer pre-clinical models"
- [discussion] Requirement to center and scale input data: "input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty"
- [abstract] Recapitulation scope of the classifier: "This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total)."
- [discussion] Not recommended for non-primary-CRC stromal contexts: "CMScaller in its current implementation is not recommended for use with samples with a different human stromal component than primary CRCs"
