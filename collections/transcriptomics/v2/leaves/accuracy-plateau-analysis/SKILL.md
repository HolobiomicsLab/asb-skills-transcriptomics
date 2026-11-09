---
name: accuracy-plateau-analysis
description: Use when you have a ranked gene/feature list and need to decide how many
  top-ranked features maximize classifier accuracy before additional genes add no
  benefit.
when_to_use_negative:
- Input is an unranked gene set with no DI (or equivalent) scores — ranking must be
  performed first, e.g. via Wx
- The goal is comparing the classification performance of fixed biomarker panels (e.g.
  Wx-14-UGCB vs Peng-14-UGCB via LOOCV) rather than determining optimal panel size
- Only independent validation data (e.g. GEO sets like GSE72056) is available and
  no training set exists to sweep accuracy on
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Wx
  role: Produces the DI-score-ranked gene list whose top-k subsets are swept
  repo: https://github.com/deargen/DearWXpub
- name: Keras
  role: Neural network framework used to train the classifier for each top-k feature
    subset
  repo: https://github.com/deargen/DearWXpub
- name: TensorFlow GPU
  role: Backend (tensorflow-gpu 1.4.0) on which the Keras classifiers run
  repo: https://github.com/deargen/DearWXpub
- name: Python
  role: Scripting environment (python 3.4) driving the sweep via src/wx_tcga.py
  repo: https://github.com/deargen/DearWXpub
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-019-47016-8
    title: 'Wx: a neural network-based feature selection algorithm for transcriptomic
      data'
  source_content_hash: sha256:f62eb919c7b813d25b6c44ae642050746f90577ad8e1d4080078e33e8db391a2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deargen/DearWXpub
  tools:
  - Keras
  - Python
  - TensorFlow GPU
  - Wx
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# accuracy-plateau-analysis

## Summary

Sweeps the number of top-ranked features (genes) included in a classifier from k=1 to 1,000 to locate the gene count at which average cancer vs normal classification accuracy plateaus, identifying a compact, non-redundant feature subset size.

## When to use

Use when you have a ranked gene/feature list and need to decide how many top-ranked features maximize classifier accuracy before additional genes add no benefit.

## When NOT to use

- Input is an unranked gene set with no DI (or equivalent) scores — ranking must be performed first, e.g. via Wx
- The goal is comparing the classification performance of fixed biomarker panels (e.g. Wx-14-UGCB vs Peng-14-UGCB via LOOCV) rather than determining optimal panel size
- Only independent validation data (e.g. GEO sets like GSE72056) is available and no training set exists to sweep accuracy on

## Inputs

- Preprocessed TCGA pan-cancer gene-expression dataset (12 cancer types plus normal controls)
- Gene list ranked by average Wx discriminative index (DI) score from 10,000 Wx iterations
- Randomly divided training set of cancer and normal samples

## Outputs

- Gene-count vs average classification accuracy table for k = 1 to 1,000
- Identified plateau point (approximately top 100 genes) defining the optimal feature subset size

## How to apply

Load the preprocessed TCGA pan-cancer gene-expression dataset and the gene list ranked by average Wx discriminative index (DI) score (obtained by iterating Wx 10,000 times). For each k from 1 to 1,000, construct a feature subset containing only the top-k DI-ranked genes, train a Keras neural network (TensorFlow GPU) classifier on the randomly divided training set, and record average cancer vs normal accuracy into a gene-count vs accuracy table. Inspect the curve for a plateau: in the paper, approximately the top 100 genes showed the highest average accuracy with no further increase when more genes were added. Use this plateau point as the cutoff for the final biomarker candidate set rather than arbitrarily fixing gene count.

## Related tools

- **Wx** (Produces the DI-score-ranked gene list whose top-k subsets are swept) — https://github.com/deargen/DearWXpub
- **Keras** (Neural network framework used to train the classifier for each top-k feature subset) — https://github.com/deargen/DearWXpub
- **TensorFlow GPU** (Backend (tensorflow-gpu 1.4.0) on which the Keras classifiers run) — https://github.com/deargen/DearWXpub
- **Python** (Scripting environment (python 3.4) driving the sweep via src/wx_tcga.py) — https://github.com/deargen/DearWXpub

## Examples

```
python src/wx_tcga.py
```

## Evaluation signals

- The gene-count vs accuracy table exists for all k values from 1 to 1,000
- Maximum average accuracy is reached near k≈100, matching the reported plateau
- No further increase in average accuracy is observed for k beyond the plateau point
- Accuracy at the plateau is computed on the training set with the same Keras/TensorFlow classifier settings across all k

## Limitations

- Plateau position is specific to the training set and NN algorithm; selected features and optimal counts vary depending on the algorithm and backend framework used (paper used naive TensorFlow, repo uses Keras)
- Neural network-based approaches require a large number of samples for good classification performance
- The plateau identifies subset size but not experimental validation of the genes' biomarker usefulness

## Evidence

- [results] Plateau point definition: "Approximately the top 100 genes showed the highest average accuracy and no further increase in average accuracy was observed when more genes were added"
- [results] Sweep design: "each set containing the top genes (1 to 1,000) was constructed to evaluate the average accuracy of cancer and normal sample classifications in the training set"
- [results] Ranking provenance: "iterating the Wx algorithm 10,000 times and the highest genes (features) ranked by the average value of the DI score were selected as UGCBs"
- [readme] Environment: "$ python 3.4
$ tensorflow gpu 1.4.0
$ keras 2.1.2"
