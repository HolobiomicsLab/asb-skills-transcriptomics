---
name: fastq-umi-clipping
description: Use when the input is a quality-filtered single-end FASTQ whose reads
  carry the UMI at the 5' start and you must move it into the read header before alignment
  and UMI counting.
when_to_use_negative:
- Input reads do not carry the UMI at the 5' start (e.g. UMI located elsewhere or
  already stored in a separate index read)
- The FASTQ has not yet been demultiplexed/quality-filtered and you also need quality-based
  read selection — run fastq_qual_filter first
- Library preparation does not use 5' UMI placement, so clip_length would exceed the
  read length or remove biological sequence
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: fastq_umi_clipper (C, src/c in ncgr/UMI-analysis)
  role: Clips the UMI from the start of each read and writes it into the FASTQ header
    in place of the index
  repo: https://github.com/ncgr/UMI-analysis
- name: Better String Library
  role: C string-manipulation dependency whose shared object must be in LD_LIBRARY_PATH
    and whose bstring.h must be on the include path to build fastq_umi_clipper
  repo: https://github.com/msteinert/bstring
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-018-31064-7
    title: Unique Molecular Identifiers reveal a novel sequencing artefact with implications
      for RNA-Seq based gene expression analysis
  source_content_hash: sha256:23797c79360490ed9b3aedb1b2f36407c8e2c9138e5cc90fc2580fb166244939
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ncgr/UMI-analysis
  tools:
  - Better String Library
  - C
  - fastq_umi_clipper (C, src/c in ncgr/UMI-analysis)
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# fastq-umi-clipping

## Summary

Removes the UMI from the 5' start of each FASTQ read and stores it in the FASTQ header by replacing the index at the end of header 1, using the C program fastq_umi_clipper from the UMI-analysis pipeline. This preserves the UMI for downstream UMI-collapse steps while preventing UMI and template-switching G-motif bases from entering alignments.

## When to use

Use when the input is a quality-filtered single-end FASTQ whose reads carry the UMI at the 5' start and you must move it into the read header before alignment and UMI counting.

## When NOT to use

- Input reads do not carry the UMI at the 5' start (e.g. UMI located elsewhere or already stored in a separate index read)
- The FASTQ has not yet been demultiplexed/quality-filtered and you also need quality-based read selection — run fastq_qual_filter first
- Library preparation does not use 5' UMI placement, so clip_length would exceed the read length or remove biological sequence

## Inputs

- Quality-filtered single-end FASTQ file (output of fastq_qual_filter) with UMI at the 5' start of each read
- clip_length parameter (UMI length in nucleotides)
- g_length parameter (number of 5' G bases to delete; 0, 3, or 4)

## Outputs

- FASTQ file in which each read header ends with the UMI (replacing the index) and sequence/quality are shortened by clip_length + g_length bases
- Clipped reads ready for alignment (e.g. with STAR) and downstream UMI counting

## How to apply

Build the C project src/c/fastq_umi_clipper after editing the makefile so the Better String Library shared object is resolvable via LD_LIBRARY_PATH and bstring.h is on the include path. Run fastq_umi_clipper on the quality-filtered FASTQ with clip_length set to the UMI length (number of nucleotides moved from the start of the read into header 1) and g_length set to the number of G nucleotides to delete after the UMI — use 0 unless the library was prepared with the Clontech SMARTer template switching system, which leaves GGG/GGGG at the 5' end, in which case use 3 or 4. The program replaces the index at the end of the FASTQ header with the UMI and truncates both sequence and quality strings by clip_length + g_length bases. Note the program performs little argument checking and overwrites output files without warning, so verify parameters before running.

## Related tools

- **fastq_umi_clipper (C, src/c in ncgr/UMI-analysis)** (Clips the UMI from the start of each read and writes it into the FASTQ header in place of the index) — https://github.com/ncgr/UMI-analysis
- **Better String Library** (C string-manipulation dependency whose shared object must be in LD_LIBRARY_PATH and whose bstring.h must be on the include path to build fastq_umi_clipper) — https://github.com/msteinert/bstring

## Examples

```
fastq_umi_clipper good.fq clipped.fq 10 0
```

## Evaluation signals

- Every output read header ends with the UMI extracted from the start of its sequence
- Read lengths (sequence and quality strings) are reduced by exactly clip_length + g_length bases
- No UMI bases and no GGG/GGGG 5' template-switching motif remain at the start of the clipped sequence
- Output remains in valid 4-line FASTQ format with sequence and quality strings still of equal length
- Downstream alignment (STAR) receives reads free of UMI/G-motif bases before UMI collapsing via umi_counterizer.pl

## Limitations

- The program performs very little argument checking: a clip_length longer than the read length or negative values cause unexpected behavior, and output files are overwritten without warning
- g_length greater than 0 should only be used when the library was made with the Clontech SMARTer template switching system, which leaves GGG or GGGG at the 5' end
- The pipeline authors' use case involves single-end sequencing only

## Evidence

- [readme] You tell the program how long the UMI is at the start of each read. It replaces the index at the end of the FASTQ header with the UMI.: "You tell the program how long the UMI is at the start of each read. It replaces the index at the end of the FASTQ header with the UMI."
- [readme] clip_length: number of nucleotides to move from the start of the read and put in header 1; g_length: number of nucleotides (Gs) to delete after the UMI (can be 0): "clip\_length: number of nucleotides to move from the start of the read and put in header 1
   * g\_length: number of nucleotides (Gs) to delete after the UMI (can be 0)"
- [readme] Warning: not much arg checking is done so unexpected behavior may be encountered. For example if you enter a clip_length longer than the read length, or negative values.: "Warning: not much arg checking is done so unexpected behavior may be encountered. For example if you enter a clip\_length longer than the read length, or negative values."
- [methods] This requires the Better String Library shared object to be in your LD_LIBRARY_PATH: "This requires the Better String Library (https://github.com/msteinert/bstring) shared object to be in your LD_LIBRARY_PATH"
