---
name: dna-sequence-motif-scoring
description: Use when scoring a DNA sequence against a position weight matrix and
  the GC content of the input sequence must be accounted for via a background model
  P(G)=P(C)=GC/2, P(A)=P(T)=(1-GC)/2.
when_to_use_negative:
- Input is already a set of scored/predicted core promoter elements (e.g. the CORE
  database outputs) rather than raw sequence to score
- The task requires evolutionary conservation assessment or TSS dataset quality evaluation
  rather than PWM-based sequence scoring
- The user needs to search a user-defined motif that is not representable as a PWM
  over nucleotide positions
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_3511
tools:
- name: ElemeNT 2023
  role: Tool whose GC-normalized log2 PWM scoring scheme is reproduced; provides the
    PWM table and reference implementation
  repo: https://github.com/OritAdato/ElemeNT
- name: Perl
  role: Implementation language of the ElemeNT command-line scoring script
  repo: https://github.com/OritAdato/ElemeNT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae110
    title: 'ElemeNT 2023: an enhanced tool for detection and curation of core promoter
      elements'
  source_content_hash: sha256:a63497cb984bef48cb703476ffa69b09ce4bac6dccd5951108b0806baad1ee1f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OritAdato/ElemeNT
  tools:
  - ElemeNT 2023
  - Perl
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_3511
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# dna-sequence-motif-scoring

## Summary

Computes GC-content-normalized per-position log2 log-odds scores for a PWM against a user-specified DNA sequence, as implemented in ElemeNT 2023 (Perl) for detecting core promoter elements.

## When to use

Use when scoring a DNA sequence against a position weight matrix and the GC content of the input sequence must be accounted for via a background model P(G)=P(C)=GC/2, P(A)=P(T)=(1-GC)/2.

## When NOT to use

- Input is already a set of scored/predicted core promoter elements (e.g. the CORE database outputs) rather than raw sequence to score
- The task requires evolutionary conservation assessment or TSS dataset quality evaluation rather than PWM-based sequence scoring
- The user needs to search a user-defined motif that is not representable as a PWM over nucleotide positions

## Inputs

- ElemeNT 2023 PWM table (from github.com/OritAdato/ElemeNT)
- User-provided DNA sequence to be scored
- User-specified GC fraction of the sequence

## Outputs

- Per-position log2 log-odds score matrix, score(i,j)=log2(P(i,j)/background(j))
- Position-wise motif scores for the test DNA sequence at the specified GC fraction
- Verification report of emitted position-nucleotide scores

## How to apply

Retrieve the ElemeNT 2023 PWM table from github.com/OritAdato/ElemeNT and load it into the scoring script. For the user-specified GC fraction, define the background model with P(G)=P(C)=GC/2 and P(A)=P(T)=(1-GC)/2 so the normalization reflects the sequence's base composition. Compute the per-position log2 log-odds score matrix, score(i,j)=log2(P(i,j)/background(j)), for each nucleotide j at each PWM position i. Apply this score matrix to the test DNA sequence and record position-wise scores; the tool is available as a web interface or Perl command-line version. Validate that the script runs end-to-end and that every position-nucleotide pair of the PWM yields a finite numeric log2 score consistent with the GC-normalized background.

## Related tools

- **ElemeNT 2023** (Tool whose GC-normalized log2 PWM scoring scheme is reproduced; provides the PWM table and reference implementation) — https://github.com/OritAdato/ElemeNT
- **Perl** (Implementation language of the ElemeNT command-line scoring script) — https://github.com/OritAdato/ElemeNT

## Examples

```
perl ElemeNT2023/src/ElemeNT --sequence test_dna.fa --gc 0.5  # score positions via log2(PWM/background), background P(G)=P(C)=gc/2
```

## Evaluation signals

- Script runs end-to-end without error on the test DNA sequence
- Emitted score matrix contains a finite numeric log2 score for every position-nucleotide pair of the PWM
- Scores are consistent with the GC-normalized background at the user-specified GC fraction (P(G)=P(C)=GC/2, P(A)=P(T)=(1-GC)/2)
- Position-wise scores recorded in the verification report match score(i,j)=log2(P(i,j)/background(j)) recomputed independently

## Limitations

- The normalization assumes equal probabilities within the GC pair and within the AT pair; sequences deviating from this symmetry are not modeled
- The GC fraction must be user-specified, so a mis-estimated GC content shifts all log-odds scores

## Evidence

- [full_text] computes background probabilities as GC/2 for G and C and (1 − GC)/2 for A and T, then calculates the per-position nucleotide score as log2 of the PWM probability divided by the corresponding background probability: [evidence span withheld — anchor status: not_found]
- [abstract] The source code and command line version of ElemeNT 2023 are available at https://github.com/OritAdato/ElemeNT.: "The source code and command line version of ElemeNT 2023 are available at https://github.com/OritAdato/ElemeNT."
- [intro] ElemeNT is an interactive tool (implemented in Perl) for rapid and convenient detection of core promoter elements within any given sequences.: "ElemeNT is an interactive tool (implemented in Perl) for rapid and convenient detection of core promoter elements within any given sequences."
- [readme] It can be executed via the following web interface: [ElemeNT 2023](https://www.juven-gershonlab.org/resources/element-v2023/) or via a command line.: "It can be executed via the following web interface: [ElemeNT 2023](https://www.juven-gershonlab.org/resources/element-v2023/) or via a command line."
