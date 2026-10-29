---
name: kmer-nucleotide-frequency-encoding
description: Use when encoding short RNA sequence segments (e.g., 41 bp m5C site flanks
  from A. thaliana or M. musculus benchmark data) into fixed-length numeric features
  for machine learning, specifically when building the Staem5 feature fusion input.
when_to_use_negative:
- The input is already a numeric feature table or embedding — k-mer encoding applies
  to raw nucleotide sequences
- A general cross-species m5C model is needed; the article notes it may be difficult
  to develop a general model and species-specific models are necessary
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0099
tools:
- name: Staem5
  role: The stacked ensemble prediction framework whose feature fusion pipeline consumes
    the k-mer encoding block as input features
  repo: https://github.com/Cxd-626/Staem5.git
- name: iLearn
  role: Used in the same feature-encoding workflow to compute the complementary PCPseDNC
    block (38 dinucleotide physicochemical properties) fused with the k-mer features
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:5ddf31f994578295f927c2d81cb6f1aa736f7e056393464a84898740f02a55ca
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Cxd-626/Staem5.git
  tools:
  - Staem5
  - iLearn
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# kmer-nucleotide-frequency-encoding

## Summary

Computes k-mer (k = 1, 2, and 3) nucleotide frequency features over RNA sequence segments as one of the four input feature blocks for the Staem5 m5C site prediction model, later concatenated with PSP, PCPseDNC, and PseEIIP into a fused feature matrix.

## When to use

Use when encoding short RNA sequence segments (e.g., 41 bp m5C site flanks from A. thaliana or M. musculus benchmark data) into fixed-length numeric features for machine learning, specifically when building the Staem5 feature fusion input.

## When NOT to use

- The input is already a numeric feature table or embedding — k-mer encoding applies to raw nucleotide sequences
- A general cross-species m5C model is needed; the article notes it may be difficult to develop a general model and species-specific models are necessary

## Inputs

- Labeled 41 bp RNA sequence segments (positive and negative m5C sites) in FASTA or sequence-segment form for A. thaliana and M. musculus
- Benchmark training and independent test sets (e.g., 5,298 positive + 5,298 negative for A. thaliana; 4,563 + 4,563 for M. musculus)

## Outputs

- k-mer frequency feature columns (k = 1, 2, 3) per sequence segment
- Fused feature matrix (fused_features.csv) combining k-mer with PSP, PCPseDNC, and PseEIIP blocks, one row per training sample

## How to apply

Load the labeled 41 bp m5C training and independent test sequence segments for A. thaliana (from GEO: GSE94065 benchmark data) and M. musculus. For each sequence, count occurrences of all oligomers with k = 1, 2, and 3 over the nucleotide alphabet to produce the k-mer feature block, alongside PSP, PCPseDNC (computed with iLearn using 38 dinucleotide physicochemical properties), and PseEIIP blocks. Concatenate the k-mer block with the other three encoding blocks into a single fused input matrix (fused_features.csv) with one row per sample. Because the fused feature space is large, downstream F score feature selection with a step of 5 can be applied to reduce dimension and computing time. The k-mer counts serve as informatic sequence profiles feeding the stacked ensemble (SVM, XGBoost, LightGBM base classifiers with logistic regression meta-model).

## Related tools

- **Staem5** (The stacked ensemble prediction framework whose feature fusion pipeline consumes the k-mer encoding block as input features) — https://github.com/Cxd-626/Staem5.git
- **iLearn** (Used in the same feature-encoding workflow to compute the complementary PCPseDNC block (38 dinucleotide physicochemical properties) fused with the k-mer features)

## Examples

```
python m5C_predict.py --input query_sequences.fa --species Arabidopsis --output predictions.csv
```

## Evaluation signals

- fused_features.csv exists and contains one row per training sample (5,298 positive + 5,298 negative for A. thaliana; 4,563 + 4,563 for M. musculus)
- All four encoding blocks (k-mer, PSP, PCPseDNC, PseEIIP) are present as column groups in the fused matrix
- k-mer features cover k = 1, 2, and 3 over the nucleotide alphabet with consistent counts across samples
- Downstream 10-fold cross-validation accuracy of the stacked model (73.89% accuracy, 0.479 MCC for A. thaliana with SVM, XGBoost, LightGBM) provides an end-to-end sanity check

## Limitations

- k-mer encoding alone is insufficient; it must be fused with PSP, PCPseDNC, and PseEIIP blocks and then reduced via F score to control dimension and computing time
- The stacked model with these features underperformed competing methods on the independent A. thaliana test set
- The source code repo README documents only the prediction script (m5C_predict.py) and its Python dependency pins (numpy 1.19.5, pandas 1.2.4, sklearn 0.21.3, etc.), not the feature-encoding training pipeline details

## Evidence
