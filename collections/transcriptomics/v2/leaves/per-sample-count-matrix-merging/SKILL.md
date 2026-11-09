---
name: per-sample-count-matrix-merging
description: Use when per-sample RiboSeq alignments (BAM) exist and a single merged
  count matrix, one column per sample indexed by transcript/gene ID, is needed as
  input to DESeq2 differential expression analysis.
when_to_use_negative:
- The merged count matrix already exists and the goal is the downstream DESeq2 normalization/comparison
  step itself
- The data are still raw or trimmed FASTQ and have not yet been contaminant-depleted
  and aligned to BAM (run the upstream RiboDoc QC/trimming/alignment steps first)
- The goal is P-site offset estimation or trinucleotide periodicity analysis, which
  is handled by riboWaltz/TRiP, not read counting
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: HTSeq (htseq-count)
  role: Performs the per-sample read counting on the BAM files in intersection-strict
    mode restricted to CDS features (v0.12.4 in RiboDoc)
  repo: https://htseq.readthedocs.io/en/release_0.11.1/count.html
- name: RiboDoc
  role: Docker/Snakemake pipeline that orchestrates the htseq-count step and merges
    the per-sample outputs into the count matrix consumed by DESeq2
  repo: https://github.com/equipeGST/RiboDoc
- name: DESeq2
  role: Downstream consumer of the merged count matrix for normalization and differential
    gene expression analysis
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:fbbc2497186829e52f79bad5e937141049f5d8fa555a427fdbb48dc7eb04af22
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/equipeGST/RiboDoc
  tools:
  - DESeq2
  - HTSeq (htseq-count)
  - RiboDoc
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# per-sample-count-matrix-merging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the RiboDoc read-counting step in which htseq-count (v0.12.4) quantifies each length-filtered RiboSeq BAM (e.g. BAM.25-35/Wild_Type.1.25-35.bam) against a GFF in intersection-strict mode restricted to CDS features, then merges the per-sample count files into one transcript/gene-by-sample matrix for DESeq2.

## When to use

Use when per-sample RiboSeq alignments (BAM) exist and a single merged count matrix, one column per sample indexed by transcript/gene ID, is needed as input to DESeq2 differential expression analysis.

## When NOT to use

- The merged count matrix already exists and the goal is the downstream DESeq2 normalization/comparison step itself
- The data are still raw or trimmed FASTQ and have not yet been contaminant-depleted and aligned to BAM (run the upstream RiboDoc QC/trimming/alignment steps first)
- The goal is P-site offset estimation or trinucleotide periodicity analysis, which is handled by riboWaltz/TRiP, not read counting

## Inputs

- Length-filtered per-sample BAM alignment files (e.g. BAM.25-35/Wild_Type.1.25-35.bam)
- GFF annotation file for the reference organism (e.g. Saccharomyces_cerevisiae.R64-1-1 from Ensembl)
- Project config defining the gff_cds_feature vocabulary (CDS-type features to count)
- Sample names parsed from FASTQ file names ([CONDITION]_[NAME].[REPLICATE].fastq.gz)

## Outputs

- Per-sample htseq-count count files stored under HTSeq-counts/htseqcount_CDS
- Merged count matrix written to DESeq2_CDS.25-35/count_matrix_by_transcript_IDs.csv, rows indexed by transcript/gene ID, one column per sample with sample names in the header

## How to apply

Load the length-filtered per-sample BAM files (e.g. Wild_Type.1.25-35.bam from the BAM.25-35 folder) and the reference organism's GFF annotation. Run htseq-count on each BAM against the GFF using the 'intersection-strict' overlap-resolution mode (as done in RiboDoc with HTSeq v0.12.4), restricting counting to CDS-type features as defined by the gff_cds_feature vocabulary of the project config, so reads overlapping annotated CDS regions are assigned to the parent gene/transcript identifier. Collect the raw per-sample htseq-count outputs under HTSeq-counts/htseqcount_CDS and merge them row-wise by transcript/gene ID. Derive column headers from the sample names parsed out of the FASTQ file names (RiboDoc convention: [CONDITION]_[NAME].[REPLICATE].fastq.gz, e.g. Wild_Type.1, Mutant.2). Write the merged table to DESeq2_CDS.25-35/count_matrix_by_transcript_IDs.csv so it can feed the downstream DESeq2 normalization and comparison step.

## Related tools

- **HTSeq (htseq-count)** (Performs the per-sample read counting on the BAM files in intersection-strict mode restricted to CDS features (v0.12.4 in RiboDoc)) — https://htseq.readthedocs.io/en/release_0.11.1/count.html
- **RiboDoc** (Docker/Snakemake pipeline that orchestrates the htseq-count step and merges the per-sample outputs into the count matrix consumed by DESeq2) — https://github.com/equipeGST/RiboDoc
- **DESeq2** (Downstream consumer of the merged count matrix for normalization and differential gene expression analysis)

## Examples

```
htseq-count -f bam -s no -t CDS -m intersection-strict BAM.25-35/Wild_Type.1.25-35.bam annotation.gff > HTSeq-counts/htseqcount_CDS/Wild_Type.1.txt
```

## Evaluation signals

- The merged output CSV exists at DESeq2_CDS.25-35/count_matrix_by_transcript_IDs.csv
- The CSV header contains exactly the sample names parsed from the input FASTQ file names (e.g. Wild_Type.1, Mutant.2), matching the number of input BAM samples
- Rows are indexed by transcript/gene IDs and per-sample column values are consistent with the raw per-sample htseq-count outputs under HTSeq-counts/htseqcount_CDS
- Counting was restricted to CDS-type features and overlap resolved with intersection-strict, so counts reproduce when htseq-count is rerun with the same mode and GFF

## Limitations

- Counting is restricted to CDS features with intersection-strict resolution; using a different mode or feature type will change the counts and break comparability with the RiboDoc output
- Sample naming relies on the strict FASTQ convention [CONDITION]_[NAME].[REPLICATE].fastq.gz with no dashes, parentheses, blank spaces or other special characters (Windows may hide extensions), which affects header generation
- The merged matrix does not account for P-site offsets (riboWaltz/TRiP handle offset prediction separately), and the README-linked htseq-count documentation is for release 0.11.1 while RiboDoc reports HTSeq v0.12.4

## Evidence

- [other] Counts are performed on the BAM files with Htseq (v0.12.4), using the 'intersection-strict' mode restricted to CDS regions; all per-sample count files are then merged: [evidence span withheld — anchor status: not_found]
- [other] Run htseq-count on each BAM against the GFF using intersection-strict overlap resolution so reads overlapping annotated CDS features are assigned: [evidence span withheld — anchor status: not_found]
