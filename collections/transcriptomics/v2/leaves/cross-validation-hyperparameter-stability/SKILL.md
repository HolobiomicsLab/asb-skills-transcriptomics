---
name: cross-validation-hyperparameter-stability
description: Use when training a classifier on a small labelled gene set (e.g. 136
  TSGs and 76 OGs from CGC) where overfitting risk is high and hyper-parameters must
  be estimated stably before predictions are trusted.
when_to_use_negative:
- When a large labelled training set is available and simple grid-search CV suffices
  without stability checks
- When the input is raw somatic mutation data that has not yet been converted into
  the 37-feature matrix (use feature-matrix construction first)
- When the goal is driver-gene prediction on unlabelled genes rather than model training
  and hyper-parameter validation
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: cTaG
  role: Provides the random-forest training pipeline (TSG_OG_Feat_final.py) and iteration-evaluation
    script (evaluate_randiter.py) used to estimate stable hyper-parameters across
    fivefold CV and random iterations
  repo: https://github.com/RamanLab/cTaG
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:ada88ddb6207a9303194d906ab3f0f4ba77386bdd24e262d8800bf4ce7b1e508
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RamanLab/cTaG
  tools:
  - cTaG
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cross-validation-hyperparameter-stability

## Summary

A protocol for avoiding overfitting when training random-forest classifiers on small labelled gene sets (136 TSGs, 76 OGs) by combining fivefold cross-validation with multiple random iterations to identify a stable hyper-parameter set, as used to train the cTaG pan-cancer model.

## When to use

Use when training a classifier on a small labelled gene set (e.g. 136 TSGs and 76 OGs from CGC) where overfitting risk is high and hyper-parameters must be estimated stably before predictions are trusted.

## When NOT to use

- When a large labelled training set is available and simple grid-search CV suffices without stability checks
- When the input is raw somatic mutation data that has not yet been converted into the 37-feature matrix (use feature-matrix construction first)
- When the goal is driver-gene prediction on unlabelled genes rather than model training and hyper-parameter validation

## Inputs

- Pre-processed 37-feature matrix (FeatureMat.tsv) with genes as rows and ratio-metric/entropy features as columns
- CGC-derived gene labels: 136 tumour suppressor genes (TSGs) and 76 oncogenes (OGs) used for training

## Outputs

- Stable hyper-parameter set for the random-forest model, validated across multiple random iterations
- Per-fold, per-class training and test accuracies (mean ± standard deviation for TSG and OG)
- Consolidated metrics table (balanced_bagging_metrics.csv) benchmarking against baselines

## How to apply

Load the pre-processed 37-feature matrix (FeatureMat.tsv from the IdentifyTSGOG repository, rows = genes/transcripts, columns = 37 ratio-metric and entropy features) and CGC-derived TSG/OG labels. Train multiple random-forest models using a subset (80%) of 136 TSGs and 76 OGs per fold, performing fivefold cross-validation for hyper-parameter estimation. Follow the CV with multiple random iterations, recording training and test accuracies per fold and per class (TSG and OG separately) to verify the chosen hyper-parameter set is stable rather than fold-specific. Use the cTaG repository's evaluate_randiter.py to check performance as a function of the number of random iterations. Benchmark against naive alternatives (e.g. a balanced bagging classifier with standard grid-search CV, which reaches ~0.94 training but only ~0.73 test TSG accuracy) to demonstrate that the stability procedure genuinely curbs overfitting; write the consolidated metrics to balanced_bagging_metrics.csv.

## Related tools

- **cTaG** (Provides the random-forest training pipeline (TSG_OG_Feat_final.py) and iteration-evaluation script (evaluate_randiter.py) used to estimate stable hyper-parameters across fivefold CV and random iterations) — https://github.com/RamanLab/cTaG

## Examples

```
python code/evaluate_randiter.py  # evaluates performance for different numbers of random iterations to verify hyper-parameter stability
```

## Evaluation signals

- Training and test accuracies diverge only modestly for cTaG (contrast: balanced bagging baseline shows OG train 0.93 ± 0.05 vs test 0.69 ± 0.06, TSG train 0.94 ± 0.04 vs test 0.73 ± 0.06, indicating overfitting)
- Chosen hyper-parameters remain consistent (stable) when the number of random iterations is varied, verified via evaluate_randiter.py outputs
- Each cross-validation fold uses exactly a subset (80%) of the 136 TSGs and 76 OGs for training
- AUROC of the cTaG random-forest model (~0.76 ± 0.07) clearly exceeds naive baselines (e.g. 0.54 ± 0.07 for balanced bagging)
- Consolidated metrics fall within the reported variability and are written to balanced_bagging_metrics.csv

## Limitations

- Only 136 TSGs and 76 OGs are available for training, so overfitting risk is inherently high and stability checking is essential rather than optional
- Test-set performance cannot be compared with Davoli et al. as their test set results have not been published
- Binary classification of genes into TSG or OG only (not multi-class) is a stated limitation of cTaG

## Evidence

- [results] fivefold CV with multiple random iterations: "We performed fivefold cross-validation while estimating hyper-parameters for the model, followed by multiple random iterations to estimate stable hyper-parameters and avoid overfitting"
- [results] 80% training subset per fold: "We trained multiple random forest models using a subset (80%) of 136 TSGs and 76 OGs for each fold of the cross-validation."
- [readme] overfitting rationale from README: "We overcome overfitting due to the small set of genes to train by estimating stable hyper-parameter set using multiple random iterations."
- [other] balanced-bagging overfitting contrast: [evidence span withheld — anchor status: not_found]
- [readme] feature matrix provenance from README: "The mutation data was used to generate feature matrix were the rows and columns correspond to genes/transcripts and 37 features respectively."
