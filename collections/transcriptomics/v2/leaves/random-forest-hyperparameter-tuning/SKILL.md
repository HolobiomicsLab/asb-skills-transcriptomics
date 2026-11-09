---
name: random-forest-hyperparameter-tuning
description: Use when classifying genes as TSGs vs OGs from a pre-processed mutation
  feature matrix (e.g. FeatureMat.tsv with 37 ratio-metric/entropy features) with
  CGC labels, and stable hyper-parameters are needed to avoid overfitting on a small
  labelled set.
when_to_use_negative:
- When a background mutation rate model is desired instead — cTaG is deliberately
  not biased towards genes with high mutation frequency, so it is the wrong tool if
  mutation-rate-based ranking (e.g. MutSigCV) is the goal
- When the goal is a three-way driver/TSG/OG classification on a single cancer type
  — the pan-cancer model uses binary TSG-vs-OG classification on COSMIC pan-cancer
  data, and tissue-specific models require feature matrices from tissues with greater
  than 1000 samples
- When input is a raw COSMIC mutation file — this skill starts from the already-built
  37-feature matrix (FeatureMat.tsv); building features from mutation data is a separate
  step (identifyTSGOG.py)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0622
tools:
- name: cTaG
  role: Pan-cancer random-forest model classifying genes as TSGs or OGs; provides
    the training/evaluation code (TSG_OG_Feat_final.py, evaluate_randiter.py) for
    cross-validated hyper-parameter estimation
  repo: https://github.com/RamanLab/cTaG
- name: Random Forest
  role: scikit-learn classifier trained per cross-validation fold; hyper-parameters
    (n_estimator, max_features, max_depth, criterion) tuned via fivefold CV and multiple
    random iterations
  repo: https://github.com/RamanLab/cTaG
- name: IdentifyTSGOG
  role: Hosts the pre-processed 37-feature matrix (FeatureMat.tsv) and the CGC TSG/OG
    gene lists used as training inputs
  repo: https://github.com/RamanLab/IdentifyTSGOG/tree/master/data/cgc_genes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:4eb9c7711492cc51b7cf820d08284782d4de14c142d426290e8823baeca07e20
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RamanLab/cTaG
  tools:
  - IdentifyTSGOG
  - Random Forest
  - cTaG
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# random-forest-hyperparameter-tuning

## Summary

Reproduces the cTaG random-forest classification of tumour suppressor genes (TSGs) and oncogenes (OGs) by performing fivefold cross-validation with iterative hyper-parameter estimation on a 37-feature pan-cancer mutation feature matrix, yielding stable per-class accuracy, precision, recall and F1 metrics despite a small training set of 136 TSGs and 76-77 OGs.

## When to use

Use when classifying genes as TSGs vs OGs from a pre-processed mutation feature matrix (e.g. FeatureMat.tsv with 37 ratio-metric/entropy features) with CGC labels, and stable hyper-parameters are needed to avoid overfitting on a small labelled set.

## When NOT to use

- When a background mutation rate model is desired instead — cTaG is deliberately not biased towards genes with high mutation frequency, so it is the wrong tool if mutation-rate-based ranking (e.g. MutSigCV) is the goal
- When the goal is a three-way driver/TSG/OG classification on a single cancer type — the pan-cancer model uses binary TSG-vs-OG classification on COSMIC pan-cancer data, and tissue-specific models require feature matrices from tissues with greater than 1000 samples
- When input is a raw COSMIC mutation file — this skill starts from the already-built 37-feature matrix (FeatureMat.tsv); building features from mutation data is a separate step (identifyTSGOG.py)

## Inputs

- Pre-processed 37-feature pan-cancer feature matrix (FeatureMat.tsv, rows = genes/transcripts, columns = features)
- CGC-derived TSG/OG gene labels (136 TSGs, 76-77 OGs) from the cgc_genes list
- Somatic coding mutation data from COSMIC (v79) filtered to samples with fewer than 2000 mutations

## Outputs

- Stable random-forest hyper-parameter set (n_estimator, max_features, max_depth, criterion)
- Per-fold training and test metrics (accuracy, precision, recall, F1) for TSG and OG classes
- Consolidated average ± standard deviation metric summary table per class (test TSG accuracy ~0.83 ± 0.02; test OG accuracy ~0.76 ± 0.03)
- Trained per-fold random-forest models for consensus prediction of new TSGs and OGs

## How to apply

Load the pre-processed 37-feature matrix (FeatureMat.tsv from the RamanLab/IdentifyTSGOG data folder) and attach Cancer Gene Census (CGC)-derived TSG/OG labels to the 136 TSGs and 76-77 OGs used for training. Split the labelled set so each fold trains on a random 80% subset, holding out 20% as test, and use scikit-learn Random Forest with fivefold cross-validation to estimate hyper-parameters (n_estimator, max_features, max_depth, criterion). Because the gene set is small, repeat the cross-validation over multiple random iterations to identify a stable hyper-parameter set, which the authors use explicitly to overcome overfitting. Train per-fold models with the stable hyper-parameters, compute per-class accuracy, precision, recall and F1 on both training and test partitions, then consolidate per-fold metrics into average ± standard deviation summaries per class. Validation: reproduced test metrics should match the reported values — test TSG accuracy ~0.83 ± 0.02 and test OG accuracy ~0.76 ± 0.03 (OG precision 0.79 ± 0.12, recall 0.50 ± 0.19) — within the reported standard deviations.

## Related tools

- **cTaG** (Pan-cancer random-forest model classifying genes as TSGs or OGs; provides the training/evaluation code (TSG_OG_Feat_final.py, evaluate_randiter.py) for cross-validated hyper-parameter estimation) — https://github.com/RamanLab/cTaG
- **Random Forest** (scikit-learn classifier trained per cross-validation fold; hyper-parameters (n_estimator, max_features, max_depth, criterion) tuned via fivefold CV and multiple random iterations) — https://github.com/RamanLab/cTaG
- **IdentifyTSGOG** (Hosts the pre-processed 37-feature matrix (FeatureMat.tsv) and the CGC TSG/OG gene lists used as training inputs) — https://github.com/RamanLab/IdentifyTSGOG/tree/master/data/cgc_genes

## Examples

```
python code/TSG_OG_Feat_final.py  # builds random-forest models on FeatureMat.tsv with CGC TSG/OG labels; hyper-parameters estimated via fivefold CV (see also code/evaluate_randiter.py)
```

## Evaluation signals

- Reproduced test TSG accuracy falls within 0.83 ± 0.02 and test OG accuracy within 0.76 ± 0.03 of the published metrics
- OG test precision and recall approximate 0.79 ± 0.12 and 0.50 ± 0.19 respectively
- Each cross-validation fold trains on exactly 80% of 136 TSGs and 76-77 OGs with a 20% held-out test partition
- The evaluate_randiter.py output (output/evalRandIter) confirms metric stability across the number of random iterations used for hyper-parameter selection
- Feature matrix has 37 columns per gene, matching FeatureMat.tsv before model training

## Limitations

- The small training set (136 TSGs, 76-77 OGs) makes the model prone to overfitting, hence the need for multiple random iterations to find stable hyper-parameters
- Test-set performance could not be compared with Davoli et al. because their test set results have not been published
- OG recall is low (0.50 ± 0.19), reflecting the difficulty of the OG class under this binary classification scheme
- The README notes the data was downloaded from COSMIC (v79 in the paper, v9 referenced in README data section), and only samples with fewer than 2000 mutations were considered

## Evidence

- [results] Trained random forest models on a subset (80%) of 136 TSGs and 76 OGs per cross-validation fold: "We trained multiple random forest models using a subset (80%) of 136 TSGs and 76 OGs for each fold of the cross-validation."
- [results] Fivefold cross-validation with multiple random iterations: "We performed fivefold cross-validation while estimating hyper-parameters for the model, followed by multiple random iterations to estimate stable hyper-parameters and avoid overfitting"
- [results] Test accuracy reported for TSGs and OGs: "Our model (cTaG) presents a significant improvement in recall for TSGs."
- [readme] CGC labels and 37-feature matrix as model inputs: "The genes were labelled as TSG or OG based on labels given by Cancer Gene Census (CGC), and the rest were marked Unlabelled."
- [readme] Overfitting rationale for stable hyper-parameters: "We overcome overfitting due to the small set of genes to train by estimating stable hyper-parameter set using multiple random iterations."
