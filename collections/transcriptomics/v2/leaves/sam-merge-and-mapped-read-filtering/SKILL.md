---
name: sam-merge-and-mapped-read-filtering
description: Use when a ribosome profiling pipeline aligns the same FASTA with two
  mappers (HISAT2 then Bowtie2) and the resulting SAM outputs must be merged, filtered
  to mapped reads, and converted to BAM/BAI.
when_to_use_negative:
- Reads have not yet been adapter-trimmed, quality-trimmed, and rRNA/tRNA/viral-RNA-depleted
  with Cutadapt and Bowtie2 — this step expects depleted, 25-35 nt selected reads.
- The analysis requires splice junction detection itself — that is the role of the
  preceding HISAT2 splice-aware mapping step, not of the merge/filter step.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: HISAT2
  role: Performs the first, splice-aware alignment of depleted RPF reads against the
    reference genome to capture footprints spanning exon junctions.
- name: Bowtie 2
  role: Remaps reads left unaligned by HISAT2 against the same FASTA with less strict
    parameters allowing at most 1 mismatch (v2.4.2).
- name: samtools
  role: Converts, sorts and indexes the merged HISAT2/Bowtie2 alignment to produce
    BAM and BAI files per sample.
- name: RiboDoc
  role: Docker/Snakemake pipeline orchestrating the double mapping, merge, mapped-read
    filtering and BAM generation steps.
  repo: https://github.com/equipeGST/RiboDoc
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:439959a626b8cdc80947aa2cca313045e00a51ce1a511f6fcfe08726cad04d0b
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
  - RiboDoc
  - samtools
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# SAM merge and mapped-read filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Merges the HISAT2 splice-aware and Bowtie2 non-strict alignments of rRNA-depleted, 25-35 nt RPF reads into a single alignment per sample, converts it with samtools to sorted/indexed BAM+BAI, and discards reads that remain unmapped so only usable mapped footprints proceed.

## When to use

Use when a ribosome profiling pipeline aligns the same FASTA with two mappers (HISAT2 then Bowtie2) and the resulting SAM outputs must be merged, filtered to mapped reads, and converted to BAM/BAI.

## When NOT to use

- Reads have not yet been adapter-trimmed, quality-trimmed, and rRNA/tRNA/viral-RNA-depleted with Cutadapt and Bowtie2 — this step expects depleted, 25-35 nt selected reads.
- The analysis requires splice junction detection itself — that is the role of the preceding HISAT2 splice-aware mapping step, not of the merge/filter step.

## Inputs

- rRNA-depleted, length-filtered FastQ reads (25-35 nt RPF selection)
- pre-built reference genome index from the annex_database/index_files folder
- SAM outputs from HISAT2 (splice-aware) and Bowtie2 (≤1 mismatch) alignments

## Outputs

- Merged, sorted BAM file per sample in the BAM.25-35 results folder
- Corresponding BAI index file for each BAM
- Multimapping/uniquely-mapped read counts reported in Analysis_Table_summary outputs

## How to apply

After rRNA depletion and RPF length selection (25-35 nt), first align the depleted reads against the pre-built genome index (from annex_database/index_files) with HISAT2 in splice-aware mode to capture footprints spanning exon junctions. Collect the reads HISAT2 left unaligned and remap them against the same reference FASTA with Bowtie2 (v2.4.2) using less strict parameters that accept more complementarity errors, at most 1. Merge the HISAT2 and Bowtie2 SAM outputs into a single alignment per sample and discard reads that remain unmapped, retaining only uniquely usable mapped footprints (the multimapping filter keeps uniquely mapped reads as reported in the Analysis_Table_summary outputs). Finally convert, sort and index each merged alignment with samtools to produce BAM and BAI files in the BAM.25-35 results folder.

## Related tools

- **HISAT2** (Performs the first, splice-aware alignment of depleted RPF reads against the reference genome to capture footprints spanning exon junctions.)
- **Bowtie 2** (Remaps reads left unaligned by HISAT2 against the same FASTA with less strict parameters allowing at most 1 mismatch (v2.4.2).)
- **samtools** (Converts, sorts and indexes the merged HISAT2/Bowtie2 alignment to produce BAM and BAI files per sample.)
- **RiboDoc** (Docker/Snakemake pipeline orchestrating the double mapping, merge, mapped-read filtering and BAM generation steps.) — https://github.com/equipeGST/RiboDoc

## Examples

```
samtools merge merged.sam hisat2.sam bowtie2.sam && samtools view -F 4 -b merged.sam | samtools sort -o BAM.25-35/sample.bam && samtools index BAM.25-35/sample.bam
```

## Evaluation signals

- Each sample yields a non-empty, valid BAM file in the BAM.25-35 results folder together with its .bai index.
- samtools index succeeds on every merged BAM (BAI present and readable).
- The multimapping filter step has kept uniquely mapped reads, as reported in the Analysis_Table_summary outputs.
- No reads that remained unmapped after both HISAT2 and Bowtie2 are present in the final BAM.
- Raw/Filtered/Multimapping/Unique map counts in the Analysis_Table_summary are internally consistent (unique ≤ filtered ≤ raw).

## Limitations

- Bowtie2 remapping accepts up to 1 complementarity error, so alignments with more than 1 mismatch will be lost.
- RiboDoc does not state a version number for the Docker package, so exact tool versions beyond Bowtie2 v2.4.2 may need to be confirmed per deployment.
- On complex genomes, reads unaligned by both mappers are discarded, which can reduce library depth.

## Evidence

- [methods] HISAT2 first, Bowtie2 remap with 1 mismatch: [evidence span withheld — anchor status: not_found]
