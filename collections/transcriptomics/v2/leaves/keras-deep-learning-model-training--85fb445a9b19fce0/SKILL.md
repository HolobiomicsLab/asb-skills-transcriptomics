---
name: keras-deep-learning-model-training
description: Use when evaluating the cancer-versus-normal classification performance
  of a fixed gene panel (e.g. the 14-gene Wx-14-UGCB) on TCGA RNA-seq cohorts with
  tumor/normal labels such as BRCA (1,214 samples), LUAD (576), and LUSC (553).
when_to_use_negative:
- Input is not RNA-seq expression data with tumor/normal labels (e.g. already-aggregated
  feature tables, methylation, or copy-number data)
- The gene panel has not been fixed/selected yet — this skill evaluates a fixed set;
  gene selection itself is the separate Wx feature-selection step iterating the DI
  score 10,000 times
- Only a handful of samples per class are available; the paper notes a large number
  of samples are needed for good neural-network performance
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Keras
  role: Neural-network framework used to train the cancer/normal classifier on the
    14-gene set
  repo: https://github.com/deargen/DearWXpub
- name: TensorFlow GPU 1.4.0
  role: Backend framework running beneath Keras for model training
  repo: https://github.com/deargen/DearWXpub
- name: Python 3.4
  role: Runtime environment for the classification pipeline (src/wx_tcga.py)
  repo: https://github.com/deargen/DearWXpub
- name: TCGA-Assembler
  role: R tool (Module_A.R) used to download the TCGA RNA-seq data used as classifier
    input
  repo: http://www.compgenome.org/TCGA-Assembler/index.php
- name: R
  role: Runs tcga_download.R and its dependencies (curl, httr, stringr, r) to assemble
    the TCGA_DATAS expression matrix
  repo: https://github.com/deargen/DearWXpub
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-019-47016-8
    title: 'Wx: a neural network-based feature selection algorithm for transcriptomic
      data'
  source_content_hash: sha256:d79264f921f4be09a420eb83d9d441bbb3d7b7ce308c94b83ddf3e6d3b146cfd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deargen/DearWXpub
  tools:
  - Keras
  - Python
  - Python 3.4
  - R
  - TCGA-Assembler
  - TensorFlow
  - TensorFlow GPU 1.4.0
  - Wx
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# keras-deep-learning-model-training

## Summary

Trains a Keras 2.1.2 (TensorFlow GPU 1.4.0, Python 3.4) neural-network classifier on TCGA RNA-seq expression of the Wx-14-UGCB 14-gene biomarker set to classify tumor vs normal samples, reporting per-cancer-type AUC (e.g. BRCA 0.9944, LUAD 0.9943, LUSC 0.9936).

## When to use

Use when evaluating the cancer-versus-normal classification performance of a fixed gene panel (e.g. the 14-gene Wx-14-UGCB) on TCGA RNA-seq cohorts with tumor/normal labels such as BRCA (1,214 samples), LUAD (576), and LUSC (553).

## When NOT to use

- Input is not RNA-seq expression data with tumor/normal labels (e.g. already-aggregated feature tables, methylation, or copy-number data)
- The gene panel has not been fixed/selected yet — this skill evaluates a fixed set; gene selection itself is the separate Wx feature-selection step iterating the DI score 10,000 times
- Only a handful of samples per class are available; the paper notes a large number of samples are needed for good neural-network performance

## Inputs

- Preprocessed TCGA RNA-seq expression matrix (TCGA_DATAS folder, downloaded via TCGA-Assembler)
- Tumor/normal sample labels for cohorts such as BRCA (1,101 tumor / 113 normal), LUAD (517/59), LUSC (502/51)
- Fixed 14-gene Wx-14-UGCB gene list

## Outputs

- Trained Keras neural-network classifier for cancer-vs-normal classification
- Per-cancer-type AUC values (auc_results.csv with rows for BRCA, LUAD, LUSC)
- ROC curves for each cancer type (e.g. roc_wx14_brca, roc_wx14_luad, roc_wx14_lusc)

## How to apply

Load the preprocessed TCGA RNA-seq matrix from the TCGA_DATAS folder (downloaded via TCGA-Assembler Module_A.R and tcga_download.R), subset to the cohorts of interest, and restrict features to the fixed 14 genes (EEF1A1, FN1, GAPDH, SFTPC, AHNAK, KLK3, UMOD, CTSB, COL1A1, GPX3, GNAS, ATP1A1, SFTPB, ACTB). Randomly split each cancer and its corresponding normal group into training and validation sets (the README used half for feature selection and half for validation). Train a Keras 2.1.2 neural network on the 14 genes (backend: TensorFlow GPU 1.4.0, Python 3.4) and score held-out samples to compute the AUC of cancer-vs-normal classification per cancer type. Write the per-type AUC values (e.g. to auc_results.csv) and validate that each is within [0, 1] and consistent with the reported performance (BRCA 0.9944, LUAD 0.9943, LUSC 0.9936).

## Related tools

- **Keras** (Neural-network framework used to train the cancer/normal classifier on the 14-gene set) — https://github.com/deargen/DearWXpub
- **TensorFlow GPU 1.4.0** (Backend framework running beneath Keras for model training) — https://github.com/deargen/DearWXpub
- **Python 3.4** (Runtime environment for the classification pipeline (src/wx_tcga.py)) — https://github.com/deargen/DearWXpub
- **TCGA-Assembler** (R tool (Module_A.R) used to download the TCGA RNA-seq data used as classifier input) — http://www.compgenome.org/TCGA-Assembler/index.php
- **R** (Runs tcga_download.R and its dependencies (curl, httr, stringr, r) to assemble the TCGA_DATAS expression matrix) — https://github.com/deargen/DearWXpub

## Examples

```
$ python src/wx_tcga.py
```

## Evaluation signals

- Output file (auc_results.csv) exists and contains a row for each cancer type evaluated (BRCA, LUAD, LUSC)
- Each reported AUC is within the valid [0, 1] range
- AUC values are consistent with the paper's reported performance: 0.9944 (BRCA), 0.9943 (LUAD), 0.9936 (LUSC)
- Only the 14 Wx-14-UGCB genes (EEF1A1, FN1, GAPDH, SFTPC, AHNAK, KLK3, UMOD, CTSB, COL1A1, GPX3, GNAS, ATP1A1, SFTPB, ACTB) are used as input features
- Held-out/validation samples were never used during training (random split of each cancer and its corresponding control group into training and validation sets)

## Limitations

- Selected features can differ depending on which backend learning framework is used; the README's Keras implementation may select different genes than the paper's native TensorFlow run
- TCGA samples were updated relative to the paper, so the README cohorts contain more samples than the paper used
- Neural-network approaches require a large number of samples to achieve good classification performance
- The 14-gene Wx-14-UGCB signature lacks extensive experimental validation and further experimental validation is needed to prove the algorithm's usefulness

## Evidence

- [results] The Wx-14-UGCB set achieved AUC values of 0.9944 for BRCA, 0.9943 for LUAD, and 0.9936 for LUSC: [evidence span withheld — anchor status: not_found]
- [results] The samples in each cancer and their corresponding control group were randomly divided: "The samples in each cancer and their corresponding control group were randomly divided into two sets, a training set and validation set"
- [results] the training set was used to train a model using a neural network (NN) algorithm: "the training set was used to train a model using a neural network (NN) algorithm and the validation set was used to assess how well the model had been trained"
- [methods] we downloaded tool (`Module_A.R`) via: "we downloaded tool (`Module_A.R`) via  http://www.compgenome.org/TCGA-Assembler/index.php"
- [readme] Selected Features can be different as which backend learning framework used: "Selected Features can be different as which backend learning framework used"
