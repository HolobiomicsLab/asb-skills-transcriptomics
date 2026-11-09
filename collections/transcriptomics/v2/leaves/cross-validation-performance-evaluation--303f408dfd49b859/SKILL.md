---
name: cross-validation-performance-evaluation
description: Use when reproducing or benchmarking a small-sample gene-classification
  model (e.g. 136 TSGs and 76 OGs labelled from CGC) and you need stable, class-wise
  performance metrics under k-fold cross-validation rather than a single train/test
  split.
when_to_use_negative:
- Input is already a model prediction or consensus output (e.g. the classified unlabelled
  genes) rather than the labelled 37-feature matrix — evaluation metrics cannot be
  computed without CGC TSG/OG ground-truth labels
- You are reproducing the Davoli et al. comparison — the paper notes their test set
  results have not been published, so test-set metric comparison is not possible for
  that baseline
- Feature values include Tokheim et al. data — missense and frameshift entropy and
  frequency features could not be calculated there, so the 37-feature protocol is
  not fully applicable
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_2269
tools:
- name: cTaG
  role: Provides the random-forest pan-cancer model, the 37-feature matrix, CGC gene
    labels and CV code (TSG_OG_Feat_final.py, evaluate_randiter.py) that this evaluation
    skill reproduces
  repo: https://github.com/RamanLab/cTaG
- name: Random Forest (scikit-learn)
  role: Classifier trained per CV fold with hyper-parameters n_estimator, max_features,
    max_depth and criterion tuned by fivefold CV plus repeated random iterations
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:bd30c82eaf84d6e9b93f603cbfb910dafaf9d45718f0ddef1df45b376af4f12e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RamanLab/cTaG
  tools:
  - Random Forest (scikit-learn)
  - cTaG
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-validation-performance-evaluation

## Summary

Reproduces the cTaG random-forest fivefold cross-validation protocol to estimate per-class accuracy, precision, recall and F1 for tumour suppressor gene (TSG) and oncogene (OG) classification over the 37-feature pan-cancer mutation feature matrix, using iterated hyper-parameter search to avoid overfitting on the small labelled gene set.

## When to use

Use when reproducing or benchmarking a small-sample gene-classification model (e.g. 136 TSGs and 76 OGs labelled from CGC) and you need stable, class-wise performance metrics under k-fold cross-validation rather than a single train/test split.

## When NOT to use

- Input is already a model prediction or consensus output (e.g. the classified unlabelled genes) rather than the labelled 37-feature matrix — evaluation metrics cannot be computed without CGC TSG/OG ground-truth labels
- You are reproducing the Davoli et al. comparison — the paper notes their test set results have not been published, so test-set metric comparison is not possible for that baseline
- Feature values include Tokheim et al. data — missense and frameshift entropy and frequency features could not be calculated there, so the 37-feature protocol is not fully applicable

## Inputs

- 37-feature pan-cancer mutation feature matrix (FeatureMat.tsv, genes as rows, ratio-metric and entropy mutation-type features as columns)
- CGC-derived gene labels (TSG vs OG vs Unlabelled) from IdentifyTSGOG/data/cgc_genes
- Training gene lists: 136 TSGs and 76 OGs

## Outputs

- Per-class (TSG, OG) accuracy, precision, recall and F1 on training and test partitions for each CV fold
- Average ± standard deviation summary table of per-class metrics across folds (reference: test TSG accuracy 0.83 ± 0.02, test OG accuracy 0.76 ± 0.03)
- Stable hyper-parameter set (n_estimator, max_features, max_depth, criterion) selected by repeated random iterations

## How to apply

Load the pre-processed 37-feature matrix (FeatureMat.tsv from RamanLab/cTaG) and attach CGC-derived TSG/OG labels. For each of five folds, train a scikit-learn Random Forest on a random 80% subset of the 136 TSGs and 76 OGs, holding out the remaining 20% as test. First run fivefold cross-validation over the hyper-parameter grid (n_estimator, max_features, max_depth, criterion), then repeat over multiple random iterations to identify a stable hyper-parameter set — this is the paper's explicit defence against overfitting on the small gene set. Train each fold with the stable hyper-parameters and compute per-class accuracy, precision, recall and F1 on both training and test partitions. Consolidate per-fold metrics into average ± standard deviation per class; the target reference values are test TSG accuracy ~0.83 ± 0.02 and test OG accuracy ~0.76 ± 0.03 (OG precision 0.79 ± 0.12, recall 0.50 ± 0.19).

## Related tools

- **cTaG** (Provides the random-forest pan-cancer model, the 37-feature matrix, CGC gene labels and CV code (TSG_OG_Feat_final.py, evaluate_randiter.py) that this evaluation skill reproduces) — https://github.com/RamanLab/cTaG
- **Random Forest (scikit-learn)** (Classifier trained per CV fold with hyper-parameters n_estimator, max_features, max_depth and criterion tuned by fivefold CV plus repeated random iterations)

## Examples

```
python code/TSG_OG_Feat_final.py  # loads data/FeatureMat/FeatureMat.tsv and builds the per-fold random-forest CV models
```

## Evaluation signals

- Reproduced test accuracy falls within reported standard deviations: ~0.83 ± 0.02 for TSGs and ~0.76 ± 0.03 for OGs
- Per-class OG precision and recall land in the reported wide ranges (precision 0.79 ± 0.12, recall 0.50 ± 0.19), reflecting the small OG class
- Stable hyper-parameter set is reproducible across multiple random iterations (output/evalRandIter confirms the iteration count is sufficient)
- Each fold trains on exactly 80% of the 136 TSGs and 76 OGs with the remaining 20% held out — verify fold splits partition the labelled set without leakage
- Metrics are reported as average ± standard deviation per class over folds, not as a single pooled value

## Limitations

- The labelled gene set is small (136 TSGs, 76 OGs), so per-fold metrics have large variance, especially OG recall (±0.19)
- The model uses binary TSG/OG classification, which the authors list as a limitation of cTaG
- Training is restricted to CGC-labelled genes; genes known as drivers but absent from CGC (e.g. ZNF750) fall outside the evaluation
- Consensus prediction across models is required downstream, since no single fold model is used for final TSG/OG calls

## Evidence

- [results] We performed fivefold cross-validation while estimating hyper-parameters for the model, followed by multiple random iterations to estimate stable hyper-parameters and avoid overfitting: "We performed fivefold cross-validation while estimating hyper-parameters for the model, followed by multiple random iterations to estimate stable hyper-parameters and avoid overfitting"
- [results] 80% subset of 136 TSGs and 76 OGs: "We trained multiple random forest models using a subset (80%) of 136 TSGs and 76 OGs for each fold of the cross-validation."
- [readme] Stable hyper-parameters rationale (README): "We overcome overfitting due to the small set of genes to train by estimating stable hyper-parameter set using multiple random iterations."
- [readme] Metrics reported per class with deviations: "The list of TSGs and OGs used for training can be found [here](https://github.com/RamanLab/IdentifyTSGOG/tree/master/data/cgc_genes)."
