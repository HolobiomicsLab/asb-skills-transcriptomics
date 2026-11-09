---
name: rna-seq-adapter-quality-trimming
description: Use when raw RNA-seq FASTQ files (e.g., pESC/aESC reads retrieved from
  SRA per Supplemental Table S1) must be adapter- and quality-trimmed before alignment
  to GRCh38/hg38 with STAR and expression quantification with RSEM/DESeq2.
when_to_use_negative:
- Input is already processed RNA-seq data — CCIVR itself consumes processed expression
  tables (CSV with TPM, FPKM, fold_change, padj columns), not raw FASTQ, so trimming
  is unnecessary at that stage
- Input is a gene-annotation CSV (id, Chr, Strand, Start, End) intended for direct
  CCIVR cis-NAT extraction rather than read processing
- Reads have already been aligned (e.g., BAM from STAR); re-trimming aligned alignments
  is out of scope
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3169
tools:
- name: Trim Galore!
  role: Trims adapters and low-quality bases from raw RNA-seq FASTQ reads as the first
    step before STAR alignment and RSEM/DESeq2 analysis
- name: STAR
  role: Consumes the trimmed reads for splice-aware alignment to the GRCh38/hg38 human
    reference
- name: RSEM
  role: Quantifies gene-level TPM expression from the trimmed-and-aligned reads
- name: DESeq2
  role: Performs statistical analysis on gene counts from trimmed reads to call DEGs
    at padj < 0.05 with twofold difference
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-19782-5
    title: CCIVR facilitates comprehensive identification of cis-natural antisense
      transcripts with their structural characteristics and expression profiles
  source_content_hash: sha256:c997ac38c799d5f90e1e602290223e459b4ce3369df5a47d41f14aaec9c85f8f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CCIVR
  - DESeq2
  - RSEM
  - STAR
  - Trim Galore!
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# rna-seq-adapter-quality-trimming

## Summary

Cleans raw RNA-seq FASTQ reads by removing adapter sequences and low-quality bases with Trim Galore! before splice-aware alignment, so that downstream STAR mapping, RSEM quantification, and DESeq2 differential analysis (e.g., pESC vs aESC DEG calling at padj < 0.05, twofold) rest on accurate read data.

## When to use

Use when raw RNA-seq FASTQ files (e.g., pESC/aESC reads retrieved from SRA per Supplemental Table S1) must be adapter- and quality-trimmed before alignment to GRCh38/hg38 with STAR and expression quantification with RSEM/DESeq2.

## When NOT to use

- Input is already processed RNA-seq data — CCIVR itself consumes processed expression tables (CSV with TPM, FPKM, fold_change, padj columns), not raw FASTQ, so trimming is unnecessary at that stage
- Input is a gene-annotation CSV (id, Chr, Strand, Start, End) intended for direct CCIVR cis-NAT extraction rather than read processing
- Reads have already been aligned (e.g., BAM from STAR); re-trimming aligned alignments is out of scope

## Inputs

- Raw RNA-seq FASTQ files (e.g., published pESC and aESC reads from SRA, accessions per Supplemental Table S1)
- SRA accession list linking samples to run identifiers

## Outputs

- Adapter- and quality-trimmed FASTQ files ready for STAR alignment to GRCh38/hg38
- Per-sample Trim Galore! trimming reports (adapter content and base-quality summaries)

## How to apply

Retrieve the raw RNA-seq FASTQ files (for the pESC vs aESC comparison, the published SRA accessions listed in Supplemental Table S1) and run Trim Galore! on the raw reads to trim adapters and low-quality bases. Trimming is the first step of the pipeline before aligning the trimmed reads to the GRCh38/hg38 human reference with STAR; its rationale is that the downstream tools (RSEM for expression profiling, DESeq2 for statistical analysis) require processed RNA-seq data, so untrimmed adapter/quality artifacts would corrupt mapping and TPM/fold_change/padj estimates. After trimming, proceed directly to STAR alignment using the GRCh38/hg38 genome and Ensembl gene annotation, then quantify with RSEM and call DEGs in DESeq2 at padj < 0.05 with more than a twofold difference. Judge success both at the trimming stage (per-sample Trim Galore! reports show adapter removal and improved base-quality profiles) and downstream (the DEG tables approximately reproduce the reported 1,661 DEGs in pESCs and 1,311 DEGs in aESCs and recover the imprinted KCNQ1/KCNQ1OT1 cis-NAT pair with expression data).

## Related tools

- **Trim Galore!** (Trims adapters and low-quality bases from raw RNA-seq FASTQ reads as the first step before STAR alignment and RSEM/DESeq2 analysis)
- **STAR** (Consumes the trimmed reads for splice-aware alignment to the GRCh38/hg38 human reference)
- **RSEM** (Quantifies gene-level TPM expression from the trimmed-and-aligned reads)
- **DESeq2** (Performs statistical analysis on gene counts from trimmed reads to call DEGs at padj < 0.05 with twofold difference)

## Evaluation signals

- Per-sample Trim Galore! trimming reports show adapter sequences were detected and removed and low-quality bases were clipped from read ends
- Trimmed FASTQ files align to GRCh38/hg38 with STAR without adapter-derived soft-clipping artifacts dominating the alignments
- Downstream RSEM/DESeq2 output CSV carries TPM, fold_change, and padj columns as required for CCIVR input formatting
- Differential analysis at padj < 0.05 with more than a twofold difference approximately reproduces the reported 1,661 DEGs in pESCs and 1,311 DEGs in aESCs
- The imprinted KCNQ1/KCNQ1OT1 cis-NAT pair is recovered in the final expression table

## Limitations

- Trimming parameters are not specified in the article, so exact adapter/quality cutoffs must be chosen by the practitioner and may affect downstream DEG counts
- The pipeline depends on third-party programs (STAR, RSEM, DESeq2) rather than a single integrated tool, so trimming quality must be validated indirectly via mapping and DEG reproduction
- Results depend on the accuracy of reference genome annotation (structure and strand direction) deposited in databases such as Ensembl

## Evidence

- [other] Trim adapters and low-quality bases from raw reads with Trim Galore!.: [evidence span withheld — anchor status: not_found]
- [other] Retrieve the published pESC and aESC RNA-seq FASTQ files from SRA using the accession list given in Supplemental Table S1: [evidence span withheld — anchor status: not_found]
- [discussion] it uses processed RNA-seq data using third-party programs such as STAR51 for mapping, RSEM32 for expression profiling, and DESeq252 for statistical analysis: "it uses processed RNA-seq data using third-party programs such as STAR51 for mapping, RSEM32 for expression profiling, and DESeq252 for statistical analysis"
- [readme] Attaching information for expression profiling obtained from RNA-seq analysis is also available during the CCIVR process by adding extra columns such as **"TPM"**: "Attaching information for expression profiling obtained from RNA-seq analysis is also available during the CCIVR process by adding extra columns such as **"TPM"**"
