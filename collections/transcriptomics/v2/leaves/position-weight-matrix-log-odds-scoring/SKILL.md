---
name: position-weight-matrix-log-odds-scoring
description: Use when scoring nucleotide sequences against a PWM for motif detection
  and the scoring must be normalized to the GC fraction of the user-supplied sequence
  to avoid GC-bias in motif hits.
when_to_use_negative:
- When searching for arbitrary user-defined motifs that are not represented in the
  ElemeNT 2023 PWM table — although ElemeNT 2023 can search any user-defined motif,
  the GC-normalized log-odds scoring described here is grounded in its PWM-based motif
  search.
- When a background model other than a simple GC-fraction model is required (e.g.
  Markov-order-1 backgrounds), since this skill's normalization uses only P(G)=P(C)=GC/2
  and P(A)=P(T)=(1-GC)/2.
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3511
tools:
- name: ElemeNT 2023
  role: Tool whose PWM-based motif search implements the GC-normalized log2 log-odds
    scoring; used via its command line or source code to compute score matrices
  repo: https://github.com/OritAdato/ElemeNT
- name: Perl
  role: Implementation language of ElemeNT; the scoring computation is carried out
    in the Perl source under ElemeNT2023/src/ElemeNT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae110
    title: 'ElemeNT 2023: an enhanced tool for detection and curation of core promoter
      elements'
  source_content_hash: sha256:3bd38b7ebbc751f3da32cf1622ecc1a05b150f59f92b88994fa62827f3e1dff0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OritAdato/ElemeNT
  tools:
  - ElemeNT 2023
  - Perl
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3511
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# position-weight-matrix-log-odds-scoring

## Summary

Computes GC-content-normalized log2 log-odds scores from a PWM by dividing each position-nucleotide probability by a GC-dependent background model (P(G)=P(C)=GC/2, P(A)=P(T)=(1-GC)/2), as implemented in ElemeNT 2023 for core promoter element detection.

## When to use

Use when scoring nucleotide sequences against a PWM for motif detection and the scoring must be normalized to the GC fraction of the user-supplied sequence to avoid GC-bias in motif hits.

## When NOT to use

- When searching for arbitrary user-defined motifs that are not represented in the ElemeNT 2023 PWM table — although ElemeNT 2023 can search any user-defined motif, the GC-normalized log-odds scoring described here is grounded in its PWM-based motif search.
- When a background model other than a simple GC-fraction model is required (e.g. Markov-order-1 backgrounds), since this skill's normalization uses only P(G)=P(C)=GC/2 and P(A)=P(T)=(1-GC)/2.

## Inputs

- ElemeNT 2023 PWM table (retrieved from github.com/OritAdato/ElemeNT)
- User-provided DNA nucleotide sequence
- User-specified GC fraction of the sequence

## Outputs

- Per-position log2 log-odds score matrix (score for every position-nucleotide pair of the PWM)
- Position-wise motif scores for the test DNA sequence at the specified GC fraction
- Verification report of computed scores

## How to apply

Retrieve the ElemeNT 2023 PWM table from the official repository (github.com/OritAdato/ElemeNT). For a user-specified GC fraction, build the background model with P(G)=P(C)=GC/2 and P(A)=P(T)=(1-GC)/2. Compute the per-position log2 log-odds score matrix, score(i,j)=log2(P(i,j)/background(j)), for each nucleotide j at each PWM position i. Apply the computed score matrix to the target DNA sequence at the specified GC fraction and record the position-wise scores. The GC normalization ensures that motif scores reflect enrichment over the sequence's actual base composition rather than raw PWM probabilities.

## Related tools

- **ElemeNT 2023** (Tool whose PWM-based motif search implements the GC-normalized log2 log-odds scoring; used via its command line or source code to compute score matrices) — https://github.com/OritAdato/ElemeNT
- **Perl** (Implementation language of ElemeNT; the scoring computation is carried out in the Perl source under ElemeNT2023/src/ElemeNT)

## Examples

```
perl ElemeNT2023/src/ElemeNT/elementnt.pl --input test_sequence.fa --GC 0.5
```

## Evaluation signals

- The script runs end-to-end without error.
- The emitted score matrix contains a finite numeric log2 score for every position-nucleotide pair of the PWM.
- Each score equals log2 of the PWM probability divided by the GC-normalized background probability at the user-specified GC fraction (e.g. at GC=0.5 all background terms are 0.25).
- Position-wise scores recorded for the test DNA sequence are consistent with the GC-normalized background model.
- Score results are stable when the GC fraction parameter is varied in a way consistent with the background formulas P(G)=P(C)=GC/2, P(A)=P(T)=(1-GC)/2.

## Limitations

- The background model is a simple GC-fraction model; it does not capture higher-order nucleotide dependencies.
- Scores depend on the user-specified GC fraction, so incorrect GC estimation of the input sequence will bias the log-odds scores.
- ElemeNT is focused on core promoter elements; using this PWM scoring outside that motif family requires supplying an appropriate PWM.

## Evidence

- [intro] ElemeNT is an interactive tool (implemented in Perl) for rapid and convenient detection of core promoter elements within any given sequences.: "ElemeNT is an interactive tool (implemented in Perl) for rapid and convenient detection of core promoter elements within any given sequences."
- [abstract] Source code availability: "The source code and command line version of ElemeNT 2023 are available at https://github.com/OritAdato/ElemeNT."
- [readme] Repository structure: "ElemeNT/ElemeNT2023/src/ElemeNT - the sources of ElemeNT2023."
- [abstract] Web and command line access: "accessible both as a fast web-based tool and via command line (no coding skills are required to run the tool)."
