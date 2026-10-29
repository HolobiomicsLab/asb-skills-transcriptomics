---
name: simulated-randomers-training-data-augmentation
description: Use when building training datasets for ONT dRNA basecalling/anomaly
  models from public or IVT data of questionable quality, and inference artefacts
  or 'hallucinations' from near-identical modified structures must be mitigated before
  training NanoListener/IsolationForest models.
when_to_use_negative:
- When clean, high-quality IVT training data with known modification sites is already
  available and no public-data quality concerns exist
- When the goal is basecalling with a pre-trained modification-aware basecaller (e.g.
  Dorado, m6ABasecaller) rather than building a custom training set
- When working with DNA nanopore data — this strategy is grounded in ONT direct RNA
  (dRNA) reads from SQK-RNA002/SQK-RNA004 chemistries
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: NanoListener
  role: Extracts current chunks and kmers from f5c eventalign-processed dRNA reads
    to create the training datasets that are then augmented in silico with simulated
    randomers
- name: f5c
  role: Event-aligns dRNA reads so raw current chunks can be anchored to corresponding
    reference kmer sequences before augmentation
- name: scikit-learn
  role: Provides the IsolationForest anomaly model trained on the randomer-augmented,
    context-balanced dataset
- name: NanoSpeech
  role: Downstream transformer basecaller whose training data pipeline (NanoListener)
    benefits from randomer augmentation for robustness
  repo: https://github.com/F0nz0/NanoSpeech_basecaller
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf709
    title: <i>Ab initio</i> detection of multiple epitranscriptomic modifications
      from Oxford nanopore technology direct RNA sequencing data
  source_content_hash: sha256:0fe9148771aadecfc09025723fb0e069e68fdd799fe96c7c88d1da9b74311942
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
  - NanoListener
  - NanoSpeech
  - f5c
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# simulated-randomers-training-data-augmentation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Augments NanoListener-extracted ONT dRNA training datasets in silico by inserting simulated random sequence segments (randomers) into current chunks and reference kmers, so the downstream model becomes robust to inference artefacts or 'hallucinations' caused by low-quality public modification data.

## When to use

Use when building training datasets for ONT dRNA basecalling/anomaly models from public or IVT data of questionable quality, and inference artefacts or 'hallucinations' from near-identical modified structures must be mitigated before training NanoListener/IsolationForest models.

## When NOT to use

- When clean, high-quality IVT training data with known modification sites is already available and no public-data quality concerns exist
- When the goal is basecalling with a pre-trained modification-aware basecaller (e.g. Dorado, m6ABasecaller) rather than building a custom training set
- When working with DNA nanopore data — this strategy is grounded in ONT direct RNA (dRNA) reads from SQK-RNA002/SQK-RNA004 chemistries

## Inputs

- f5c eventalign output anchoring ONT dRNA read signal events to reference kmers
- NanoListener-extracted paired current chunks and reference kmer sequences (CTRL and COND samples)
- Reference fasta/fastq sequences

## Outputs

- Randomer-augmented training dataset of paired current chunks and kmers
- Context-balanced training set (random chunk sampling across sequence contexts)
- Trained model (e.g. scikit-learn IsolationForest) with per-read anomaly predictions at bona fide modification positions

## How to apply

First extract paired current chunks (variable length) and reference kmers from f5c eventalign-processed dRNA reads (CTRL and COND) using NanoListener, which works for any ONT sequencing chemistry and kit. Then, in silico, insert simulated randomers — random sequence segments — into the extracted chunks/kmers so the training set spans sequence contexts not observed in the source data; this mitigates 'hallucinations' arising from highly similar chemical structures, intrinsic errors in modified samples, and the questionable quality of public RNA modification datasets. Balance the augmented dataset with random chunk sampling across sequence contexts so frequent kmers do not dominate training. Finally train the downstream model (e.g. a scikit-learn IsolationForest anomaly model) on the augmented data and score reads at bona fide modification positions, flagging COND reads anomalous relative to CTRL counterparts. Acceptance of the resulting per-read prediction table is a qualitative completeness check, since the article reports no quantitative threshold for this validation step.

## Related tools

- **NanoListener** (Extracts current chunks and kmers from f5c eventalign-processed dRNA reads to create the training datasets that are then augmented in silico with simulated randomers)
- **f5c** (Event-aligns dRNA reads so raw current chunks can be anchored to corresponding reference kmer sequences before augmentation)
- **scikit-learn** (Provides the IsolationForest anomaly model trained on the randomer-augmented, context-balanced dataset)
- **NanoSpeech** (Downstream transformer basecaller whose training data pipeline (NanoListener) benefits from randomer augmentation for robustness) — https://github.com/F0nz0/NanoSpeech_basecaller

## Evaluation signals

- The augmented dataset contains both original and randomer-inserted chunk/kmer pairs derived from f5c eventalign output
- Random chunk sampling yields balanced kmer representation across sequence contexts — frequent kmers no longer dominate the training set
- The trained IsolationForest produces a per-read prediction table covering bona fide modification positions in both CTRL and COND samples with anomaly labels per read (qualitative completeness, no quantitative threshold reported)
- COND reads at bona fide modification positions are flagged as anomalies relative to CTRL counterparts (Fig. 2B-C behaviour)
- Model remains chemistry-independent: augmentation should not tie training to a specific ONT chemistry or kit

## Limitations

- Augmentation mitigates but cannot fully remove inference artefacts/'hallucinations' arising from highly similar chemical structures or intrinsic errors in modified samples
- The quality of public modification data is often questionable, limiting how reliable any training set built from it can be; there is also a lack of reliable benchmarks for training and validation
- Sample-level factors — number of available reads and their quality, contaminations, and sometimes unknown IVT experimental conditions — hamper training set generation even with augmentation
- Validation of the per-read prediction table is qualitative; the article reports no quantitative threshold

## Evidence

- [other] Augment the extracted dataset in silico with simulated randomers: [evidence span withheld — anchor status: not_found]
