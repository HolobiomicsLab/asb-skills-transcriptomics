---
name: mutation-type-entropy-feature-interpretation
description: Use when a trained random-forest TSG/OG classifier (37-feature matrix,
  e.g. FeatureMat.tsv) must be interpreted to identify which mutation-type features
  (entropy, frequency, ratio-metric) drive the classification.
when_to_use_negative:
- Input is raw somatic mutation data rather than the pre-processed 37-feature matrix;
  feature-matrix construction (identifyTSGOG.py) must happen first
- The goal is driver-gene prediction on unlabelled genes rather than interpreting
  which features drive classification
- Data lacks mutation-type annotations (e.g. Tokheim et al. data), where missense
  and frameshift entropy/frequency features cannot be computed
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: cTaG
  role: Pan-cancer random-forest model whose trained fold classifiers provide the
    per-fold feature importance values that are ranked and averaged
  repo: https://github.com/RamanLab/cTaG
- name: RandomForest
  role: Classifier whose feature importance measure is extracted from each cross-validation
    fold model for ranking
  repo: https://github.com/RamanLab/cTaG
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-021-04015-y
    title: Novel ratio-metric features enable the identification of new driver genes
      across cancer types
  source_content_hash: sha256:3af5439878a5f58ce35ed64a6e6b12f804b1d31bfabf65c34a950ca480cb791d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/RamanLab/cTaG
  tools:
  - RandomForest
  - cTaG
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# mutation-type-entropy-feature-interpretation

## Summary

Interprets a random-forest-based pan-cancer TSG/OG classifier by ranking its 37 mutation-type entropy and ratio-metric features, revealing that nonsense/LOF- and missense-related features dominate TSG-vs-OG classification.

## When to use

Use when a trained random-forest TSG/OG classifier (37-feature matrix, e.g. FeatureMat.tsv) must be interpreted to identify which mutation-type features (entropy, frequency, ratio-metric) drive the classification.

## When NOT to use

- Input is raw somatic mutation data rather than the pre-processed 37-feature matrix; feature-matrix construction (identifyTSGOG.py) must happen first
- The goal is driver-gene prediction on unlabelled genes rather than interpreting which features drive classification
- Data lacks mutation-type annotations (e.g. Tokheim et al. data), where missense and frameshift entropy/frequency features cannot be computed

## Inputs

- Pre-processed gene-by-feature matrix FeatureMat.tsv (rows = genes/transcripts, columns = 37 mutation-type entropy/frequency/ratio-metric features)
- CGC-derived TSG/OG labels for 136 TSGs and 76 OGs used for training
- Five cross-validation random forest models with stable hyper-parameters

## Outputs

- Consensus feature importance ranking table feature_importance_ranking.csv (average per-feature rank across the five fold models)
- Ranked list of top features (dominated by LOF- and missense-related features such as Nonsense entropy, High missense frequency, High Frameshift Frequency, Damaging/kb)

## How to apply

Load the pre-processed gene-by-feature matrix (37 features per gene) from FeatureMat.tsv together with the CGC-derived labels for the 136 TSGs and 76 OGs used for training. Train a random forest classifier per fold of the fivefold cross-validation, using the stable hyper-parameter set estimated via multiple random iterations to avoid overfitting on this small training set. Extract the feature importance measure from each fold's model and convert the 37 importances to per-fold ranks. Average each feature's rank across all five fold models to obtain a single consensus importance ranking (saved as feature_importance_ranking.csv). Optionally compare feature-value distributions between TSGs and OGs in the training set to support the ranking-based interpretation.

## Related tools

- **cTaG** (Pan-cancer random-forest model whose trained fold classifiers provide the per-fold feature importance values that are ranked and averaged) — https://github.com/RamanLab/cTaG
- **RandomForest** (Classifier whose feature importance measure is extracted from each cross-validation fold model for ranking) — https://github.com/RamanLab/cTaG

## Examples

```
python code/TSG_OG_Feat_final.py  # loads FeatureMat.tsv + CGC TSG/OG labels, trains 5 CV random forest folds, outputs average feature ranks
```

## Evaluation signals

- The consensus ranking must place nonsense-, LOF-, frameshift- and missense-related features (e.g. nonsense entropy, frameshift frequency) in the top ranks
- feature_importance_ranking.csv must contain an average rank for all 37 features, each averaged over exactly five cross-validation fold models
- Top-ranked features should include Nonsense entropy, High missense frequency, Compound/benign, High Frameshift Frequency, Damaging/kb, Compound/kB, Damaging/LoFI and HiFI/benign
- Training data must contain exactly 136 TSGs and 76 OGs labelled from CGC, matching the paper's training set
- Per-fold importances must be converted to ranks before averaging, so the output is an average-rank (not average-importance) ordering

## Limitations

- Overfitting is a risk with only 136 TSGs and 76 OGs; stable hyper-parameters via multiple random iterations are required before interpreting feature importances
- Missense and frameshift entropy/frequency features could not be computed for the Tokheim et al. comparison data, limiting external validation of feature importance
- cTaG uses binary TSG/OG classification, so feature importance does not capture multi-class distinctions among all driver genes

## Evidence

- [results] To identify features important for the classification of TSGs and OGs, we calculated the average rank of each feature across all models.: "To identify features important for the classification of TSGs and OGs, we calculated the average rank of each feature across all models."
- [results] We performed fivefold cross-validation while estimating hyper-parameters for the model, followed by multiple random iterations to estimate stable hyper-parameters and avoid overfitting: "We performed fivefold cross-validation while estimating hyper-parameters for the model, followed by multiple random iterations to estimate stable hyper-parameters and avoid overfitting"
- [abstract] We overcome overfitting and show that certain mutation types, such as nonsense mutations, are more important for classification.: "We overcome overfitting and show that certain mutation types, such as nonsense mutations, are more important for classification."
- [readme] The mutation data was used to generate feature matrix were the rows and columns correspond to genes/transcripts and 37 features respectively.: "The mutation data was used to generate feature matrix were the rows and columns correspond to genes/transcripts and 37 features respectively."
