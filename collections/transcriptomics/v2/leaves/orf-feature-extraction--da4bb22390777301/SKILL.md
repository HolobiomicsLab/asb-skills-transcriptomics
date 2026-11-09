---
name: orf-feature-extraction
description: Use when building a feature matrix from FASTA transcripts (e.g. H-Train
  mRNA/CDS/lncRNA files) for coding-potential classification and you need ORF-derived
  features beyond the single longest-ORF convention.
when_to_use_negative:
- Input is already an imputed/scaled feature matrix or a prediction output csv — this
  skill operates on raw transcript FASTA sequences
- Only the single longest ORF is desired (e.g. when replicating a longest-ORF tool
  like CPC-style pipelines) rather than LncDC's four flexible ORF types
- You need secondary structure features (SSFs) — those require separate RNAfold folding
  and k-mer scoring, not ORF feature extraction
edam_operation: http://edamontology.org/operation_0436
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3474
tools:
- name: LncDC
  role: Source implementation defining the four FEELnc-style ORF types and the ORF-associated
    features (max ORF length, coverage, hexamer score, RCB)
  repo: https://github.com/lim74/LncDC
- name: Python
  role: Implementation language; loads FASTA files and assembles the per-transcript
    feature matrix
  repo: https://github.com/lim74/LncDC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-22082-7
    title: 'LncDC: a machine learning-based tool for long non-coding RNA detection
      from RNA-Seq data'
  source_content_hash: sha256:0f17f3738c97402255a89b5047f59315ddf61da36ee7e343e7c3cfd5a7ec5881
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lim74/LncDC
  tools:
  - LncDC
  - Python
  - ViennaRNA
  edam_operation: http://edamontology.org/operation_0436
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# orf-feature-extraction

## Summary

Extract ORF-associated coding-potential features from RNA transcripts following LncDC's four FEELnc-style ORF types (type 0-3), computing maximum ORF length, ORF coverage, ORF hexamer score, and relative codon bias (RCB) per ORF category as part of the sequence intrinsic features (SIFs) feeding the XGBoost lncRNA/mRNA classifier.

## When to use

Use when building a feature matrix from FASTA transcripts (e.g. H-Train mRNA/CDS/lncRNA files) for coding-potential classification and you need ORF-derived features beyond the single longest-ORF convention.

## When NOT to use

- Input is already an imputed/scaled feature matrix or a prediction output csv — this skill operates on raw transcript FASTA sequences
- Only the single longest ORF is desired (e.g. when replicating a longest-ORF tool like CPC-style pipelines) rather than LncDC's four flexible ORF types
- You need secondary structure features (SSFs) — those require separate RNAfold folding and k-mer scoring, not ORF feature extraction

## Inputs

- mRNA transcript FASTA (plain or gzip-compressed)
- Corresponding CDS FASTA with identical order and count to the mRNA file
- lncRNA FASTA
- Hexamer table csv built from the training data (e.g. train_hexamer_table.csv)

## Outputs

- Per-transcript feature rows with ORF features for four ORF types (max ORF length, ORF coverage, ORF hexamer score, RCB) plus Fickett score, hexamer score, and GC content
- Feature matrix (e.g. h_train_feature_matrix.csv) with 57 feature columns and one row per H-Train transcript, raw (pre-imputation/scaling) values

## How to apply

Load the mRNA, CDS, and lncRNA FASTA files (plain or gzip-compressed) with Biopython, matching each mRNA to its CDS record in the given order. Define four ORF types as in FEELnc and, for each type, compute the maximum ORF length, the ORF coverage of the transcript, an ORF hexamer score against the training hexamer table, and the relative codon bias (RCB); pair these with the Fickett score, transcript-level hexamer score, and GC content to form the SIFs. Translate ORFs to compute ProtParam protein features (PFs), then assemble one row per transcript into a feature matrix such as h_train_feature_matrix.csv (57 columns for H-Train). Retain raw values at this stage because imputation/scaling are performed later by separately persisted imputer and scaler pkl files.

## Related tools

- **LncDC** (Source implementation defining the four FEELnc-style ORF types and the ORF-associated features (max ORF length, coverage, hexamer score, RCB)) — https://github.com/lim74/LncDC
- **Python** (Implementation language; loads FASTA files and assembles the per-transcript feature matrix) — https://github.com/lim74/LncDC

## Examples

```
python bin/lncDC-train.py -m mrna_mouse.fasta -c cds_mouse.fasta -l lncrna_mouse.fasta -o self_mouse -t 8
```

## Evaluation signals

- Output matrix has exactly one row per H-Train transcript and 57 populated feature columns
- Each mRNA record is matched to its CDS record in the same order (order and number of CDS sequences must equal the mRNA sequences)
- All four ORF types (type 0-3) yield the four ORF-associated values (max ORF length, coverage, hexamer score, RCB) per transcript
- Raw feature values are retained prior to imputation/scaling — imputer and scaler are separate pkl artifacts (e.g. imputer_SIF_PF.pkl, scaler_SIF_PF.pkl)

## Limitations

- CDS FASTA must contain the same sequences in the same order as the mRNA FASTA; mismatched order breaks mRNA-CDS feature pairing
- ViennaRNA/RNAfold is not needed for ORF/SIF/PF features alone but is a time-consuming optional dependency when SSFs are also computed
- SIFs and PFs already achieved high accuracy on their own, so ORF-type flexibility mainly helps capture CDS characteristics rather than large standalone gains

## Evidence

- [intro] In order to fully capture the characteristics of CDS, we defined four types of ORF similar to those described in FEELnc: "In order to fully capture the characteristics of CDS, we defined four types of ORF similar to those described in FEELnc"
- [discussion] different types of ORF and their associated features effectively improved the ability of LncDC in capturing CDS characteristics of mRNAs and discriminated them from lncRNAs: "different types of ORF and their associated features effectively improved the ability of LncDC in capturing CDS characteristics of mRNAs and discriminated them from lncRNAs"
- [readme] The order and number of the CDS sequences should be the same as the mRNA sequences.: "The order and number of the CDS sequences should be the same as the mRNA sequences."
- [readme] The fasta file could be regular text file or gzip compressed file (*.gz).: "The fasta file could be regular text file or gzip compressed file (*.gz)."
