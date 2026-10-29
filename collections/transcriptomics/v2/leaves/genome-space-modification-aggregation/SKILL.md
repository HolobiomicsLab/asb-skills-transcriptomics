---
name: genome-space-modification-aggregation
description: Use when NanoSpeech basecalled fasta/q read headers carry modification
  (inosine or multi-mod) indexes and you need them mapped to reference coordinates
  and aggregated into per-site modification counts/frequencies.
when_to_use_negative:
- Reads were basecalled with a modification-unaware basecaller (standard Guppy/Dorado),
  so no modification indexes exist in the fasta/q headers for the detector scripts
  to convert
- The BAM is not filtered (e.g. secondary/supplementary alignments not removed with
  'samtools view -b -F 2304'), sorted, and indexed
- You need ab initio modification detection from raw currents — that is the basecalling
  step (NanoSpeech.py / NanoSpeech_multi_mod.py), not this aggregation step
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0160
tools:
- name: NanoSpeech (inosine_detector.py / modification_detector.py)
  role: Accessory scripts that read BAM files and modification indexes in fasta/q
    headers to produce per-read BED and genome-space aggregated TSV outputs
  repo: https://github.com/F0nz0/NanoSpeech_basecaller
- name: minimap2
  role: Maps NanoSpeech basecalled fasta/fastq reads to the reference genome (splice
    preset) or transcriptome (map-ont preset) prior to aggregation
- name: samtools
  role: Filters (-F 2304), sorts, and indexes the SAM/BAM required as detector input
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf709
    title: <i>Ab initio</i> detection of multiple epitranscriptomic modifications
      from Oxford nanopore technology direct RNA sequencing data
  source_content_hash: sha256:84a2a0e1ed35730312317b770d84787d7d208abc217bf3cc6f0b3d9d8a851b6c
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
  - NanoSpeech (inosine_detector.py / modification_detector.py)
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# genome-space-modification-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts per-read modification calls from NanoSpeech modification-aware basecalling into reference-coordinate annotations and aggregated genome/transcriptome-space modification counts and frequencies, using NanoSpeech's accessory detector scripts on sorted, indexed BAM files.

## When to use

Use when NanoSpeech basecalled fasta/q read headers carry modification (inosine or multi-mod) indexes and you need them mapped to reference coordinates and aggregated into per-site modification counts/frequencies.

## When NOT to use

- Reads were basecalled with a modification-unaware basecaller (standard Guppy/Dorado), so no modification indexes exist in the fasta/q headers for the detector scripts to convert
- The BAM is not filtered (e.g. secondary/supplementary alignments not removed with 'samtools view -b -F 2304'), sorted, and indexed
- You need ab initio modification detection from raw currents — that is the basecalling step (NanoSpeech.py / NanoSpeech_multi_mod.py), not this aggregation step

## Inputs

- Sorted, indexed BAM file produced by filtering the minimap2 SAM ('samtools view -b -F 2304 | samtools sort -O BAM' then 'samtools index')
- NanoSpeech basecalled fasta/fastq file with modification indexes encoded in read headers
- Minimum modification/inosine quality threshold (-q parameter)
- Optional: REDIportal A-to-I editing annotations for overlap analysis

## Outputs

- {bam}.per_read.bed — per-read modification calls (7 columns: region, start, stop, read-id, inosine quality, strand, reference base)
- {bam}.genome_space.tsv — genome/transcriptome-space aggregated predictions (6 columns: region, start, strand, I_count, depth, I_freq)

## How to apply

After NanoSpeech basecalling (e.g. with the RNA002 inosine model NanoSpeech_Inosine_m23M_e71_d19Mrec_SINGLE_MOD_VER.h5) and mapping with minimap2 presets like 'minimap2 -ax splice -uf -k 14 --secondary=no --MD', filter and convert SAM to sorted, indexed BAM with 'samtools view -b -F 2304 | samtools sort -O BAM' plus samtools index. Then run inosine_detector.py (or modification_detector.py for the multi-mod version) with -b {bam}, -f {fasta/q} and a minimum modification quality threshold (-q); the script converts modification indexes in the read headers into reference coordinates. This yields {bam}.per_read.bed (7 columns: region, start, stop, read-id, inosine quality, strand, reference base) and {bam}.genome_space.tsv (6 columns: region, start, strand, I_count, depth, I_freq). Because NanoSpeech calls modified bases ab initio during basecalling, no post-processing re-squiggling tools (e.g. ModiKit or similar) are required. Downstream, intersect genome-space calls with REDIportal A-to-I annotations and compare per-sample inosine counts/frequencies (e.g. hOE vs hKO vs hWT at dsRNA loci).

## Related tools

- **NanoSpeech (inosine_detector.py / modification_detector.py)** (Accessory scripts that read BAM files and modification indexes in fasta/q headers to produce per-read BED and genome-space aggregated TSV outputs) — https://github.com/F0nz0/NanoSpeech_basecaller
- **minimap2** (Maps NanoSpeech basecalled fasta/fastq reads to the reference genome (splice preset) or transcriptome (map-ont preset) prior to aggregation)
- **samtools** (Filters (-F 2304), sorts, and indexes the SAM/BAM required as detector input)

## Examples

```
python3 inosine_detector.py -b {bam} -f {fasta/q} -q {minimum inosine quality}
```

## Evaluation signals

- {bam}.per_read.bed is produced with exactly the required 7-column layout: region, start, stop, read-id, inosine quality, strand, reference base
- {bam}.genome_space.tsv is produced with exactly the required 6-column layout: region, start, strand, I_count, depth, I_freq
- I_freq values in genome_space.tsv are consistent with I_count/depth at each site
- On hWT/hKO/hOE HEK293T dsRNA data, aggregated calls show more inosines in hOE than hKO, with ~92% of hOE inosines overlapping REDIportal A-to-I events
- Raising the -q quality threshold reduces the number of retained per-read modification entries

## Limitations

- Results depend on the chosen minimum modification quality threshold (-q); no universal cutoff is specified in the context
- NanoSpeech was primarily tested on synthetic constructs as a proof of concept; public RNA modification data can be of questionable quality, and lack of reliable benchmarks impairs validation
- Inference artefacts or 'hallucinations' can arise for highly similar chemical structures; streamlined models for such structures are still under development

## Evidence
