---
name: pod5-raw-signal-processing
description: Use when raw ONT dRNA pod5 or fast5 signal files must be basecalled in
  a single step that simultaneously calls canonical (A,C,G,U) and modified bases (I,
  m6A, m1A, m5C, Psi), avoiding re-squiggling and separate modification models.
when_to_use_negative:
- Input is already basecalled FASTQ/BAM data without raw ionic signals — NanoSpeech
  requires raw pod5/fast5 signal input
- The analysis needs modification probabilities requiring post-processing with ModKit-style
  tools — NanoSpeech calls modified bases directly during basecalling without such
  steps
- Only DNA nanopore reads are present — the pipeline is designed for direct RNA sequencing
  data from ONT chemistries (SQK-RNA002, SQK-RNA004)
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: NanoSpeech
  role: Modification-aware transformer basecaller that converts raw pod5/fast5 dRNA
    signals into fasta/fastq with modified-base indexes in read headers
  repo: https://github.com/F0nz0/NanoSpeech_basecaller
- name: minimap2
  role: Maps NanoSpeech basecalled fasta/fastq reads to reference transcriptome/genome
    using map-ont or splice -uf presets (-k 14 --secondary=no --MD)
- name: samtools
  role: Filters SAM output ('samtools view -b -F 2304'), converts to sorted and indexed
    BAM files
- name: modification_detector.py
  role: NanoSpeech accessory script that uses BAM files and modification indexes in
    fasta/q headers to generate per-read BED and genome-space aggregated predictions
  repo: https://github.com/F0nz0/NanoSpeech_basecaller
- name: Dorado
  role: ONT's modification-aware basecaller used as the state-of-the-art comparison
    baseline (with ModKit post-processing) for benchmarking NanoSpeech outputs
  repo: https://github.com/nanoporetech/dorado
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf709
    title: <i>Ab initio</i> detection of multiple epitranscriptomic modifications
      from Oxford nanopore technology direct RNA sequencing data
  source_content_hash: sha256:0d8279d933a464191dae249405e1f32d6845a6e25d149df0f9357bbe323fe2e1
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
  - minimap2
  - modification_detector.py
  - samtools
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# pod5-raw-signal-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A skill for converting ONT direct RNA raw pod5/fast5 ionic-current signals into modification-aware basecalls via NanoSpeech's transformer model, then mapping, filtering, and aggregating modification calls at per-read and genome/transcriptome-space resolution.

## When to use

Use when raw ONT dRNA pod5 or fast5 signal files must be basecalled in a single step that simultaneously calls canonical (A,C,G,U) and modified bases (I, m6A, m1A, m5C, Psi), avoiding re-squiggling and separate modification models.

## When NOT to use

- Input is already basecalled FASTQ/BAM data without raw ionic signals — NanoSpeech requires raw pod5/fast5 signal input
- The analysis needs modification probabilities requiring post-processing with ModKit-style tools — NanoSpeech calls modified bases directly during basecalling without such steps
- Only DNA nanopore reads are present — the pipeline is designed for direct RNA sequencing data from ONT chemistries (SQK-RNA002, SQK-RNA004)

## Inputs

- Raw ONT dRNA pod5 or fast5 signal files (folder of reads)
- Pretrained NanoSpeech model weights (.h5) with paired .h5.cfg configuration file
- Reference transcriptome (map-ont preset) or reference genome (splice -uf preset) in FASTA
- Minimum base/inosine quality threshold for modification_detector.py

## Outputs

- FASTA/FASTQ basecalled reads with per-modification base indexes in read headers
- Sorted, indexed BAM alignments filtered with 'samtools view -b -F 2304'
- Per-read modification calls in *.per_read.bed format
- Genome-space (transcriptome-space) aggregated modification predictions in *.genome_space.tsv with alternative modified nucleotide column

## How to apply

Run NanoSpeech_multi_mod.py on the raw pod5/fast5 folder with the chemistry-appropriate model weights (-m, e.g. the RNA002 nine-mod model NanoSpeech_RNA002_16classes_9mods_m50Mplus_DS4M_E54.h5 or the RNA004 multi-modification model) plus -d for the input folder, -t for parallel model threads, and -o for the output fasta/q; the paired *.h5.cfg file auto-loads transformer parameters. NanoSpeech emits a fasta/fastq whose read headers carry per-modification base indexes (e.g. I_A=..., M_A=...). Map the basecalled reads with minimap2 using '-ax map-ont -k 14 --secondary=no --MD' against a reference transcriptome or '-ax splice -uf -k 14 --secondary=no --MD' against a genome; filter the SAM with 'samtools view -b -F 2304', sort and index the BAM, and delete the intermediate SAM. Then run modification_detector.py (-b BAM, -f fasta/q, -q minimum base-quality threshold) to translate read-header modification indexes into per-read BED calls and a genome-space aggregated table. Rationale: the transformer's expanded dictionary removes the classical pre-processing chain (modification-unaware basecalling, re-squiggling, ad hoc models), so modification detection happens during basecalling itself.

## Related tools

- **NanoSpeech** (Modification-aware transformer basecaller that converts raw pod5/fast5 dRNA signals into fasta/fastq with modified-base indexes in read headers) — https://github.com/F0nz0/NanoSpeech_basecaller
- **minimap2** (Maps NanoSpeech basecalled fasta/fastq reads to reference transcriptome/genome using map-ont or splice -uf presets (-k 14 --secondary=no --MD))
- **samtools** (Filters SAM output ('samtools view -b -F 2304'), converts to sorted and indexed BAM files)
- **modification_detector.py** (NanoSpeech accessory script that uses BAM files and modification indexes in fasta/q headers to generate per-read BED and genome-space aggregated predictions) — https://github.com/F0nz0/NanoSpeech_basecaller
- **Dorado** (ONT's modification-aware basecaller used as the state-of-the-art comparison baseline (with ModKit post-processing) for benchmarking NanoSpeech outputs) — https://github.com/nanoporetech/dorado

## Examples

```
python3 NanoSpeech_multi_mod.py -d {pod5_folder} -o {out.fastq} -m models/NanoSpeech_RNA002_16classes_9mods_m50Mplus_DS4M_E54.h5 -t 15 -f
```

## Evaluation signals

- Output fasta/q read headers contain per-modification base indexes in the expected format (e.g. I_A=..., M_A=...) for each read
- BAM is sorted, indexed, and was filtered with -F 2304 (no secondary/supplementary alignments); intermediate plain-text SAM removed
- modification_detector.py emits both a *.per_read.bed and a *.genome_space.tsv file
- Computed per-read identity, consensus accuracy, and insertion rates are comparable to state-of-the-art ONT software (Dorado) and reproduce the reported Fig. 3A–K relationships
- Per-sample modification identification shows low false-positive rate on unmodified IVT controls; beware the known m1A-miscalled-as-m6A pattern (cross-model correlations r=0.451 vs 0.709)

## Limitations

- In the nine-modification RNA002 multi-model, m1A was erroneously miscalled as its chemical isomer m6A; models for highly similar chemical structures are still under development
- Primarily tested on synthetic constructs (IVT oligos, fully modified constructs) as a proof of concept; real-world public data quality is questionable
- At least 3 GB of GPU memory per parallel basecalling thread is required (e.g. ~15 threads on an NVIDIA A100 40GB); tested on CentOS-7 with CUDA 11.8, cuDNN, TensorFlow 2.7.0
- Training datasets and reliable benchmarks for training/validation are lacking, limiting unbiased and reproducible performance comparisons and potentially causing inference artefacts or 'hallucinations'

## Evidence
