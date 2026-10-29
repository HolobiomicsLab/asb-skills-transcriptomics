---
name: position-specific-propensity-sequence-encoding
description: Use when building machine-learning inputs for m5C site prediction from
  labeled 41-nt RNA sequence segments (e.g., A. thaliana or M. musculus benchmarks)
  and you need positional, compositional, and physicochemical feature encodings fused
  into one matrix.
when_to_use_negative:
- Input sequences are not 41-bp segments centered on a candidate m5C site, or come
  from a species other than Arabidopsis or Mouse — the article notes it is necessary
  to set up species-specific models
- Input is already a numeric feature table; this skill operates on raw nucleotide
  sequences
- Only sequence-motif visualization is needed (Two Sample Logo analysis) rather than
  quantitative feature encoding for classification
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3316
tools:
- name: Staem5
  role: Downstream stacked-ensemble m5C predictor that consumes the fused PSP/k-mer/PCPseDNC/PseEIIP
    feature matrix
  repo: https://github.com/Cxd-626/Staem5.git
- name: iLearn
  role: Computes the PCPseDNC encoding based on 38 dinucleotide physicochemical properties
- name: PEA-m5C
  role: Prior RF-based m5C predictor whose feature-encoding-then-predict workflow
    this skill parallels for sequence segments
  repo: https://github.com/cma2015/PEA-m5C
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:7fc387d187951e2449fff5380d6df64f583c182d4531ddd7f0ece20eaafe73db
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Cxd-626/Staem5.git
  tools:
  - PEA-m5C
  - Staem5
  - iLearn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# position-specific-propensity-sequence-encoding

## Summary

Encodes 41-bp RNA segments centered on candidate m5C sites using position-specific propensity (PSP) together with k-mer (k=1,2,3), PCPseDNC, and PseEIIP encodings, then fuses them into a single input matrix for the Staem5 stacked-ensemble m5C predictor.

## When to use

Use when building machine-learning inputs for m5C site prediction from labeled 41-nt RNA sequence segments (e.g., A. thaliana or M. musculus benchmarks) and you need positional, compositional, and physicochemical feature encodings fused into one matrix.

## When NOT to use

- Input sequences are not 41-bp segments centered on a candidate m5C site, or come from a species other than Arabidopsis or Mouse — the article notes it is necessary to set up species-specific models
- Input is already a numeric feature table; this skill operates on raw nucleotide sequences
- Only sequence-motif visualization is needed (Two Sample Logo analysis) rather than quantitative feature encoding for classification

## Inputs

- Labeled 41-bp RNA sequence segments centered on candidate m5C sites (positive and negative sets, FASTA/CSV)
- A. thaliana benchmark sequences from GEO: GSE94065 and M. musculus benchmark sequences
- Trinucleotide electron-ion interaction pseudo-potential values for PseEIIP
- 38 dinucleotide physicochemical property definitions (via iLearn) for PCPseDNC

## Outputs

- fused_features.csv with one row per training sample and columns for PSP, k-mer (k=1,2,3), PCPseDNC, and PseEIIP encoding blocks
- Encoded feature matrices for training and independent test sets ready for Staem5's F-score feature selection and stacked ensemble

## How to apply

Load the labeled m5C training and independent test 41-bp sequence segments for A. thaliana (from GEO: GSE94065) and M. musculus. Compute PSP (position-specific propensity) features per sequence segment using the definitions and formulas in the supplemental information, and compute k-mer features with k = 1, 2, and 3 over the nucleotide alphabet. Compute PCPseDNC (parallel correlation pseudo dinucleotide composition) with iLearn, which encodes 38 dinucleotide physicochemical properties, and compute PseEIIP from trinucleotide electron-ion interaction pseudo-potential values. Concatenate the PSP, k-mer, PCPseDNC, and PseEIIP blocks into one fused matrix (e.g., fused_features.csv); downstream, Staem5 applies F score feature selection (step size 5) before feeding the stacked ensemble of SVM, XGBoost, and LightGBM base classifiers with logistic regression as the meta-model.

## Related tools

- **Staem5** (Downstream stacked-ensemble m5C predictor that consumes the fused PSP/k-mer/PCPseDNC/PseEIIP feature matrix) — https://github.com/Cxd-626/Staem5.git
- **iLearn** (Computes the PCPseDNC encoding based on 38 dinucleotide physicochemical properties)
- **PEA-m5C** (Prior RF-based m5C predictor whose feature-encoding-then-predict workflow this skill parallels for sequence segments) — https://github.com/cma2015/PEA-m5C

## Examples

```
python m5C_predict.py --input query_seqs.fa --output predictions.csv --species Arabidopsis
```

## Evaluation signals

- fused_features.csv exists and contains exactly one row per training sample (e.g., 5,298 positive + 5,298 negative rows for A. thaliana; 4,563 + 4,563 for M. musculus)
- All four encoding blocks (PSP, k-mer k=1/2/3, PCPseDNC, PseEIIP) are present as column groups in the fused matrix
- PCPseDNC columns match the 38 dinucleotide physicochemical properties encoded by iLearn
- Feature dimensions before and after F-score selection (step size 5) are consistent with the reduced dimension used by the base classifiers
- Encoded matrices reproduce reported Staem5 performance when fed to the SVM/XGBoost/LightGBM stacking with logistic regression meta-classifier

## Limitations

- No general cross-species model: species-specific models are required for A. thaliana and M. musculus
- Staem5 underperformed competing methods (iRNA-m5C_SVM and m5CPred-SVM) on the independent A. thaliana test set
- Detailed feature-selection results are only available in supplementary Tables S5–S7, and selected hyperparameters only in Table S2

## Evidence
