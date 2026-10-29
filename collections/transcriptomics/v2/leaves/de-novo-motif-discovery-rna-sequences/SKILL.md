---
name: de-novo-motif-discovery-rna-sequences
description: Use when you have nucleotide sequences (e.g., circRNA host regions) linked
  to candidate RBPs and want to test whether their binding motifs resemble those of
  established regulators, independent of physical interaction evidence.
when_to_use_negative:
- The candidate RBP already has validated physical circRNA-RBP interactions (e.g.,
  CLIP evidence), making motif similarity inference redundant
- Input is a precomputed feature/interaction matrix (e.g., CRIT features MATRIX.txt)
  rather than nucleotide sequences suitable for motif discovery
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_3511
tools:
- name: MEME
  role: De novo discovery of enriched consensus binding motifs from candidate-RBP-linked
    circRNA sequences
- name: TOMTOM
  role: Compares discovered motifs against the gold-regulator motif set with a GC-matched
    background, E < 0.01 threshold, writing tomtom_results.tsv
- name: RNAct / catRAPID
  role: Source of protein-RNA interaction scores (score >= 30) used to link candidate
    RBPs to circRNAs
  repo: https://rnact.crg.eu/download
- name: CircBase
  role: Provides human circRNA chromosomal position annotations (hsa_hg19_circRNA.txt)
    used to align circRNAs to RNAs
- name: CRIT
  role: Upstream NMF pipeline that identifies the candidate RBPs whose circRNA sequences
    undergo motif discovery and comparison
  repo: https://github.com/BioinformaticsSTU/CRIT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:8d7f9f1bb9b721b8a756ac571589388bf04244ba11f4250088f9390aea256f05
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BioinformaticsSTU/CRIT
  tools:
  - CRIT
  - CircBase
  - MEME
  - RNAct
  - RNAct / catRAPID
  - TOMTOM
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_3511
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# de-novo-motif-discovery-rna-sequences

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Discovers enriched RNA-binding consensus motifs de novo from circRNA sequences associated with candidate RBPs, then compares them against gold-regulator motifs using MEME and TOMTOM to infer functional similarity between candidate and known circRNA regulators.

## When to use

Use when you have nucleotide sequences (e.g., circRNA host regions) linked to candidate RBPs and want to test whether their binding motifs resemble those of established regulators, independent of physical interaction evidence.

## When NOT to use

- The candidate RBP already has validated physical circRNA-RBP interactions (e.g., CLIP evidence), making motif similarity inference redundant
- Input is a precomputed feature/interaction matrix (e.g., CRIT features MATRIX.txt) rather than nucleotide sequences suitable for motif discovery

## Inputs

- circRNA nucleotide sequences or chromosomal positions (from CircBase hsa_hg19_circRNA.txt / circRNA-to-RBP assignment from RNAct catrapid_human_basic.zip)
- Gold-regulator motif set (MEME/PWM format)
- GC background model specification (GC-matched)

## Outputs

- De novo consensus motifs per candidate RBP (MEME motif output)
- TOMTOM comparison table with E-values and matched gold-regulator motifs (tomtom_results.tsv)
- Per-candidate count of significant (E < 0.01) gold-regulator motif matches

## How to apply

Extract circRNA sequences for the candidate RBPs using chromosomal position annotations from CircBase (hsa_hg19_circRNA.txt) and RBP-RNA interactions from RNAct/catRAPID, filtering weak interactions with interaction score < 30. Run de novo motif discovery with the MEME suite on each candidate's associated circRNA sequences to derive enriched consensus motifs. Compare each discovered motif against the collected motif set of gold regulators using TOMTOM with a GC-matched background model and a significance threshold of E < 0.01. Tabulate significant matches per candidate; in the CRIT study all 16 non-physically-associated candidates (e.g., MFAP1, CELF4, ZFC3H1) shared motifs with at least one gold regulator, and MFAP1 matched 12 gold regulators.

## Related tools

- **MEME** (De novo discovery of enriched consensus binding motifs from candidate-RBP-linked circRNA sequences)
- **TOMTOM** (Compares discovered motifs against the gold-regulator motif set with a GC-matched background, E < 0.01 threshold, writing tomtom_results.tsv)
- **RNAct / catRAPID** (Source of protein-RNA interaction scores (score >= 30) used to link candidate RBPs to circRNAs) — https://rnact.crg.eu/download
- **CircBase** (Provides human circRNA chromosomal position annotations (hsa_hg19_circRNA.txt) used to align circRNAs to RNAs)
- **CRIT** (Upstream NMF pipeline that identifies the candidate RBPs whose circRNA sequences undergo motif discovery and comparison) — https://github.com/BioinformaticsSTU/CRIT

## Examples

```
tomtom -evalue -thresh 0.01 -o tomtom_out candidate_motifs.meme gold_regulators.meme
```

## Evaluation signals

- All candidate RBPs should show at least one motif match to a gold regulator at E < 0.01 (as observed for all 16 candidates in the CRIT study)
- tomtom_results.tsv contains E-values and matched gold-regulator motif identifiers for every queried candidate motif
- MEME output motifs are derived only from circRNA sequences whose chromosomal positions align to candidate-RBP-bound RNAs in RNAct (interaction score >= 30)
- Per-candidate match counts are reproducible, e.g., MFAP1 sharing consensus motifs with 12 gold regulators at E < 0.01

## Limitations

- catRAPID interaction scores are predictions, not experimental binding evidence; weak scores (< 30) are excluded but residuals may still be indirect
- Motif similarity (E < 0.01 via TOMTOM) is a functional association only and does not establish physical circRNA-RBP interaction
- Results depend on the completeness of the gold-regulator motif set collected from prior literature; motifs absent there cannot be matched

## Evidence

- [other] Run de novo motif discovery on these circRNA sequences with the MEME suite to identify enriched consensus motifs for each candidate RBP.: [evidence span withheld — anchor status: not_found]
- [other] Compare each discovered motif against the motif set of the collected gold regulators using TOMTOM with a GC-matched background model and a significance threshold of E < 0.01.: [evidence span withheld — anchor status: not_found]
