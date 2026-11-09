---
name: isolation-forest-anomaly-detection-of-modified-reads
description: Use when you have event-aligned ONT direct RNA reads from paired CTRL
  and COND samples and need per-read modified/unmodified labels at bona fide modification
  positions without a pretrained modification-aware basecaller.
when_to_use_negative:
- A modification-aware basecaller with an expanded dictionary (e.g. NanoSpeech or
  Dorado modification models) is already available and pretrained for your chemistry/modification.
- Input reads are DNA nanopore reads rather than ONT direct RNA (dRNA) sequencing
  data.
- No unmodified CTRL counterpart sample exists to define the baseline signal distribution
  for anomaly scoring.
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3474
tools:
- name: NanoListener
  role: Extracts current chunks and kmers from event-aligned dRNA reads and applies
    the simulated randomers strategy to build the training dataset
- name: f5c
  role: Performs event alignment that anchors each read's signal events to reference
    kmers, defining chunk-to-kmer pairing
- name: scikit-learn
  role: Provides the IsolationForest implementation used to train the anomaly model
    and label per-read predictions
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf709
    title: <i>Ab initio</i> detection of multiple epitranscriptomic modifications
      from Oxford nanopore technology direct RNA sequencing data
  source_content_hash: sha256:0db214431539b62d67c84deb4d09705b0beaa36d2d9da81ebcf240cf0c3ab61e
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
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# isolation-forest-anomaly-detection-of-modified-reads

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a NanoListener training set by anchoring raw ONT dRNA current chunks to reference kmers via f5c eventalign, augments it in silico with simulated randomers, and trains an IsolationForest (scikit-learn) model whose anomaly labels flag modified reads in COND samples relative to CTRL counterparts.

## When to use

Use when you have event-aligned ONT direct RNA reads from paired CTRL and COND samples and need per-read modified/unmodified labels at bona fide modification positions without a pretrained modification-aware basecaller.

## When NOT to use

- A modification-aware basecaller with an expanded dictionary (e.g. NanoSpeech or Dorado modification models) is already available and pretrained for your chemistry/modification.
- Input reads are DNA nanopore reads rather than ONT direct RNA (dRNA) sequencing data.
- No unmodified CTRL counterpart sample exists to define the baseline signal distribution for anomaly scoring.

## Inputs

- Raw ONT dRNA fast5/pod5 reads from CTRL and COND samples
- Reference transcriptome/genome sequence (fasta)
- f5c eventalign output aligning signal events to reference kmers
- Bona fide modification positions (known modified sites for scoring)

## Outputs

- Training dataset of paired current chunks and reference kmers
- Trained IsolationForest anomaly model (scikit-learn)
- Per-read prediction table with anomaly labels at bona fide modification positions for CTRL and COND samples

## How to apply

First align dRNA reads from CTRL and COND samples to the reference with f5c eventalign so each read's signal events are anchored to reference kmers, then use NanoListener to extract paired variable-length current chunks and their reference kmers (works for any ONT chemistry/kit). Augment the extracted dataset in silico with simulated randomers (random sequence insertions) to make training data robust against inference artefacts or 'hallucinations' stemming from low-quality public modification data. Apply balanced random chunk sampling across sequence contexts so that frequent kmers do not dominate the training set. Train an IsolationForest anomaly model from scikit-learn on this dataset, then score reads at bona fide modification positions, labelling COND reads as anomalous when their current patterns deviate from the CTRL-trained distribution (Fig. 2B-C).

## Related tools

- **NanoListener** (Extracts current chunks and kmers from event-aligned dRNA reads and applies the simulated randomers strategy to build the training dataset)
- **f5c** (Performs event alignment that anchors each read's signal events to reference kmers, defining chunk-to-kmer pairing)
- **scikit-learn** (Provides the IsolationForest implementation used to train the anomaly model and label per-read predictions)

## Evaluation signals

- The per-read prediction table covers bona fide modification positions in both CTRL and COND samples with an anomaly label per read (article reports no quantitative threshold; acceptance is a qualitative completeness check of labelled per-read predictions).
- The extracted chunk-kmer pairs are consistent with the f5c eventalign anchoring (each variable-length chunk maps to its corresponding reference kmer).
- Balanced random sampling is verified across sequence contexts so frequent kmers do not dominate the training set.
- Modified reads in COND samples are preferentially flagged as anomalies relative to their CTRL counterparts.
- Randomers augmentation reduces hallucinated modification calls attributable to low-quality public modification data.

## Limitations

- Publicly available modification data are often of questionable quality and can generate inference artefacts or 'hallucinations' in the training set, which the randomers strategy only mitigates.
- There is no quantitative threshold for validating the per-read prediction table; the check is a qualitative completeness review.
- Sample-level factors (number of reads, quality, contaminations, unknown IVT conditions) hamper training set generation.

## Evidence

- [full_text] IsolationForest training and per-read scoring workflow: [evidence span withheld — anchor status: not_found]
