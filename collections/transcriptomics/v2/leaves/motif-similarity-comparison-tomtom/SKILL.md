---
name: motif-similarity-comparison-tomtom
description: Use when candidate RBPs lack physical interactions with gold regulators
  and you must test whether their circRNA-derived binding motifs statistically resemble
  known regulator motifs (E-value cutoff E < 0.01).
when_to_use_negative:
- Candidate RBPs already have physical interactions (protein-protein or protein-RNA)
  with gold regulators — motif similarity is used here specifically for non-physically-associated
  candidates
- De novo motifs have not yet been discovered — run MEME on circRNA sequences first;
  TOMTOM requires query motifs in MEME format
- Comparing motifs without a GC-matched background model — significance of TOMTOM
  matches would not be comparable to the article's E < 0.01 results
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_3308
tools:
- name: MEME
  role: de novo discovery of enriched consensus binding motifs from circRNA sequences
    linked to each candidate RBP
- name: TOMTOM
  role: motif comparison of discovered candidate motifs against the gold-regulator
    motif set with a GC-matched background, reporting E-values
- name: RNAct / catRAPID
  role: source of protein-RNA interactions (interaction score >= 30) used to link
    RBPs to circRNAs whose sequences feed motif discovery
- name: CircBase
  role: provides human circRNA chromosomal position annotations (hsa_hg19_circRNA.txt)
    used to align circRNAs to RNAct RNAs and extract sequences
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:2e92c5801738cb0746647bd18c20697441538d7bcc0c2e8b4e7826a62835e3f4
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
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# motif-similarity-comparison-tomtom

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compares de novo RNA-binding motifs discovered by MEME for candidate circRNA regulators against a gold-regulator motif set using TOMTOM with a GC-matched background, to show non-physically-associated RBPs are functionally connected to known circRNA regulators via similar binding motifs.

## When to use

Use when candidate RBPs lack physical interactions with gold regulators and you must test whether their circRNA-derived binding motifs statistically resemble known regulator motifs (E-value cutoff E < 0.01).

## When NOT to use

- Candidate RBPs already have physical interactions (protein-protein or protein-RNA) with gold regulators — motif similarity is used here specifically for non-physically-associated candidates
- De novo motifs have not yet been discovered — run MEME on circRNA sequences first; TOMTOM requires query motifs in MEME format
- Comparing motifs without a GC-matched background model — significance of TOMTOM matches would not be comparable to the article's E < 0.01 results

## Inputs

- circRNA sequences for candidate RBPs (from CircBase chromosomal position annotations aligned to RNAct/catRAPID interactome)
- de novo motifs discovered by MEME from candidate-RBP-linked circRNA sequences (MEME motif file)
- motif set of collected gold circRNA regulators
- GC-matched background model for TOMTOM

## Outputs

- tomtom_results.tsv: full TOMTOM comparison table with E-values and matched gold-regulator motifs
- per-candidate tabulation of significant motif matches (E < 0.01) to gold-regulator motifs

## How to apply

First extract circRNA sequences for the candidate RBPs by aligning CircBase chromosomal positions (hsa_hg19_circRNA.txt) to RNAct/catRAPID protein-RNA interactions, excluding interaction scores <30. Run de novo motif discovery with the MEME suite on these circRNA sequences to obtain enriched consensus motifs per candidate RBP. Then run TOMTOM comparing each candidate motif against the collected gold-regulator motif set, using a GC-matched background model and a significance threshold of E < 0.01. Tabulate significant motif matches per candidate; in the CRIT study this established that all 16 non-physically-associated candidates (e.g., MFAP1, matching 12 gold regulators) share consensus motifs with gold circRNA regulators. Write the full TOMTOM comparison, including E-values and matched gold-regulator motifs, to tomtom_results.tsv.

## Related tools

- **MEME** (de novo discovery of enriched consensus binding motifs from circRNA sequences linked to each candidate RBP)
- **TOMTOM** (motif comparison of discovered candidate motifs against the gold-regulator motif set with a GC-matched background, reporting E-values)
- **RNAct / catRAPID** (source of protein-RNA interactions (interaction score >= 30) used to link RBPs to circRNAs whose sequences feed motif discovery)
- **CircBase** (provides human circRNA chromosomal position annotations (hsa_hg19_circRNA.txt) used to align circRNAs to RNAct RNAs and extract sequences)

## Examples

```
tomtom -evalue -bfile GC-matched-background candidate_motifs.meme gold_regulator_motifs.meme > tomtom_results.tsv
```

## Evaluation signals

- All 16 non-physically-associated candidate RBPs (ADARB1, ALKBH8, ARL6IP4, CDK11B, CELF4, CELF5, CSTF2T, GIGYF2, MFAP1, MTO1, NOL10, RBMX2, REXO4, RNF113A, ZC3H12A, ZFC3H1) have at least one significant match (E < 0.01) to a gold-regulator motif
- tomtom_results.tsv contains both E-values and the matched gold-regulator motif identifiers for every query motif
- Spot-check: MFAP1's motifs match 12 gold regulators with E < 0.01
- TOMTOM run used a GC-matched background model; background composition matches circRNA sequence GC content
- Number of query motifs in the TOMTOM run equals the number of motifs reported by MEME for the 16 candidates

## Limitations

- Motif similarity is an indirect association — the article uses it alongside physical interaction, TF modulation, and co-expression to argue functional connection, not direct binding
- Binding-site prediction/motif models extrapolate poorly: the paper notes such methods 'relied heavily on training datasets and only obtained limited RBP-specific models'
- Generalization beyond cancers is untested; the discussion flags that extension to other diseases would require additional expression profiles and fine-tuning

## Evidence

- [other] Compare each discovered motif against the motif set of the collected gold regulators using TOMTOM with a GC-matched background model and a significance threshold of E < 0.01.: [evidence span withheld — anchor status: not_found]
