---
name: reverse-complement-strand-scanning
description: Use when scanning a user-provided nucleotide sequence (e.g. FASTA) for
  core promoter elements (TATA, Inr, DPE, MTE) or user-defined motifs and hits on
  both strands must be reported without prior TSS determination.
when_to_use_negative:
- Input is already a set of predicted elements or a feature table (e.g. ElemeNT's
  CORE database predictions around annotated TSSs) rather than raw nucleotide sequence
  to scan
- The analysis requires prior determination of the transcription start site (TSS)
  before element calling — ElemeNT 2023 explicitly operates without prior TSS determination
- The goal is protein or RNA motif scanning where reverse-complement semantics of
  double-stranded DNA do not apply
edam_operation: http://edamontology.org/operation_0432
edam_topics:
- http://edamontology.org/topic_0749
tools:
- name: ElemeNT 2023
  role: Interactive Perl tool that performs the PWM log-likelihood scan on both strands,
    applies element-specific cutoffs and combination logic (e.g. DPE dependency on
    Inr), and reports element location and score
  repo: https://github.com/OritAdato/ElemeNT
- name: Perl
  role: Implementation language of ElemeNT 2023 command-line version used to run the
    bidirectional scan
  repo: https://github.com/OritAdato/ElemeNT
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae110
    title: 'ElemeNT 2023: an enhanced tool for detection and curation of core promoter
      elements'
  source_content_hash: sha256:54fd318ec5cb938aefe23b67bef4e709135158187182bc1d39cbbcb1f3385a52
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OritAdato/ElemeNT
  tools:
  - ElemeNT 2023
  - Perl
  edam_operation: http://edamontology.org/operation_0432
  edam_topics:
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# reverse-complement-strand-scanning

## Summary

A motif-scanning strategy that scores a nucleotide sequence against a PWM in the forward orientation and again on its reverse complement, so element hits are reported on both sense and antisense strands with PWM log scores ≥ the element-specific cutoff.

## When to use

Use when scanning a user-provided nucleotide sequence (e.g. FASTA) for core promoter elements (TATA, Inr, DPE, MTE) or user-defined motifs and hits on both strands must be reported without prior TSS determination.

## When NOT to use

- Input is already a set of predicted elements or a feature table (e.g. ElemeNT's CORE database predictions around annotated TSSs) rather than raw nucleotide sequence to scan
- The analysis requires prior determination of the transcription start site (TSS) before element calling — ElemeNT 2023 explicitly operates without prior TSS determination
- The goal is protein or RNA motif scanning where reverse-complement semantics of double-stranded DNA do not apply

## Inputs

- User-provided nucleotide sequence in FASTA format
- Position weight matrices (PWMs) for core promoter element motifs (e.g. TATA, Inr, DPE, MTE) or user-defined motifs
- Element-specific cutoff thresholds on PWM log-likelihood scores

## Outputs

- Report of element names, strand (sense/antisense), coordinates and PWM log scores (e.g. elements_report.tsv), restricted to hits with scores ≥ the defined cutoff

## How to apply

Load the user-provided sample nucleotide sequence in FASTA format (e.g. with Biopython). For each core promoter element motif, compute PWM log-likelihood scores along the sequence in the forward orientation. Repeat the identical PWM log-likelihood scan on the reverse complement of the sequence so the opposite strand is covered. Retain only matches whose PWM log score meets or exceeds the element-specific cutoff threshold, and record strand, coordinates and score. Apply combination logic afterwards (e.g. DPE reported only when an Inr hit is present at the biologically relevant spacing) rather than deciding strand or TSS beforehand. Write element names, strand, coordinates and scores to a report such as elements_report.tsv.

## Related tools

- **ElemeNT 2023** (Interactive Perl tool that performs the PWM log-likelihood scan on both strands, applies element-specific cutoffs and combination logic (e.g. DPE dependency on Inr), and reports element location and score) — https://github.com/OritAdato/ElemeNT
- **Perl** (Implementation language of ElemeNT 2023 command-line version used to run the bidirectional scan) — https://github.com/OritAdato/ElemeNT

## Evaluation signals

- elements_report.tsv exists and contains element/position/score fields
- Every reported hit has a PWM log score meeting or exceeding its element-specific cutoff
- Hits appear for both the forward scan and the reverse-complement scan, with strand labels distinguishing sense from antisense
- Spacing-dependent elements such as DPE appear only in combination with an Inr hit at the expected spacing (qualitative acceptance check)
- Reverse-complement hits correspond to forward-orientation hits on the reverse-complement sequence (strand symmetry sanity check)

## Limitations

- The scan reports candidate hits with scores ≥ cutoff but does not require prior determination of the TSS, so hits are strand-local predictions rather than TSS-anchored annotations
- Combination logic means some hits (e.g. DPE) are only reported conditionally on another element (Inr) at the biologically relevant spacing, so raw scan output alone is not the final call
- Command-line version requires installation via the provided installPackage zip; no coding skills are required for the web or command-line versions per the article, but Perl availability is needed

## Evidence

- [results] ElemeNT 2023 reports the location and PWM log score of identified elements (scores ≥ the defined cutoff) on both sense and antisense strands: [evidence span withheld — anchor status: not_found]
- [intro] ElemeNT is an interactive tool (implemented in Perl) for rapid and convenient detection of core promoter elements within any given sequences.: "ElemeNT is an interactive tool (implemented in Perl) for rapid and convenient detection of core promoter elements within any given sequences."
- [abstract] it can also be used for searching any user-defined motif including sequence-specific DNA binding sites: "accessible both as a fast web-based tool and via command line (no coding skills are required to run the tool). While this tool is focused on core promoter elements, it can also be used for searching"
- [readme] ElemeNT/ElemeNT2023/installPackage - zip of the installation package of ElemeNT2023 command line version.: "ElemeNT/ElemeNT2023/installPackage - zip of the installation package of ElemeNT2023 command line version."
