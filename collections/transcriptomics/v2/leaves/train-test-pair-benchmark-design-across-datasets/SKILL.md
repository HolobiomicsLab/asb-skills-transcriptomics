---
name: train-test-pair-benchmark-design-across-datasets
description: 'Use when evaluating whether a single-cell classifier generalizes across
  datasets rather than within one batch: multiple labeled scRNA-seq datasets of the
  same tissue exist, and each dataset should serve once as training and test set to
  form all pairwise combinations.'
when_to_use_negative:
- The analysis goal is pseudo-time inference or unsupervised clustering rather than
  supervised cell-type classification
- Only one labeled dataset is available, so no cross-dataset holdout pair can be constructed
- A within-dataset cross-validation estimate of accuracy is required instead of cross-dataset
  generalization
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDHA
  role: Provides the scDHA.class function that classifies test-set cells using a training
    embedding from the hierarchical autoencoder
  repo: github.com/duct317/scDHA
- name: H2O
  role: Backend framework used to train the competing XGBoost, Random Forest, Deep
    Learning, and GBM classifiers on the same 20 pairs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:4cee388b8ab0779918b7f817d6f7fb4440ecbda7fc5e4a140c23a63bcd9c3f16
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Gradient Boosting Machine (GBM)
  - H2O
  - Random Forest (RF)
  - scDHA
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# train-test-pair-benchmark-design-across-datasets

## Summary

Design a cross-dataset cell-classification benchmark in which each of five human pancreatic scRNA-seq datasets (Baron, Segerstolpe, Muraro, Xin, Wang) is held out in turn as the test set, yielding 20 train-test pairs, and compare scDHA.class against XGBoost, Random Forest, Deep Learning, and GBM classifiers on per-pair accuracy.

## When to use

Use when evaluating whether a single-cell classifier generalizes across datasets rather than within one batch: multiple labeled scRNA-seq datasets of the same tissue exist, and each dataset should serve once as training and test set to form all pairwise combinations.

## When NOT to use

- The analysis goal is pseudo-time inference or unsupervised clustering rather than supervised cell-type classification
- Only one labeled dataset is available, so no cross-dataset holdout pair can be constructed
- A within-dataset cross-validation estimate of accuracy is required instead of cross-dataset generalization

## Inputs

- Five labeled human pancreatic scRNA-seq datasets (Baron, Segerstolpe, Muraro, Xin, Wang) with GSE accessions
- Raw count matrices preprocessed by gene filtering, library-size normalization, and log2 transformation
- Cell-type label vectors for each dataset

## Outputs

- classification_accuracy_table.csv with per-pair accuracy for scDHA, XGBoost, RF, DL, and GBM across all 20 train-test pairs
- Average accuracy per method (scDHA 0.96, XGB 0.77, RF 0.69, DL 0.43, GBM 0.72)
- One-sided Wilcoxon significance test result comparing scDHA to the best competitor

## How to apply

Download the five pancreas scRNA-seq datasets via their GSE accessions and preprocess each with gene filtering, library-size normalization, and log transformation. Construct 20 train-test pairs by taking each dataset in turn as the training set and every dataset as a test set. For each pair, project cells into the scDHA low-dimensional representation (non-negative kernel autoencoder followed by the stacked Bayesian self-learning network) and train scDHA.class on the training embedding to predict test cell-type labels. On the same 20 pairs, train the H2O-based XGBoost, Random Forest, Deep Learning, and Gradient Boosting Machine classifiers and predict test labels. Compute per-pair classification accuracy for all five methods, assemble the accuracy table, and test the average scDHA accuracy against the best competitor with a one-sided paired Wilcoxon test.

## Related tools

- **scDHA** (Provides the scDHA.class function that classifies test-set cells using a training embedding from the hierarchical autoencoder) — github.com/duct317/scDHA
- **H2O** (Backend framework used to train the competing XGBoost, Random Forest, Deep Learning, and GBM classifiers on the same 20 pairs)

## Examples

```
install.packages('scDHA'); library(scDHA); pred <- scDHA.class(train = train_data, trainLabel = train_label, new = test_data)
```

## Evaluation signals

- The accuracy table contains exactly 20 train-test pairs (each of 5 datasets used once as training against each of 5 test sets)
- Average scDHA accuracy across pairs is approximately 0.96, exceeding XGB (0.77), GBM (0.72), RF (0.69), and DL (0.43)
- One-sided paired Wilcoxon test comparing scDHA to the best competitor yields p ≈ 2.1 × 10−8
- classification_accuracy_table.csv is written with per-pair accuracy values for all five methods
- All methods receive identical train-test splits and identically preprocessed input matrices, so accuracy differences reflect the classifier, not data handling

## Limitations

- The comparison is restricted to the five human pancreatic scRNA-seq datasets; generalization to other tissues is not demonstrated by this benchmark
- Deep Learning (H2O) performs very poorly (average accuracy 0.43), so the competitor field may not represent the strongest deep-learning baseline
- The scVI comparison is deferred to Supplementary Section 6 and Fig. 25–32 rather than the main benchmark, and no explicit limitations or caveats section is acknowledged in the discussion

## Evidence

- [intro] Dominant classification methods include XGBoost, Random Forest (RF), Deep Learning (DL), and Gradient Boosting Machine (GBM): "Dominant classification methods include XGBoost19, Random Forest (RF)20, Deep Learning (DL)21, and Gradient Boosting Machine (GBM)22"
- [results] scDHA achieves the highest classification accuracy across all 20 train-test pairs with average accuracy 0.96 versus XGB 0.77, RF 0.69, DL 0.43: [evidence span withheld — anchor status: not_found]
- [readme] scDHA.class takes a train data matrix, train data label and new data matrix to classify new data: [evidence span withheld — anchor status: external_doc]
- [readme] Data are log-transformed (log2(data + 1)) before scDHA processing, as shown in the package example: "Log transform the data: `data <- log2(data + 1)`"
