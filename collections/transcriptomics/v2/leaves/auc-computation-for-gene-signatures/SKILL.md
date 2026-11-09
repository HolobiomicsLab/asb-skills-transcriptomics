---
name: auc-computation-for-gene-signatures
description: Use when a fixed gene signature from TCGA RNA-seq data (tumor vs normal
  labels) must be scored for classification performance, and you need a concrete AUC
  value per cancer type as the deliverable.
when_to_use_negative:
- 'You are still selecting genes: use the Wx feature-selection loop (10,000 iterations,
  average DI-score ranking) first, then apply this skill only to the fixed signature.'
- Comparing different UGCB sets by generalization accuracy with LOOCV — that evaluation
  used LOOCV/accuracy, not per-cancer AUC curves.
- Data are independent GEO single-cell or small cohorts (GSE72056, GSE40419, GSE103322)
  where the paper used 5-fold cross validation accuracy instead of the TCGA AUC protocol.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0092
tools:
- name: Wx
  role: Supplies the DI-score-ranked 14-gene signature (Wx-14-UGCB) whose cancer-vs-normal
    classification AUC is computed
  repo: https://github.com/deargen/DearWXpub
- name: Keras
  role: Neural network framework used to train the classifier on the 14 genes before
    scoring the validation set (version 2.1.2)
  repo: https://github.com/deargen/DearWXpub
- name: TensorFlow GPU
  role: Backend for the Keras neural network classifier (version 1.4.0)
- name: TCGA-Assembler
  role: Downloads the TCGA RNA-seq data (Module_A.R) that form the evaluation matrix
- name: Python
  role: Runtime environment (Python 3.4) in which the TCGA AUC evaluation (src/wx_tcga.py)
    is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-019-47016-8
    title: 'Wx: a neural network-based feature selection algorithm for transcriptomic
      data'
  source_content_hash: sha256:ca6c10076b99699f11b9668d329ddf14bd5b6d2a144e316d74bf6b1bdff542de
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# auc-computation-for-gene-signatures

## Summary

Evaluates a fixed gene-expression signature (e.g., the 14-gene Wx-14-UGCB set) as a cancer-vs-normal classifier by training a neural network on TCGA RNA-seq expression and reporting per-cancer-type AUC values (BRCA, LUAD, LUSC).

## When to use

Use when a fixed gene signature from TCGA RNA-seq data (tumor vs normal labels) must be scored for classification performance, and you need a concrete AUC value per cancer type as the deliverable.

## When NOT to use

- You are still selecting genes: use the Wx feature-selection loop (10,000 iterations, average DI-score ranking) first, then apply this skill only to the fixed signature.
- Comparing different UGCB sets by generalization accuracy with LOOCV — that evaluation used LOOCV/accuracy, not per-cancer AUC curves.
- Data are independent GEO single-cell or small cohorts (GSE72056, GSE40419, GSE103322) where the paper used 5-fold cross validation accuracy instead of the TCGA AUC protocol.

## Inputs

- Preprocessed TCGA RNA-seq expression matrix from the TCGA_DATAS folder (BRCA, LUAD, LUSC cohorts with tumor/normal labels)
- Fixed 14-gene Wx-14-UGCB signature gene list
- Sample label table mapping each sample to cancer type and tumor/normal class

## Outputs

- auc_results.csv with one AUC row per cancer type (BRCA, LUAD, LUSC)
- ROC curves for the Wx-14 signature on LUSC, LUAD, and BRCA

## How to apply

Load the preprocessed TCGA RNA-seq expression matrix from the TCGA_DATAS folder (produced via TCGA-Assembler Module_A.R and tcga_download.R) and subset to BRCA (1,214 samples), LUAD (576 samples), and LUSC (553 samples) with tumor/normal labels. Restrict the feature columns to the fixed 14-gene Wx-14-UGCB signature ('EEF1A1','FN1','GAPDH','SFTPC','AHNAK','KLK3','UMOD','CTSB','COL1A1','GPX3','GNAS','ATP1A1','SFTPB','ACTB'). Randomly split each cancer group and its corresponding normal group into training and validation sets, then train the neural network classifier (Keras 2.1.2 on TensorFlow GPU 1.4.0, Python 3.4) using the cond_wx14_tcga condition. Score the held-out samples and compute the AUC of the cancer-versus-normal decision per cancer type; the paper reports AUC = 0.9944 (BRCA), 0.9943 (LUAD), and 0.9936 (LUSC). Rationale: the Wx DI-score ranks genes by classification power, so the AUC directly quantifies how well the top-ranked 14-gene signature distinguishes cancer from normal.

## Related tools

- **Wx** (Supplies the DI-score-ranked 14-gene signature (Wx-14-UGCB) whose cancer-vs-normal classification AUC is computed) — https://github.com/deargen/DearWXpub
- **Keras** (Neural network framework used to train the classifier on the 14 genes before scoring the validation set (version 2.1.2)) — https://github.com/deargen/DearWXpub
- **TensorFlow GPU** (Backend for the Keras neural network classifier (version 1.4.0))
- **TCGA-Assembler** (Downloads the TCGA RNA-seq data (Module_A.R) that form the evaluation matrix)
- **Python** (Runtime environment (Python 3.4) in which the TCGA AUC evaluation (src/wx_tcga.py) is executed)

## Examples

```
python src/wx_tcga.py
```

## Evaluation signals

- auc_results.csv exists and contains exactly one row each for BRCA, LUAD, and LUSC.
- Each reported AUC lies in the valid [0, 1] range.
- Reported AUCs are consistent with the paper's values: 0.9944 (BRCA), 0.9943 (LUAD), 0.9936 (LUSC).
- The classifier was trained on the exact 14 genes of Wx-14-UGCB (EEF1A1, FN1, GAPDH, SFTPC, AHNAK, KLK3, UMOD, CTSB, COL1A1, GPX3, GNAS, ATP1A1, SFTPB, ACTB), not on the full expression matrix.
- Cohort sizes match the TCGA download: BRCA 1,214 samples, LUAD 576 samples, LUSC 553 samples, with tumor/normal labels per the README table.

## Limitations

- Selected features and accuracies can differ when the NN backend changes (README: Keras vs the paper's naive TensorFlow implementation) and TCGA sample counts were updated after the paper.
- Only half of the TCGA data is used for validation in the README protocol (half feature selection, half validation), so AUC estimates carry sampling variability.
- The 14-gene Wx-14-UGCB signature still lacks extensive experimental validation per the article discussion, and large sample numbers are needed for good neural network performance.

## Evidence

- [results] Wx-14-UGCB achieved AUC values of 0.9944 for BRCA, 0.9943 for LUAD, and 0.9936 for LUSC: [evidence span withheld — anchor status: not_found]
- [abstract] The Wx algorithm ranks genes based on the discriminative index (DI) score: "The Wx algorithm ranks genes based on the discriminative index (DI) score that represents the classification power for distinguishing given groups."
- [results] Samples in each cancer and control group randomly divided into training and validation sets: "The samples in each cancer and their corresponding control group were randomly divided into two sets, a training set and validation set"
- [readme] WX 14 AUC curve (LUSC, LUAD, BRCA): "**WX 14 AUC curve (LUSC, LUAD, BRCA)**"
- [readme] Keras Wx 14 selected gene markers list: "`['EEF1A1','FN1','GAPDH','SFTPC','AHNAK','KLK3','UMOD','CTSB','COL1A1','GPX3','GNAS','ATP1A1','SFTPB','ACTB']`"
