---
name: htseq-cds-read-counting
description: Use when RiboSeq BAM alignments (e.g. 25-35 nt length-selected files
  like Wild_Type.1.25-35.bam) and a GFF3 annotation are available and per-gene counts
  over annotated CDS features are needed for DESeq2 differential translation analysis.
when_to_use_negative:
- Input is already a count matrix or feature table — counting should only run on sorted/indexed
  BAM alignments
- The analysis needs differential expression results themselves; this skill only produces
  the count matrix that DESeq2 subsequently consumes
- Counts are desired over non-CDS features (e.g. UTRs or transcripts), since RiboDoc
  restricts htseq-count to CDS-type features defined in the config
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: HTSeq
  role: htseq-count assigns reads overlapping annotated CDS features using intersection-strict
    mode to produce per-sample count files
- name: samtools
  role: sorts and indexes the BAM alignments so they are suitable input for htseq-count
- name: RiboDoc
  role: orchestrates the counting step within its Snakemake/Docker pipeline and merges
    counts into the count matrix feeding DESeq2
  repo: https://github.com/equipeGST/RiboDoc
- name: DESeq2
  role: 'downstream consumer: normalizes the merged CDS count matrix and performs
    the differential expression analysis'
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:06a1e9ebcce34603009c48969952b81adfc16ff84fc41d23fbce73b775c449a6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/equipeGST/RiboDoc
  repo_url: https://github.com/equipeGST/RiboDoc
  tools:
  - Bowtie 2
  - Cutadapt
  - DESeq2
  - FastQC
  - HISAT2
  - HTSeq
  - RiboDoc
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# htseq-cds-read-counting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate per-gene ribosome-footprint read counts from length-selected RiboSeq BAM alignments by running htseq-count in intersection-strict mode over CDS features of a GFF annotation, then merging per-sample outputs into the count matrix consumed by DESeq2 in the RiboDoc pipeline.

## When to use

Use when RiboSeq BAM alignments (e.g. 25-35 nt length-selected files like Wild_Type.1.25-35.bam) and a GFF3 annotation are available and per-gene counts over annotated CDS features are needed for DESeq2 differential translation analysis.

## When NOT to use

- Input is already a count matrix or feature table — counting should only run on sorted/indexed BAM alignments
- The analysis needs differential expression results themselves; this skill only produces the count matrix that DESeq2 subsequently consumes
- Counts are desired over non-CDS features (e.g. UTRs or transcripts), since RiboDoc restricts htseq-count to CDS-type features defined in the config

## Inputs

- Length-selected per-sample BAM alignments (e.g. Wild_Type.1.25-35.bam) from the BAM.25-35 folder
- GFF3 annotation of the reference organism (e.g. Saccharomyces cerevisiae R64-1-1 from Ensembl)
- Project config.yaml specifying the gff_cds_feature vocabulary and sample/condition naming

## Outputs

- Merged count matrix at DESeq2_CDS.25-35/count_matrix_by_transcript_IDs.csv, one column per sample, header containing sample names
- Raw per-sample htseq-count outputs under HTSeq-counts/htseqcount_CDS

## How to apply

Take the length-filtered per-sample BAM files from the BAM.25-35 folder (produced after FastQC QC, Cutadapt trimming with 25-35 nt read-length selection, Bowtie2 contaminant/rRNA depletion, and joint HISAT2/Bowtie2 genome-transcriptome alignment) and the organism GFF annotation (e.g. S. cerevisiae R64-1-1 GFF3 from Ensembl). First sort and index the alignments with samtools, then run htseq-count (v0.12.4 in RiboDoc) on each BAM restricting counting to CDS-type features as defined by the project config's gff_cds_feature vocabulary, using the 'intersection-strict' overlap-resolution mode so reads overlapping annotated CDS regions are assigned to those features, grouped by the parent gene/transcript identifier. Merge the per-sample htseq-count outputs into a single matrix indexed by transcript/gene ID, using sample names parsed from the FASTQ file names (e.g. Wild_Type.1, Mutant.2) as the column header, and write it to DESeq2_CDS.25-35/count_matrix_by_transcript_IDs.csv while storing raw per-sample outputs under HTSeq-counts/htseqcount_CDS. Rationale: RiboDoc standardizes this counting step so the resulting matrix feeds directly into DESeq2 normalization and differential expression with the configured p-val and logFC thresholds.

## Related tools

- **HTSeq** (htseq-count assigns reads overlapping annotated CDS features using intersection-strict mode to produce per-sample count files)
- **samtools** (sorts and indexes the BAM alignments so they are suitable input for htseq-count)
- **RiboDoc** (orchestrates the counting step within its Snakemake/Docker pipeline and merges counts into the count matrix feeding DESeq2) — https://github.com/equipeGST/RiboDoc
- **DESeq2** (downstream consumer: normalizes the merged CDS count matrix and performs the differential expression analysis)

## Examples

```
htseq-count -f BAM -t CDS -m intersection-strict Wild_Type.1.25-35.bam Saccharomyces_cerevisiae.R64-1-1.gff3
```

## Evaluation signals

- DESeq2_CDS.25-35/count_matrix_by_transcript_IDs.csv exists and its columns match the number and names of input samples (header contains sample names such as Wild_Type.1, Mutant.2)
- Per-sample counts in the merged matrix are consistent with the raw per-sample htseq-count outputs under HTSeq-counts/htseqcount_CDS
- Counts are restricted to CDS-type features as defined by the config's gff_cds_feature vocabulary, not all GFF features
- Counts derive from the 25-35 nt read-length selection (RiboDoc default, since RPF are usually around 30 nucleotides long)
- Counting was run in intersection-strict mode so reads overlapping CDS features are assigned to those features

## Limitations

- RiboDoc requires a precise project-folder architecture (fastq/ and database/ subfolders plus config.yaml); subfolder names must not contain uppercase letters and FASTQ file names must follow the [CONDITION]_[NAME].[REPLICATE].fastq.gz pattern
- Only CDS-type features are counted; non-CDS regions are excluded by design
- The read-length selection defaults to 25-35 nt; reads outside this window are filtered upstream and therefore never counted

## Evidence
