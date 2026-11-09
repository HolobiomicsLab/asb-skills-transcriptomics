---
name: gold-standard-regulator-validation
description: Use when a set of candidate RBPs (e.g. CRIT-derived non-physically-associated
  candidates) must be connected to known circRNA regulators through binding-motif
  similarity, given circRNA sequences and a gold-regulator motif set.
when_to_use_negative:
- Candidate RBPs already have documented physical interactions with gold regulators
  — physical interaction, not motif similarity, is the appropriate evidence channel
  for those candidates
- CircRNA sequences cannot be mapped to chromosomal positions in CircBase/GENCODE,
  so candidate RBP–circRNA associations (C value) cannot be established
- The aim is regulator discovery from expression/alteration data (the CRIT NMF pipeline
  on ρm, Zp, Zs, Zg, C value features) rather than post-hoc validation of an already-identified
  candidate set
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0080
tools:
- name: MEME
  role: De novo motif discovery on circRNA sequences linked to each candidate RBP
    to derive consensus binding motifs
- name: TOMTOM
  role: Compares each MEME-discovered motif against the gold-regulator motif set with
    a GC-matched background model; significant matches reported at E < 0.01 in tomtom_results.tsv
- name: RNAct / catRAPID
  role: Source of protein-RNA interaction prediction scores (interaction score >=
    30) linking candidate RBPs to RNAs/circRNAs
  repo: https://rnact.crg.eu/download
- name: CircBase
  role: Provides human circRNA chromosomal position annotations (hsa_hg19_circRNA.txt)
    used to align circRNAs to RNAs and assign each circRNA to a protein
  repo: http://circbase.org/download/hsa_hg19_circRNA.txt
- name: CRIT
  role: Upstream pipeline whose NMF-based candidate identification supplies the candidate
    RBPs that this motif-similarity validation supports
  repo: https://github.com/BioinformaticsSTU/CRIT
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:053613a9c84134245eeadbf4c1158cb063a2f0d44f6771bfcfb9e841deedd75d
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# gold-standard-regulator-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates candidate circRNA-regulating RBPs that lack physical interactions with gold regulators by testing whether their de novo RNA-binding motifs (discovered with MEME on circRNA sequences) significantly resemble gold-regulator motifs via TOMTOM comparison. This provides an orthogonal, motif-based line of evidence that novel RBPs participate in the circRNA life cycle.

## When to use

Use when a set of candidate RBPs (e.g. CRIT-derived non-physically-associated candidates) must be connected to known circRNA regulators through binding-motif similarity, given circRNA sequences and a gold-regulator motif set.

## When NOT to use

- Candidate RBPs already have documented physical interactions with gold regulators — physical interaction, not motif similarity, is the appropriate evidence channel for those candidates
- CircRNA sequences cannot be mapped to chromosomal positions in CircBase/GENCODE, so candidate RBP–circRNA associations (C value) cannot be established
- The aim is regulator discovery from expression/alteration data (the CRIT NMF pipeline on ρm, Zp, Zs, Zg, C value features) rather than post-hoc validation of an already-identified candidate set

## Inputs

- CircRNA sequences linked to candidate RBPs, derived from RNAct/catRAPID interactome scores (interaction score >= 30) and CircBase chromosomal position annotations (hsa_hg19_circRNA.txt)
- Human RNA annotation file gencode.v37.annotation.gtf (GENCODE release 37) for chromosomal position mapping
- Motif set of collected gold circRNA regulators (gold_regulators list from prior literature)
- GC-matched background model for TOMTOM comparison

## Outputs

- tomtom_results.tsv — full TOMTOM comparison table with E-values and matched gold-regulator motifs per candidate
- Per-candidate consensus motifs discovered de novo by MEME from the candidates' circRNA sequences
- Tabulation of significant (E < 0.01) motif matches between each candidate RBP and gold regulators

## How to apply

First, extract the circRNA sequences linked to each candidate RBP: take protein-RNA interactions from the RNAct/catRAPID interactome (excluding interaction scores < 30) and map circRNAs to RNAs using chromosomal position annotations from CircBase (hsa_hg19_circRNA.txt) and GENCODE (gencode.v37.annotation.gtf), considering a protein–circRNA interaction only when the circRNA position completely aligns to the corresponding RNA. Second, run de novo motif discovery with the MEME suite on each candidate RBP's associated circRNA sequences to derive enriched consensus motifs. Third, compare each discovered motif against the collected motif set of gold regulators using TOMTOM with a GC-matched background model and a significance threshold of E < 0.01. Finally, tabulate significant motif matches per candidate and write the full comparison table, including E-values and matched gold-regulator motifs, to tomtom_results.tsv; the validation succeeds if every candidate shows at least one significant gold-regulator match.

## Related tools

- **MEME** (De novo motif discovery on circRNA sequences linked to each candidate RBP to derive consensus binding motifs)
- **TOMTOM** (Compares each MEME-discovered motif against the gold-regulator motif set with a GC-matched background model; significant matches reported at E < 0.01 in tomtom_results.tsv)
- **RNAct / catRAPID** (Source of protein-RNA interaction prediction scores (interaction score >= 30) linking candidate RBPs to RNAs/circRNAs) — https://rnact.crg.eu/download
- **CircBase** (Provides human circRNA chromosomal position annotations (hsa_hg19_circRNA.txt) used to align circRNAs to RNAs and assign each circRNA to a protein) — http://circbase.org/download/hsa_hg19_circRNA.txt
- **CRIT** (Upstream pipeline whose NMF-based candidate identification supplies the candidate RBPs that this motif-similarity validation supports) — https://github.com/BioinformaticsSTU/CRIT

## Examples

```
tomtom -oc tomtom_out -evalue -thresh 0.01 -bfile gc_matched_background candidate_motifs.meme gold_regulators.meme
```

## Evaluation signals

- tomtom_results.tsv exists and contains E-values plus matched gold-regulator motif identifiers for every analyzed candidate
- All 16 non-physically-associated candidates (ADARB1, ALKBH8, ARL6IP4, CDK11B, CELF4, CELF5, CSTF2T, GIGYF2, MFAP1, MTO1, NOL10, RBMX2, REXO4, RNF113A, ZC3H12A, ZFC3H1) have at least one significant motif match to a gold regulator
- MFAP1's motifs match 12 gold regulators with E < 0.01, serving as a known positive control row
- All reported matches pass the significance threshold E < 0.01 under the GC-matched background model
- RNAct interactions used for circRNA extraction all satisfy interaction score >= 30, and circRNA-to-RNA assignments show complete chromosomal positional alignment

## Limitations

- Motif similarity is an indirect association: it functionally connects candidates to authentic regulators but does not itself demonstrate regulation of circRNAs
- The protein-RNA associations derive from catRAPID predictions in RNAct, not experimental CLIP evidence, and rely on complete positional alignment of circRNAs to RNAs, which can miss partial overlaps
- Validation depends on the completeness of the curated gold-regulator motif set; candidates may share motifs with regulators absent from the gold list

## Evidence
