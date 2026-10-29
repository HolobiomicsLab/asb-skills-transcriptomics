---
name: rna-secondary-structure-feature-encoding
description: Use when building deep learning inputs for RNA site prediction where
  secondary structure context should complement sequence, e.g. 51 nt fragments centered
  on a uridine/Ψ site.
when_to_use_negative:
- Fragments whose center nucleotide is not uridine — the article considers only uridine-centered
  fragments for further prediction
- Tasks requiring raw secondary-structure strings rather than fixed-size categorical
  encodings, or sequences whose length differs from 51 nt
edam_operation: http://edamontology.org/operation_0278
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3474
tools:
- name: RNAshapes
  role: Generates abstract secondary structure dot-bracket notation for each 51 nt
    fragment
- name: EDeN
  role: Converts RNAshapes dot-bracket structures into six sub-shape context categories
    (S, M, H, I, F, T) used as encoding alphabet
  repo: https://github.com/fabriziocosta/EDeN
- name: MU-PseUDeep
  role: Consumes the 51 × 6 structure matrices as the second input channel of the
    dual-input CNN
  repo: https://github.com/smk5g5/MU-PseUDeep
- name: BEDTools
  role: Extracts each Ψ site with 25 bases of upstream and downstream flanking sequence
    from reference genomes
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2020.07.010
    title: 'MU-PseUDeep: A deep learning method for prediction of pseudouridine sites'
  source_content_hash: sha256:7ee1b2fa6def3ed09e022ee426b64653176adffe86f324210d288aa14f390a44
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fabriziocosta/EDeN
  tools:
  - BEDTools
  - EDeN
  - Hyperas
  - Hyperopt
  - MU-PseUDeep
  - PseUI
  - RNAshapes
  - TensorFlow
  - XG-PseU
  - iRNA-PseU
  edam_operation: http://edamontology.org/operation_0278
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# rna-secondary-structure-feature-encoding

## Summary

Encodes predicted RNA secondary structure context as a fixed-size one-of-K binary matrix (51 × 6) for use as a second input channel to a dual-input CNN, alongside the one-hot encoded sequence (51 × 4), so that structural signal can improve RNA modification site prediction.

## When to use

Use when building deep learning inputs for RNA site prediction where secondary structure context should complement sequence, e.g. 51 nt fragments centered on a uridine/Ψ site.

## When NOT to use

- Fragments whose center nucleotide is not uridine — the article considers only uridine-centered fragments for further prediction
- Tasks requiring raw secondary-structure strings rather than fixed-size categorical encodings, or sequences whose length differs from 51 nt

## Inputs

- RNA fragments of 51 nt centered on candidate Ψ/uridine sites (Ψ plus 25 flanking bases each side) derived from RMBase v2.0 annotations
- Reference genome FASTA files (hg19, mm10, sacCer3) for flanking sequence extraction with BEDTools

## Outputs

- One-of-K binary structure matrix of size 51 × 6 (one column per sub-shape context S, M, H, I, F, T) per fragment
- Paired one-of-K binary sequence matrix of size 51 × 4 per fragment
- Dot-bracket secondary structure strings from RNAshapes

## How to apply

Extract each candidate site as a 51 nt fragment (the modified nucleotide plus 25 bases upstream and downstream). Generate abstract secondary structure dot-bracket notation with the RNAshapes package, then convert the structure using EDeN into six sub-shape context categories (S, M, H, I, F, T). Encode the structure as a one-of-K binary matrix of size 51 × 6 and the nucleotide sequence in parallel as a 51 × 4 matrix; both matrices feed paired 1D CNN layers in Keras. Retain only fragments with uridine at their center before prediction. The rationale is that structure contexts provide orthogonal signal to sequence, improving Ψ site prediction over sequence-only CNNs by 3–4% accuracy/F1 on balanced data.

## Related tools

- **RNAshapes** (Generates abstract secondary structure dot-bracket notation for each 51 nt fragment)
- **EDeN** (Converts RNAshapes dot-bracket structures into six sub-shape context categories (S, M, H, I, F, T) used as encoding alphabet) — https://github.com/fabriziocosta/EDeN
- **MU-PseUDeep** (Consumes the 51 × 6 structure matrices as the second input channel of the dual-input CNN) — https://github.com/smk5g5/MU-PseUDeep
- **BEDTools** (Extracts each Ψ site with 25 bases of upstream and downstream flanking sequence from reference genomes)

## Examples

```
from hyperas import optim; X_seq, X_struct = encode_one_of_K(fragments)  # X_struct: (N, 51, 6), X_seq: (N, 51, 4)
```

## Evaluation signals

- Every encoded matrix has shape exactly 51 × 6 (structure) and 51 × 4 (sequence), with one '1' per position row
- Structure alphabet restricted to the six EDeN sub-shape contexts S, M, H, I, F, T — no other symbols present
- Each encoded fragment has uridine at position 26 (the center of the 51 nt window)
- Downstream check: the dual-input CNN using these matrices should outperform a sequence-only CNN by ~3–4% accuracy and F1 (up to 9% sensitivity) on the balanced human dataset

## Limitations

- Secondary structure is predicted (abstract shapes via RNAshapes), not experimentally determined, so encoding quality depends on the structure predictor
- Encoding is fixed to 51 nt windows and six shape contexts; it does not generalize to variable-length inputs or alternative structure representations
- No source code or webserver was available for iPseU-CNN, preventing direct comparison with the sequence-only deep learning baseline

## Evidence

- [methods] Sequence data was converted into a one-of-K encoding binary matrix of size 51 × 4: "Sequence data was converted into a one-of-K encoding binary matrix of size 51 × 4"
- [methods] abstract secondary structure dot-bracket notation was generated using the RNAshapes package: "the abstract secondary structure dot-bracket notation was generated using the RNAshapes package"
- [methods] we extracted the Ψ and surrounding 25 bases upstream and downstream nucleotides using BEDTools: "we extracted the Ψ and surrounding 25 bases upstream and downstream nucleotides using BEDTools"
- [methods] Only those fragments with uridine at their center were considered for further prediction: "Only those fragments with uridine at their center were considered for further prediction"
- [results] The improvement was by 3–4% for accuracy and F1 and up to 9% for sensitivity in the balanced dataset: "The improvement was by 3–4% for accuracy and F1 and up to 9% for sensitivity in the balanced dataset in comparison to sequence CNN"
