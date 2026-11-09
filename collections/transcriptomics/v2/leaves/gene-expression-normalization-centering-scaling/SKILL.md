---
name: gene-expression-normalization-centering-scaling
description: Use when a gene expression matrix (e.g. GEO/ArrayExpress pre-clinical
  CRC datasets, TCGA pCRC, organoid or PDX matrices) is to be classified with CMScaller
  or its NTP algorithm, since input must be centered and scaled before prediction.
when_to_use_negative:
- Classifying single-cell RNA-sequencing data, which is exceedingly noisy and for
  which CMScaller is not optimized
- Classifying samples with a different human stromal component than primary CRCs,
  e.g. patient biopsies of metastatic CRC — CMScaller in its current implementation
  is not recommended for these
- Datasets with fewer than approximately forty samples, where centering/scaling inescapably
  introduces prediction uncertainty
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CMScaller
  role: Consumes the centered/scaled expression matrix for nearest-template prediction
    of CMS1-CMS4 classes; its RNAseq flag selects RNA-seq vs microarray input handling
  repo: github.com/Lothelab/CMScaller
- name: Biobase
  role: Loads and represents the gene expression data as an ExpressionSet prior to
    normalization and classification
- name: R
  role: Runtime environment in which the centering/scaling and CMScaller package functions
    are executed
- name: devtools
  role: Installs the CMScaller package from GitHub before normalization and classification
    can run
  repo: https://github.com/peterawe/CMScaller
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_002
  - task_008
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:11634a1d0ab5a28164d2996b392435cff975f6e389394c9f6b1133b5910ff66e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Biobase
  - CMScaller
  - R
  - devtools
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# gene-expression-normalization-centering-scaling

## Summary

Prepares gene expression matrices (RNA-seq or microarray) for CMScaller nearest-template prediction by loading them as Biobase ExpressionSet objects and centering and scaling the expression values, as CMScaller requires for classification of colorectal cancer pre-clinical models.

## When to use

Use when a gene expression matrix (e.g. GEO/ArrayExpress pre-clinical CRC datasets, TCGA pCRC, organoid or PDX matrices) is to be classified with CMScaller or its NTP algorithm, since input must be centered and scaled before prediction.

## When NOT to use

- Classifying single-cell RNA-sequencing data, which is exceedingly noisy and for which CMScaller is not optimized
- Classifying samples with a different human stromal component than primary CRCs, e.g. patient biopsies of metastatic CRC — CMScaller in its current implementation is not recommended for these
- Datasets with fewer than approximately forty samples, where centering/scaling inescapably introduces prediction uncertainty

## Inputs

- Pre-processed gene expression matrix (RNA-seq counts/TPM or microarray intensities) from GEO/ArrayExpress, TCGA pCRC, cell line, organoid, or PDX datasets
- Biobase ExpressionSet object wrapping the expression data

## Outputs

- Centered and scaled ExpressionSet suitable for CMScaller nearest-template prediction
- CMScaller-ready expression matrices feeding CMS1-CMS4 class assignment and CMSgsa/subCamera gene set analysis

## How to apply

Retrieve the pre-processed gene expression matrix for the target dataset (e.g. GSE59857, GSE76402, GSE35144, GSE64392, GSE74843, E-MTAB-991, Gao PDX) and load it into R as a Biobase ExpressionSet. Center and scale the expression matrix within dataset — CMScaller's nearest-template prediction (NTP, Hoshida 2010) requires this normalization, and the article explicitly states 'input gene expression data should be centered and scaled'. Decide between RNAseq=TRUE (e.g. crcTCGAsubset RNA-seq input) and microarray mode when calling CMScaller on the scaled matrix. Note that centering/scaling makes the prediction dataset-size dependent: small datasets (fewer than ~40 samples) inescapably introduce prediction uncertainty, so this is a genuine decision point for whether NTP calls can be trusted. Downstream, the scaled ExpressionSet feeds CMScaller classification, CMSgsa/Camera gene set analysis, or subDEG workflows.

## Related tools

- **CMScaller** (Consumes the centered/scaled expression matrix for nearest-template prediction of CMS1-CMS4 classes; its RNAseq flag selects RNA-seq vs microarray input handling) — github.com/Lothelab/CMScaller
- **Biobase** (Loads and represents the gene expression data as an ExpressionSet prior to normalization and classification)
- **R** (Runtime environment in which the centering/scaling and CMScaller package functions are executed)
- **devtools** (Installs the CMScaller package from GitHub before normalization and classification can run) — https://github.com/peterawe/CMScaller

## Examples

```
library(Biobase); es <- ExpressionSet(emat_scaled); res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE)
```

## Evaluation signals

- The expression matrix has been centered (per-gene mean ~0) and scaled (per-gene SD ~1) within dataset before any CMScaller call
- Data are carried in a Biobase ExpressionSet, matching the package's Bioconductor-based design
- The RNAseq flag passed to CMScaller matches the technology of the input matrix (e.g. RNAseq=TRUE for TCGA RNA-seq)
- Downstream CMS predictions are present across all four subtypes — e.g. all 551 pre-clinical samples were classified with CMS4 at 7.5-22% per dataset — indicating normalization did not suppress subtypes
- No unclassified/CMS4-starved failure mode appears that would indicate unscaled or stromal-contaminated input (the classifyCMS.RF default run left 219/551 unclassified and only 3 CMS4)

## Limitations

- Input gene expression data must be centered and scaled, which introduces prediction uncertainty for small datasets (fewer than approximately forty samples)
- CMScaller is not recommended for samples with a different human stromal component than primary CRCs, e.g. metastatic CRC biopsies
- Caution is warranted for highly selected datasets where molecular distributions deviate from pCRC cohorts, including metastatic samples
- CMScaller is not optimized for single-cell RNA-sequencing data, which is exceedingly noisy

## Evidence

- [discussion] Normalization requirement from discussion: "input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty"
- [readme] Quick start showing RNA-seq input to CMScaller: "res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE)"
- [abstract] Downstream capability enabled by the normalized input: "This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total)."
