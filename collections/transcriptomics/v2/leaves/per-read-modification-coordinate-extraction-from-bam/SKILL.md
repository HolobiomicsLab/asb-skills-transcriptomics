---
name: per-read-modification-coordinate-extraction-from-bam
description: Use when NanoSpeech basecalled dRNA pod5/fast5 reads into fasta/fastq
  whose headers embed indexes of candidate modified nucleotides (e.g. inosines), and
  you need per-read and transcriptome-space modification coordinates to score calls
  against known modification positions.
when_to_use_negative:
- The basecaller output does not encode modification indexes in read headers (e.g.
  calls from a canonical-only basecaller like standard Guppy/Dorado without a modification-aware
  model)
- You need modification probabilities from standard ONT tools without ab initio basecalling
  — a re-squiggling pipeline (basecall, align, modification model) or Dorado's modification
  models would be the alternative approach
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: NanoSpeech
  role: Provides the accessory modification_detector.py script that reads BAM files
    plus inosine indexes in the fasta/q headers to emit per-read and genome-space
    modification coordinates
  repo: https://github.com/F0nz0/NanoSpeech_basecaller
- name: minimap2
  role: Maps the basecalled fasta/fastq reads to a reference transcriptome or genome
    so modification indexes can be converted to reference coordinates
- name: samtools
  role: Filters the SAM (view -b -F 2304), converts to sorted BAM, and indexes it
    as input for modification detection
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf709
    title: <i>Ab initio</i> detection of multiple epitranscriptomic modifications
      from Oxford nanopore technology direct RNA sequencing data
  source_content_hash: sha256:096a25631de072279143db600b10cc154f9c8ca788bab460a12ec0c4c0de1c13
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/F0nz0/NanoSpeech_basecaller
  repo_url: https://github.com/F0nz0/NanoSpeech_basecaller
  tools:
  - NanoSpeech
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# per-read-modification-coordinate-extraction-from-BAM

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract per-read (and aggregated transcriptome-space) coordinates of modified nucleotides called by NanoSpeech, by parsing read headers that carry modification indexes and mapping them against a reference with minimap2 and samtools. Produces {bam}.per_read.bed and {bam}.genome_space.tsv tables for downstream precision/recall evaluation.

## When to use

Use when NanoSpeech basecalled dRNA pod5/fast5 reads into fasta/fastq whose headers embed indexes of candidate modified nucleotides (e.g. inosines), and you need per-read and transcriptome-space modification coordinates to score calls against known modification positions.

## When NOT to use

- The basecaller output does not encode modification indexes in read headers (e.g. calls from a canonical-only basecaller like standard Guppy/Dorado without a modification-aware model)
- You need modification probabilities from standard ONT tools without ab initio basecalling — a re-squiggling pipeline (basecall, align, modification model) or Dorado's modification models would be the alternative approach

## Inputs

- NanoSpeech-basecalled fasta/fastq file whose headers embed 0-based indexes of candidate modified nucleotides (e.g. inosines)
- Sorted, indexed BAM file of the basecalled reads aligned with minimap2 (presets map-ont for transcriptome or splice -uf for genome)
- Minimum base quality threshold for calling modifications (modification_detector.py -q parameter)

## Outputs

- {bam}.per_read.bed: per-read mapping coordinates of every called modified nucleotide
- {bam}.genome_space.tsv: transcriptome/genome-space aggregated modification predictions

## How to apply

First map the NanoSpeech-basecalled fasta/fastq to a reference transcriptome with minimap2 -ax map-ont -k 14 --secondary=no --MD (or -ax splice -uf -k 14 --secondary=no --MD for a reference genome). Convert the SAM to a sorted, indexed BAM, filtering secondary alignments with samtools view -b -F 2304 followed by samtools sort -O BAM and samtools index. Then run NanoSpeech's accessory script modification_detector.py with -b {BAM}, -f {fasta/q}, and -q {minimum base quality threshold}; it reads the modification indexes stored in the fasta/fastq headers and emits mapping coordinates for every modified nucleotide as {bam}.per_read.bed plus aggregated {bam}.genome_space.tsv. Decision point: the -q quality threshold filters which called modifications are kept, trading sensitivity against false positives.

## Related tools

- **NanoSpeech** (Provides the accessory modification_detector.py script that reads BAM files plus inosine indexes in the fasta/q headers to emit per-read and genome-space modification coordinates) — https://github.com/F0nz0/NanoSpeech_basecaller
- **minimap2** (Maps the basecalled fasta/fastq reads to a reference transcriptome or genome so modification indexes can be converted to reference coordinates)
- **samtools** (Filters the SAM (view -b -F 2304), converts to sorted BAM, and indexes it as input for modification detection)

## Examples

```
samtools view -b -F 2304 {out_filepath}.sam | samtools sort -O BAM; samtools index {out_filepath}.sorted.bam; python modification_detector.py -b {BAM} -f {fasta/fastq} -q {minimum base quality}
```

## Evaluation signals

- The output files {bam}.per_read.bed and {bam}.genome_space.tsv exist and the .tsv is indexed in the same transcriptome/genome space as the minimap2 --MD alignment
- Every 0-based modification index present in a read's fasta/fastq header has a corresponding coordinate row in the per-read BED for that read
- Per-class precision of >=0.98 and recall of ~0.99 for I and m6A calls against the known positions of synthetic IVT constructs, with zero I<->m6A cross-misclassification
- Increasing the -q minimum quality threshold monotonically reduces reported modification calls (fewer low-confidence sites retained)
- Alignments validated after filtering (-F 2304, --secondary=no) contain no secondary alignments in the indexed BAM

## Limitations

- NanoSpeech models have been primarily tested on synthetic constructs as a proof of concept; public modification data quality and missing reliable benchmarks impair validation on real-world samples
- Inference artefacts or 'hallucinations' can arise for highly similar chemical structures, and streamlined models for detecting them are still under development
- The modification-aware index is specific to the modification vocabulary of the NanoSpeech model used (e.g. inosine-only vs. multi-mod I+m6A weights) and ONT dRNA reads; it does not generalize to other read types or chemistries without retraining

## Evidence
