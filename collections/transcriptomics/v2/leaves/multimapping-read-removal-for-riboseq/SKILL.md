---
name: multimapping-read-removal-for-riboseq
description: Use when aligning ribosome-profiling footprints (25-35 nt) to a genome
  with the Hisat2 then Bowtie2 double-mapping strategy and you must retain only uniquely
  mapped reads before counting.
when_to_use_negative:
- Reads have not yet been adapter-trimmed or depleted of contaminant RNA (rRNA, tRNA,
  viral RNA) — RiboDoc performs that earlier Bowtie2 step first, and running the alignment/multimapping
  filter before it wastes mappings on contaminants
- Input is already a sorted BAM from a previous RiboDoc run where the multimapping
  filter was applied — the pipeline can be rerun 'only performing the steps for which
  recalculation is required'
- The analysis goal is quantifying reads shared across paralogs/repeats rather than
  gene-level differential expression, where discarding multimappers loses signal by
  design
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: RiboDoc
  role: Docker/Snakemake pipeline that orchestrates the double-mapping, multimapping
    filter and BAM generation steps
  repo: https://github.com/equipeGST/RiboDoc
- name: HISAT2
  role: First-pass splice-aware genome alignment capturing footprints spanning exon
    junctions
- name: Bowtie 2
  role: Second-pass remapping of reads unaligned by HISAT2, and earlier removal of
    contaminant rRNA/tRNA reads
  repo: https://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- name: samtools
  role: Convert, sort and index the merged HISAT2/Bowtie2 alignments into BAM/BAI
    after unmapped reads are discarded
  repo: https://sourceforge.net/projects/samtools/files/samtools/
- name: HTSeq
  role: htseq-count reads-counting step applied to the uniquely mapped alignments
  repo: https://htseq.readthedocs.io/en/release_0.11.1/count.html#
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:fd5150e3aba2999aa3f1b4109f649c41b1c44fce9d5114e41967b7b97f67e4fd
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
  - HISAT2
  - HTSeq
  - RiboDoc
  - samtools
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# multimapping-read-removal-for-riboseq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Removal of multimapping reads after the Hisat2/Bowtie2 double-mapping step in RiboDoc, keeping only uniquely mapped ribosome footprints (25-35 nt RPFs) so that downstream counting with htseq-count and DESeq2 differential analysis is not biased by ambiguously placed reads.

## When to use

Use when aligning ribosome-profiling footprints (25-35 nt) to a genome with the Hisat2 then Bowtie2 double-mapping strategy and you must retain only uniquely mapped reads before counting.

## When NOT to use

- Reads have not yet been adapter-trimmed or depleted of contaminant RNA (rRNA, tRNA, viral RNA) — RiboDoc performs that earlier Bowtie2 step first, and running the alignment/multimapping filter before it wastes mappings on contaminants
- Input is already a sorted BAM from a previous RiboDoc run where the multimapping filter was applied — the pipeline can be rerun 'only performing the steps for which recalculation is required'
- The analysis goal is quantifying reads shared across paralogs/repeats rather than gene-level differential expression, where discarding multimappers loses signal by design

## Inputs

- Gzip-compressed FastQ files (*.fastq.gz) of rRNA-depleted, length-filtered reads (25-35 nt RPFs)
- Pre-built reference genome index from annex_database/index_files folder
- Reference genome FASTA (e.g. Saccharomyces_cerevisiae.R64-1-1.dna.toplevel.fa.gz from Ensembl)

## Outputs

- Sorted, indexed BAM and .bai files per sample in the BAM.25-35 results folder containing only uniquely mapped footprints
- Analysis_Table_summary outputs reporting the raw reads / filtered reads / multimapping / unique map counts per sample

## How to apply

After rRNA depletion and length filtering (Cutadapt keeps 25-35 nt RPFs), align reads splice-aware to the pre-built genome index with HISAT2 to capture footprints spanning exon junctions. Collect the reads HISAT2 leaves unaligned and remap them against the same FASTA with Bowtie2 (v2.4.2) using less strict defaults accepting at most 1 mismatch. Merge HISAT2 and Bowtie2 SAM outputs into a single alignment per sample, discarding reads that remain unmapped, then apply the multimapping filter so only uniquely mapped reads are retained; convert, sort and index the merged alignments with samtools. The rationale is that multimapping footprints cannot be assigned unambiguously to one gene, so they would corrupt the unique-map counts reported in the Analysis_Table_summary and used by DESeq2.

## Related tools

- **RiboDoc** (Docker/Snakemake pipeline that orchestrates the double-mapping, multimapping filter and BAM generation steps) — https://github.com/equipeGST/RiboDoc
- **HISAT2** (First-pass splice-aware genome alignment capturing footprints spanning exon junctions)
- **Bowtie 2** (Second-pass remapping of reads unaligned by HISAT2, and earlier removal of contaminant rRNA/tRNA reads) — https://bowtie-bio.sourceforge.net/bowtie2/index.shtml
- **samtools** (Convert, sort and index the merged HISAT2/Bowtie2 alignments into BAM/BAI after unmapped reads are discarded) — https://sourceforge.net/projects/samtools/files/samtools/
- **HTSeq** (htseq-count reads-counting step applied to the uniquely mapped alignments) — https://htseq.readthedocs.io/en/release_0.11.1/count.html#

## Examples

```
docker pull equipegst/ribodoc
```

## Evaluation signals

- Each sample yields a non-empty, valid BAM file in the BAM.25-35 folder together with its .bai index
- The multimapping filter kept uniquely mapped reads, as reported in the Analysis_Table_summary outputs (raw reads / filtered reads / multimapping / unique map)
- Alignments are sorted and indexable — samtools can read each BAM together with its .bai index without errors
- Uniquely mapped counts are consistent with the expected footprint length window (majority of reads ~28 nt, RPF selection 25-35 nt)
- Downstream htseq-count and DESeq2 normalization run on the unique-map BAMs without alignment-related warnings

## Limitations

- Bowtie2's second pass accepts a maximum of 1 complementarity error (less strict defaults), so highly degraded or heavily mismatched footprints may still remain unmapped and be discarded
- Discarding multimappers removes reads from duplicated regions, paralogs and repeats, so genes in duplicated loci will be underrepresented in unique-map counts
- RiboDoc's defaults select 25-35 nt reads as RPFs; footprints outside this length window never reach the alignment/multimapping stage
- No version number is stated for the RiboDoc Docker package itself, which complicates exact reproduction of the filter behavior across releases

## Evidence

- [methods] Reads not aligned in the first mapping step are remapped on the same fasta file with Bowtie2 (v2.4.2), using less strict default parameters that accept more complementarity errors (maximum of 1).: [evidence span withheld — anchor status: not_found]
- [methods] Multimapping filter step has kept uniquely mapped reads as reported in the Analysis_Table_summary outputs.: [evidence span withheld — anchor status: not_found]
