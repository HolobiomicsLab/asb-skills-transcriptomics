---
name: transcript-abundance-quantification-and-gene-level-import
description: Use when starting from raw bulk RNA-seq FASTQ files (e.g. anti-PD-1 vs
  control CD8 TIL samples from GSE62771) and gene-level counts or a differential expression
  ranking are needed for GSEA or signature enrichment.
when_to_use_negative:
- Input is already a gene-level count matrix — skip trimming/quantification and go
  directly to DESeq2
- Input is single-cell RNA-seq data requiring cell-state classification (use TILPRED
  or ProjecTILs reference projection instead)
- Samples are not mouse or no mouse transcriptome index is available, since the workflow
  specifies quasi-mapping against the mouse transcriptome
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Trimmomatic
  role: Trims adapters and low-quality bases from raw reads before quantification
- name: Salmon
  role: Quantifies transcript abundances per sample via quasi-mapping against the
    mouse transcriptome
  repo: https://github.com/COMBINE-lab/salmon/releases
- name: tximport
  role: Aggregates transcript-level Salmon estimates to gene-level counts for DESeq2
- name: DESeq2
  role: Fits the differential expression model contrasting anti-PD-1 vs control CD8
    TILs
- name: clusterProfiler
  role: Runs GSEA with the EM-like CD8 TIL state signature as the query gene set on
    log2FC-ranked genes
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1080/2162402x.2020.1737369
    title: Deciphering the transcriptomic landscape of tumor-infiltrating CD8 lymphocytes
      in B16 melanoma tumors with single-cell RNA-Seq
  source_content_hash: sha256:5efa29da1fd516004711b3a22bd5974a6229df9c59cab452a54a2ff37a4e9e7e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/carmonalab/TILPRED
  tools:
  - BiocManager
  - DESeq2
  - R
  - Salmon
  - Trimmomatic
  - clusterProfiler
  - tximport
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Transcript abundance quantification and gene-level import

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A bulk RNA-seq pipeline that trims raw FASTQ reads, quantifies transcript abundances with Salmon quasi-mapping against the mouse transcriptome, and aggregates transcript-level estimates to gene-level counts with tximport for DESeq2 differential expression. It underpins downstream signature-enrichment tests such as the EM-like CD8 TIL GSEA on the Gubin et al. murine sarcoma dataset (GSE62771).

## When to use

Use when starting from raw bulk RNA-seq FASTQ files (e.g. anti-PD-1 vs control CD8 TIL samples from GSE62771) and gene-level counts or a differential expression ranking are needed for GSEA or signature enrichment.

## When NOT to use

- Input is already a gene-level count matrix — skip trimming/quantification and go directly to DESeq2
- Input is single-cell RNA-seq data requiring cell-state classification (use TILPRED or ProjecTILs reference projection instead)
- Samples are not mouse or no mouse transcriptome index is available, since the workflow specifies quasi-mapping against the mouse transcriptome

## Inputs

- Raw bulk RNA-seq FASTQ files (e.g. GSE62771 anti-PD-1 and control CD8 TIL samples from GEO/SRA)
- Sample metadata table mapping samples to condition (anti-PD-1 vs control)
- Mouse transcriptome index/reference for Salmon quasi-mapping
- Transcript-to-gene mapping (tx2gene) for tximport aggregation
- Query gene set: EM-like CD8 TIL state signature for clusterProfiler GSEA

## Outputs

- Per-sample Salmon quant files (transcript-level abundances: TPM and NumReads)
- Gene-level count matrix imported via tximport
- DESeq2 differential expression table (anti-PD-1 vs control, with log2 fold changes)
- GSEA enrichment statistics for the EM-like signature, written to gsea_emlike_enrichment.csv

## How to apply

Download the raw FASTQ files and sample metadata (anti-PD-1 vs control CD8 TIL samples) from GEO/SRA, then trim adapters and low-quality bases with Trimmomatic. Quantify transcript abundances per sample with Salmon in quasi-mapping mode against the mouse transcriptome, which avoids full alignments and outputs per-sample quant files. Aggregate transcript-level estimates to gene-level counts with tximport and feed them into a DESeq2 differential expression model contrasting anti-PD-1 vs control — tximport preserves the transcript-level uncertainty so DESeq2 can account for it. Rank genes by log2 fold change and run GSEA in clusterProfiler with the EM-like CD8 TIL state signature as the query gene set. Correctness is judged by reproducing the reported nominal enrichment (p = 0.035) of the EM-like signature in anti-PD-1-treated CD8 TILs and exporting the DE table plus GSEA statistics.

## Related tools

- **Trimmomatic** (Trims adapters and low-quality bases from raw reads before quantification)
- **Salmon** (Quantifies transcript abundances per sample via quasi-mapping against the mouse transcriptome) — https://github.com/COMBINE-lab/salmon/releases
- **tximport** (Aggregates transcript-level Salmon estimates to gene-level counts for DESeq2)
- **DESeq2** (Fits the differential expression model contrasting anti-PD-1 vs control CD8 TILs)
- **clusterProfiler** (Runs GSEA with the EM-like CD8 TIL state signature as the query gene set on log2FC-ranked genes)

## Examples

```
salmon quant -i mouse_index -l A -1 sample_R1.fastq.gz -2 sample_R2.fastq.gz -o quants/sample
```

## Evaluation signals

- EM-like CD8 TIL signature shows selective enrichment in anti-PD-1-treated CD8 TILs, reproducing the reported nominal p = 0.035
- DE table plus GSEA statistics are written to gsea_emlike_enrichment.csv
- Each Salmon quant directory contains complete quant.sf files for all samples in the metadata before tximport
- Gene-level matrix dimensions match the transcript-to-gene mapping after tximport aggregation
- Sample condition labels (anti-PD-1 vs control) in the DESeq2 design match the GEO/SRA metadata

## Limitations

- The enrichment result is a nominal p value (p = 0.035) from a single public dataset (Gubin et al. GSE62771), not a multiple-testing-corrected meta-analysis
- The workflow is species- and condition-specific (murine sarcoma, anti-PD-1 vs control); adapting to other organisms requires a different transcriptome index
- The EM-like signature enrichment is inferred from bulk RNA-seq of sorted CD8 TILs, so within-population state composition changes are indirect

## Evidence

- [methods] Quantify transcript abundances per sample with Salmon using quasi-mapping against the mouse transcriptome.: [evidence span withheld — anchor status: not_found]
- [methods] Aggregate transcript-level estimates to gene-level counts with tximport and fit a DESeq2 differential expression model (anti-PD-1 vs control).: [evidence span withheld — anchor status: not_found]
- [abstract] Checkpoint blockade led to a selective enrichment of the EM-like CD8 TIL gene signature in bulk RNA-seq of CD8 TILs following anti-PD-1 therapy (Gubin et al. dataset), with p = .035.: [evidence span withheld — anchor status: not_found]
- [abstract] Meta-analyses of public bulk and single-cell RNA-seq data suggested that anti-PD-1 treatment induces the expansion of EM-like cells: [evidence span withheld — anchor status: not_found]
