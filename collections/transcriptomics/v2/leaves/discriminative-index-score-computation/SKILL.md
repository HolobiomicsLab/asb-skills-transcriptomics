---
name: discriminative-index-score-computation
description: Use when ranking genes from high-dimensional labeled RNA-seq/expression
  matrices by classification power, as an alternative to DEG analysis (e.g., edgeR)
  for selecting biomarker candidates.
when_to_use_negative:
- Input is already a small curated feature set or gene signature — Wx is designed
  to rank from thousands of features in high-dimensional NGS data
- The analysis goal is differential-expression testing with adjusted p-values; edgeR-style
  DEG analysis is the complementary approach described in the paper
- Labeled sample groups (e.g., cancer vs normal) are unavailable, since the DI score
  requires training a supervised softmax classifier on class labels
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Wx
  role: Neural network-based feature selection algorithm whose DI score is computed
    and used to rank genes
  repo: https://github.com/deargen/DearWXpub
- name: Keras
  role: Framework (v2.1.2) for training the softmax regression single-layer network
    whose parameter vectors feed the DI-score inner products
  repo: https://github.com/deargen/DearWXpub
- name: TensorFlow GPU
  role: Backend (v1.4.0) for the Keras neural network training step
- name: Python
  role: Runtime (v3.4) for the Wx scripts (src/wx_tcga.py, src/wx_examples.py)
- name: TCGA-Assembler
  role: R tool (Module_A.R) used to download the TCGA RNA-seq data that form the expression
    matrix input
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-019-47016-8
    title: 'Wx: a neural network-based feature selection algorithm for transcriptomic
      data'
  source_content_hash: sha256:5db6726ca5ed5dca262532f444194c235564fc98f9e658991354d38d59a03fae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deargen/DearWXpub
  tools:
  - Keras
  - Python
  - TCGA-Assembler
  - TensorFlow GPU
  - Wx
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# discriminative-index-score-computation

## Summary

Computes the Wx algorithm's discriminative index (DI) score for every gene by taking inner products of softmax regression parameter vectors with class-average expression vectors, then ranks genes in descending DI-score order. DI score represents classification power for distinguishing given groups (e.g., cancer vs normal) in high-dimensional NGS expression data.

## When to use

Use when ranking genes from high-dimensional labeled RNA-seq/expression matrices by classification power, as an alternative to DEG analysis (e.g., edgeR) for selecting biomarker candidates.

## When NOT to use

- Input is already a small curated feature set or gene signature — Wx is designed to rank from thousands of features in high-dimensional NGS data
- The analysis goal is differential-expression testing with adjusted p-values; edgeR-style DEG analysis is the complementary approach described in the paper
- Labeled sample groups (e.g., cancer vs normal) are unavailable, since the DI score requires training a supervised softmax classifier on class labels

## Inputs

- Labeled preprocessed TCGA pan-cancer gene-expression matrix from the TCGA_DATAS folder (12 cancer types plus normal controls, e.g., 6226 samples: 5609 tumor / 617 normal)
- Class labels distinguishing cancer and normal samples per cancer type

## Outputs

- di_ranked_genes.csv: gene list with per-gene DI scores sorted in strictly descending DI-score order
- Average DI score per gene across 10,000 iterations

## How to apply

Load the labeled preprocessed TCGA gene-expression matrix (from the TCGA_DATAS folder produced by TCGA-Assembler) and train a softmax regression classifier (single-layer neural network) using Keras 2.1.2 backed by TensorFlow GPU 1.4.0 on Python 3.4. Compute the class-average expression vector for each class (e.g., cancer and normal groups). For every pair of classes, take the inner product of the softmax parameter vectors with the class-average vectors to derive a per-gene DI score. Because selected features vary with random initial values, iterate the algorithm 10,000 times and rank genes by the average DI score across iterations. Sort genes in descending DI-score order and write the ranked gene list with scores to di_ranked_genes.csv.

## Related tools

- **Wx** (Neural network-based feature selection algorithm whose DI score is computed and used to rank genes) — https://github.com/deargen/DearWXpub
- **Keras** (Framework (v2.1.2) for training the softmax regression single-layer network whose parameter vectors feed the DI-score inner products) — https://github.com/deargen/DearWXpub
- **TensorFlow GPU** (Backend (v1.4.0) for the Keras neural network training step)
- **Python** (Runtime (v3.4) for the Wx scripts (src/wx_tcga.py, src/wx_examples.py))
- **TCGA-Assembler** (R tool (Module_A.R) used to download the TCGA RNA-seq data that form the expression matrix input)

## Examples

```
$ python src/wx_tcga.py
```

## Evaluation signals

- di_ranked_genes.csv exists and contains a DI score for every gene in the input expression matrix
- The output gene list is sorted in strictly descending DI-score order
- Run-to-run stability: results are averaged over 10,000 iterations so the ranking is not driven by random initial values
- Top-ranked genes should include the paper's Keras Wx 14 markers (EEF1A1, FN1, GAPDH, SFTPC, AHNAK, KLK3, UMOD, CTSB, COL1A1, GPX3, GNAS, ATP1A1, SFTPB, ACTB) when run on comparable TCGA data
- Downstream check: classification accuracy using approximately the top 100 genes should be the highest, with no further increase when more genes are added

## Limitations

- Selected features vary depending on the algorithm used and on which backend learning framework is used (Keras vs the original naive TensorFlow); the README notes 'Selected Features can be different as which backend learning framework used'
- The DI scores are unstable against random initial values, requiring 10,000 iterations of averaging
- Neural network-based approaches require a large number of samples for good classification or regression performance
- The 14-gene signature (Wx-14-UGCB) lacks extensive experimental validation; further experimental validation is needed to prove the algorithm's usefulness

## Evidence

- [abstract] DI score represents classification power: "The Wx algorithm ranks genes based on the discriminative index (DI) score that represents the classification power for distinguishing given groups."
- [results] 10,000 iterations rationale: "we avoided this irregularity by iterating the Wx algorithm 10,000 times and the highest genes (features) ranked by the average value of the DI score were selected as UGCBs"
- [methods] software environment: "$ python 3.4
$ tensorflow gpu 1.4.0
$ keras 2.1.2"
- [readme] backend-dependence caveat: "Selected Features can be different as which backend learning framework used"
- [results] top-100 accuracy plateau: "Approximately the top 100 genes showed the highest average accuracy and no further increase in average accuracy was observed when more genes were added"
