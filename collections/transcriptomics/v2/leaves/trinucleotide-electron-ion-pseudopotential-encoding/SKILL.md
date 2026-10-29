---
name: trinucleotide-electron-ion-pseudopotential-encoding
description: Use when converting 41-bp RNA sequence segments (e.g., m5C candidate
  sites from A. thaliana or M. musculus) into PseEIIP numerical features for machine
  learning input.
when_to_use_negative:
- Input sequences are not RNA/DNA sequence segments (e.g., a numeric feature table
  is already provided — no re-encoding is needed)
- The target task requires species-agnostic cross-species m5C prediction; the article
  notes it could be difficult to develop a general model for cross-species prediction
  and species-specific models are necessary
tools:
- name: Staem5
  role: Source pipeline that defines and consumes the PseEIIP encoding as part of
    its four-scheme feature fusion for m5C prediction
  repo: https://github.com/Cxd-626/Staem5.git
- name: iLearn
  role: Toolkit used to compute the companion pseudo-dinucleotide composition encodings
    (e.g., PCPseDNC with 38 dinucleotide physicochemical properties) fused alongside
    PseEIIP
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:4f3d36f99c61ec79aaf9eea360ba5e1322cb0c5671e8b4f95b9d1c721b8769b3
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Cxd-626/Staem5.git
  tools:
  - Staem5
  - iLearn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# trinucleotide-electron-ion-pseudopotential-encoding

## Summary

Computes PseEIIP (electron-ion interaction pseudo potentials of trinucleotide) features for RNA sequence segments, as one of the four encoding schemes (PSP, k-mer k=1–3, PCPseDNC, PseEIIP) fused into the Staem5 m5C-site prediction input matrix.

## When to use

Use when converting 41-bp RNA sequence segments (e.g., m5C candidate sites from A. thaliana or M. musculus) into PseEIIP numerical features for machine learning input.

## When NOT to use

- Input sequences are not RNA/DNA sequence segments (e.g., a numeric feature table is already provided — no re-encoding is needed)
- The target task requires species-agnostic cross-species m5C prediction; the article notes it could be difficult to develop a general model for cross-species prediction and species-specific models are necessary

## Inputs

- Labeled m5C training/independent test RNA sequence segments (41 bp) in FASTA or tabular form, for A. thaliana (GEO: GSE94065) and M. musculus
- Trinucleotide EIIP pseudo-potential value lookup table (from the Staem5 supplemental information)

## Outputs

- PseEIIP feature block (one row per sequence segment, one column per EIIP-derived feature)
- Fused feature matrix (PSP + k-mer + PCPseDNC + PseEIIP blocks concatenated, e.g., fused_features.csv)

## How to apply

Take each labeled m5C sequence segment (41 bp) from the Staem5 benchmark datasets and slide over its trinucleotides, mapping each trinucleotide to its electron-ion interaction pseudo potential value to build the PseEIIP feature block. Compute the three companion blocks — PSP (position-specific propensity), k-mer with k = 1, 2, and 3, and PCPseDNC (parallel correlation pseudo dinucleotide composition, which encodes 38 dinucleotide physicochemical properties via iLearn) — then concatenate all four blocks into a single fused input matrix (e.g., written to fused_features.csv). The rationale is that Staem5's feature fusion tactic leverages these complementary informatic sequence profiles before stacking classifiers (SVM, XGBoost, LightGBM) with F-score-based feature dimension reduction.

## Related tools

- **Staem5** (Source pipeline that defines and consumes the PseEIIP encoding as part of its four-scheme feature fusion for m5C prediction) — https://github.com/Cxd-626/Staem5.git
- **iLearn** (Toolkit used to compute the companion pseudo-dinucleotide composition encodings (e.g., PCPseDNC with 38 dinucleotide physicochemical properties) fused alongside PseEIIP)

## Evaluation signals

- fused_features.csv exists and contains one row per training sample (e.g., 5,298 positive + 5,298 negative rows for A. thaliana; 4,563 + 4,563 for M. musculus)
- All four encoding blocks (PSP, k-mer k=1–3, PCPseDNC, PseEIIP) are present as column groups in the fused matrix
- PseEIIP values fall within the range of the published trinucleotide EIIP values (same numeric scale as the supplemental lookup table)
- Downstream F-score feature selection and stacked model (SVM/XGBoost/LightGBM with logistic regression meta-classifier) trains without schema errors on the fused matrix

## Limitations

- Encoding is defined for the standard nucleotide alphabet of the 41-bp Staem5 segments; sequences with ambiguous or non-canonical bases are outside the paper's scope
- Feature selection and detailed performance metrics are only available in supplementary tables (Tables S1, S5–S7), so expected feature counts cannot be verified from the main text

## Evidence
