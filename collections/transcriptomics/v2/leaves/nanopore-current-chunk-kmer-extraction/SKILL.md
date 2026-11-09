---
name: nanopore-current-chunk-kmer-extraction
description: Use when building a training dataset from ONT dRNA fast5/pod5 signal
  data and you must pair variable-length raw current chunks with their reference kmers
  to train a modification-detection model.
when_to_use_negative:
- You need modification-aware basecalling directly from raw signals — use NanoSpeech
  or Dorado instead, since the extraction pipeline is a training-set builder, not
  a basecaller
- You already possess a validated training dataset — the article notes that training
  datasets used by current basecalling tools are often unavailable but that rebuilding
  via randomer augmentation is only needed when robust data are missing
- Input reads are already basecalled FASTQ/BAM without attached raw signal; the skill
  anchors current chunks to kmers, which requires raw squiggle events aligned via
  f5c eventalign
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: NanoListener
  role: Extracts paired current chunks and reference kmers from f5c eventalign-ed
    dRNA reads; supports any ONT chemistry and kit
- name: f5c
  role: Produces the eventalign output used to anchor variable-length current chunks
    on the reference sequence
- name: scikit-learn
  role: Provides the IsolationForest anomaly model trained on the extracted chunk/kmer
    dataset to flag modified reads
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf709
    title: <i>Ab initio</i> detection of multiple epitranscriptomic modifications
      from Oxford nanopore technology direct RNA sequencing data
  source_content_hash: sha256:a206ad8bcc73d58e849525daad98a996a95eaeb2393401d56cccbcc39bdf1ef9
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
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# nanopore-current-chunk-kmer-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

NanoListener-based training-dataset construction for ONT direct RNA (dRNA) reads: raw ionic-current chunks are anchored to their corresponding reference kmer sequences via f5c eventalign, augmented in silico with simulated randomers, and used to train an IsolationForest anomaly model for detecting modified reads.

## When to use

Use when building a training dataset from ONT dRNA fast5/pod5 signal data and you must pair variable-length raw current chunks with their reference kmers to train a modification-detection model.

## When NOT to use

- You need modification-aware basecalling directly from raw signals — use NanoSpeech or Dorado instead, since the extraction pipeline is a training-set builder, not a basecaller
- You already possess a validated training dataset — the article notes that training datasets used by current basecalling tools are often unavailable but that rebuilding via randomer augmentation is only needed when robust data are missing
- Input reads are already basecalled FASTQ/BAM without attached raw signal; the skill anchors current chunks to kmers, which requires raw squiggle events aligned via f5c eventalign

## Inputs

- ONN dRNA fast5/pod5 raw signal reads from CTRL and COND samples (SQK-RNA002 or SQK-RNA004 chemistry)
- Reference transcript/genome FASTA sequence
- f5c eventalign output anchoring signal events to reference positions

## Outputs

- Paired (current chunk, reference kmer) training instances suitable for IsolationForest training
- Per-read anomaly prediction table covering bona fide modification positions in both CTRL and COND samples with anomaly labels per read
- Balanced training set across sequence contexts after simulated-randomer augmentation

## How to apply

First align dRNA reads from CTRL and COND samples against the reference sequence with f5c eventalign so each read's signal events are anchored to reference kmers. Then use NanoListener to extract paired current chunks and kmers from the event-aligned reads; this works for any ONT sequencing chemistry and kit. Augment the extracted dataset in silico with simulated randomers (random sequence insertions) to mitigate inference artefacts or 'hallucinations' arising from the questionable quality of public RNA-modification data. Apply balanced random chunk sampling across sequence contexts so that frequent kmers do not dominate the training set. Finally, train an IsolationForest anomaly model (scikit-learn) and score reads at bona fide modification positions, flagging modified reads in COND samples relative to CTRL counterparts as anomalies.

## Related tools

- **NanoListener** (Extracts paired current chunks and reference kmers from f5c eventalign-ed dRNA reads; supports any ONT chemistry and kit)
- **f5c** (Produces the eventalign output used to anchor variable-length current chunks on the reference sequence)
- **scikit-learn** (Provides the IsolationForest anomaly model trained on the extracted chunk/kmer dataset to flag modified reads)

## Evaluation signals

- Every extracted current chunk can be traced back to an f5c eventalign anchor, so each chunk has a known reference kmer label
- The per-read prediction table covers all bona fide modification positions in both CTRL and COND samples with an anomaly label per read (the article reports no quantitative threshold, so acceptance is a qualitative completeness check)
- Kmer/sequence-context frequency distribution in the final training set is balanced after random chunk sampling, i.e. frequent kmers do not dominate
- Trained IsolationForest flags COND reads at bona fide modification positions as anomalies while CTRL counterparts are scored normally (as in Fig. 2B-C)
- Augmented reads containing simulated randomers are basecalled/handled without inference artefacts or 'hallucinations' relative to the unmodified reference

## Limitations

- Public modification data of questionable quality can hamper training-set generation and lead to inference artefacts or 'hallucinations' from highly similar chemical structures or intrinsic errors in modified samples
- Sample-level factors such as the number of available reads, their quality, contaminations, and unknown IVT conditions impair training-set generation
- The article reports no quantitative threshold for validating the per-read prediction table, so this step is only a qualitative completeness check

## Evidence
