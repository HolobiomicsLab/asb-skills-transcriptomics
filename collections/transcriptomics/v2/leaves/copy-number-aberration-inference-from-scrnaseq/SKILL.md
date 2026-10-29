---
name: copy-number-aberration-inference-from-scrnaseq
description: Use when malignant identity of a scRNA-seq subcluster must be validated
  via CNV inference from a gene-expression count table (10X 5'/3'-tagged or full-transcript
  data) without a matched normal control, e.g. after SCISSORS reclustering of epithelial
  tumor clusters.
when_to_use_negative:
- Input is bulk RNA-seq or DNA-seq rather than single-cell expression data — CONICS/CONICSmat
  are designed for single-cell CNV calling
- The goal is subtype scoring rather than malignancy validation — use signature-based
  tools (VAM, DECODER) instead; CONICSmat adds CNV evidence but does not assign basal-like/classical
  labels
- No CNV segment definitions and no control dataset are available and the BAM-based
  CONICS workflow cannot be configured (Python, Perl, samtools, bedtools >2.2.5 requirements
  unmet)
edam_operation: http://edamontology.org/operation_3224
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_2648
tools:
- name: CONICSmat
  role: 'Core tool: infers per-cell CNV presence/absence directly from the scRNA-seq
    count table to confirm malignant identity of the reclustered subcluster'
  repo: https://github.com/Neurosurgery-Brain-Tumor-Center-DiazLab/CONICS
- name: SCISSORS
  role: 'Upstream: multi-step semi-supervised reclustering that resolves the rare
    subcluster (e.g. 148-cell basal-like tumor cluster) on which CONICSmat is run'
  repo: https://github.com/jr-leary7/SCISSORS
- name: Seurat
  role: Provides the single-cell object and count/normalized expression matrix passed
    to CONICSmat
  repo: https://github.com/jr-leary7/SCISSORS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad449
    title: <u>S</u>ub-<u>C</u>luster <u>I</u>dentification through <u>S</u>emi-<u>S</u>upervised
      <u>O</u>ptimization of <u>R</u>are-Cell <u>S</u>ilhouettes (SCISSORS) in single-cell
      RNA-sequencing
  source_content_hash: sha256:ae896203ec0878ae76d4c0886e25df384ffcb4fcf13ef734a12f68e2b0bb6840
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Neurosurgery-Brain-Tumor-Center-DiazLab/CONICS
  tools:
  - CONICSmat
  - R
  - SCISSORS
  - Seurat
  - dplyr
  - ggplot2
  - magrittr
  edam_operation: http://edamontology.org/operation_3224
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_2648
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# copy-number-aberration-inference-from-scrnaseq

## Summary

Infers per-cell copy-number aberrations from single-cell RNA-seq count tables using CONICSmat, providing an orthogonal line of evidence (large-scale Amp/Del events) that a reclustered subcluster — e.g. a rare basal-like PDAC tumor population recovered by SCISSORS — is genuinely malignant.

## When to use

Use when malignant identity of a scRNA-seq subcluster must be validated via CNV inference from a gene-expression count table (10X 5'/3'-tagged or full-transcript data) without a matched normal control, e.g. after SCISSORS reclustering of epithelial tumor clusters.

## When NOT to use

- Input is bulk RNA-seq or DNA-seq rather than single-cell expression data — CONICS/CONICSmat are designed for single-cell CNV calling
- The goal is subtype scoring rather than malignancy validation — use signature-based tools (VAM, DECODER) instead; CONICSmat adds CNV evidence but does not assign basal-like/classical labels
- No CNV segment definitions and no control dataset are available and the BAM-based CONICS workflow cannot be configured (Python, Perl, samtools, bedtools >2.2.5 requirements unmet)

## Inputs

- Single-cell scRNA-seq gene expression count table for the reclustered cells (CONICSmat path)
- Aligned scRNA-seq BAM files for tumor cells and a matched/control normal dataset (advanced CONICS path)
- BED file of genomic coordinates of candidate CNV segments, tab-delimited with labels Amp/Del
- VCF files from exome-seq of control and tumor (optional, for B-allele frequency integration)

## Outputs

- incidenceMatrix.csv — presence/absence matrix of all CNVs per individual cell
- Read-count distribution violin plots per CNV segment
- Hierarchical clustering (dendrogram) of single cells by CNV status
- Estimated false discovery rate (cross-validation or empirical testing outputs)

## How to apply

Take the normalized single-cell gene expression count table for the reclustered population of interest (e.g. SCISSORS-reclustered epithelial cells from the Elyada et al. human PDAC dataset) and run CONICSmat, which infers CNVs directly from a gene expression table without an explicit normal control dataset. CONICSmat accepts both full-transcript (Fluidigm C1) and 5'/3'-tagged (10X Genomics) data, so it is compatible with standard Seurat/SCISSORS pipelines. For the advanced CONICS variant (aligned BAMs), supply a tumor BAM directory, a normal control BAM directory, and a BED file of CNV segments with the exact 4-column format 'chrom start end chrom:start:end:Amp|Del' (e.g. '7 19533 157408385 7:19533:157408385:Amp'), ensuring bedtools >2.2.5 for correct coverage calculation. Interpret the output incidenceMatrix.csv (per-cell presence/absence of each CNV), read-count violin plots per CNV segment, and hierarchical clustering of cells by CNV status: a bona fide malignant subcluster should cluster together and show strong, coherent CNV aberrations. Because false positives are possible when no explicit control is used, assess the false discovery rate via the cross-validation or empirical-testing workflows described in the CONICS documentation before declaring the subcluster malignant.

## Related tools

- **CONICSmat** (Core tool: infers per-cell CNV presence/absence directly from the scRNA-seq count table to confirm malignant identity of the reclustered subcluster) — https://github.com/Neurosurgery-Brain-Tumor-Center-DiazLab/CONICS
- **SCISSORS** (Upstream: multi-step semi-supervised reclustering that resolves the rare subcluster (e.g. 148-cell basal-like tumor cluster) on which CONICSmat is run) — https://github.com/jr-leary7/SCISSORS
- **Seurat** (Provides the single-cell object and count/normalized expression matrix passed to CONICSmat) — https://github.com/jr-leary7/SCISSORS

## Examples

```
bash run_CONICS.sh tumor_bams/ normal_control_bams/ cnv_segments.bed conics_out
```

## Evaluation signals

- incidenceMatrix.csv is produced with per-cell presence/absence calls for every CNV segment
- Hierarchical clustering of cells by CNV status places the candidate tumor subcluster in a distinct malignant branch
- Violin plots of read counts in CNV segments show shifted distributions consistent with called Amp/Del states
- False discovery rate is estimated via cross validation or empirical testing rather than left unquantified
- CNV-inferred malignant cells coincide with the rare subcluster recovered by SCISSORS (e.g. the 148-cell, 0.59%-of-dataset basal-like cluster in the Elyada PDAC data)

## Limitations

- The BAM-based CONICS workflow requires Python, Perl, samtools, bedtools >2.2.5 (bedtools >2.2.5 is explicitly required for correct coverage calculation) plus an explicit normal control dataset
- CONICSmat infers CNVs from expression alone, so false discoveries must be controlled via cross-validation or empirical FDR estimation
- CNV calls depend on the supplied genomic segment definitions; poorly chosen segments undermine the malignancy evidence

## Evidence

- [readme] CONICSmat is an R package that can be used to identify CNVs in single cell RNA-seq data from a gene expression table, without the need of an explicit normal control dataset.: [evidence span withheld — anchor status: not_found]
- [readme] CONICS works with either full transcript (e.g. Fluidigm C1) or 5'/3' tagged (e.g. 10X Genomics) data!: [evidence span withheld — anchor status: external_doc]
- [readme] 1. __incidenceMatrix.csv__: matrix of presence/absence for all CNVs, in individual cells: [evidence span withheld — anchor status: not_found]
- [readme] IMPORTANT: Bedtools >2.2.5 is needed in order to correctly calculate the coverage using CONICS.: [evidence span withheld — anchor status: external_doc]
- [abstract] SCISSORS is wrapped around the popular Seurat R package and can be easily integrated into existing Seurat pipelines: "SCISSORS is wrapped around the popular Seurat R package and can be easily integrated into existing Seurat pipelines"
