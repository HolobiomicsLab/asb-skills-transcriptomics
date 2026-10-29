---
name: minimap2-read-alignment
description: Use when NanoSpeech (or a comparable ONT) basecaller has produced fasta/fastq
  reads from direct RNA sequencing and the modification indexes carried in read headers
  must be mapped to a reference transcriptome or genome for per-read and genome-space
  modification calling.
when_to_use_negative:
- Input is still raw pod5/fast5 signal data — NanoSpeech basecalling (NanoSpeech.py
  or NanoSpeech_multi_mod.py) must run first to produce fasta/fastq with modification
  indexes.
- Reference coordinates are already available or the analysis operates purely on read-level
  statistics (e.g. per-read identity, consensus accuracy) that do not require alignment
  to a reference.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: minimap2
  role: Aligns basecalled fasta/fastq reads to the reference using map-ont (transcriptome)
    or splice -uf (genome) presets with -k 14, --secondary=no and --MD
  repo: https://github.com/F0nz0/NanoSpeech_basecaller
- name: samtools
  role: Filters (-F 2304), sorts, and indexes the minimap2 SAM into the BAM consumed
    by inosine_detector.py / modification_detector.py
  repo: https://github.com/F0nz0/NanoSpeech_basecaller
- name: NanoSpeech
  role: Produces the basecalled fasta/fastq whose read headers carry the modification
    indexes that alignment and detection convert to reference coordinates
  repo: https://github.com/F0nz0/NanoSpeech_basecaller
provenance:
  source_task_ids:
  - task_005
  - task_001
  - task_002
  - task_004
  - task_006
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf709
    title: <i>Ab initio</i> detection of multiple epitranscriptomic modifications
      from Oxford nanopore technology direct RNA sequencing data
  source_content_hash: sha256:634642f94a9270ada5abd3b64216cf19a2d788f651ebbdad064e73bc3d751079
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
  - Dorado
  - ModiKit
  - NanoSpeech
  - TensorFlow
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# minimap2-read-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps NanoSpeech basecalled dRNA fasta/fastq reads onto a reference transcriptome or genome with minimap2 (map-ont or splice presets, secondary alignments suppressed, MD tags kept), then converts the SAM to a sorted, indexed BAM with samtools so modification indexes in read headers can be projected to reference coordinates.

## When to use

Use when NanoSpeech (or a comparable ONT) basecaller has produced fasta/fastq reads from direct RNA sequencing and the modification indexes carried in read headers must be mapped to a reference transcriptome or genome for per-read and genome-space modification calling.

## When NOT to use

- Input is still raw pod5/fast5 signal data — NanoSpeech basecalling (NanoSpeech.py or NanoSpeech_multi_mod.py) must run first to produce fasta/fastq with modification indexes.
- Reference coordinates are already available or the analysis operates purely on read-level statistics (e.g. per-read identity, consensus accuracy) that do not require alignment to a reference.

## Inputs

- Basecalled fasta/fastq from NanoSpeech (headers carrying modification base indexes)
- Reference transcriptome (multi-fasta) or reference genome for the splice preset

## Outputs

- Plain SAM file from minimap2 (intermediate, removed after conversion)
- Sorted and indexed BAM (via samtools view -b -F 2304 | samtools sort -O BAM, then samtools index)
- Per-read modification BED and genome-space TSV produced downstream by inosine_detector.py / modification_detector.py

## How to apply

After NanoSpeech basecalling emits fasta/fastq files whose headers carry indexes of predicted modified bases (e.g. inosines), align the reads with minimap2 v2.24: use the transcriptome preset 'minimap2 -ax map-ont -k 14 --secondary=no --MD' or, for a reference genome, 'minimap2 -ax splice -uf -k 14 --secondary=no --MD' so reads spanning splice junctions map correctly. The --secondary=no flag suppresses secondary alignments and --MD retains the reference mismatch string needed to compare basecalls against the reference. Filter the resulting SAM with 'samtools view -b -F 2304', sort with 'samtools sort -O BAM', index with 'samtools index', and delete the intermediate plain-text SAM. The downstream scripts (inosine_detector.py, modification_detector.py) consume the BAM plus the basecalled fasta/fastq to emit {bam}.per_read.bed and {bam}.genome_space.tsv; mappability itself can serve as a benchmark (NanoSpeech-mapped read fraction above 90% versus ~40% for Guppy in the pureI/pureG tests).

## Related tools

- **minimap2** (Aligns basecalled fasta/fastq reads to the reference using map-ont (transcriptome) or splice -uf (genome) presets with -k 14, --secondary=no and --MD) — https://github.com/F0nz0/NanoSpeech_basecaller
- **samtools** (Filters (-F 2304), sorts, and indexes the minimap2 SAM into the BAM consumed by inosine_detector.py / modification_detector.py) — https://github.com/F0nz0/NanoSpeech_basecaller
- **NanoSpeech** (Produces the basecalled fasta/fastq whose read headers carry the modification indexes that alignment and detection convert to reference coordinates) — https://github.com/F0nz0/NanoSpeech_basecaller

## Examples

```
minimap2 -ax map-ont -k 14 --secondary=no --MD ref_transcriptome.fa basecalled.fq > mapped.sam
```

## Evaluation signals

- SAM is produced with the correct preset flags (-ax map-ont or -ax splice -uf, -k 14, --secondary=no, --MD) and no secondary alignments survive.
- The SAM is filtered with 'samtools view -b -F 2304', converted to a sorted BAM, and successfully indexed with 'samtools index'.
- Downstream inosine_detector.py / modification_detector.py runs emit {bam}.per_read.bed (7 columns) and {bam}.genome_space.tsv (6 columns: region, start, strand, I_count, depth, I_freq), implying the BAM is coordinate-sorted and indexed.
- Mappability exceeds the modification-unaware baseline: >99% of reads correctly aligned on the coli-IVT construct and NanoSpeech mapping fraction above 90% versus ~40% for Guppy.
- The intermediate plain-text SAM file is removed after BAM conversion.

## Limitations

- Preset choice depends on the reference type: map-ont for a reference transcriptome, splice -uf for a reference genome — using the wrong preset degrades mapping of dRNA reads.
- Mappability depends on the basecaller: modification-unaware basecallers like Guppy can map only ~40% of reads on inosine-containing constructs, so poor alignment may reflect basecalling rather than the alignment step.

## Evidence
