---
name: splice-aware-genome-alignment
description: Use when raw or adapter-trimmed RNA-seq FASTQ reads must be mapped to
  a reference genome while correctly handling splice junctions across introns, prior
  to transcript quantification (RSEM) and DEG calling (DESeq2).
when_to_use_negative:
- The input is small RNA or other non-spliced sequencing data where splice-junction
  awareness is unnecessary and a standard short-read aligner suffices
- The analysis goal is de novo transcriptome assembly without a reference genome (as
  in NATpipe's de novo assembled transcriptomes), rather than genome-guided alignment
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3308
tools:
- name: STAR
  role: Splice-aware aligner used to map trimmed RNA-seq reads to the reference genome
    before RSEM/DESeq2 steps
- name: Trim Galore!
  role: Preprocessing step that trims adapters and low-quality bases from raw FASTQ
    reads before STAR alignment
- name: RSEM
  role: Consumes STAR alignments for expression profiling (TPM quantification)
- name: DESeq2
  role: Performs statistical analysis of gene counts downstream of alignment to call
    DEGs
- name: CCIVR
  role: Downstream program that uses the processed RNA-seq data (STAR/RSEM/DESeq2
    outputs) to identify cis-NAT pairs
  repo: github.com/CCIVR/ccivr
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-19782-5
    title: CCIVR facilitates comprehensive identification of cis-natural antisense
      transcripts with their structural characteristics and expression profiles
  source_content_hash: sha256:a46e84d4fa8f72d83653ae8b2145f3c1641c6fc53af106984b632789f6fc28fb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CCIVR
  - DESeq2
  - RSEM
  - STAR
  - Trim Galore!
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# splice-aware-genome-alignment

## Summary

Map trimmed RNA-seq reads to a reference genome with the STAR splice-aware aligner, as used in the CCIVR pipeline (e.g., pESC vs aESC reads against GRCh38/hg38) to produce alignments for downstream RSEM expression profiling.

## When to use

Use when raw or adapter-trimmed RNA-seq FASTQ reads must be mapped to a reference genome while correctly handling splice junctions across introns, prior to transcript quantification (RSEM) and DEG calling (DESeq2).

## When NOT to use

- The input is small RNA or other non-spliced sequencing data where splice-junction awareness is unnecessary and a standard short-read aligner suffices
- The analysis goal is de novo transcriptome assembly without a reference genome (as in NATpipe's de novo assembled transcriptomes), rather than genome-guided alignment

## Inputs

- Adapter- and quality-trimmed RNA-seq FASTQ reads (from Trim Galore!)
- Reference genome FASTA (e.g., GRCh38/hg38)
- Gene annotation (e.g., Ensembl GTF/annotation for GRCh38/hg38)

## Outputs

- Splice-aware genome alignments (BAM/SAM) of RNA-seq reads suitable for RSEM expression profiling

## How to apply

Retrieve the RNA-seq FASTQ files (e.g., published pESC and aESC reads listed by accession in Supplemental Table S1) and trim adapters and low-quality bases with Trim Galore!. Align the trimmed reads to the reference genome — GRCh38/hg38 with Ensembl gene annotation in the pESC/aESC workflow — using STAR, which is splice-aware and therefore handles reads spanning exon-exon junctions. STAR is used as the third-party mapping program feeding the CCIVR expression-profile analysis: its alignments are passed to RSEM for gene-level TPM quantification and then to DESeq2 for statistical comparison. Judge success by the alignment supporting downstream recovery of known results, e.g., reproducing 1,661 DEGs in pESCs and 1,311 DEGs in aESCs (padj < 0.05, twofold) and the imprinted KCNQ1/KCNQ1OT1 cis-NAT pair.

## Related tools

- **STAR** (Splice-aware aligner used to map trimmed RNA-seq reads to the reference genome before RSEM/DESeq2 steps)
- **Trim Galore!** (Preprocessing step that trims adapters and low-quality bases from raw FASTQ reads before STAR alignment)
- **RSEM** (Consumes STAR alignments for expression profiling (TPM quantification))
- **DESeq2** (Performs statistical analysis of gene counts downstream of alignment to call DEGs)
- **CCIVR** (Downstream program that uses the processed RNA-seq data (STAR/RSEM/DESeq2 outputs) to identify cis-NAT pairs) — github.com/CCIVR/ccivr

## Examples

```
STAR --genomeDir GRCh38_hg38_index --readFilesIn sample_trimmed_R1.fq.gz sample_trimmed_R2.fq.gz --readFilesCommand zcat --outFileNamePrefix sample_STAR_
```

## Evaluation signals

- Alignment rate and mapping statistics are plausible for the trimmed reads (no systematic unmapped/overtrimmed failures)
- Downstream RSEM TPM quantification succeeds and yields a CSV gene table with TPM, fold_change, and padj columns
- DESeq2 comparison of pESC vs aESC approximately reproduces the reported 1,661 DEGs in pESCs and 1,311 DEGs in aESCs (padj < 0.05, more than twofold)
- The imprinted KCNQ1/KCNQ1OT1 cis-NAT pair is recovered from the expression data
- Per-contrast DEG tables are formatted so the five annotation columns (id, Chr, Strand, Start, End) plus expression columns can serve as CCIVR input

## Limitations

- CCIVR scripts do not cover the full analysis pipeline; STAR (and RSEM/DESeq2) are third-party programs whose outputs must be processed before CCIVR input
- CCIVR's cis-NAT prediction depends on the accuracy of gene annotation (structure and strand direction) deposited in databases, so an incorrect or outdated reference annotation degrades downstream results

## Evidence

- [supplementary] Alignment of pESC/aESC reads to GRCh38/hg38 with STAR: [evidence span withheld — anchor status: not_found]
- [discussion] STAR as third-party mapping program: "it uses processed RNA-seq data using third-party programs such as STAR51 for mapping, RSEM32 for expression profiling, and DESeq252 for statistical analysis"
- [discussion] Stated limitation on annotation accuracy: "For predicting the composition of various cis-NATs by CCIVR, it depends on the accuracy of gene annotation including their structure and strand direction deposited in the databases."
- [supplementary] Trimming precedes alignment: [evidence span withheld — anchor status: not_found]
