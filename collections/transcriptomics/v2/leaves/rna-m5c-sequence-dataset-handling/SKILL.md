---
name: rna-m5c-sequence-dataset-handling
description: Use when the input is RNA cytosine-site sequence data (FASTA windows
  around candidate m5C sites) that must be split into species-specific positive/negative
  training and independent test sets for m5C prediction.
when_to_use_negative:
- Input is already an encoded feature table rather than raw FASTA sequence windows
- The task requires a general cross-species m5C model — the paper states species-specific
  models are necessary
- Data are non-RNA (DNA methylated sites or protein modifications) outside the m5C
  scope
edam_operation: http://edamontology.org/operation_0253
edam_topics:
- http://edamontology.org/topic_0097
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: Staem5
  role: Stacked-ensemble m5C predictor whose CLI (m5C_predict.py) consumes FASTA query
    sequences with a species flag
  repo: https://github.com/Cxd-626/Staem5.git
- name: PEA-m5C
  role: Alternative R/JAVA random-forest m5C predictor; its README documents reading
    FASTA files, motif scanning (extra_motif_seq) and feature encoding (FeatureExtract)
    of m5C window sequences
  repo: https://github.com/cma2015/PEA-m5C
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:c8afdcbab39ad777628addbc319c5cd72252937d6ce722c5ca33bb88510f991b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Cxd-626/Staem5.git
  tools:
  - PEA-m5C
  - Staem5
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0253
  edam_topics:
  - http://edamontology.org/topic_0097
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# rna-m5c-sequence-dataset-handling

## Summary

Handling and preparing RNA m5C site sequence datasets (positive/negative site FASTA collections such as Met1320, Met1900, Met935, Met240, Test1157, DatasetCV, DatasetHT, and GSE90963/GSE93749/GSE83432-derived A. thaliana and M. musculus sets) so they can be encoded and passed to species-specific m5C predictors like Staem5.

## When to use

Use when the input is RNA cytosine-site sequence data (FASTA windows around candidate m5C sites) that must be split into species-specific positive/negative training and independent test sets for m5C prediction.

## When NOT to use

- Input is already an encoded feature table rather than raw FASTA sequence windows
- The task requires a general cross-species m5C model — the paper states species-specific models are necessary
- Data are non-RNA (DNA methylated sites or protein modifications) outside the m5C scope

## Inputs

- FASTA file of query RNA sequences (m5C candidate sites / windows)
- Species identifier ('Arabidopsis' or 'Mouse') selecting the pre-trained model
- Positive/negative m5C site training sets (e.g. A. thaliana 5,298:5,298; Met1320, Met1900, Met935, Met240)
- Independent test sets (e.g. Test1157, DatasetHT, DatasetCV)

## Outputs

- Encoded feature matrix (PSP, k-mer k=1,2,3, PCPseDNC, PseEIIP) per sequence
- Per-sequence m5C site prediction results written to the specified output file
- Species-specific training/test dataset tables used for cross-validation

## How to apply

Load m5C site sequences as FASTA, keeping positive and negative site windows separate per species — the Staem5 A. thaliana training set contains 5,298 positive and 5,298 negative m5C site sequences. Maintain species-specific files ('Arabidopsis' or 'Mouse') rather than pooling, since cross-species models were found impractical. Feed query sequences in FASTA format into the predictor via the --input flag, selecting the species with --species so the correct pre-trained Staem5 model and its PSP, k-mer, PCPseDNC, and PseEIIP feature encodings are applied. Ensure the pinned Python environment (python 3.7, sklearn 0.21.3, xgboost 0.90, lightgbm 2.3.0, mlxtend 0.17.3, Bio 0.4.1) is installed before running predictions.

## Related tools

- **Staem5** (Stacked-ensemble m5C predictor whose CLI (m5C_predict.py) consumes FASTA query sequences with a species flag) — https://github.com/Cxd-626/Staem5.git
- **PEA-m5C** (Alternative R/JAVA random-forest m5C predictor; its README documents reading FASTA files, motif scanning (extra_motif_seq) and feature encoding (FeatureExtract) of m5C window sequences) — https://github.com/cma2015/PEA-m5C

## Examples

```
python m5C_predict.py --input query_sites.fa --output m5c_predictions.txt --species Arabidopsis
```

## Evaluation signals

- All query sequences are accepted only in FASTA format; malformed input should fail rather than silently predict
- Species argument resolves strictly to 'Arabidopsis' or 'Mouse'; any other value should be rejected
- Balanced positive:negative counts in training sets (e.g. A. thaliana 5,298:5,298; Met240 120:120) preserved after loading
- Environment matches pinned versions (python 3.7, sklearn 0.21.3, xgboost 0.90, lightgbm 2.3.0) or predictions may not reproduce
- Output file contains one prediction result row per input sequence

## Limitations

- Only two species are currently supported by Staem5 ('Arabidopsis' or 'Mouse'); no general cross-species model exists
- Specific Python package versions are required (e.g. tensorflow 1.14, keras 2.30), which can be difficult to install in modern environments
- Staem5 underperformed competing methods (iRNA-m5C_SVM, m5CPred-SVM) on the independent A. thaliana test set

## Evidence
