---
name: class-average-vector-inner-product-analysis
description: Use when a labeled gene-expression matrix (e.g., TCGA RNA-seq tumor vs.
  normal) must be reduced to a ranked feature list, and ranking should reflect a classifier's
  learned weights rather than differential-expression p values.
when_to_use_negative:
- The goal is differential-expression testing with statistical significance measures
  rather than classifier-based feature ranking — edgeR DEG analysis would be the complementary
  approach.
- There are no class labels, so class-average vectors and softmax parameter vectors
  cannot be formed.
- Only a small sample set is available — the article notes a large number of samples
  are needed for neural-network-based classification to perform well.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Wx
  role: Feature selection algorithm whose softmax weights and class-average inner
    products produce DI-score gene rankings
  repo: https://github.com/deargen/DearWXpub
- name: Keras
  role: Framework used to train the single-layer softmax regression network (version
    2.1.2)
  repo: https://github.com/deargen/DearWXpub
- name: TensorFlow GPU
  role: Backend running the Keras classifier (version 1.4.0)
- name: Python
  role: Environment and scripting language (version 3.4) used with pandas to load
    the expression matrix
- name: TCGA-Assembler
  role: Module_A.R tool used to download the TCGA RNA-seq data before feature selection
  repo: http://www.compgenome.org/TCGA-Assembler/index.php
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-019-47016-8
    title: 'Wx: a neural network-based feature selection algorithm for transcriptomic
      data'
  source_content_hash: sha256:2933fe4345bd972202cf74c225a67a98fd18e36be92b4e5da0eeccd0b9ebd287
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# class-average-vector-inner-product-analysis

## Summary

Ranks genes from high-dimensional NGS expression data using the Wx algorithm: a Keras softmax-regression classifier's weight vectors are inner-producted with class-average expression vectors to compute a per-gene discriminative index (DI) score representing classification power.

## When to use

Use when a labeled gene-expression matrix (e.g., TCGA RNA-seq tumor vs. normal) must be reduced to a ranked feature list, and ranking should reflect a classifier's learned weights rather than differential-expression p values.

## When NOT to use

- The goal is differential-expression testing with statistical significance measures rather than classifier-based feature ranking — edgeR DEG analysis would be the complementary approach.
- There are no class labels, so class-average vectors and softmax parameter vectors cannot be formed.
- Only a small sample set is available — the article notes a large number of samples are needed for neural-network-based classification to perform well.

## Inputs

- Labeled preprocessed TCGA gene-expression matrix from the TCGA_DATAS folder (e.g., 6,226 samples: 5,609 tumor / 617 normal across 12 cancer types)
- Class labels for each sample (cancer type vs. normal control)

## Outputs

- Ranked gene list with DI scores in strictly descending order (e.g., di_ranked_genes.csv)
- Top-k gene panel, e.g., the Wx-14-UGCB: EEF1A1, FN1, GAPDH, SFTPC, AHNAK, KLK3, UMOD, CTSB, COL1A1, GPX3, GNAS, ATP1A1, SFTPB, ACTB

## How to apply

Load the labeled, preprocessed TCGA gene-expression matrix (TCGA_DATAS folder, assembled via TCGA-Assembler Module_A.R) with Python 3.4 and pandas. Train a single-layer softmax regression network with Keras 2.1.2 on TensorFlow GPU 1.4.0 to classify the sample classes (e.g., 12 cancer types vs. normal). Compute the class-average expression vector for each class, then for each pair of classes take the inner product of the softmax parameter vectors with the class-average vectors to derive a per-gene DI score. Sort genes in descending DI score and export the ranked list (e.g., di_ranked_genes.csv). Because softmax weights depend on random initialization, iterate the procedure 10,000 times and rank genes by the average DI score across iterations, as done to select the Wx-14-UGCB biomarker set.

## Related tools

- **Wx** (Feature selection algorithm whose softmax weights and class-average inner products produce DI-score gene rankings) — https://github.com/deargen/DearWXpub
- **Keras** (Framework used to train the single-layer softmax regression network (version 2.1.2)) — https://github.com/deargen/DearWXpub
- **TensorFlow GPU** (Backend running the Keras classifier (version 1.4.0))
- **Python** (Environment and scripting language (version 3.4) used with pandas to load the expression matrix)
- **TCGA-Assembler** (Module_A.R tool used to download the TCGA RNA-seq data before feature selection) — http://www.compgenome.org/TCGA-Assembler/index.php

## Examples

```
python src/wx_tcga.py
```

## Evaluation signals

- di_ranked_genes.csv exists and contains a DI score for every gene in the input matrix.
- The output is sorted in strictly descending DI-score order.
- Stability check: genes are ranked by the average DI score over 10,000 iterations, and the top approximately 100 genes should give the highest average classification accuracy with no further increase beyond that.
- Downstream sanity check: a classifier built on the top 14 ranked genes should reach accuracy comparable to or above edgeR-based 14-gene panels (Keras Wx 14: 96.72% overall vs. EdgeR 14: 94.81% on 3,119 TCGA validation samples).

## Limitations

- Selected features vary depending on the algorithm and even on the backend framework used; Keras-based Wx rankings can differ from the original naive TensorFlow implementation.
- Softmax outputs depend on random initialization, so the algorithm must be iterated 10,000 times to avoid irregular DI rankings.
- The 14-gene signature (Wx-14-UGCB) lacks extensive experimental validation, and neural-network approaches require a large number of samples for good performance.

## Evidence

- [abstract] Wx ranks genes based on a discriminative index (DI) score representing classification power for distinguishing given groups: "The Wx algorithm ranks genes based on the discriminative index (DI) score that represents the classification power for distinguishing given groups."
- [results] 10,000 iterations averaging DI scores: "we avoided this irregularity by iterating the Wx algorithm 10,000 times and the highest genes (features) ranked by the average value of the DI score were selected as UGCBs"
- [results] top 100 genes saturate accuracy: "Approximately the top 100 genes showed the highest average accuracy and no further increase in average accuracy was observed when more genes were added"
- [readme] environment versions: "$ python 3.4
$ tensorflow gpu 1.4.0
$ keras 2.1.2"
- [readme] features differ by backend framework: "Selected Features can be different as which backend learning framework used"
