---
name: f5c-eventalign-kmer-anchoring
description: Use when building training data from ONT direct RNA reads that pairs
  variable-length raw current chunks with reference kmers — e.g. before NanoListener
  dataset construction, simulated-randomer augmentation, or IsolationForest training
  on f5c eventalign output.
when_to_use_negative:
- The goal is direct modification-aware basecalling of raw reads in a single step
  (e.g. Dorado, m6ABasecaller, NanoSpeech) rather than building custom training datasets
  via re-squiggling-style anchoring
- No reference sequence is available — the chunk-to-kmer anchoring fundamentally requires
  aligning reads to a reference via f5c eventalign
- Input is not ONT dRNA raw signal data (the chunk/kmer pairing depends on ionic-current
  events from nanopore dRNA runs)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: f5c
  role: Performs event alignment of ONT dRNA raw signals against the reference so
    current chunks can be anchored to their reference kmers
- name: NanoListener
  role: Consumes the f5c eventalign output to extract paired current chunks and reference
    kmers for training-dataset construction
- name: minimap2
  role: Maps basecalled reads to the reference (splice-aware, secondary alignments
    disabled) as the prerequisite for event alignment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf709
    title: <i>Ab initio</i> detection of multiple epitranscriptomic modifications
      from Oxford nanopore technology direct RNA sequencing data
  source_content_hash: sha256:ec31831d4d0a3033c9562547a0d9dc76cb1a6406a23d9acfbeabaee7490dbeaf
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/F0nz0/NanoSpeech_basecaller
  tools:
  - NanoListener
  - NanoSpeech
  - f5c
  - minimap2
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# f5c eventalign kmer anchoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Anchors raw ONT dRNA ionic-current chunks to their corresponding reference kmer sequences using f5c eventalign output, creating the paired (current chunk, kmer) training data consumed by NanoListener and downstream IsolationForest modification models.

## When to use

Use when building training data from ONT direct RNA reads that pairs variable-length raw current chunks with reference kmers — e.g. before NanoListener dataset construction, simulated-randomer augmentation, or IsolationForest training on f5c eventalign output.

## When NOT to use

- The goal is direct modification-aware basecalling of raw reads in a single step (e.g. Dorado, m6ABasecaller, NanoSpeech) rather than building custom training datasets via re-squiggling-style anchoring
- No reference sequence is available — the chunk-to-kmer anchoring fundamentally requires aligning reads to a reference via f5c eventalign
- Input is not ONT dRNA raw signal data (the chunk/kmer pairing depends on ionic-current events from nanopore dRNA runs)

## Inputs

- ONT direct RNA (dRNA) raw signal files (fast5/pod5)
- Basecalled reads (FASTA/FASTQ) from modification-unaware basecalling
- Reference transcriptome/genome FASTA with known modification positions
- Sorted, indexed BAM alignments of basecalled reads

## Outputs

- Event-aligned reads with signal events anchored to reference kmers (f5c eventalign output)
- Paired (current chunk, reference kmer) training instances extracted by NanoListener
- Per-read prediction table with anomaly labels at bona fide modification positions (CTRL vs COND)

## How to apply

First map basecalled dRNA reads to the reference transcriptome with minimap2 using the article's splice preset (e.g. '-ax splice -uf -k 14 --secondary=no --MD'), then filter the SAM files and convert them to sorted, indexed BAM files ('samtools view -b -F 2304' followed by 'samtools sort -O BAM'). Run f5c eventalign on the signal-aligned reads so that each read's signal events are anchored to reference positions. NanoListener then leverages the f5c eventalign output to extract current chunks of variable length together with the corresponding reference kmers, and this works for any ONT sequencing chemistry and kit. The extracted paired dataset can subsequently be augmented in silico with simulated randomers (random sequence insertions) and balanced-random chunk sampling across sequence contexts to prevent frequent kmers from dominating training of the IsolationForest anomaly model.

## Related tools

- **f5c** (Performs event alignment of ONT dRNA raw signals against the reference so current chunks can be anchored to their reference kmers)
- **NanoListener** (Consumes the f5c eventalign output to extract paired current chunks and reference kmers for training-dataset construction)
- **minimap2** (Maps basecalled reads to the reference (splice-aware, secondary alignments disabled) as the prerequisite for event alignment)

## Examples

```
minimap2 -ax splice -uf -k 14 --secondary=no --MD ref.fa basecalled.fastq > aligned.sam
```

## Evaluation signals

- Every extracted current chunk is paired with the correct reference kmer from the eventalign output (no unanchored chunks in the training set)
- The per-read prediction table covers bona fide modification positions in both CTRL and COND samples with an anomaly label per read — the article reports no quantitative threshold, so acceptance is a qualitative completeness check
- Balanced random chunk sampling across sequence contexts is applied so frequent kmers do not dominate the training set
- Alignment files are correctly filtered and sorted BAMs (samtools view -b -F 2304, then samtools sort -O BAM) before event alignment
- The pipeline remains chemistry-independent — the same extraction works on reads from different ONT chemistries/kits (e.g. SQK-RNA002 and SQK-RNA004)

## Limitations

- No quantitative threshold is reported for validating the per-read prediction table; acceptance is a qualitative completeness check of labelled per-read predictions
- Low-quality public modification data and highly similar chemical structures hamper robust training-set generation, leading to inference artefacts or 'hallucinations' that the simulated-randomer augmentation only mitigates
- Frequent kmers can dominate the training set unless balanced random chunk sampling across sequence contexts is applied
- Primarily tested on synthetic constructs as a proof of concept; sample-level factors (read number/quality, contaminations, unknown IVT conditions) can impair training-set generation

## Evidence
