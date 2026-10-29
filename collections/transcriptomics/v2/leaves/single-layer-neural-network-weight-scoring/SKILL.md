---
name: single-layer-neural-network-weight-scoring
description: 'Use when you must perform feature selection for scRNA-seq classification
  under low resources: an .h5ad/anndata gene-expression matrix is too large to train
  one model on all features, so per-subset weight scoring is needed to pick top-K
  genes.'
when_to_use_negative:
- You need feature attributions from a multi-layer model — use SHAP (shap_scorer)
  instead, since linear weight scoring only interprets the single-layer network's
  coefficients
- You need class-wise (per-class) top features rather than a global top-K ranking
  — use the classwise_abs or ClasswisePromoters selector instead of abs_mean
- The comparison tool you are using (e.g. scBalance, scvi, devCellPy, CellPLM) does
  not expose feature importances — weight-based top-K extraction is not applicable
  there
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scaLR
  role: 'Platform implementing subset-wise feature extraction: feature subsetting,
    SequentialModel training, LinearScorer, and AbsMean top-K selector'
  repo: https://github.com/infocusp/scaLR
- name: Python
  role: Language in which the scaLR feature-extraction pipeline (pyScaLR / pipeline.py
    with config.yaml) is executed
  repo: https://github.com/infocusp/scaLR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:623e5b82ef28fb984e798093104d684d07132b7c7e5bb354c2107d19e041b6ab
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/infocusp/scaLR
  repo_url: https://github.com/infocusp/scaLR
  tools:
  - Python
  - scaLR
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# single-layer-neural-network-weight-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

scaLR's feature-extraction mechanism: train a single-layer network (SequentialModel with weights_init_zero: True) on each feature subset, aggregate the class-wise weight matrices, and score every feature by the mean of absolute weights (LinearScorer + abs_mean selector) to rank and retain top-K features.

## When to use

Use when you must perform feature selection for scRNA-seq classification under low resources: an .h5ad/anndata gene-expression matrix is too large to train one model on all features, so per-subset weight scoring is needed to pick top-K genes.

## When NOT to use

- You need feature attributions from a multi-layer model — use SHAP (shap_scorer) instead, since linear weight scoring only interprets the single-layer network's coefficients
- You need class-wise (per-class) top features rather than a global top-K ranking — use the classwise_abs or ClasswisePromoters selector instead of abs_mean
- The comparison tool you are using (e.g. scBalance, scvi, devCellPy, CellPLM) does not expose feature importances — weight-based top-K extraction is not applicable there

## Inputs

- Preprocessed anndata .h5ad scRNA-seq matrix (adata.X log1p-normalized counts in range 0-10, adata.obs with a target column, adata.var gene names)
- Feature-selection config: model layers, feature_subsetsize, SimpleModelTrainer settings (optimizer, loss, epochs), and scoring/selector config
- Class labels (e.g. cell_type or disease) used as the classification target

## Outputs

- feature_scores.csv containing an importance score for every input feature
- Ranked top-K feature list (e.g. k=5000 genes) feeding the final model training stage
- Aggregated class-wise weight matrices from the subset models

## How to apply

Partition the preprocessed single-cell expression matrix into feature subsets (e.g. feature_subsetsize: 5000) using scalr.feature.feature_subsetting. For each subset, train a single-layer SequentialModel via SimpleModelTrainer with weights_init_zero: True and an identical optimizer across subsets (e.g. SGD, lr 1.0e-3, weight_decay 0.1, CrossEntropyLoss, ~10 epochs, batch_size 25000 with padding 5000), so all features and samples are utilized batch-wise. Collect each subset model's weight matrix, since these weights quantify each feature's contribution toward predicting specific classes, and score features with LinearScorer, applying the AbsMean selector with the desired k (e.g. k: 5000) to compute the mean of absolute class-wise weights. Rank all features by importance and retain the top-K features to train the final model; the run writes feature_scores.csv with one importance score per input feature.

## Related tools

- **scaLR** (Platform implementing subset-wise feature extraction: feature subsetting, SequentialModel training, LinearScorer, and AbsMean top-K selector) — https://github.com/infocusp/scaLR
- **Python** (Language in which the scaLR feature-extraction pipeline (pyScaLR / pipeline.py with config.yaml) is executed) — https://github.com/infocusp/scaLR

## Examples

```
python pipeline.py --config /path/to/config.yaml -l -m  # with feature_selection: {feature_subsetsize: 5000, scoring_config: LinearScorer, features_selector: {name: AbsMean, k: 5000}}
```

## Evaluation signals

- feature_scores.csv exists and contains an importance score for every input feature
- The top-K rows of feature_scores.csv correspond to the features with the highest mean absolute class-wise weights from the aggregated subset weight matrices
- All feature subsets were trained with the identical optimizer and zero-initialized weights (weights_init_zero: True), so the aggregate scores are comparable across subsets
- The number of retained features equals the selector's k parameter (e.g. k: 5000) and downstream final training consumes exactly those top-K genes
- The subset count matches ceil(total_features / feature_subsetsize) with feature_subsetsize: 5000 covering all genes

## Limitations

- Memory usage is inversely proportional to execution time — reducing memory consumption may increase processing time during subset training
- Performance may vary based on the input data format, requiring users to ensure proper data preprocessing (log1p normalization, range 0-10) for optimal results
- Weight-based scoring applies to linear/single-layer models; for multi-layer architectures SHAP-based scoring is required instead
- Specific hyperparameters (epochs, layers) affect score stability; the paper only reports sensitivity analyses without final values

## Evidence

- [readme] A model is trained on feature subsets in a batch-wise process, so all features and samples are utilized in the feature selection process.: [evidence span withheld — anchor status: external_doc]
- [readme] name: SequentialModel ... layers: [5000, 10] ... weights_init_zero: True ... name: LinearScorer ... name: AbsMean ... k: 5000: [evidence span withheld — anchor status: external_doc]
- [readme] Then, the top-k features are selected to train the final model, using a feature score based on the model's coefficients/weights or SHAP analayis.: [evidence span withheld — anchor status: not_found]
