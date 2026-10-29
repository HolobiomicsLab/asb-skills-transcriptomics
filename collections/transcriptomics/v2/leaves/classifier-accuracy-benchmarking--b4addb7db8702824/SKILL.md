---
name: classifier-accuracy-benchmarking
description: Use when the goal is to compare cell-classification accuracy of a learned
  scRNA-seq representation (e.g., scDHA embedding) versus standard ML classifiers
  across multiple train-test dataset combinations.
when_to_use_negative:
- The task is clustering, visualization, or pseudo-time inference rather than supervised
  cell classification — use scDHA, scDHA.vis, or scDHA.pt instead
- Only a single dataset is available with no cross-dataset generalization requirement,
  so the 20 train-test pairs cannot be constructed and cross-dataset benchmarking
  is not the question at hand
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDHA
  role: Generates the low-dimensional embedding via non-negative kernel autoencoder
    and stacked Bayesian self-learning network, and classifies new data with scDHA.class
  repo: https://github.com/duct317/scDHA
- name: XGBoost
  role: Baseline dominant classifier trained on the same 20 train-test pairs for accuracy
    comparison
- name: H2O
  role: Platform hosting the Random Forest, Deep Learning, and Gradient Boosting Machine
    baseline classifiers
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:a11c4133997f66b60b00216e4b62fe9d8992113a6af0f3e3fbc28e90ac860b12
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duct317/scDHA
  tools:
  - Gradient Boosting Machine (GBM)
  - H2O
  - Random Forest (RF)
  - XGBoost
  - scDHA
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# classifier-accuracy-benchmarking

## Summary

Benchmarks cell-type classification accuracy across scRNA-seq datasets by comparing scDHA's low-dimensional embedding classifier (scDHA.class) against dominant machine learning classifiers — XGBoost, Random Forest, Deep Learning, and Gradient Boosting Machine — over all train-test dataset pairs.

## When to use

Use when the goal is to compare cell-classification accuracy of a learned scRNA-seq representation (e.g., scDHA embedding) versus standard ML classifiers across multiple train-test dataset combinations.

## When NOT to use

- The task is clustering, visualization, or pseudo-time inference rather than supervised cell classification — use scDHA, scDHA.vis, or scDHA.pt instead
- Only a single dataset is available with no cross-dataset generalization requirement, so the 20 train-test pairs cannot be constructed and cross-dataset benchmarking is not the question at hand

## Inputs

- Five human pancreatic scRNA-seq count matrices (Baron, Segerstolpe, Muraro, Xin, Wang) with GSE accessions from Table 1
- Cell-type label vectors for each dataset
- 20 cross-dataset train-test pair definitions (each dataset as training set, each as test set)

## Outputs

- classification_accuracy_table.csv with per-pair classification accuracy for all five methods
- Average accuracies per method (scDHA 0.96 vs XGB 0.77, RF 0.69, DL 0.43, GBM 0.72)
- Statistical significance result from the paired comparison (one-sided Wilcoxon test p-value)

## How to apply

Download the five human pancreatic scRNA-seq datasets (Baron, Segerstolpe, Muraro, Xin, Wang) via their GSE accessions and preprocess them with gene filtering, library-size normalization, and log transformation (log2(data + 1)). Construct all 20 train-test pairs by taking each dataset in turn as the training set and each dataset as the test set. For each pair, project cells into the low-dimensional representation with the scDHA hierarchical autoencoder (non-negative kernel autoencoder followed by the stacked Bayesian self-learning network), then train scDHA.class on the training embedding and predict test cell-type labels. Train the four H2O-based classifiers (XGBoost, Random Forest, Deep Learning, GBM) on the same 20 pairs for a fair comparison. Compute per-pair classification accuracy for all five methods, assemble the accuracy table, and test whether the proposed method's average accuracy exceeds the best competitor (e.g., paired one-sided Wilcoxon test, which gave p = 2.1 × 10−8 with scDHA averaging 0.96 vs 0.77 for XGB).

## Related tools

- **scDHA** (Generates the low-dimensional embedding via non-negative kernel autoencoder and stacked Bayesian self-learning network, and classifies new data with scDHA.class) — https://github.com/duct317/scDHA
- **XGBoost** (Baseline dominant classifier trained on the same 20 train-test pairs for accuracy comparison)
- **H2O** (Platform hosting the Random Forest, Deep Learning, and Gradient Boosting Machine baseline classifiers)

## Examples

```
library(scDHA); data <- log2(data + 1); emb <- scDHA(train, seed = 1); pred <- scDHA.class(train, train.label, test)
```

## Evaluation signals

- classification_accuracy_table.csv exists and contains accuracy entries for all 20 train-test pairs × 5 methods
- Per-pair accuracies are within [0, 1] and scDHA's average accuracy is the highest among the five methods
- The paired significance test (one-sided Wilcoxon) is reported with its p-value and the direction of the effect
- Baseline classifiers (XGBoost, RF, DL, GBM) were trained on the identical 20 train-test splits, ensuring a fair paired comparison
- Results are reproducible when the scDHA functions are run with a fixed seed (README: 'The result is reproducible by setting seed')

## Limitations

- The benchmark is grounded in five human pancreas datasets; generality to other domains is claimed but not demonstrated by cited experiments in the discussion
- The comparative claim against scVI is deferred to supplementary materials (Supplementary Section 6 and Fig. 25–32) rather than shown in the main discussion
- Accuracy claims in the discussion are partly stated as expectations rather than demonstrated results

## Evidence

- [intro] Dominant classification methods include XGBoost, Random Forest (RF), Deep Learning (DL), and Gradient Boosting Machine (GBM): "Dominant classification methods include XGBoost19, Random Forest (RF)20, Deep Learning (DL)21, and Gradient Boosting Machine (GBM)22"
- [readme] scDHA.class: classification new data using available one. The inputs consist of train data matrix, train data label and new data matrix.: [evidence span withheld — anchor status: external_doc]
- [discussion] We demonstrate that scDHA outperforms state-of-the-art techniques in each research sub-field.: "We demonstrate that scDHA outperforms state-of-the-art techniques in each research sub-field."
- [readme] Log transform the data: data <- log2(data + 1): "Log transform the data: `data <- log2(data + 1)`"
