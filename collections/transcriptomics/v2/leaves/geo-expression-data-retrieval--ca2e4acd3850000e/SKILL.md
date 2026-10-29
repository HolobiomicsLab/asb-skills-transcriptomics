---
name: geo-expression-data-retrieval
description: Use when a workflow needs bulk gene expression data for public pre-clinical
  CRC datasets identified by GEO (GSE*) or ArrayExpress (E-MTAB-*) accessions, as
  input to downstream classification or subtyping.
when_to_use_negative:
- You need raw sequencing files (FASTQ/CEL) rather than the pre-processed expression
  matrices the original analysis used
- The target samples are primary CRC tumor biopsies or metastatic biopsies rather
  than in vitro/in vivo pre-clinical models (CMScaller is not recommended for samples
  with a different human stromal component than primary CRCs)
- Single-cell RNA-seq data, which CMScaller is not optimized for
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0204
tools:
- name: R
  role: Environment for downloading and loading the retrieved expression matrices
    into ExpressionSet objects with Biobase
  repo: http://www.R-project.org/
- name: Biobase
  role: Constructs ExpressionSet objects from the retrieved matrices as required by
    CMScaller
- name: CMScaller
  role: Downstream consumer of the retrieved datasets; matrices must be centered and
    scaled within dataset before its NTP-based prediction
  repo: github.com/Lothelab/CMScaller
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:98d3b18a5ecba26ce9267ba5e0d5fd9175e25ee5d4d81b006eb3f3c76cb24052
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Biobase
  - CMScaller
  - R
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# geo-expression-data-retrieval

## Summary

Retrieve pre-processed gene expression matrices for colorectal cancer pre-clinical models (cell lines, organoids, PDXs) from GEO/ArrayExpress accession IDs so they can be centered, scaled, and subtype-classified with CMScaller.

## When to use

Use when a workflow needs bulk gene expression data for public pre-clinical CRC datasets identified by GEO (GSE*) or ArrayExpress (E-MTAB-*) accessions, as input to downstream classification or subtyping.

## When NOT to use

- You need raw sequencing files (FASTQ/CEL) rather than the pre-processed expression matrices the original analysis used
- The target samples are primary CRC tumor biopsies or metastatic biopsies rather than in vitro/in vivo pre-clinical models (CMScaller is not recommended for samples with a different human stromal component than primary CRCs)
- Single-cell RNA-seq data, which CMScaller is not optimized for

## Inputs

- GEO/ArrayExpress accession IDs (GSE59857, GSE76402, GSE35144, GSE64392, GSE74843, E-MTAB-991, Gao PDX)
- Pre-processed gene expression matrices (sample x gene) from the public repositories

## Outputs

- Per-dataset gene expression matrices ready for within-dataset centering/scaling and ExpressionSet construction with Biobase

## How to apply

Identify the accession IDs for the target datasets — in this study the seven pre-clinical sets GSE59857, GSE76402, GSE35144, GSE64392, GSE74843, E-MTAB-991, and the Gao PDX dataset — and download the pre-processed gene expression matrices from GEO or ArrayExpress. Collect the matrices per dataset rather than merged, because CMScaller requires each expression matrix to be centered and scaled within dataset before prediction (loaded via Biobase into an ExpressionSet). Verify sample counts and annotation (gene identifiers, RNA-seq vs. microarray platform) per accession, since the platform type determines the RNAseq argument in downstream CMScaller calls. The retrieval is correct when each dataset yields a complete sample-by-gene matrix covering the models used in the 551-sample pre-clinical cohort.

## Related tools

- **R** (Environment for downloading and loading the retrieved expression matrices into ExpressionSet objects with Biobase) — http://www.R-project.org/
- **Biobase** (Constructs ExpressionSet objects from the retrieved matrices as required by CMScaller)
- **CMScaller** (Downstream consumer of the retrieved datasets; matrices must be centered and scaled within dataset before its NTP-based prediction) — github.com/Lothelab/CMScaller

## Examples

```
library(Biobase); eset <- ExpressionSet(assayData=as.matrix(scaled_emat)); res <- CMScaller(eset, RNAseq=TRUE, doPlot=TRUE)
```

## Evaluation signals

- All seven accessions resolve to complete sample-by-gene expression matrices summing to the 551 pre-clinical samples
- Each matrix is retrieved and processed per dataset (not merged), matching the within-dataset centering/scaling requirement of CMScaller
- Dataset-level CMS4 fractions after classification fall within the reported 7.5–22% range, indicating the retrieved data support the expected subtype distributions
- Platform type (RNA-seq vs. microarray) is recorded per accession so the correct RNAseq argument is used downstream

## Limitations

- Expression data must be centered and scaled, which introduces prediction uncertainty for small datasets (fewer than approximately forty samples)
- Caution is warranted for highly selected datasets where molecular distributions deviate from primary CRC cohorts, including metastatic samples

## Evidence

- [other] Retrieve the pre-processed gene expression matrices for the seven public pre-clinical datasets (GSE59857, GSE76402, GSE35144, GSE64392, GSE74843, E-MTAB-991, Gao PDX) from GEO/ArrayExpress.: [evidence span withheld — anchor status: not_found]
- [discussion] CMScaller is platform independent and available as an R package.: "CMScaller is platform independent and available as an R package."
- [discussion] Input gene expression data must be centered and scaled, which introduces prediction uncertainty for small datasets: "input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty"
- [abstract] CMScaller recapitulated subtypes in 551 in vitro and in vivo models: "This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total)."
- [readme] CMScaller predicts from ExpressionSet-style expression matrices via NTP: "res <- CMScaller(crcTCGAsubset, RNAseq=TRUE, doPlot=TRUE)"
