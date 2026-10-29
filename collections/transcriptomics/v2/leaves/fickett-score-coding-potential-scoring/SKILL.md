---
name: fickett-score-coding-potential-scoring
description: Use when reconstructing LncDC feature extraction from H-Train-style mRNA/CDS/lncRNA
  FASTA inputs, or when needing an alignment-free coding-potential value per transcript
  to feed a coding/noncoding classifier.
when_to_use_negative:
- Input is already a feature matrix or imputer/scaler outputs exist — the Fickett
  score is a raw per-transcript feature, not a post-processing step on scaled data
- Only secondary structure features (SSFs) are needed — Fickett scoring belongs to
  the SIFs and requires only sequence, not RNAfold folding
- Aligning transcripts to protein databases is the goal (e.g. CPC-style homology search)
  — LncDC is explicitly an alignment-free approach
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3474
tools:
- name: LncDC
  role: Computes the Fickett score as part of SIFs during feature extraction for lncRNA/mRNA
    classification
  repo: https://github.com/lim74/LncDC
- name: Python
  role: Implementation language of LncDC; Biopython used to load FASTA inputs for
    feature computation
  repo: https://github.com/lim74/LncDC
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-22082-7
    title: 'LncDC: a machine learning-based tool for long non-coding RNA detection
      from RNA-Seq data'
  source_content_hash: sha256:13af6661beb7106db7bdbc5909dddc3ed8fa02bc6c961c296694bb2f6bcf9fde
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lim74/LncDC
  tools:
  - LncDC
  - Python
  - ViennaRNA
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# fickett-score-coding-potential-scoring

## Summary

Compute the Fickett coding-potential score for RNA transcripts as one of the sequence intrinsic features (SIFs) in LncDC's 57-column feature matrix used by the XGBoost model to discriminate lncRNAs from mRNAs.

## When to use

Use when reconstructing LncDC feature extraction from H-Train-style mRNA/CDS/lncRNA FASTA inputs, or when needing an alignment-free coding-potential value per transcript to feed a coding/noncoding classifier.

## When NOT to use

- Input is already a feature matrix or imputer/scaler outputs exist — the Fickett score is a raw per-transcript feature, not a post-processing step on scaled data
- Only secondary structure features (SSFs) are needed — Fickett scoring belongs to the SIFs and requires only sequence, not RNAfold folding
- Aligning transcripts to protein databases is the goal (e.g. CPC-style homology search) — LncDC is explicitly an alignment-free approach

## Inputs

- Transcript sequences in FASTA format (regular text or gzip-compressed *.gz), e.g. mrna.fa and lncrna.fa
- CDS sequences in FASTA format, ordered and numbered identically to the mRNA sequences (cds.fa)
- Training hexamer table (e.g. train_hexamer_table.csv) when computing companion SIFs in the same pass

## Outputs

- Fickett coding-potential score per transcript, retained as a raw (un-imputed, unscaled) value
- One row per H-Train transcript in the assembled 57-column feature matrix (e.g. h_train_feature_matrix.csv)

## How to apply

Load each transcript's FASTA sequence (plain or gzip-compressed) with Biopython, matching each mRNA record to its CDS record in the given order as required by lncDC-train.py. Compute the Fickett score from the transcript sequence as one of the SIFs, alongside the hexamer score against the training hexamer table, GC content, and ORF features for the four FEELnc-style ORF types (type 0-3: max ORF length, ORF coverage, ORF hexamer score, and RCB). The Fickett score is retained as a raw value in the per-transcript row; imputation and scaling happen downstream via the imputer/scaler pkl files (e.g. imputer_SIF_PF.pkl, scaler_SIF_PF.pkl) produced during training. Note the article's caveat that SIFs and PFs already achieve high accuracy on their own, so the Fickett score is a strong single contributor to the 57-feature matrix rather than a standalone classifier.

## Related tools

- **LncDC** (Computes the Fickett score as part of SIFs during feature extraction for lncRNA/mRNA classification) — https://github.com/lim74/LncDC
- **Python** (Implementation language of LncDC; Biopython used to load FASTA inputs for feature computation) — https://github.com/lim74/LncDC

## Examples

```
python bin/lncDC-train.py -m mrna_mouse.fasta -c cds_mouse.fasta -l lncrna_mouse.fasta -o self_mouse -t 8
```

## Evaluation signals

- Output matrix contains exactly one row per H-Train transcript with all 57 feature columns populated
- Fickett score column contains raw values prior to the imputation/scaling stage (imputer.pkl and scaler.pkl applied only downstream)
- Each mRNA record is matched to its CDS record in the given order — mismatches in count or order invalidate the feature row
- XGBoost model trained on features including the Fickett score outperforms CPAT, PLEK, CPC2, CPPred, LncFinder, and COME on human and mouse test sets

## Limitations

- Fickett score is one SIF among 57 features; the article notes SIFs and PFs already achieve high accuracy, so further gains from adding SSFs were difficult
- Feature quality depends on correctly paired mRNA/CDS FASTA files (same order and number of sequences), a training-input requirement of lncDC-train.py
- For non-model organisms without well-annotated lncRNAs, the training hexamer table and downstream model must come from an evolutionarily close organism (e.g. Zebrafish, Drosophila melanogaster)

## Evidence

- [other] Compute sequence intrinsic features (SIFs): Fickett coding-potential score, hexamer score against the training hexamer table, GC content, and ORF features: [evidence span withheld — anchor status: not_found]
- [intro] LncDC integrates sequence intrinsic features (SIFs), secondary structure features (SSFs), and protein features (PFs): "we developed an alignment-free, machine learning-based tool named LncDC, which integrates sequence intrinsic features (SIFs), secondary structure features (SSFs), and protein features (PFs)"
- [readme] The fasta file could be regular text file or gzip compressed file (*.gz): "The fasta file could be regular text file or gzip compressed file (*.gz)."
- [readme] The order and number of the CDS sequences should be the same as the mRNA sequences: "The order and number of the CDS sequences should be the same as the mRNA sequences."
- [intro] CPC2 integrated a SVM model with Fickett score, ORF length, ORF integrity, and protein pH isoelectric point (PI) features: "CPC2 is an upgraded version of CPC, which integrated a SVM model with Fickett score, ORF length, ORF integrity, and protein pH isoelectric point (PI) features"
