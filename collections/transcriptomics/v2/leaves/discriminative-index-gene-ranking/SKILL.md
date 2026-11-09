---
name: discriminative-index-gene-ranking
description: Use when you need to prioritize genes from high-dimensional RNA-seq data
  (e.g. TCGA pan-cancer, 20,501 genes) by classification power rather than differential-expression
  p-values, to build a compact biomarker panel.
when_to_use_negative:
- The analysis goal is differential-expression testing with statistical significance
  (adjusted p-values); edgeR is the appropriate DEG tool here, and Wx is positioned
  as a complement, not a DEG method
- The dataset has few samples — neural network-based selection needs a large number
  of samples for good classification or regression performance
- You need reproducible feature identity across backends/frameworks — selected features
  can differ depending on which NN backend is used (Keras vs naive TensorFlow) and
  which algorithm is applied
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2640
tools:
- name: Wx
  role: Neural network-based feature selection algorithm that computes per-gene DI
    scores; iterated 10,000 times to produce the average-DI ranking
  repo: https://github.com/deargen/DearWXpub
- name: Keras 2.1.2
  role: Neural network framework used to run Wx and to train cancer vs normal classifiers
    on top-k DI-ranked gene subsets
  repo: https://github.com/deargen/DearWXpub
- name: TensorFlow GPU 1.4.0
  role: Backend for Keras used in the Wx computation environment
- name: TCGA-Assembler
  role: Module_A.R tool used to download the TCGA RNA-seq data that feed the DI ranking
- name: Python 3.4
  role: Runtime environment; src/wx_tcga.py preprocesses the TCGA cohort and runs
    feature selection/scoring
  repo: https://github.com/deargen/DearWXpub
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1038/s41598-019-47016-8
    title: 'Wx: a neural network-based feature selection algorithm for transcriptomic
      data'
  source_content_hash: sha256:a2974a1f0c948e271a34aac2f85610c0a7d0285d844b66fd1044164db7f1be73
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
  - TensorFlow GPU
  - TensorFlow GPU 1.4.0
  - Wx
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# discriminative-index-gene-ranking

## Summary

Rank genes from a large transcriptomic cohort by their Wx discriminative index (DI) score, computed by iterating a neural network-based feature selection algorithm many times to suppress run-to-run irregularity. The ranked list determines the top gene subsets (e.g. top 14 UGCBs, top 100 genes) used for cancer vs normal classification.

## When to use

Use when you need to prioritize genes from high-dimensional RNA-seq data (e.g. TCGA pan-cancer, 20,501 genes) by classification power rather than differential-expression p-values, to build a compact biomarker panel.

## When NOT to use

- The analysis goal is differential-expression testing with statistical significance (adjusted p-values); edgeR is the appropriate DEG tool here, and Wx is positioned as a complement, not a DEG method
- The dataset has few samples — neural network-based selection needs a large number of samples for good classification or regression performance
- You need reproducible feature identity across backends/frameworks — selected features can differ depending on which NN backend is used (Keras vs naive TensorFlow) and which algorithm is applied

## Inputs

- Preprocessed TCGA pan-cancer gene-expression matrix (TCGA RNA-seq illuminahiseq_rnaseqv2-RSEM_genes_normalized, 6,226 samples: 5,609 tumor / 617 normal)
- TCGA RNA-seq data downloaded via TCGA-Assembler Module_A.R into the TCGA_DATAS folder
- Cancer/normal sample labels for the 12 cancer types (BLCA, BRCA, COAD, HNSC, KICH, KIRC, KIRP, LIHC, LUAD, LUSC, PRAD, THCA)

## Outputs

- Ranked gene list with average DI scores (e.g. wx_tcga_di_ranked_genes.csv covering 20,501 genes)
- Top-14 universal gene-expression cancer biomarker candidates (Wx-14-UGCB), e.g. ['EEF1A1','FN1','GAPDH','SFTPC','AHNAK','KLK3','UMOD','CTSB','COL1A1','GPX3','GNAS','ATP1A1','SFTPB','ACTB']
- Gene-count vs average classification accuracy table for top-k subsets (k = 1 to 1,000)

## How to apply

Run the Wx neural network-based feature selection algorithm (Python 3.4, TensorFlow GPU 1.4.0, Keras 2.1.2) on a preprocessed expression matrix, e.g. the 6,226-sample TCGA pan-cancer matrix (5,609 tumor / 617 normal) produced by src/wx_tcga.py after downloading data with TCGA-Assembler Module_A.R. Because individual Wx runs are irregular, iterate the algorithm 10,000 times and rank all 20,501 genes by the average DI score across iterations to suppress run-to-run instability. To choose the panel size, construct feature subsets of the top 1 to 1,000 DI-ranked genes, train a Keras neural network classifier on the randomly divided training set, and record gene count vs average cancer/normal classification accuracy; accuracy plateaus near the top 100 genes, and the highest-ranked 14 genes form the Wx-14-UGCB biomarker set. Validate the ranked list against known behavior (housekeeping genes such as GAPDH and ACTB should rank highly) before using it for downstream classification.

## Related tools

- **Wx** (Neural network-based feature selection algorithm that computes per-gene DI scores; iterated 10,000 times to produce the average-DI ranking) — https://github.com/deargen/DearWXpub
- **Keras 2.1.2** (Neural network framework used to run Wx and to train cancer vs normal classifiers on top-k DI-ranked gene subsets) — https://github.com/deargen/DearWXpub
- **TensorFlow GPU 1.4.0** (Backend for Keras used in the Wx computation environment)
- **TCGA-Assembler** (Module_A.R tool used to download the TCGA RNA-seq data that feed the DI ranking)
- **Python 3.4** (Runtime environment; src/wx_tcga.py preprocesses the TCGA cohort and runs feature selection/scoring) — https://github.com/deargen/DearWXpub

## Examples

```
git clone https://github.com/deargen/DearWXpub.git && python src/wx_tcga.py  # preprocesses TCGA (6,226 samples), iterates Wx, writes DI-ranked genes
```

## Evaluation signals

- The 10,000-iteration average-DI ranking of the TCGA cohort must place GAPDH among the top 14 genes and ACTB at rank 14 of 20,501 genes
- The top-14 ranked set must reproduce the reported Wx-14-UGCB: ['EEF1A1','FN1','GAPDH','SFTPC','AHNAK','KLK3','UMOD','CTSB','COL1A1','GPX3','GNAS','ATP1A1','SFTPB','ACTB']
- The gene-count vs accuracy table must show maximum average accuracy near k≈100 with no further increase beyond
- The ranked list must cover all 20,501 genes evaluated from the 6,226-sample pan-cancer matrix (5,609 tumor / 617 normal)
- Comparative sanity check: Peng-14 and edgeR-14 genes should receive markedly lower Wx rankings (e.g. UBE2C rank 9039, ESRRB rank 9630) than Wx-selected genes

## Limitations

- Selected features from the same data set vary depending on algorithms, and none of the Wx UGCBs overlapped with those identified by Peng et al. or Martinez-Ledesma et al., so it is difficult to establish which algorithm is better without experimental verification
- The 14-gene signature (Wx-14-UGCB) lacks extensive experimental validation; further experimental validation will be necessary to prove the Wx algorithm's usefulness
- Neural network-based approaches require a large number of samples to achieve good classification or regression performance
- Selected features can differ depending on which NN backend/framework is used (README reports Keras results differ from the naive TensorFlow implementation in the original preprint), and updated TCGA samples mean more samples than the paper

## Evidence

- [results] Iterated Wx 10,000 times and averaged DI scores: "we avoided this irregularity by iterating the Wx algorithm 10,000 times and the highest genes (features) ranked by the average value of the DI score were selected as UGCBs"
- [results] Top-100 gene accuracy plateau: "Approximately the top 100 genes showed the highest average accuracy and no further increase in average accuracy was observed when more genes were added"
- [results] Top-k subset construction: "each set containing the top genes (1 to 1,000) was constructed to evaluate the average accuracy of cancer and normal sample classifications in the training set"
- [abstract] Wx ranks genes by DI score: "The Wx algorithm ranks genes based on the discriminative index (DI) score that represents the classification power for distinguishing given groups."
- [readme] README: Keras Wx-14 selected genes: "['EEF1A1','FN1','GAPDH','SFTPC','AHNAK','KLK3','UMOD','CTSB','COL1A1','GPX3','GNAS','ATP1A1','SFTPB','ACTB']"
- [readme] README: environment and run command: "$ python 3.4
$ tensorflow gpu 1.4.0
$ keras 2.1.2"
- [discussion] Discussion: features vary by algorithm: "In addition, selected features from the same data set vary depending on algorithms."
