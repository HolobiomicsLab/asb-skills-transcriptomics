---
name: nanopore-drna-modification-aware-basecalling
description: Use when raw ONT dRNA fast5/pod5 files (SQK-RNA002 or SQK-RNA004) must
  be basecalled while simultaneously detecting modifications (e.g., inosine, m6A)
  without re-squiggling or separate modification-scoring models.
when_to_use_negative:
- The input is already basecalled FASTQ/BAM without raw signal — modification-aware
  basecalling requires raw fast5/pod5 ionic currents
- Latest ONT chemistries (e.g., SQK-RNA004 with Dorado models) are in use and a production-grade
  supported basecaller is preferred over this prototype
- The goal is highly similar chemical isomers (e.g., m1A vs m6A) with a model not
  specifically trained to distinguish them — the nine-mod model erroneously miscalled
  m1A as m6A
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: NanoSpeech
  role: Transformer-based modification-aware basecaller that converts raw dRNA currents
    into fasta/fastq with expanded dictionary (A, C, G, U + mod bases) and supplies
    inosine_detector.py / modification_detector.py accessory scripts
  repo: https://github.com/F0nz0/NanoSpeech_basecaller
- name: minimap2
  role: Maps basecalled fasta/fastq reads to reference genome (-ax splice -uf) or
    transcriptome (-ax map-ont) with -k 14 --secondary=no --MD
- name: samtools
  role: Filters SAM (-b -F 2304), sorts and indexes to BAM for downstream modification
    coordinate conversion
- name: TensorFlow
  role: Deep-learning framework underlying the NanoSpeech transformer models (tested
    with TensorFlow v.2.7.0)
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_006
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf709
    title: <i>Ab initio</i> detection of multiple epitranscriptomic modifications
      from Oxford nanopore technology direct RNA sequencing data
  source_content_hash: sha256:bc5895339346959e35a0e9e2ac23343136faacd47386acb0cf418dbcbee762dc
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
  - TensorFlow
  - minimap2
  - modification_detector.py
  - samtools
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# nanopore-drna-modification-aware-basecalling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Single-step ab initio detection of modified RNA bases from Oxford Nanopore dRNA raw signals using the NanoSpeech transformer basecaller with an expanded nucleotide vocabulary (A, C, G, U + modifications such as I and m6A), followed by reference mapping and per-read/genome-space modification localization.

## When to use

Use when raw ONT dRNA fast5/pod5 files (SQK-RNA002 or SQK-RNA004) must be basecalled while simultaneously detecting modifications (e.g., inosine, m6A) without re-squiggling or separate modification-scoring models.

## When NOT to use

- The input is already basecalled FASTQ/BAM without raw signal — modification-aware basecalling requires raw fast5/pod5 ionic currents
- Latest ONT chemistries (e.g., SQK-RNA004 with Dorado models) are in use and a production-grade supported basecaller is preferred over this prototype
- The goal is highly similar chemical isomers (e.g., m1A vs m6A) with a model not specifically trained to distinguish them — the nine-mod model erroneously miscalled m1A as m6A

## Inputs

- Raw ONT dRNA fast5 files (single-mod script) or pod5/fast5 files (multi-mod script)
- Pretrained NanoSpeech model weights (.h5) with coupled *.h5.cfg configuration (e.g., NanoSpeech_Inosine_m23M_e71_d19Mrec_SINGLE_MOD_VER.h5, NanoSpeech_RNA002_16classes_9mods_m50Mplus_DS4M_E54.h5)
- Reference genome or transcriptome (fasta) for minimap2 mapping
- Known modified-base positions (e.g., synthetic construct design or REDIportal A-to-I annotations) for validation

## Outputs

- fasta/fastq basecalled reads with modified-base indexes (e.g., I_A=..., M_A=...) in read headers
- Sorted, indexed BAM files from minimap2 alignments
- {bam}.per_read.bed (7 columns: region, start, stop, read-id, inosine quality, strand, reference base)
- {bam}.genome_space.tsv (6 columns: region, start, strand, I_count, depth, I_freq)

## How to apply

Basecall the raw fast5/pod5 folder with NanoSpeech.py (single-mod, e.g. the RNA002 multi-species inosine model NanoSpeech_Inosine_m23M_e71_d19Mrec_SINGLE_MOD_VER.h5) or NanoSpeech_multi_mod.py (e.g. NanoSpeech_RNA002_16classes_9mods_m50Mplus_DS4M_E54.h5), using --clip_outliers 20-185 and ~3 GB GPU memory per parallel worker thread (~15 threads on an A100-40GB); the output fasta/fastq read headers carry indexes of predicted modified bases. Map basecalled reads with minimap2 using -k 14 --secondary=no --MD with preset -ax splice -uf for a reference genome or -ax map-ont for a reference transcriptome, then filter and convert the SAM with 'samtools view -b -F 2304' piped to 'samtools sort -O BAM' and index. Run inosine_detector.py or modification_detector.py with -b {BAM}, -f {fasta/q} and a minimum quality threshold (-q) to translate read-header modification indexes into reference coordinates, producing {bam}.per_read.bed and {bam}.genome_space.tsv (region, start, strand, I_count, depth, I_freq). Decide on the model (single-mod vs nine-mod multi-model) based on the expected modification set; score calls against known modified positions to validate.

## Related tools

- **NanoSpeech** (Transformer-based modification-aware basecaller that converts raw dRNA currents into fasta/fastq with expanded dictionary (A, C, G, U + mod bases) and supplies inosine_detector.py / modification_detector.py accessory scripts) — https://github.com/F0nz0/NanoSpeech_basecaller
- **minimap2** (Maps basecalled fasta/fastq reads to reference genome (-ax splice -uf) or transcriptome (-ax map-ont) with -k 14 --secondary=no --MD)
- **samtools** (Filters SAM (-b -F 2304), sorts and indexes to BAM for downstream modification coordinate conversion)
- **TensorFlow** (Deep-learning framework underlying the NanoSpeech transformer models (tested with TensorFlow v.2.7.0))

## Examples

```
python3 NanoSpeech.py -d {fast5_folder_path} -o {output.fastq} -m models/NanoSpeech_Inosine_m23M_e71_d19Mrec_SINGLE_MOD_VER.h5 -t 15 -c 20-185
```

## Evaluation signals

- Inosine detection on pureI/pureG synthetic reads must reach accuracy >96%, F1-score >95% and AUC ~1.0 with negligible false negatives in pureG samples
- Mappability check: NanoSpeech-mapped read fraction above 90% versus ~40% for modification-unaware Guppy basecalling
- m6A-I-aware model: per-class precision ≥0.98 and recall ≈0.99 for canonical, inosine and m6A classes with zero I↔m6A cross-misclassification
- Output schema check: per_read.bed has 7 columns and genome_space.tsv has 6 columns (region, start, strand, I_count, depth, I_freq)
- On hWT/hKO/hOE HEK293T samples, hOE shows more inosines than hKO, and ~92% of hOE inosines overlap REDIportal A-to-I events

## Limitations

- The single-modification NanoSpeech.py version will be deprecated in future releases in favour of the multi-mod version
- The nine-modification multi-model erroneously miscalled m1A as its chemical isomer m6A; streamlined models for highly similar chemical structures are still under development
- Performance was primarily demonstrated on synthetic IVT constructs as proof of concept; questionable quality of public RNA modification data and lack of reliable benchmarks hamper real-world training and validation
- Requires GPU resources — at least 3 GB of GPU memory per parallel basecaller worker thread

## Evidence

- [results] NanoSpeech detected inosines in the pureI sample with high accuracy (accuracy >96%, f1-score >95%, and AUC ~1.0): [evidence span withheld — anchor status: not_found]
