---
name: neural-network-feature-selection
description: Use when you must rank/select genes from a high-dimensional labeled RNA-seq
  expression matrix (e.g., TCGA tumor/normal) and DEG analysis alone cannot pick an
  optimal biomarker combination.
when_to_use_negative:
- The analysis goal is differential-expression p-value ranking (e.g., edgeR DEG-14-UGCB)
  rather than classification-power ranking of genes.
- Only a small sample size is available, since neural network-based approaches need
  a large number of samples to achieve good classification performance.
- A single-run ranking suffices without validation — Wx selected features vary between
  runs/algorithms, so unstable one-shot output is a known failure mode, not a use
  case.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Wx
  role: Neural network-based feature (gene) selection algorithm producing the DI-score
    ranked gene list
  repo: https://github.com/deargen/DearWXpub
- name: Keras 2.1.2
  role: Framework for training the softmax regression (single-layer NN) used to compute
    DI scores
- name: TensorFlow GPU 1.4.0
  role: Backend for the Keras classifier during DI-score computation
- name: Python 3.4
  role: Runtime for running the Wx pipeline (e.g., src/wx_tcga.py with pandas)
- name: TCGA-Assembler
  role: Module_A.R tool driven by tcga_download.R to download TCGA RNA-seq data into
    TCGA_DATAS
provenance:
  source_task_ids:
  - task_002
  - task_001
  - task_005
  source_papers:
  - doi: 10.1038/s41598-019-47016-8
    title: 'Wx: a neural network-based feature selection algorithm for transcriptomic
      data'
  source_content_hash: sha256:062576944b8a6609a3beb5b9e337149e02f8cfe47d99fcd33079d7c8bd4ef175
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deargen/DearWXpub
  tools:
  - Keras 2.1.2
  - Python
  - Python 3.4
  - R
  - TCGA-Assembler
  - TensorFlow
  - TensorFlow GPU
  - TensorFlow GPU 1.4.0
  - Wx
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# neural-network-feature-selection

## Summary

The Wx algorithm ranks genes from high-dimensional NGS expression matrices by a discriminative index (DI) score derived from softmax regression parameters and class-average expression vectors, to select universal cancer-vs-normal biomarker candidates.

## When to use

Use when you must rank/select genes from a high-dimensional labeled RNA-seq expression matrix (e.g., TCGA tumor/normal) and DEG analysis alone cannot pick an optimal biomarker combination.

## When NOT to use

- The analysis goal is differential-expression p-value ranking (e.g., edgeR DEG-14-UGCB) rather than classification-power ranking of genes.
- Only a small sample size is available, since neural network-based approaches need a large number of samples to achieve good classification performance.
- A single-run ranking suffices without validation — Wx selected features vary between runs/algorithms, so unstable one-shot output is a known failure mode, not a use case.

## Inputs

- Labeled preprocessed TCGA RNA-seq gene-expression matrix (illuminahiseq_rnaseqv2-RSEM_genes_normalized) with tumor/normal labels from the TCGA_DATAS folder
- TCGA pan-cancer cohort covering 12 cancer types (e.g., BLCA, BRCA, COAD, HNSC, KICH, KIRC, KIRP, LIHC, LUAD, LUSC, PRAD, THCA)

## Outputs

- Ranked gene list with per-gene DI scores (di_ranked_genes.csv / wx_tcga_di_ranked_genes.csv) sorted in strictly descending DI-score order
- Top-k biomarker candidate sets (e.g., Wx-14-UGCB) and their average cancer-vs-normal classification accuracy

## How to apply

Load the labeled preprocessed TCGA expression matrix (e.g., the 6,226-sample pan-cancer matrix: 5,609 tumor / 617 normal) from the TCGA_DATAS folder and train a softmax regression (single-layer neural network) classifier in Keras 2.1.2 backed by TensorFlow GPU 1.4.0 on Python 3.4. Compute the class-average expression vector for each class (cancer and normal), then for every pair of classes take the inner product of the softmax parameter vectors with the class-average vectors to derive a per-gene DI score representing classification power. Sort genes in descending DI score and, to suppress random-initialization irregularity, iterate the algorithm 10,000 times and rank genes by the average DI score across iterations. Write the ranked gene list with DI scores to a CSV (e.g., di_ranked_genes.csv / wx_tcga_di_ranked_genes.csv), then evaluate top-k gene sets (1 to 1,000) by average classification accuracy to choose the biomarker set size (accuracy plateaus around the top 100 genes).

## Related tools

- **Wx** (Neural network-based feature (gene) selection algorithm producing the DI-score ranked gene list) — https://github.com/deargen/DearWXpub
- **Keras 2.1.2** (Framework for training the softmax regression (single-layer NN) used to compute DI scores)
- **TensorFlow GPU 1.4.0** (Backend for the Keras classifier during DI-score computation)
- **Python 3.4** (Runtime for running the Wx pipeline (e.g., src/wx_tcga.py with pandas))
- **TCGA-Assembler** (Module_A.R tool driven by tcga_download.R to download TCGA RNA-seq data into TCGA_DATAS)

## Examples

```
$ python src/wx_tcga.py
```

## Evaluation signals

- di_ranked_genes.csv exists, contains a DI score for every gene in the input matrix (20,501 genes for TCGA pan-cancer), and is sorted in strictly descending DI-score order.
- The TCGA pan-cancer top-14 ranked set reproduces the expected Wx-14 gene list containing GAPDH, with ACTB ranked 14th of 20,501 genes: ['EEF1A1','FN1','GAPDH','SFTPC','AHNAK','KLK3','UMOD','CTSB','COL1A1','GPX3','GNAS','ATP1A1','SFTPB','ACTB'].
- Average classification accuracy peaks near the top 100 genes, with no further increase when more genes are added.
- Across 10,000 iterations, the averaged DI-score ranking is stable (no run-to-run irregularity in the selected top genes).

## Limitations

- Selected features vary depending on the algorithm and backend framework used; Keras-selected features can differ from the naive-TensorFlow original.
- Neural network-based approaches require a large number of samples for good classification or regression performance.
- The 14-gene Wx-14-UGCB signature lacks extensive experimental validation; further experimental verification is needed to prove the algorithm's usefulness.
- Single Wx runs are irregular, hence the 10,000-iteration averaging requirement.

## Evidence

- [abstract] The Wx algorithm ranks genes based on the discriminative index (DI) score that represents the classification power for distinguishing given groups.: "The Wx algorithm ranks genes based on the discriminative index (DI) score that represents the classification power for distinguishing given groups."
- [results] Iterating 10,000 times and selecting by average DI: "we avoided this irregularity by iterating the Wx algorithm 10,000 times and the highest genes (features) ranked by the average value of the DI score were selected as UGCBs"
- [results] Top 100 genes plateau accuracy: "Approximately the top 100 genes showed the highest average accuracy and no further increase in average accuracy was observed when more genes were added"
- [readme] Environment requirements from README: "$ python 3.4
$ tensorflow gpu 1.4.0
$ keras 2.1.2"
- [discussion] Large sample requirement: "a large number of samples are needed to achieve good classification or regression performance"
