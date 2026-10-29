---
name: cancer-normal-biomarker-evaluation
description: Use when a predefined gene set must be scored for its ability to distinguish
  tumor from normal samples in TCGA RNA-seq cohorts, and per-cancer-type AUC or classification
  accuracy is the target output.
when_to_use_negative:
- The gene set has not yet been selected — use the Wx DI-score ranking skill first
  to derive the top-14 genes from the 6,226-sample pan-cancer matrix (5,609 tumor
  / 617 normal)
- The goal is differential-expression-based biomarker discovery (edgeR DEG-14-UGCB)
  rather than evaluating a fixed neural-network-selected signature
- Input is single-cell or non-TCGA data without tumor/normal labels matching the Keras-trained
  binary cancer/normal setup (independent GEO sets used a separate 5-fold cross-validation
  protocol)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: Wx
  role: Neural network-based feature selection/evaluation framework; scores the fixed
    14-gene set for cancer/normal classification
  repo: https://github.com/deargen/DearWXpub
- name: Keras
  role: Neural network backend (version 2.1.2) used to train and validate the cancer/normal
    classifier
- name: TensorFlow GPU
  role: Deep-learning backend (version 1.4.0) underlying the Keras classifier
- name: TCGA-Assembler
  role: Module_A.R tool used to download the TCGA RNA-seq data populating TCGA_DATAS
- name: Python
  role: Runtime environment (Python 3.4) executing wx_tcga.py and the evaluation scripts
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1038/s41598-019-47016-8
    title: 'Wx: a neural network-based feature selection algorithm for transcriptomic
      data'
  source_content_hash: sha256:77b82807d196827cb1a05abde698f4b182a03e7f3c824145f971ed1737676ebe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deargen/DearWXpub
  tools:
  - Keras
  - Python
  - R
  - TCGA-Assembler
  - TensorFlow
  - TensorFlow GPU
  - Wx
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# cancer-normal-biomarker-evaluation

## Summary

Evaluate a fixed gene-expression biomarker set (e.g., the Wx-14-UGCB) for cancer-versus-normal classification by training a neural network classifier on TCGA RNA-seq cohorts (BRCA, LUAD, LUSC) and reporting per-cancer AUC/accuracy on held-out samples.

## When to use

Use when a predefined gene set must be scored for its ability to distinguish tumor from normal samples in TCGA RNA-seq cohorts, and per-cancer-type AUC or classification accuracy is the target output.

## When NOT to use

- The gene set has not yet been selected — use the Wx DI-score ranking skill first to derive the top-14 genes from the 6,226-sample pan-cancer matrix (5,609 tumor / 617 normal)
- The goal is differential-expression-based biomarker discovery (edgeR DEG-14-UGCB) rather than evaluating a fixed neural-network-selected signature
- Input is single-cell or non-TCGA data without tumor/normal labels matching the Keras-trained binary cancer/normal setup (independent GEO sets used a separate 5-fold cross-validation protocol)

## Inputs

- Preprocessed TCGA RNA-seq expression matrix (illuminahiseq_rnaseqv2-RSEM_genes_normalized) in the TCGA_DATAS folder
- Tumor/normal sample labels for BRCA (1,214 samples), LUAD (576), and LUSC (553) cohorts
- Fixed 14-gene biomarker set (Wx-14-UGCB) to be evaluated

## Outputs

- auc_results.csv with one AUC row per cancer type (BRCA, LUAD, LUSC)
- Per-cancer-type classification accuracy/hit counts on held-out validation samples
- ROC/AUC curves for each cancer-versus-normal classification

## How to apply

Load the preprocessed TCGA RNA-seq (illuminahiseq_rnaseqv2-RSEM_genes_normalized) matrix from the TCGA_DATAS folder and subset to the target cohorts — BRCA (1,214 samples), LUAD (576), LUSC (553) — keeping tumor/normal labels. Apply the Wx neural-network evaluation (cond_wx14_tcga condition) with the fixed 14-gene set ['EEF1A1','FN1','GAPDH','SFTPC','AHNAK','KLK3','UMOD','CTSB','COL1A1','GPX3','GNAS','ATP1A1','SFTPB','ACTB'], running Keras 2.1.2 on TensorFlow GPU 1.4.0 under Python 3.4. Randomly split each cancer group and its corresponding control group into training and validation sets, train the NN classifier on the 14 genes, and score held-out samples. Compute AUC of cancer-versus-normal classification per cancer type (paper values: BRCA 0.9944, LUAD 0.9943, LUSC 0.9936) and/or hit/accuracy counts (README: BRCA 98.19%, LUAD 97.92%, LUSC 98.19% for Wx-14 on half-split validation data). Write per-type AUC values to auc_results.csv and validate against the paper-reported ranges.

## Related tools

- **Wx** (Neural network-based feature selection/evaluation framework; scores the fixed 14-gene set for cancer/normal classification) — https://github.com/deargen/DearWXpub
- **Keras** (Neural network backend (version 2.1.2) used to train and validate the cancer/normal classifier)
- **TensorFlow GPU** (Deep-learning backend (version 1.4.0) underlying the Keras classifier)
- **TCGA-Assembler** (Module_A.R tool used to download the TCGA RNA-seq data populating TCGA_DATAS)
- **Python** (Runtime environment (Python 3.4) executing wx_tcga.py and the evaluation scripts)

## Examples

```
python src/wx_tcga.py
```

## Evaluation signals

- auc_results.csv exists and contains rows for BRCA, LUAD, and LUSC
- Each reported AUC falls within [0, 1] and matches paper values (BRCA 0.9944, LUAD 0.9943, LUSC 0.9936) within tolerance
- Classification accuracies are consistent with README expectations (e.g., ~98.19% BRCA, 97.92% LUAD, 98.19% LUSC for Wx-14 on the half-split validation)
- The evaluated gene set is exactly the 14-gene Wx-14-UGCB including GAPDH and ACTB as the 14th gene
- Cohort sizes match the documented TCGA table (BRCA 1,214; LUAD 576; LUSC 553 total samples)

## Limitations

- The 14-gene signature (Wx-14-UGCB) lacks extensive experimental validation; the authors state its usefulness should be validated with experimental evidence
- Selected features vary depending on the algorithm and backend framework used — the README notes selected features can differ when Keras replaces the original naive TensorFlow framework, and some TCGA samples were updated so the repo has more samples than the paper
- Neural network-based approaches require a large number of samples for good classification performance, so small cohorts may underperform

## Evidence

- [results] The Wx-14-UGCB set achieved AUC values of 0.9944 for BRCA, 0.9943 for LUAD, and 0.9936 for LUSC, showing excellent cancer-versus-normal classification performance.: [evidence span withheld — anchor status: not_found]
- [results] Training/validation split per cancer and control group: "The samples in each cancer and their corresponding control group were randomly divided into two sets, a training set and validation set"
- [readme] Environment pinning from README: "$ python 3.4
$ tensorflow gpu 1.4.0
$ keras 2.1.2"
- [readme] Keras Wx 14 selected gene list: "['EEF1A1','FN1','GAPDH','SFTPC','AHNAK','KLK3','UMOD','CTSB','COL1A1','GPX3','GNAS','ATP1A1','SFTPB','ACTB']"
- [discussion] Discussion caveat on validation: "the usefulness of the 14 genes (Wx-14-UGCB) for cancer biomarkers should be validated with extensive experimental evidence in the near future"
- [readme] README note on backend differences: "Selected Features can be different as which backend learning framework used"
