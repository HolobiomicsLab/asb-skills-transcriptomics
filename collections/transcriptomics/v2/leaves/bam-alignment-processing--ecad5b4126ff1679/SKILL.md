---
name: bam-alignment-processing
description: Use when sorted, length-filtered RiboSeq BAM files (e.g. BAM.25-35 folder)
  and a GFF annotation are available and a gene-level count matrix is needed for downstream
  DESeq2 differential expression analysis.
when_to_use_negative:
- Input is already a gene/sample count table ready for DESeq2 — recounting from BAMs
  would be redundant
- BAM files are unsorted or not length-filtered RiboSeq fragments (e.g. raw alignments
  outside the 25-35 nt RPF window)
- The goal is P-site offset estimation or periodicity plots rather than quantification
  — use riboWaltz/TRiP instead of htseq-count
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0622
tools:
- name: HTSeq
  role: htseq-count (v0.12.4) assigns reads overlapping annotated CDS features to
    genes using intersection-strict mode and produces per-sample count files
  repo: https://htseq.readthedocs.io/en/release_0.11.1/count.html#
- name: RiboDoc
  role: Docker/Snakemake pipeline that orchestrates htseq-count on the length-filtered
    BAMs, merges the per-sample counts into the DESeq2 input matrix, and enforces
    sample naming from FASTQ files
  repo: https://github.com/equipeGST/RiboDoc
- name: samtools
  role: Sorts and indexes alignments upstream so BAMs are count-ready for htseq-count
  repo: https://sourceforge.net/projects/samtools/files/samtools/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:66535b3a7f14c70623dd8169c68ee9913940886c4eb765b329b6d6be03f24377
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/equipeGST/RiboDoc
  tools:
  - HTSeq
  - RiboDoc
  - samtools
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# bam-alignment-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Per-gene read counting from length-filtered RiboSeq BAM alignments using htseq-count in intersection-strict mode restricted to CDS features, followed by merging per-sample count files into a single gene-by-sample matrix consumed by DESeq2. This is the quantitative bridge between alignment and differential translation analysis in RiboDoc.

## When to use

Use when sorted, length-filtered RiboSeq BAM files (e.g. BAM.25-35 folder) and a GFF annotation are available and a gene-level count matrix is needed for downstream DESeq2 differential expression analysis.

## When NOT to use

- Input is already a gene/sample count table ready for DESeq2 — recounting from BAMs would be redundant
- BAM files are unsorted or not length-filtered RiboSeq fragments (e.g. raw alignments outside the 25-35 nt RPF window)
- The goal is P-site offset estimation or periodicity plots rather than quantification — use riboWaltz/TRiP instead of htseq-count

## Inputs

- Sorted, length-filtered per-sample BAM alignment files (e.g. Wild_Type.1.25-35.bam from BAM.25-35 folder)
- GFF annotation file for the reference organism (e.g. from Ensembl)
- Project config specifying gff_cds_feature vocabulary (CDS-type features and parent gene/transcript grouping)

## Outputs

- Merged count matrix DESeq2_CDS.25-35/count_matrix_by_transcript_IDs.csv with transcript/gene IDs as rows and sample names (e.g. Wild_Type.1, Mutant.2) as columns
- Raw per-sample htseq-count output files under HTSeq-counts/htseqcount_CDS

## How to apply

Load the length-filtered per-sample BAM files (e.g. Wild_Type.1.25-35.bam) from the BAM.25-35 folder together with the reference organism GFF. Run htseq-count (v0.12.4) on each BAM against the GFF using 'intersection-strict' overlap resolution, restricting counting to CDS-type features as defined by the gff_cds_feature vocabulary of the project config, so reads overlapping annotated CDS features are assigned to their parent gene/transcript. Merge the per-sample htseq-count outputs into one matrix indexed by transcript/gene ID with one column per sample, deriving the column header sample names (e.g. Wild_Type.1, Mutant.2) from the FASTQ file naming convention [CONDITION]_[NAME].[REPLICATE].fastq.gz. Write the merged table to DESeq2_CDS.25-35/count_matrix_by_transcript_IDs.csv and keep the raw per-sample outputs under HTSeq-counts/htseqcount_CDS for traceability. Rationale: intersection-strict on CDS features ensures only reads unambiguously overlapping annotated coding regions contribute counts, which supports the CDS-enrichment quality control and reliable DESeq2 normalization.

## Related tools

- **HTSeq** (htseq-count (v0.12.4) assigns reads overlapping annotated CDS features to genes using intersection-strict mode and produces per-sample count files) — https://htseq.readthedocs.io/en/release_0.11.1/count.html#
- **RiboDoc** (Docker/Snakemake pipeline that orchestrates htseq-count on the length-filtered BAMs, merges the per-sample counts into the DESeq2 input matrix, and enforces sample naming from FASTQ files) — https://github.com/equipeGST/RiboDoc
- **samtools** (Sorts and indexes alignments upstream so BAMs are count-ready for htseq-count) — https://sourceforge.net/projects/samtools/files/samtools/

## Examples

```
htseq-count --mode=intersection-strict --type=CDS --idattr=gene_id Wild_Type.1.25-35.bam Saccharomyces_cerevisiae.R64-1-1.gff > HTSeq-counts/htseqcount_CDS/Wild_Type.1.counts
```

## Evaluation signals

- The output CSV DESeq2_CDS.25-35/count_matrix_by_transcript_IDs.csv exists and is parseable
- Header columns match the number and names of the input samples, derived from FASTQ file names (e.g. Wild_Type.1, Mutant.2)
- Per-sample column totals in the merged matrix are consistent with the raw htseq-count outputs stored under HTSeq-counts/htseqcount_CDS
- Row identifiers correspond to transcript/gene IDs grouped from CDS-type features per the gff_cds_feature config vocabulary
- Counts derive from BAM files restricted to the 25-35 nt RPF selection used by the pipeline

## Limitations

- Counts depend on GFF annotation quality and the configured CDS feature type; mis-specified gff_cds_feature vocabulary silently yields wrong or empty counts
- htseq-count in intersection-strict mode discards reads ambiguous across overlapping features, so multimapping/ambiguous reads are not counted and total counts understate library size
- The pipeline expects the FASTQ naming convention [CONDITION]_[NAME].[REPLICATE].fastq.gz; special characters other than underscores break sample-name parsing

## Evidence

- [other] Counts are performed on the BAM files with Htseq (v0.12.4), using the 'intersection-strict' mode restricted to CDS regions: [evidence span withheld — anchor status: not_found]
- [other] Merge per-sample counts into one matrix with sample names from FASTQ file names: [evidence span withheld — anchor status: not_found]
- [readme] README pipeline step 8: Reads Count with htseq-count: "8. Reads Count with [`htseq-count`](https://htseq.readthedocs.io/en/release_0.11.1/count.html#)"
- [readme] README FASTQ file naming convention that drives sample names in the count matrix header: "**Format of file names must be as following:** `[CONDITION]_[NAME].[REPLICATE].fastq.gz`"
- [results] Article methods: DESeq2 consumes the counts for normalization and comparison: "RiboDoc uses DESeq2 [26] to normalize and compare the data"
