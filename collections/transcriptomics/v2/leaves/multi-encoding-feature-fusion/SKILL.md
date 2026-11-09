---
name: multi-encoding-feature-fusion
description: Use when building sequence-based ML predictors from labeled RNA segments
  (e.g., 41-bp m5C windows) and you need diverse informatic sequence profiles fused
  into one feature matrix.
when_to_use_negative:
- Input is already an encoded numeric feature table — re-encoding sequences would
  duplicate features and inflate dimensionality
- 'You need a general cross-species m5C predictor: the article notes it ''could be
  difficult to develop a general model for cross-species prediction, and it is necessary
  to set up species-specific models'''
- The task is single-encoding-only modeling (e.g., a quick SVM baseline on k-mer alone)
  where fusion adds unnecessary dimensionality before F-score filtering
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0097
tools:
- name: Staem5
  role: Downstream stacked-ensemble predictor consuming the fused PSP/k-mer/PCPseDNC/PseEIIP
    feature matrix; source code used to reproduce the fusion pipeline
  repo: https://github.com/Cxd-626/Staem5.git
- name: iLearn
  role: Computes PCPseDNC features encoding 38 dinucleotide physicochemical properties
- name: m5C-PseDNC
  role: Prior SVM-based m5C predictor relying on pseudo dinucleotide composition,
    motivating the PCPseDNC encoding block
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.omtn.2021.10.012
    title: 'Staem5: A novel computational approach for accurate prediction of m5C
      site'
  source_content_hash: sha256:27974dd3043e1a7b8ff729e6a817bffee8df27b8e84b38852962665e35a611dc
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Cxd-626/Staem5.git
  tools:
  - Staem5
  - iLearn
  - m5C-PseDNC
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0097
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# multi-encoding-feature-fusion

## Summary

Encodes RNA sequence segments into complementary feature blocks — position-specific propensity (PSP), k-mer (k = 1, 2, 3), PCPseDNC, and PseEIIP — and concatenates them into a single fused input matrix for machine-learning-based m5C site prediction (Staem5).

## When to use

Use when building sequence-based ML predictors from labeled RNA segments (e.g., 41-bp m5C windows) and you need diverse informatic sequence profiles fused into one feature matrix.

## When NOT to use

- Input is already an encoded numeric feature table — re-encoding sequences would duplicate features and inflate dimensionality
- You need a general cross-species m5C predictor: the article notes it 'could be difficult to develop a general model for cross-species prediction, and it is necessary to set up species-specific models'
- The task is single-encoding-only modeling (e.g., a quick SVM baseline on k-mer alone) where fusion adds unnecessary dimensionality before F-score filtering

## Inputs

- Labeled RNA sequence segments in FASTA or tabular form (41-bp windows around m5C candidate sites, positive and negative classes)
- Species-specific benchmark datasets (e.g., A. thaliana from GEO: GSE94065; M. musculus segments)
- Trinucleotide electron-ion interaction pseudo-potential values (for PseEIIP)
- Dinucleotide physicochemical property tables (38 properties, as encoded by iLearn for PCPseDNC)

## Outputs

- fused_features.csv: one row per training/test sample with concatenated PSP, k-mer (k=1,2,3), PCPseDNC, and PseEIIP column blocks
- F-score-reduced feature subset for downstream stacked-ensemble training

## How to apply

Load labeled 41-bp training/independent-test sequence segments (e.g., 5,298 positive + 5,298 negative for A. thaliana; 4,563 + 4,563 for M. musculus, with A. thaliana from GEO: GSE94065). Compute PSP features from position-specific propensity formulas in the supplemental information; compute k-mer features over the nucleotide alphabet for k = 1, 2, and 3; compute PCPseDNC with iLearn, which encodes 38 dinucleotide physicochemical properties; and compute PseEIIP from trinucleotide electron-ion interaction pseudo-potential values. Concatenate the four blocks into one fused matrix (fused_features.csv) with one row per sample. Because the fused dimensionality is high, apply F score feature selection with a small step size (step = 5) to reduce feature dimension and computing time before model training.

## Related tools

- **Staem5** (Downstream stacked-ensemble predictor consuming the fused PSP/k-mer/PCPseDNC/PseEIIP feature matrix; source code used to reproduce the fusion pipeline) — https://github.com/Cxd-626/Staem5.git
- **iLearn** (Computes PCPseDNC features encoding 38 dinucleotide physicochemical properties)
- **m5C-PseDNC** (Prior SVM-based m5C predictor relying on pseudo dinucleotide composition, motivating the PCPseDNC encoding block)

## Examples

```
python m5C_predict.py --input query.fa --output predictions.csv --species Arabidopsis
```

## Evaluation signals

- fused_features.csv exists and contains exactly one row per sample (e.g., 5,298 positive + 5,298 negative rows for A. thaliana; 4,563 + 4,563 for M. musculus)
- All four encoding blocks (PSP, k-mer, PCPseDNC, PseEIIP) are present as identifiable column groups in the fused matrix
- k-mer block contains features for k = 1, 2, and 3 over the nucleotide alphabet
- PCPseDNC block reflects the 38 dinucleotide physicochemical properties as encoded by iLearn
- After F-score selection (step = 5), downstream 10-fold cross-validation accuracy should be comparable to reported baselines (e.g., stacked model reaching 73.89% accuracy and 0.479 MCC for A. thaliana)

## Limitations

- No general cross-species model is feasible; species-specific models are required
- Hyperparameters in the original work were partly pre-set according to previous experience rather than systematically derived, and selected hyperparameters are only available in supplementary Table S2
- Staem5 underperformed competing methods (iRNA-m5C_SVM and m5CPred-SVM) on the independent A. thaliana test set
- Feature selection results (MRMD, PCC, F score) are documented only in supplementary Tables S5–S7, not the main text

## Evidence
