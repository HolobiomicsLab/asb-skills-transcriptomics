---
name: rediportal-a-to-i-editing-site-intersection
description: Use when per-read inosine calls from NanoSpeech basecalling (BAM + genome_space.tsv)
  must be validated against known A-to-I editing sites in REDIportal, or when comparing
  inosine burden across ADAR-modulated samples (e.g. hWT/hKO/hOE).
when_to_use_negative:
- Input reads were basecalled with a modification-unaware basecaller (e.g. standard
  Guppy/Dorado) and not re-processed — no inosine indexes exist in read headers, so
  inosine_detector.py cannot produce coordinate-level calls
- The biology of interest is RNA modifications other than A-to-I inosine (e.g. m6A/m1A)
  — this workflow targets inosines specifically via the RNA002 inosine model and REDIportal
  A-to-I annotations
- The goal is modification detection in the latest SQK-RNA004 chemistry or via re-squiggling
  workflows rather than direct inosine basecalling of RNA002 dRNA fast5 data
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3170
tools:
- name: NanoSpeech
  role: Inosine-aware transformer basecaller producing fasta/fastq with inosine indexes
    in read headers, from which coordinate-level inosine calls are derived
  repo: https://github.com/F0nz0/NanoSpeech_basecaller
- name: inosine_detector.py
  role: Accessory script that converts inosine indexes in read headers into reference
    coordinates, generating per_read.bed and genome_space.tsv
  repo: https://github.com/F0nz0/NanoSpeech_basecaller
- name: minimap2
  role: Maps NanoSpeech-basecalled reads to a reference genome (splice preset) or
    transcriptome before inosine coordinate conversion
- name: samtools
  role: Filters SAM files (-F 2304), converts to sorted, indexed BAM files consumed
    by inosine_detector.py
- name: Oxford Nanopore Technology
  role: Source of direct RNA (dRNA) fast5 reads from hWT/hKO/hOE HEK293T samples over
    dsRNA loci targeted by ADARs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf709
    title: <i>Ab initio</i> detection of multiple epitranscriptomic modifications
      from Oxford nanopore technology direct RNA sequencing data
  source_content_hash: sha256:e789eae46dd3e3d1d792cdf430c9dbbf7e8188c7e206608f6be223a948a72f7b
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
  - Oxford Nanopore Technology
  - inosine_detector.py
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# rediportal-a-to-i-editing-site-intersection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Intersects NanoSpeech genome-space inosine calls ({bam}.genome_space.tsv) with REDIportal A-to-I editing annotations and computes per-sample inosine counts/frequencies at ADAR-targeted dsRNA loci to validate inosine detection on real dRNA data.

## When to use

Use when per-read inosine calls from NanoSpeech basecalling (BAM + genome_space.tsv) must be validated against known A-to-I editing sites in REDIportal, or when comparing inosine burden across ADAR-modulated samples (e.g. hWT/hKO/hOE).

## When NOT to use

- Input reads were basecalled with a modification-unaware basecaller (e.g. standard Guppy/Dorado) and not re-processed — no inosine indexes exist in read headers, so inosine_detector.py cannot produce coordinate-level calls
- The biology of interest is RNA modifications other than A-to-I inosine (e.g. m6A/m1A) — this workflow targets inosines specifically via the RNA002 inosine model and REDIportal A-to-I annotations
- The goal is modification detection in the latest SQK-RNA004 chemistry or via re-squiggling workflows rather than direct inosine basecalling of RNA002 dRNA fast5 data

## Inputs

- Sorted, indexed BAM files from minimap2 alignment (e.g. 'minimap2 -ax splice -uf -k 14 --secondary=no --MD' against a genome or 'map-ont' against a transcriptome) of NanoSpeech-basecalled hWT/hKO/hOE fastq/fasta reads
- Basecalled fasta/fastq files with inosine indexes in read headers
- REDIportal A-to-I editing annotations
- Reference genome or transcriptome
- Minimum inosine quality threshold for inosine_detector.py (-q)

## Outputs

- {bam}.per_read.bed — 7 columns: region, start, stop, read-id, inosine quality, strand, reference base
- {bam}.genome_space.tsv — 6 columns: region, start, strand, I_count, depth, I_freq
- Per-sample inosine counts and frequencies at dsRNA loci (hOE vs hKO vs hWT comparison)
- REDIportal overlap statistics for called inosines

## How to apply

First convert NanoSpeech read-header inosine indexes into reference coordinates by running inosine_detector.py with -b {bam}, -f {fasta/q} and a minimum inosine quality threshold (-q), which yields {bam}.per_read.bed (7 columns) and {bam}.genome_space.tsv (6 columns: region, start, strand, I_count, depth, I_freq). Then intersect the genome-space inosine calls with REDIportal A-to-I editing annotations and aggregate per-sample inosine counts/frequencies at dsRNA loci targeted by ADARs. The biological rationale is that an inosine-aware basecaller applied to real human dRNA should recover a high fraction of known editing events and show sample-level gradients of editing burden (more inosines in ADAR-overexpressing hOE than in ADAR-knockout hKO). Judgment of success combines schema checks on the output files, the hOE>hKO>hWT frequency ordering, and the REDIportal overlap rate (92% of hOE inosines matched known A-to-I events, Fig. 1E–F).

## Related tools

- **NanoSpeech** (Inosine-aware transformer basecaller producing fasta/fastq with inosine indexes in read headers, from which coordinate-level inosine calls are derived) — https://github.com/F0nz0/NanoSpeech_basecaller
- **inosine_detector.py** (Accessory script that converts inosine indexes in read headers into reference coordinates, generating per_read.bed and genome_space.tsv) — https://github.com/F0nz0/NanoSpeech_basecaller
- **minimap2** (Maps NanoSpeech-basecalled reads to a reference genome (splice preset) or transcriptome before inosine coordinate conversion)
- **samtools** (Filters SAM files (-F 2304), converts to sorted, indexed BAM files consumed by inosine_detector.py)
- **Oxford Nanopore Technology** (Source of direct RNA (dRNA) fast5 reads from hWT/hKO/hOE HEK293T samples over dsRNA loci targeted by ADARs)

## Examples

```
python3 inosine_detector.py -b {bam} -f {fasta/q} -q {minimum inosine quality}
```

## Evaluation signals

- {bam}.per_read.bed exists with exactly 7 columns (region, start, stop, read-id, inosine quality, strand, reference base)
- {bam}.genome_space.tsv exists with exactly 6 columns (region, start, strand, I_count, depth, I_freq)
- Aggregated frequencies reproduce the reported pattern of more inosines in hOE than hKO samples
- High overlap between called inosines and known REDIportal A-to-I events (92% of hOE inosines in Fig. 1E–F)
- SAM-to-BAM conversion used 'samtools view -b -F 2304' followed by sort and index, with plain-text SAM removed

## Limitations

- Primary validation relied on synthetic constructs; performance on real-world data depends on read count, quality, and contamination, and public RNA modification data can be of questionable quality
- Inference artefacts or 'hallucinations' can arise from highly similar chemical structures or intrinsic errors in modified samples
- NanoSpeech was primarily tested on the previous SQK-RNA002 chemistry; the single-modification inosine version is deprecated in favour of the multi-mod version

## Evidence

- [other] Run inosine_detector.py with -b {bam}, -f {fasta/q} and a minimum inosine quality threshold (-q) to convert the inosine indexes in the read headers into reference coordinates: [evidence span withheld — anchor status: not_found]
- [other] 92% of hOE inosines matched known A-to-I events in REDIportal: [evidence span withheld — anchor status: not_found]
