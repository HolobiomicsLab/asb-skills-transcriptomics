---
name: rrna-binding-protein-sequence-analysis
description: Use when candidate RBPs have no physical interaction with gold circRNA
  regulators (e.g., the 16 non-physically-associated candidates from CRIT) and motif-level
  functional connection must be established from circRNA sequences.
when_to_use_negative:
- The candidate RBP already has physical-interaction evidence with a gold regulator
  via Pathway Commons Protein-Protein Interactions — motif similarity adds no new
  independence argument.
- No circRNA sequence set can be linked to the candidate RBP (no circRNA annotations
  align to its RNAct RNA targets), leaving MEME with no input sequences.
- The goal is the NMF-based regulator identification itself (CRIT features Zp, Zs,
  Zg, rho_m, C value) rather than post-hoc validation of already-predicted candidates.
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0080
tools:
- name: MEME
  role: de novo motif discovery on circRNA sequences linked to each candidate RBP
    to identify enriched consensus motifs
- name: TOMTOM
  role: compares discovered candidate motifs against the gold-regulator motif set
    with a GC-matched background and E < 0.01 threshold
- name: CRIT
  role: upstream pipeline that produced the candidate RBPs being validated by motif
    similarity
  repo: https://github.com/BioinformaticsSTU/CRIT
- name: RNAct
  role: source of catRAPID protein-RNA interaction scores used to link each candidate
    RBP to RNAs/circRNAs
- name: CircBase
  role: provides human circRNA chromosomal position annotations (hsa_hg19_circRNA.txt)
    for aligning circRNAs to RNAct RNA targets
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:2edccede27771d58b4064b179c5879eb76048901fbe01f0b7ffbbce4e3c189a2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BioinformaticsSTU/CRIT
  repo_url: https://github.com/BioinformaticsSTU/CRIT
  tools:
  - CRIT
  - CircBase
  - MEME
  - RNAct
  - TOMTOM
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# rRNA-binding-protein-sequence-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A motif-similarity validation skill from the CRIT circRNA-regulator pipeline: it performs de novo motif discovery (MEME) on circRNA sequences linked to candidate RBPs and compares the discovered motifs against gold-regulator motifs with TOMTOM, providing binding-motif evidence for candidate regulators that lack physical interaction evidence.

## When to use

Use when candidate RBPs have no physical interaction with gold circRNA regulators (e.g., the 16 non-physically-associated candidates from CRIT) and motif-level functional connection must be established from circRNA sequences.

## When NOT to use

- The candidate RBP already has physical-interaction evidence with a gold regulator via Pathway Commons Protein-Protein Interactions — motif similarity adds no new independence argument.
- No circRNA sequence set can be linked to the candidate RBP (no circRNA annotations align to its RNAct RNA targets), leaving MEME with no input sequences.
- The goal is the NMF-based regulator identification itself (CRIT features Zp, Zs, Zg, rho_m, C value) rather than post-hoc validation of already-predicted candidates.

## Inputs

- circRNA sequences with chromosomal positions (CircBase hsa_hg19_circRNA.txt)
- protein-RNA interaction scores (RNAct catrapid_human_basic.zip, interaction score >= 30)
- motif set of collected gold circRNA regulators
- list of candidate RBPs (e.g., ADARB1, ALKBH8, ARL6IP4, CDK11B, CELF4, CELF5, CSTF2T, GIGYF2, MFAP1, MTO1, NOL10, RBMX2, REXO4, RNF113A, ZC3H12A, ZFC3H1)

## Outputs

- de novo consensus binding motifs per candidate RBP (MEME motif file)
- TOMTOM comparison table with E-values and matched gold-regulator motifs (tomtom_results.tsv)

## How to apply

First, extract the circRNA sequences associated with each candidate RBP by mapping circRNA chromosomal positions (from CircBase's hsa_hg19_circRNA.txt) against the RNAct/catRAPID protein-RNA interactome (catrapid_human_basic, filtering interaction score < 30). Then run de novo motif discovery with the MEME suite on these circRNA sequences to derive each candidate RBP's enriched consensus motifs. Compare every discovered motif against the motif set of the collected gold regulators using TOMTOM with a GC-matched background model and significance threshold E < 0.01. Tabulate significant matches per candidate (e.g., MFAP1 matched 12 gold regulators) and write the full comparison table with E-values and matched gold motifs to tomtom_results.tsv; the skill is judged correct when every candidate has at least one significant match.

## Related tools

- **MEME** (de novo motif discovery on circRNA sequences linked to each candidate RBP to identify enriched consensus motifs)
- **TOMTOM** (compares discovered candidate motifs against the gold-regulator motif set with a GC-matched background and E < 0.01 threshold)
- **CRIT** (upstream pipeline that produced the candidate RBPs being validated by motif similarity) — https://github.com/BioinformaticsSTU/CRIT
- **RNAct** (source of catRAPID protein-RNA interaction scores used to link each candidate RBP to RNAs/circRNAs)
- **CircBase** (provides human circRNA chromosomal position annotations (hsa_hg19_circRNA.txt) for aligning circRNAs to RNAct RNA targets)

## Examples

```
tomtom -oc tomtom_results -evalue-thresh 0.01 -bfile gc_matched_background.txt candidate_rbp_motifs.meme gold_regulator_motifs.meme
```

## Evaluation signals

- Every candidate RBP has at least one TOMTOM match to a gold-regulator motif with E < 0.01 (as found for all 16 non-physically-associated candidates).
- tomtom_results.tsv contains full E-values and the names of matched gold-regulator motifs, not just binary significance calls.
- Only RNAct interactions with score >= 30 were used when linking RBPs to RNAs, per the CRIT feature-collection rule.
- Per-candidate counts of significant motif matches are tabulated (e.g., MFAP1's motifs match 12 gold regulators).
- A GC-matched background model was used in TOMTOM, matching the nucleotide composition of the queried motifs.

## Limitations

- Motif similarity is an indirect functional-connection signal; the paper treats it alongside physical interaction, common transcription factor modulation, and co-expression rather than as standalone proof.
- The circRNA-to-RNA linkage relies on chromosomal position alignment between hg19 CircBase annotations and GENCODE RNAs, so mapping errors or build mismatches propagate into the sequence sets.
- CLIP-based validation of circRNA-RBP interactions is a separate step; motif similarity alone is not cross-validated against CLIP experiments within this skill.

## Evidence
