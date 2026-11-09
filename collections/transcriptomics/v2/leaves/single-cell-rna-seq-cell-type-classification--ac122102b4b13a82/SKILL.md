---
name: single-cell-rna-seq-cell-type-classification
description: Use when cell-type labels must be transferred from a labeled scRNA-seq
  training set to an independent test set — especially cross-dataset (e.g. Baron,
  Segerstolpe, Muraro, Xin, Wang pancreas data) — or when benchmarking a classifier
  against ML baselines.
when_to_use_negative:
- The input is already a low-dimensional embedding or feature table — the non-negative
  kernel autoencoder and gene-filtering step assume raw gene-by-cell count data
- Only a single dataset with train-test splits within the same batch is needed and
  the question is clustering rather than label transfer — use scDHA's unsupervised
  clustering mode (scDHA) instead of scDHA.class
- Trajectory or pseudo-time ordering of cells is the goal — use scDHA.pt / Monocle
  rather than the classification module
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDHA
  role: 'Primary classifier: scDHA embeds cells into a low-dimensional representation
    via the hierarchical autoencoder, and scDHA.class trains on the labeled embedding
    to predict test cell types'
  repo: github.com/duct317/scDHA
- name: XGBoost
  role: Baseline classifier trained on the same 20 train-test pairs for accuracy comparison
- name: Random Forest (RF)
  role: Baseline classifier (average accuracy 0.69) in the cross-dataset benchmark
- name: Gradient Boosting Machine (GBM)
  role: Baseline classifier (average accuracy 0.72) in the cross-dataset benchmark
- name: H2O
  role: Framework used to run the XGBoost, Random Forest, Deep Learning, and GBM baseline
    classifiers
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-21312-2
    title: Fast and precise single-cell data analysis using a hierarchical autoencoder
  source_content_hash: sha256:7fac4f1a166c27ae926133f8e34b74db68525c80c0f39eff3c028e18b57e2e4e
schema_version: 0.2.0
metadata:
  license_tier: open
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
yes_rate: 0.8461538461538461
n_claims: 13
---

# single-cell-rna-seq-cell-type-classification

## Summary

Classify cell types in single-cell RNA-seq data by projecting query cells into the scDHA low-dimensional representation (non-negative kernel autoencoder + stacked Bayesian self-learning network) and training scDHA.class on labeled cells; validated cross-dataset on five human pancreas datasets against XGBoost, Random Forest, Deep Learning, and GBM baselines.

## When to use

Use when cell-type labels must be transferred from a labeled scRNA-seq training set to an independent test set — especially cross-dataset (e.g. Baron, Segerstolpe, Muraro, Xin, Wang pancreas data) — or when benchmarking a classifier against ML baselines.

## When NOT to use

- The input is already a low-dimensional embedding or feature table — the non-negative kernel autoencoder and gene-filtering step assume raw gene-by-cell count data
- Only a single dataset with train-test splits within the same batch is needed and the question is clustering rather than label transfer — use scDHA's unsupervised clustering mode (scDHA) instead of scDHA.class
- Trajectory or pseudo-time ordering of cells is the goal — use scDHA.pt / Monocle rather than the classification module

## Inputs

- Labeled scRNA-seq count matrices with cell-type annotations (rows as cells, columns as genes), e.g. the Baron, Segerstolpe, Muraro, Xin, and Wang human pancreas datasets from Table 1 GSE accessions
- Train data matrix, train data labels, and unlabeled new/test data matrix for scDHA.class
- Preprocessed data: gene-filtered, library-size normalized, log2(x+1) transformed expression matrix

## Outputs

- Predicted cell-type labels for the test set cells
- Per-pair classification accuracy for scDHA and the XGBoost, RF, DL, GBM baselines across the 20 train-test combinations
- Accuracy table written to classification_accuracy_table.csv

## How to apply

Download the labeled scRNA-seq datasets and preprocess them (gene filtering, library-size normalization, log2 transformation). For each train-test pair, project both sets through the scDHA hierarchical autoencoder — the non-negative kernel autoencoder removes genes or components with insignificant contributions based on encoder weight distribution, and the stacked Bayesian self-learning network compresses data into a low-dimensional, perturbation-refined representation. Train scDHA.class (inputs: train data matrix, train data label, new data matrix) on the training embedding and predict test labels; set a seed for reproducibility. Benchmark against H2O-based XGBoost, Random Forest, Deep Learning, and Gradient Boosting Machine trained on the same 20 cross-dataset pairs, compute per-pair classification accuracy for all five methods, and test that scDHA's average accuracy (0.96 vs 0.77 XGB, 0.69 RF, 0.43 DL, 0.72 GBM) exceeds the best competitor with a paired significance test (one-sided Wilcoxon, p = 2.1e-8).

## Related tools

- **scDHA** (Primary classifier: scDHA embeds cells into a low-dimensional representation via the hierarchical autoencoder, and scDHA.class trains on the labeled embedding to predict test cell types) — github.com/duct317/scDHA
- **XGBoost** (Baseline classifier trained on the same 20 train-test pairs for accuracy comparison)
- **Random Forest (RF)** (Baseline classifier (average accuracy 0.69) in the cross-dataset benchmark)
- **Gradient Boosting Machine (GBM)** (Baseline classifier (average accuracy 0.72) in the cross-dataset benchmark)
- **H2O** (Framework used to run the XGBoost, Random Forest, Deep Learning, and GBM baseline classifiers)

## Examples

```
library(scDHA); result <- scDHA.class(train = train_data, train.label = train_label, new = test_data)
```

## Evaluation signals

- Per-pair accuracy table over all 20 train-test combinations is produced and scDHA's average accuracy is ~0.96, exceeding the best baseline (XGB at 0.77)
- The scDHA-vs-best-competitor difference is significant under a paired one-sided Wilcoxon test (p = 2.1e-8)
- Results are reproducible with an explicit seed (README: 'The result is reproducible by setting seed for these functions')
- All five methods are trained and tested on identical train-test pairs so accuracy comparisons are paired, not independent

## Limitations

- Cross-dataset batch effects depress baseline performance (H2O Deep Learning averaged only 0.43 accuracy), so absolute cross-dataset accuracy values are dataset-pair dependent
- Comparative performance claims against scVI are deferred to Supplementary Section 6 and Fig. 25–32 rather than shown in the main text
- Some accuracy claims in the discussion are stated as expectations rather than demonstrated results
- The package requires libtorch (torch::install_torch()) since the framework moved from Tensorflow to Torch; Python is no longer required but Torch must be installed

## Evidence

- [readme] scDHA.class classifies new cells using labeled training data via the hierarchical autoencoder embedding: [evidence span withheld — anchor status: external_doc]
- [abstract] Paper's abstract motivating the classification task: "we demonstrate that scDHA outperforms state-of-the-art techniques in many research sub-fields of scRNA-seq analysis, including cell segregation through unsupervised learning, visualization of"
- [intro] Dominant classification baselines named in the intro: "Dominant classification methods include XGBoost19, Random Forest (RF)20, Deep Learning (DL)21, and Gradient Boosting Machine (GBM)22"
- [intro] Filtering step inside the autoencoder that precedes classification: "Based on the weight distribution of the encoder, scDHA removes genes or components that have insignificant contributions to the representation"
- [readme] Reproducibility mechanism from the README: [evidence span withheld — anchor status: external_doc]
