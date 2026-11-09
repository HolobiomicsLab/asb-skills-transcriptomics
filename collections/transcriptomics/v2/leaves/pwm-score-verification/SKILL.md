---
name: pwm-score-verification
description: Use when a PWM-based motif scanner must be validated end-to-end — e.g.
  reproducing ElemeNT 2023's log2(PWM/background) scoring with background P(G)=P(C)=GC/2,
  P(A)=P(T)=(1−GC)/2 on a test DNA sequence.
when_to_use_negative:
- The input is already a scored matrix or element-prediction output — use ElemeNT
  2023's own motif search rather than re-deriving scores
- The motif model is not a position weight matrix (e.g. regular-expression or consensus
  motif search), where log-odds normalization does not apply
- The goal is biological element discovery rather than verification of the scoring
  implementation itself
edam_operation: http://edamontology.org/operation_0432
edam_topics:
- http://edamontology.org/topic_0749
tools:
- name: ElemeNT 2023
  role: Source of the PWM table and reference implementation of GC-normalized log2
    motif scoring that the verification reproduces
  repo: https://github.com/OritAdato/ElemeNT
- name: Perl
  role: Implementation language of ElemeNT's command-line sources used to inspect
    and validate the scoring code
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae110
    title: 'ElemeNT 2023: an enhanced tool for detection and curation of core promoter
      elements'
  source_content_hash: sha256:e9e0064e8d7d1fe110fb3ad1d586dffecdcead2b8c9e5172595e791fb7be518c
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
yes_rate: 1.0
n_claims: 10
---

# pwm-score-verification

## Summary

Verifies that GC-content-normalized log2 log-odds PWM scoring, as implemented in ElemeNT 2023 for core promoter motif detection, produces a finite numeric score for every position–nucleotide pair under a user-specified GC fraction.

## When to use

Use when a PWM-based motif scanner must be validated end-to-end — e.g. reproducing ElemeNT 2023's log2(PWM/background) scoring with background P(G)=P(C)=GC/2, P(A)=P(T)=(1−GC)/2 on a test DNA sequence.

## When NOT to use

- The input is already a scored matrix or element-prediction output — use ElemeNT 2023's own motif search rather than re-deriving scores
- The motif model is not a position weight matrix (e.g. regular-expression or consensus motif search), where log-odds normalization does not apply
- The goal is biological element discovery rather than verification of the scoring implementation itself

## Inputs

- ElemeNT 2023 PWM table for a core promoter element (from github.com/OritAdato/ElemeNT)
- User-specified GC fraction (0–1) defining the background model
- Test DNA nucleotide sequence to score

## Outputs

- Per-position log2 log-odds score matrix (score(i,j) for each PWM position i and nucleotide j)
- Position-wise scores of the test DNA sequence
- Verification report recording all computed scores and validation status

## How to apply

Retrieve the ElemeNT 2023 PWM table from the official repository (github.com/OritAdato/ElemeNT), which ships the command-line sources under ElemeNT2023/src/ElemeNT. For a user-specified GC fraction, build the GC-normalized background model with P(G)=P(C)=GC/2 and P(A)=P(T)=(1−GC)/2, mirroring ElemeNT 2023's background computation. Compute the per-position log2 log-odds matrix, score(i,j)=log2(P(i,j)/background(j)), for each nucleotide j at each position i of the PWM. Apply this score matrix to a test DNA sequence at the same GC fraction and record position-wise scores in a verification report. Decision point: any non-finite value (NaN, ±Inf) at a position–nucleotide pair indicates a zero PWM probability divided by a zero background probability or a mis-specified GC fraction, and the run fails validation.

## Related tools

- **ElemeNT 2023** (Source of the PWM table and reference implementation of GC-normalized log2 motif scoring that the verification reproduces) — https://github.com/OritAdato/ElemeNT
- **Perl** (Implementation language of ElemeNT's command-line sources used to inspect and validate the scoring code)

## Examples

```
perl ElemeNT2023/src/ElemeNT/elemeNT.pl --seq test_dna.fa --GC 0.45
```

## Evaluation signals

- Script runs end-to-end without error on the retrieved PWM table and test sequence
- Emitted score matrix contains a finite numeric log2 score for every position–nucleotide pair of the PWM
- Scores are consistent with the GC-normalized background at the user-specified GC fraction (P(G)=P(C)=GC/2, P(A)=P(T)=(1−GC)/2)
- Position-wise scores over the test DNA sequence match a recomputation of log2(P(i,j)/background(j)) at those positions
- Verification report records a value for every PWM position, with no missing entries

## Limitations

- Zero PWM probabilities combined with a zero background probability at extreme GC fractions can produce undefined (non-finite) log2 log-odds scores
- The verification validates the scoring arithmetic only; it does not assess biological motif-detection performance of ElemeNT 2023

## Evidence

- [intro] ElemeNT is an interactive tool (implemented in Perl) for rapid and convenient detection of core promoter elements within any given sequences.: "ElemeNT is an interactive tool (implemented in Perl) for rapid and convenient detection of core promoter elements within any given sequences."
- [abstract] The source code and command line version of ElemeNT 2023 are available at https://github.com/OritAdato/ElemeNT.: "The source code and command line version of ElemeNT 2023 are available at https://github.com/OritAdato/ElemeNT."
- [readme] > ElemeNT/ElemeNT2023/src/ElemeNT - the sources of ElemeNT2023.: "> ElemeNT/ElemeNT2023/src/ElemeNT - the sources of ElemeNT2023."
- [abstract] accessible both as a fast web-based tool and via command line (no coding skills are required to run the tool).: "accessible both as a fast web-based tool and via command line (no coding skills are required to run the tool)."
