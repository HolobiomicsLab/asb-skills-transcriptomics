---
name: ctrl-cond-modification-contrast-analysis
description: Use when paired ONT dRNA CTRL and COND samples must be contrasted per-read
  to flag modified reads at known (bona fide) modification positions, and no modification-aware
  basecaller is available for the chemistry.
when_to_use_negative:
- When a modification-aware basecaller such as NanoSpeech or m6ABasecaller can call
  modifications ab initio in a single step without paired control conditions
- When only one sample condition is available (no CTRL counterpart to train the IsolationForest
  against)
- When a quantitative, thresholded modification-calling result is required — this
  workflow's validation is explicitly qualitative (no quantitative threshold reported)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: NanoListener
  role: Extracts paired current chunks and kmers from event-aligned dRNA reads (any
    ONT chemistry/kit) and trains the IsolationForest anomaly model used for the CTRL/COND
    contrast
- name: f5c
  role: Eventalign step that anchors each read's raw current chunks to reference kmer
    sequences
- name: scikit-learn
  role: Provides the IsolationForest anomaly model trained on CTRL chunks to score
    COND reads
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf709
    title: <i>Ab initio</i> detection of multiple epitranscriptomic modifications
      from Oxford nanopore technology direct RNA sequencing data
  source_content_hash: sha256:8b92dbbacaa2bf3a7dddd90b634ac505bdafa2138ee5f52934e8af0a42a21603
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# ctrl-cond-modification-contrast-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Per-read anomaly-based contrast between paired CTRL (unmodified) and COND (modified) ONT dRNA samples: a NanoListener-trained IsolationForest model built on CTRL current-chunk/kmer pairs is used to score COND reads at bona fide modification positions and flag modified reads as anomalies.

## When to use

Use when paired ONT dRNA CTRL and COND samples must be contrasted per-read to flag modified reads at known (bona fide) modification positions, and no modification-aware basecaller is available for the chemistry.

## When NOT to use

- When a modification-aware basecaller such as NanoSpeech or m6ABasecaller can call modifications ab initio in a single step without paired control conditions
- When only one sample condition is available (no CTRL counterpart to train the IsolationForest against)
- When a quantitative, thresholded modification-calling result is required — this workflow's validation is explicitly qualitative (no quantitative threshold reported)

## Inputs

- ONN dRNA fast5/pod5 raw reads from CTRL (unmodified) samples
- ONT dRNA fast5/pod5 raw reads from COND (modified) samples
- Reference sequence (fasta) supplying kmer sequences
- f5c eventalign output anchoring signal events to reference kmers
- List of bona fide modification positions to score

## Outputs

- Per-read prediction table covering bona fide positions in both CTRL and COND samples with an anomaly label per read
- Set of reads in the COND sample flagged as modified (anomalies) relative to CTRL counterparts

## How to apply

Align dRNA reads from both CTRL and COND samples against the reference with f5c eventalign so each read's signal events are anchored to reference kmers, then extract paired variable-length current chunks and reference kmers with NanoListener. Augment the extracted dataset in silico with simulated randomers (random sequence insertions) to mitigate inference artefacts or 'hallucinations' caused by the questionable quality of public modification data, and apply balanced random chunk sampling across sequence contexts so frequent kmers do not dominate training. Train an IsolationForest anomaly model (scikit-learn) on the CTRL-derived chunks and score reads at bona fide modification positions, flagging COND reads that behave anomalously relative to their CTRL counterparts. Validation is a qualitative completeness check: the per-read prediction table must cover all bona fide positions in both CTRL and COND samples with an anomaly label per read — the article reports no quantitative threshold for this step.

## Related tools

- **NanoListener** (Extracts paired current chunks and kmers from event-aligned dRNA reads (any ONT chemistry/kit) and trains the IsolationForest anomaly model used for the CTRL/COND contrast)
- **f5c** (Eventalign step that anchors each read's raw current chunks to reference kmer sequences)
- **scikit-learn** (Provides the IsolationForest anomaly model trained on CTRL chunks to score COND reads)

## Examples

```
from sklearn.ensemble import IsolationForest; model = IsolationForest().fit(CTRL_chunks); labels = model.predict(COND_chunks_at_bona_fide_positions)
```

## Evaluation signals

- Per-read prediction table covers every bona fide modification position in both CTRL and COND samples
- Every read in the table carries an anomaly label at each scored position
- COND reads flagged as anomalies are concentrated at bona fide modification positions and exceed the anomaly rate of CTRL counterparts at the same positions
- Training chunks are balanced across sequence contexts — frequent kmers do not dominate the set (check kmer-frequency distribution after balanced sampling)
- Simulated-randomers augmentation is present in the training set, guarding against 'hallucination' artefacts from low-quality public modification data

## Limitations

- The article reports no quantitative threshold for the validation step; acceptance is a qualitative completeness check of labelled per-read predictions
- The often questionable quality of public data for RNA modifications can hamper generation of robust training sets, leading to inference artefacts or 'hallucinations' (addressed by randomers, not eliminated)
- Relies on f5c eventalign anchoring, so alignment quality directly conditions chunk-to-kmer pairing

## Evidence

- [readme] Alternative without controls: [evidence span withheld — anchor status: external_doc]
