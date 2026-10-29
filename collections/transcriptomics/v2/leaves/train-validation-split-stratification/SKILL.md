---
name: train-validation-split-stratification
description: Use when training a neural-network classifier on high-dimensional transcriptomic
  data (e.g., TCGA pan-cancer cancer-vs-normal labels) and you need a defensible train/validation
  split before sweeping the number of ranked features to find where accuracy plateaus.
when_to_use_negative:
- The task is pure DEG analysis (e.g., edgeR adjusted p-value ranking) with no classifier-based
  evaluation, so no train/validation split is needed
- Evaluation is on independent external GEO datasets (GSE72056, GSE40419, GSE103322)
  where the paper already applies 5-fold cross validation instead of a single train/validation
  split
- Sample sizes are too small for a neural network — the paper notes a large number
  of samples are needed to achieve good classification performance
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Wx
  role: Ranks genes by DI score to generate the ordered feature list used in the top-k
    sweep on the training split
  repo: https://github.com/deargen/DearWXpub
- name: Keras
  role: Neural-network framework used to train classifiers on each top-k feature subset
    of the training set
  repo: https://github.com/deargen/DearWXpub
- name: TensorFlow GPU
  role: Backend for the Keras classifier during accuracy evaluation on the training
    split
  repo: https://github.com/deargen/DearWXpub
- name: TCGA-Assembler
  role: Downloads the TCGA RNA-seq data (Module_A.R / tcga_download.R) that is subsequently
    split into training and validation sets
  repo: http://www.compgenome.org/TCGA-Assembler/index.php
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-019-47016-8
    title: 'Wx: a neural network-based feature selection algorithm for transcriptomic
      data'
  source_content_hash: sha256:3b55122fc99f5d89c9a8a530548094f4ad9d63271636faff6f3fa304a437d2dc
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# train-validation-split-stratification

## Summary

Stratified random splitting of cancer/normal TCGA pan-cancer RNA-seq samples into training and validation partitions so that gene-count sweeps of Wx DI-ranked features (top 1–1,000) can be scored by average NN classification accuracy, revealing a plateau at ~top 100 genes.

## When to use

Use when training a neural-network classifier on high-dimensional transcriptomic data (e.g., TCGA pan-cancer cancer-vs-normal labels) and you need a defensible train/validation split before sweeping the number of ranked features to find where accuracy plateaus.

## When NOT to use

- The task is pure DEG analysis (e.g., edgeR adjusted p-value ranking) with no classifier-based evaluation, so no train/validation split is needed
- Evaluation is on independent external GEO datasets (GSE72056, GSE40419, GSE103322) where the paper already applies 5-fold cross validation instead of a single train/validation split
- Sample sizes are too small for a neural network — the paper notes a large number of samples are needed to achieve good classification performance

## Inputs

- Preprocessed TCGA pan-cancer gene-expression matrix with cancer and normal sample labels
- Gene list ranked by average Wx discriminative index (DI) score (from 10,000 Wx iterations)
- Python environment with TensorFlow GPU 1.4.0 and Keras 2.1.2

## Outputs

- Gene-count vs average classification accuracy table for k = 1 to 1,000
- Identified plateau point (approximately top 100 DI-ranked genes) with maximum average accuracy
- Trained neural-network classifier per top-k feature subset on the training set

## How to apply

Randomly divide each cancer group and its corresponding normal control group into a training set and a validation set, keeping class balance per cancer type (in the paper: 2,888 training / 723 validation / 902 test samples, i.e. 64%/16%/20%). Train a Keras-on-TensorFlow-GPU neural network on each training-set feature subset built from the top-k genes ranked by average Wx discriminative index (DI) score, iterating k from 1 to 1,000. Record average cancer-vs-normal accuracy for each k into a gene-count vs accuracy table. The rationale is that accuracy saturates — approximately the top 100 genes showed the highest average accuracy with no further increase — so the split supports selecting a compact feature set rather than all genes. Judge correctness by confirming the accuracy table peaks near k≈100 and that no larger k improves average accuracy.

## Related tools

- **Wx** (Ranks genes by DI score to generate the ordered feature list used in the top-k sweep on the training split) — https://github.com/deargen/DearWXpub
- **Keras** (Neural-network framework used to train classifiers on each top-k feature subset of the training set) — https://github.com/deargen/DearWXpub
- **TensorFlow GPU** (Backend for the Keras classifier during accuracy evaluation on the training split) — https://github.com/deargen/DearWXpub
- **TCGA-Assembler** (Downloads the TCGA RNA-seq data (Module_A.R / tcga_download.R) that is subsequently split into training and validation sets) — http://www.compgenome.org/TCGA-Assembler/index.php

## Examples

```
python src/wx_tcga.py
```

## Evaluation signals

- The gene-count vs accuracy table shows maximum average accuracy near k≈100, matching 'approximately the top 100 genes showed the highest average accuracy'
- No further increase in average accuracy is observed for any k beyond the plateau (~100 genes)
- Split proportions are recorded and sum to the full cohort (paper: 2,888 training / 723 validation / 902 test = 64%/16%/20%)
- The table contains one average accuracy entry for every k from 1 to 1,000, each evaluated on the same randomly divided training set

## Limitations

- Selected features vary depending on the algorithm used, and the README notes selected features can differ when the backend framework changes (paper used naive TensorFlow; README uses Keras)
- The 14-gene signature (Wx-14-UGCB) lacks extensive experimental validation; further experimental validation will be necessary to prove the Wx algorithm's usefulness
- The README used an updated TCGA snapshot (downloaded Dec 26, 2017) with more samples than the paper, so absolute sample counts in the split differ between sources

## Evidence

- [results] Approximately the top 100 genes showed the highest average accuracy and no further increase in average accuracy was observed when more genes were added: "Approximately the top 100 genes showed the highest average accuracy and no further increase in average accuracy was observed when more genes were added"
- [results] training set (2,888 samples, 64%), validation set (723 samples, 16%), and test set (902 samples, 20%): "training set (2,888 samples, 64%), validation set (723 samples, 16%), and test set (902 samples, 20%)"
- [results] each set containing the top genes (1 to 1,000) was constructed to evaluate the average accuracy of cancer and normal sample classifications in the training set: "each set containing the top genes (1 to 1,000) was constructed to evaluate the average accuracy of cancer and normal sample classifications in the training set"
- [readme] Half of data for feature selection / Half of data for validation: "TCGA data( Downloaded at Dec. 26th. 2017 ), Half of data for feature selection / Half of data for validation"
