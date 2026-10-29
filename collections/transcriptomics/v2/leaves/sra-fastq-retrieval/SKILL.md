---
name: sra-fastq-retrieval
description: Use when reproducing a published RNA-seq analysis and raw reads must
  be fetched from SRA before trimming, alignment (STAR), and quantification (RSEM/DESeq2);
  triggered when a supplemental accession table lists the samples.
when_to_use_negative:
- The processed read alignments or gene-level count/TPM tables are already available,
  making raw FASTQ retrieval unnecessary
- The target data are genome annotations rather than sequencing reads — obtain those
  from Ensembl (GRCh38/hg38) instead of SRA
edam_operation: http://edamontology.org/operation_0227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: Trim Galore!
  role: consumes the retrieved FASTQ files in the next pipeline step (adapter and
    low-quality base trimming)
- name: STAR
  role: aligns the retrieved FASTQ reads to the GRCh38/hg38 human reference after
    trimming
- name: RSEM
  role: quantifies gene-level TPM expression from the STAR alignments derived from
    the retrieved reads
- name: DESeq2
  role: performs statistical comparison of pESC vs aESC counts from data originating
    in the retrieved FASTQ files
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-19782-5
    title: CCIVR facilitates comprehensive identification of cis-natural antisense
      transcripts with their structural characteristics and expression profiles
  source_content_hash: sha256:94ed1a405fd62ec6901e41d3190a7273c4cab5be41822a2d10e647d014abdcfb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CCIVR
  - DESeq2
  - RSEM
  - STAR
  - Trim Galore!
  edam_operation: http://edamontology.org/operation_0227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 42
---

# sra-fastq-retrieval

## Summary

Retrieve published RNA-seq FASTQ files from the SRA using the per-sample accession list compiled in the study's Supplemental Table S1, as the first step of reproducing the pESC vs aESC differential expression analysis with the GRCh38/hg38 reference.

## When to use

Use when reproducing a published RNA-seq analysis and raw reads must be fetched from SRA before trimming, alignment (STAR), and quantification (RSEM/DESeq2); triggered when a supplemental accession table lists the samples.

## When NOT to use

- The processed read alignments or gene-level count/TPM tables are already available, making raw FASTQ retrieval unnecessary
- The target data are genome annotations rather than sequencing reads — obtain those from Ensembl (GRCh38/hg38) instead of SRA

## Inputs

- Supplemental Table S1 accession list mapping pESC and aESC samples to SRA accessions
- NCBI SRA database (run/experiment accessions for the published RNA-seq samples)

## Outputs

- Raw pESC and aESC RNA-seq FASTQ files organized per sample/accession
- Accession-to-sample mapping retained for downstream trimming, STAR alignment, RSEM quantification, and DESeq2 analysis

## How to apply

Locate the sample-level accession identifiers in Supplemental Table S1 of the pESC/aESC RNA-seq study and use them to download the raw paired-end FASTQ files from the NCBI SRA archive (e.g., via prefetch/fasterq-dump from the sra-tools toolkit). Confirm each downloaded FASTQ corresponds to the correct parthenogenetic or androgenetic ESC sample before proceeding. These raw FASTQ files are the required entry point for the downstream pipeline: adapter/quality trimming with Trim Galore!, splice-aware alignment to GRCh38/hg38 with STAR, RSEM-based TPM quantification, and DESeq2 DEG calling at padj < 0.05 with twofold difference. Keep the original accession-to-sample mapping so the final DEG counts (1,661 in pESCs, 1,311 in aESCs) can be validated against the published result.

## Related tools

- **Trim Galore!** (consumes the retrieved FASTQ files in the next pipeline step (adapter and low-quality base trimming))
- **STAR** (aligns the retrieved FASTQ reads to the GRCh38/hg38 human reference after trimming)
- **RSEM** (quantifies gene-level TPM expression from the STAR alignments derived from the retrieved reads)
- **DESeq2** (performs statistical comparison of pESC vs aESC counts from data originating in the retrieved FASTQ files)

## Examples

```
prefetch $(cat S1_accessions.txt) && fasterq-dump --split-files $(cat S1_accessions.txt)
```

## Evaluation signals

- Each accession in Supplemental Table S1 resolves to a downloaded FASTQ file with no missing or unmatched samples
- Read count and quality (e.g., per-base quality scores, adapter content via FastQC) are consistent with expected raw RNA-seq data before trimming
- After the full pipeline, DEG counts approximately reproduce the reported 1,661 DEGs in pESCs and 1,311 DEGs in aESCs (padj < 0.05, twofold)
- The imprinted KCNQ1/KCNQ1OT1 cis-NAT pair is recovered with expression data, confirming correct sample identity and strand-aware processing
- Trim Galore! trimming logs show removal of adapters and low-quality bases without unexpectedly large read loss

## Limitations

- The workflow depends on the accession list in Supplemental Table S1; if entries are missing, ambiguous, or superseded in SRA, sample mapping must be resolved manually
- The downstream CCIVR pipeline relies on third-party programs (STAR, RSEM, DESeq2) and on the accuracy of gene annotations (structure and strand direction) in the databases, so errors in reference/annotation choice propagate regardless of correct FASTQ retrieval

## Evidence

- [supplementary] Retrieve the published pESC and aESC RNA-seq FASTQ files from SRA using the accession list given in Supplemental Table S1, along with the GRCh38/hg38 reference genome and Ensembl gene annotation.: [evidence span withheld — anchor status: not_found]
- [supplementary] Trim adapters and low-quality bases from raw reads with Trim Galore!.: [evidence span withheld — anchor status: not_found]
- [supplementary] confirm the DEG counts approximately reproduce the reported 1,661 DEGs in pESCs and 1,311 DEGs in aESCs (padj < 0.05, twofold): [evidence span withheld — anchor status: not_found]
- [abstract] CCIVR analysis with RNA-seq data from parthenogenetic and androgenetic embryonic stem cells identified the imprinted cis-NAT pair KCNQ1/KCNQ1OT1: "CCIVR analysis with RNA-seq data from parthenogenetic and androgenetic embryonic stem cells identified well-known imprinted cis-NAT pair, KCNQ1/KCNQ1OT1"
- [discussion] it uses processed RNA-seq data using third-party programs such as STAR51 for mapping, RSEM32 for expression profiling, and DESeq252 for statistical analysis: "it uses processed RNA-seq data using third-party programs such as STAR51 for mapping, RSEM32 for expression profiling, and DESeq252 for statistical analysis"
