---
name: gene-expression-feature-ranking
description: Use when you need to rank/ prioritize genes from a labeled high-dimensional
  gene-expression matrix (e.g., TCGA RNA-seq) as biomarker candidates, instead of
  traditional DEG analysis.
when_to_use_negative:
- Input is already a small curated feature/gene panel rather than high-dimensional
  NGS expression data needing feature selection.
- The goal is differential expression testing with adjusted p-values rather than classification-power
  ranking — use a DEG tool like edgeR instead (the paper benchmarks edgeR-14 as the
  comparison).
- There are no class labels or group contrast, since the DI score is defined by distinguishing
  given groups.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: Wx
  role: The neural network-based feature selection algorithm itself; produces the
    DI-score-ranked gene list.
  repo: https://github.com/deargen/DearWXpub
- name: Python 3.4
  role: Runtime environment for running the Wx implementation and data handling with
    pandas.
- name: Keras 2.1.2
  role: Neural network framework used to train the softmax regression (single-layer
    NN) classifier whose weights feed the DI computation.
- name: TensorFlow GPU 1.4.0
  role: Backend for Keras, enabling GPU-accelerated training over the 10,000 Wx iterations.
- name: TCGA-Assembler
  role: Module_A.R tool used to download the TCGA RNA-seq data that form the input
    expression matrix.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-019-47016-8
    title: 'Wx: a neural network-based feature selection algorithm for transcriptomic
      data'
  source_content_hash: sha256:2779e3afc15e1f9389aca3c074b72da641233cfaa0dc4436fa9ec873d614e1b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deargen/DearWXpub
  tools:
  - Keras 2.1.2
  - Python
  - Python 3.4
  - TCGA-Assembler
  - TensorFlow GPU
  - TensorFlow GPU 1.4.0
  - Wx
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# gene-expression-feature-ranking

## Summary

Wx is a neural network (softmax regression)-based feature selection algorithm that ranks genes from high-dimensional NGS transcriptomic data by a discriminative index (DI) score representing classification power for distinguishing groups such as cancer vs. normal.

## When to use

Use when you need to rank/ prioritize genes from a labeled high-dimensional gene-expression matrix (e.g., TCGA RNA-seq) as biomarker candidates, instead of traditional DEG analysis.

## When NOT to use

- Input is already a small curated feature/gene panel rather than high-dimensional NGS expression data needing feature selection.
- The goal is differential expression testing with adjusted p-values rather than classification-power ranking — use a DEG tool like edgeR instead (the paper benchmarks edgeR-14 as the comparison).
- There are no class labels or group contrast, since the DI score is defined by distinguishing given groups.

## Inputs

- Labeled preprocessed TCGA gene-expression matrix (TCGA_DATAS folder, 12 cancer types, e.g., 6226 total samples with 5609 tumor / 617 normal in the README cohort)
- Class labels distinguishing groups (e.g., cancer vs. normal samples)
- Python 3.4 + pandas environment with TensorFlow GPU 1.4.0 and Keras 2.1.2

## Outputs

- di_ranked_genes.csv: gene list sorted in strictly descending DI-score order, with one DI score per gene in the input matrix
- Ranked gene set usable as biomarker candidates (e.g., Wx-14-UGCB: EEF1A1, FN1, GAPDH, SFTPC, AHNAK, KLK3, UMOD, CTSB, COL1A1, GPX3, GNAS, ATP1A1, SFTPB, ACTB)

## How to apply

Load the labeled preprocessed TCGA gene-expression matrix from the TCGA_DATAS folder using Python 3.4 with pandas. Train a softmax regression (single-layer neural network) classifier via Keras 2.1.2 backed by TensorFlow GPU 1.4.0. Compute the class-average expression vector for each class (e.g., cancer and normal), then for every pair of classes take the inner product of the softmax parameter vectors with the class-average vectors to derive a per-gene DI score. Sort genes in descending DI score. To stabilize against random initialization, iterate the Wx algorithm 10,000 times and rank genes by the average DI score across iterations; approximately the top 100 genes captured the highest average classification accuracy with no further gain when more genes were added.

## Related tools

- **Wx** (The neural network-based feature selection algorithm itself; produces the DI-score-ranked gene list.) — https://github.com/deargen/DearWXpub
- **Python 3.4** (Runtime environment for running the Wx implementation and data handling with pandas.)
- **Keras 2.1.2** (Neural network framework used to train the softmax regression (single-layer NN) classifier whose weights feed the DI computation.)
- **TensorFlow GPU 1.4.0** (Backend for Keras, enabling GPU-accelerated training over the 10,000 Wx iterations.)
- **TCGA-Assembler** (Module_A.R tool used to download the TCGA RNA-seq data that form the input expression matrix.)

## Examples

```
$ git clone https://github.com/deargen/DearWXpub.git && cd DearWXpub && python src/wx_tcga.py
```

## Evaluation signals

- di_ranked_genes.csv exists and is readable after running src/wx_tcga.py.
- The file contains a DI score for every gene in the input matrix (no genes dropped).
- The gene list is sorted in strictly descending DI-score order.
- Classification accuracy of the top-ranked genes should peak around the top ~100 genes, with no further increase in average training accuracy when more genes are added.
- Top-selected genes (averaged over the 10,000 iterations) should reproduce the Keras Wx 14 set (e.g., EEF1A1, FN1, GAPDH, SFTPC, ...) when run on the TCGA cohort with the same split (half feature selection / half validation, ~96.72% total accuracy).

## Limitations

- Selected features vary depending on the algorithm and backend learning framework used; the README notes 'Selected Features can be different as which backend learning framework used' (naive TensorFlow in the paper vs. Keras in the repo).
- The Wx algorithm's output varies with random initial values, which is why 10,000 iterations are required; single runs are unstable.
- Neural network-based approaches require a large number of samples for good classification performance.
- The 14-gene signature (Wx-14-UGCB) lacks extensive experimental validation; further experimental validation is needed to prove the algorithm's usefulness.

## Evidence

- [abstract] The Wx algorithm ranks genes based on the discriminative index (DI) score that represents the classification power for distinguishing given groups.: "The Wx algorithm ranks genes based on the discriminative index (DI) score that represents the classification power for distinguishing given groups."
- [results] we avoided this irregularity by iterating the Wx algorithm 10,000 times and the highest genes (features) ranked by the average value of the DI score were selected as UGCBs: "we avoided this irregularity by iterating the Wx algorithm 10,000 times and the highest genes (features) ranked by the average value of the DI score were selected as UGCBs"
- [methods] $ python 3.4: "Approximately the top 100 genes showed the highest average accuracy and no further increase in average accuracy was observed when more genes were added"
- [readme] Selected Features can be different as which backend learning framework used: "Selected Features can be different as which backend learning framework used"
