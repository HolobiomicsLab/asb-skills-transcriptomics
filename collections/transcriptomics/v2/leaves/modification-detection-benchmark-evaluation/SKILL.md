---
name: modification-detection-benchmark-evaluation
description: Use when evaluating a modification-aware basecaller on reads from synthetic
  constructs with known modification ground truth, to score per-position detection
  (accuracy/F1/AUC) and compare mappability vs standard basecallers.
when_to_use_negative:
- Data are real biological samples without known modification positions — no ground
  truth exists to compute accuracy/F1/AUC against
- The goal is de novo modification discovery rather than benchmarking a basecaller
  against a validated standard
- Reads use a chemistry other than RNA002 for which the single-mod inosine checkpoint
  was not trained (e.g. newer SQK-RNA004 data need multi-mod models)
edam_operation: http://edamontology.org/operation_0322
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: NanoSpeech
  role: Inosine-aware transformer basecaller producing FASTQ with predicted inosine
    indexes in read headers; accessory script inosine_detector.py emits per-read and
    genome-space predictions
  repo: https://github.com/F0nz0/NanoSpeech_basecaller
- name: minimap2
  role: Aligns basecalled reads to reference genome/transcriptome with --secondary=no
    --MD, -k 14, presets -ax splice -uf or -ax map-ont
- name: samtools
  role: Filters SAM (view -b -F 2304), sorts, and indexes BAM for inosine_detector.py
    input
- name: TensorFlow
  role: Runtime for the NanoSpeech transformer model (tested with TF 2.7.0, CUDA 11.8,
    cuDNN 8.6)
- name: Guppy
  role: Modification-unaware baseline basecaller used to benchmark mapped-read fraction
    (~40% vs >90% for NanoSpeech)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf709
    title: <i>Ab initio</i> detection of multiple epitranscriptomic modifications
      from Oxford nanopore technology direct RNA sequencing data
  source_content_hash: sha256:87fc84a657bf3dc6ffc5ab4e4effcb86e456bbfc80a7d24107542153f3cd2741
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
  - Guppy
  - NanoSpeech
  - TensorFlow
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_0322
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# modification-detection-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks NanoSpeech inosine-aware basecalling against known inosine positions in synthetic pureI/pureG dRNA constructs, computing per-position accuracy, F1 and AUC, plus read-mappability comparison to a modification-unaware basecaller (Guppy).

## When to use

Use when evaluating a modification-aware basecaller on reads from synthetic constructs with known modification ground truth, to score per-position detection (accuracy/F1/AUC) and compare mappability vs standard basecallers.

## When NOT to use

- Data are real biological samples without known modification positions — no ground truth exists to compute accuracy/F1/AUC against
- The goal is de novo modification discovery rather than benchmarking a basecaller against a validated standard
- Reads use a chemistry other than RNA002 for which the single-mod inosine checkpoint was not trained (e.g. newer SQK-RNA004 data need multi-mod models)

## Inputs

- fast5 folder of dRNA (SQK-RNA002) reads from pureI (inosine-positive) and pureG (inosine-negative) synthetic constructs
- NanoSpeech model weights h5 file (NanoSpeech_Inosine_m23M_e71_d19Mrec_SINGLE_MOD_VER.h5)
- Reference genome or transcriptome for minimap2 alignment
- Known inosine positions of the synthetic constructs (ground truth for positive/negative evaluation)

## Outputs

- FASTQ with read headers carrying indexes of adenosines predicted as inosines
- Sorted, indexed BAM of basecalled reads
- {bam}.per_read.bed of per-read inosine predictions
- {bam}.genome_space.tsv with region, start, strand, I_count, depth, I_freq
- Benchmark metrics: per-position accuracy, F1-score, AUC; mapped-read fraction vs Guppy baseline

## How to apply

Run NanoSpeech single-modification inosine-aware basecalling (NanoSpeech.py) on the pureI/pureG fast5 folder with the RNA002 multi-species checkpoint NanoSpeech_Inosine_m23M_e71_d19Mrec_SINGLE_MOD_VER.h5, using --clip_outliers 20-185 and ~3 GB GPU memory per worker thread (-t ~15 on an A100-40GB), producing a FASTQ whose headers carry indexes of adenosines predicted as inosines. Align basecalled reads with minimap2 using --secondary=no --MD and -k 14 with preset -ax splice -uf (reference genome) or -ax map-ont (reference transcriptome). Filter and convert with 'samtools view -b -F 2304' piped to 'samtools sort -O BAM', index, then run inosine_detector.py with -b {BAM}, -f {FASTQ} and a minimum inosine quality threshold (-q) to emit per-read predictions ({bam}.per_read.bed) and genome-space aggregated predictions ({bam}.genome_space.tsv with region, start, strand, I_count, depth, I_freq). Score detections against the known inosine positions of pureI (positive) and pureG (negative) constructs, computing per-position accuracy, F1 and AUC, and compare NanoSpeech read mappability against Guppy (~40% mapped).

## Related tools

- **NanoSpeech** (Inosine-aware transformer basecaller producing FASTQ with predicted inosine indexes in read headers; accessory script inosine_detector.py emits per-read and genome-space predictions) — https://github.com/F0nz0/NanoSpeech_basecaller
- **minimap2** (Aligns basecalled reads to reference genome/transcriptome with --secondary=no --MD, -k 14, presets -ax splice -uf or -ax map-ont)
- **samtools** (Filters SAM (view -b -F 2304), sorts, and indexes BAM for inosine_detector.py input)
- **TensorFlow** (Runtime for the NanoSpeech transformer model (tested with TF 2.7.0, CUDA 11.8, cuDNN 8.6))
- **Guppy** (Modification-unaware baseline basecaller used to benchmark mapped-read fraction (~40% vs >90% for NanoSpeech))

## Examples

```
python3 NanoSpeech.py -d {fast5_folder_path} -o {out.fastq} -m models/NanoSpeech_Inosine_m23M_e71_d19Mrec_SINGLE_MOD_VER.h5 -t 15 -c 20-185
```

## Evaluation signals

- Per-position inosine detection accuracy must exceed 96% on the pureI sample (Fig. 1A, Supp. Figs. S3-S4)
- F1-score must exceed 95% and AUC should be ~1.0 for inosine detection
- Negligible false positives on pureG negative-control samples lacking inosines
- NanoSpeech-basecalled read mapping fraction must be above 90%, versus ~40% for Guppy
- genome_space.tsv rows contain the expected columns: region, start, strand, I_count, depth, I_freq

## Limitations

- Primarily tested on synthetic constructs as a proof of concept; real-world data may behave differently
- NanoSpeech models demonstrated performance on the older RNA002 chemistry; the single-mod inosine version will be deprecated in favour of the multi-mod version
- Inference artefacts or 'hallucinations' can arise from highly similar chemical structures or intrinsic errors in modified samples
- Lack of reliable benchmarks and unavailable training datasets of existing tools impair unbiased, reproducible modification-detection comparisons

## Evidence

- [results] NanoSpeech detected inosines in the pureI sample with high accuracy (accuracy >96%, f1-score >95%, and AUC ~1.0): [evidence span withheld — anchor status: not_found]
