---
name: per-read-modification-aggregation-to-genome-space
description: Use when NanoSpeech basecalled fasta/q read headers carry per-modification
  base indexes and you need per-read BED plus genome-space aggregated modification
  calls (counts, depth, frequency) for a reference transcriptome or genome.
when_to_use_negative:
- Basecalls come from a modification-unaware caller (e.g. canonical Guppy/Dorado output)
  — the read headers will not carry modification indexes for the detector script to
  project
- Modification probabilities must be extracted from signal-level data by re-squiggling
  or an external modification model (e.g. ModiKit-style post-processing) rather than
  from NanoSpeech header indexes
- You need modification calls on raw pod5/fast5 signals directly — this skill operates
  downstream, on already-basecalled and aligned reads
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: NanoSpeech
  role: Provides the accessory inosine_detector.py / modification_detector.py scripts
    that map read-header modification indexes to reference coordinates using the BAM
    and fasta/q inputs
  repo: https://github.com/F0nz0/NanoSpeech_basecaller
- name: minimap2
  role: Aligns the NanoSpeech basecalled fasta/fastq to the reference (map-ont or
    splice presets with --secondary=no --MD) to supply coordinate context
- name: samtools
  role: Filters the SAM (samtools view -b -F 2304), sorts to BAM, and indexes it before
    running the detector script
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf709
    title: <i>Ab initio</i> detection of multiple epitranscriptomic modifications
      from Oxford nanopore technology direct RNA sequencing data
  source_content_hash: sha256:75995062f8e4d4882b0d6f86ad5bf0b6b7012f5694dcd059dcebc624c036cfcc
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
  - modification_detector.py
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# per-read-modification-aggregation-to-genome-space

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps NanoSpeech per-read modification indexes (carried in basecalled fasta/fastq headers, e.g. I_A=..., M_A=...) onto reference coordinates from a filtered BAM to produce a per-read BED and a genome-space aggregated table of modification counts, depth, and frequencies.

## When to use

Use when NanoSpeech basecalled fasta/q read headers carry per-modification base indexes and you need per-read BED plus genome-space aggregated modification calls (counts, depth, frequency) for a reference transcriptome or genome.

## When NOT to use

- Basecalls come from a modification-unaware caller (e.g. canonical Guppy/Dorado output) — the read headers will not carry modification indexes for the detector script to project
- Modification probabilities must be extracted from signal-level data by re-squiggling or an external modification model (e.g. ModiKit-style post-processing) rather than from NanoSpeech header indexes
- You need modification calls on raw pod5/fast5 signals directly — this skill operates downstream, on already-basecalled and aligned reads

## Inputs

- Filtered, sorted, indexed BAM file of NanoSpeech-basecalled direct RNA reads (SAM filtered with -F 2304)
- NanoSpeech basecalled fasta/fastq whose read headers contain per-modification base indexes (e.g. I_A=..., M_A=...)
- Minimum modification/inosine quality threshold (-q)
- Reference transcriptome or genome used for the minimap2 alignment

## Outputs

- Per-read BED file ({bam}.per_read.bed, 7 columns) with modification calls at reference coordinates
- Genome-space aggregated table ({bam}.genome_space.tsv, 6 columns: region, start, strand, I_count, depth, I_freq)

## How to apply

After NanoSpeech basecalling, align reads with minimap2 (e.g. '-ax map-ont -k 14 --secondary=no --MD' for transcriptomes or '-ax splice -uf -k 14 --secondary=no --MD' for genomes), then filter and convert with 'samtools view -b -F 2304 | samtools sort -O BAM' and index the BAM. Run the accessory detector script (inosine_detector.py or modification_detector.py) with the BAM (-b), the basecalled fasta/q (-f), and a minimum modification quality threshold (-q); the script reads the modification indexes embedded in the fasta/q headers and projects them to reference coordinates. It emits two outputs: a per-read BED ({bam}.per_read.bed, 7 columns) and a genome-space aggregated table ({bam}.genome_space.tsv, 6 columns: region, start, strand, I_count, depth, I_freq). The rationale is that NanoSpeech calls modified bases during basecalling itself, so no re-squiggling or separate modification model is needed — only coordinate mapping of header indexes onto the BAM alignment.

## Related tools

- **NanoSpeech** (Provides the accessory inosine_detector.py / modification_detector.py scripts that map read-header modification indexes to reference coordinates using the BAM and fasta/q inputs) — https://github.com/F0nz0/NanoSpeech_basecaller
- **minimap2** (Aligns the NanoSpeech basecalled fasta/fastq to the reference (map-ont or splice presets with --secondary=no --MD) to supply coordinate context)
- **samtools** (Filters the SAM (samtools view -b -F 2304), sorts to BAM, and indexes it before running the detector script)

## Examples

```
python modification_detector.py -b alignments.bam -f basecalled.fastq -q 7
```

## Evaluation signals

- Both output files exist with the documented schemas: a 7-column per_read.bed and a 6-column genome_space.tsv with fields region, start, strand, I_count, depth, I_freq
- Every modification call in the per-read BED corresponds to a header index position (e.g. I_A=... or M_A=...) of the same read in the basecalled fasta/q, mapped through the BAM
- Per-position I_freq in genome_space.tsv equals I_count/depth, and calls below the -q quality threshold are excluded
- On unmodified IVT controls, the aggregated table shows a low false-positive modification rate across samples
- Inosine calls on the I-rich E. coli AmpD (coli-IVT) construct fall at the expected reference positions with ~96% accuracy and ~95% F1

## Limitations

- Chemically similar modifications can be confused: in the nine-modification NanoSpeech model applied to unseen RNA002 IVT reads, m1A was erroneously miscalled as its isomer m6A (cross-model m6A/m1A correlations r=0.451 and r=0.709)
- The workflow is anchored to NanoSpeech's expanded-dictionary basecalling outputs; inference artefacts or 'hallucinations' can arise from highly similar chemical structures or intrinsic errors in modified samples
- The alignment must be pre-filtered (samtools view -b -F 2304), sorted, and indexed; secondary alignments must be removed (--secondary=no) for coordinate mapping to be valid

## Evidence
