---
name: gene-expression-feature-subsetting
description: Use when the scRNA-seq dataset is too large (e.g. >1 million cells) or
  resource-limited to train a feature-selection model on all genes at once, and you
  need an importance-ranked feature set before final classifier training.
when_to_use_negative:
- The input is already a small feature table or a pre-ranked feature-importance file,
  so subset-wise training adds no value.
- A SHAP-based feature score is required instead of linear weight magnitudes (scaLR
  offers shap_scorer for multi-layer models) or the goal is class-wise rather than
  overall top-K selection (use ClasswiseAbs/ClasswisePromoters instead of AbsMean).
- The dataset is small enough that a full DNN can be trained on all features without
  subsetting — the partitioning rationale (low-resource execution) does not apply.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scaLR
  role: Provides the feature_subsetting, SequentialModel, SimpleModelTrainer, LinearScorer,
    and AbsMean modules that implement subset-wise feature extraction and top-K selection
  repo: https://github.com/infocusp/scaLR
- name: Python
  role: Implementation language of the scaLR pipeline; tested with Python 3.10.20
  repo: https://github.com/infocusp/scaLR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:dd42dd376c69b1c5d2cab944f31e743180b4c2822d2b5081819147e1aa42d16e
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# gene-expression-feature-subsetting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A low-resource feature-extraction procedure from scaLR that partitions a single-cell gene expression matrix (.h5ad/anndata) into feature subsets, trains a single-layer DNN on each subset, and aggregates the resulting weight matrices to rank all features for top-K selection.

## When to use

Use when the scRNA-seq dataset is too large (e.g. >1 million cells) or resource-limited to train a feature-selection model on all genes at once, and you need an importance-ranked feature set before final classifier training.

## When NOT to use

- The input is already a small feature table or a pre-ranked feature-importance file, so subset-wise training adds no value.
- A SHAP-based feature score is required instead of linear weight magnitudes (scaLR offers shap_scorer for multi-layer models) or the goal is class-wise rather than overall top-K selection (use ClasswiseAbs/ClasswisePromoters instead of AbsMean).
- The dataset is small enough that a full DNN can be trained on all features without subsetting — the partitioning rationale (low-resource execution) does not apply.

## Inputs

- anndata (.h5ad) single-cell gene expression matrix with adata.X normalized counts (log1p, range 0-10), adata.obs containing a target column for classification, and adata.var holding gene names
- feature-selection config block (config.yaml) specifying feature_subsetsize, model layers, optimizer (e.g. SGD lr 1.0e-3, weight_decay 0.1), loss (CrossEntropyLoss), epochs, LinearScorer scoring_config, and AbsMean selector with k

## Outputs

- Aggregated class-wise weight/importance matrix for every input feature (e.g. feature_scores.csv with an importance score per gene)
- Top-K ranked feature set used to train the final classification model

## How to apply

Load the preprocessed anndata object (.h5ad, with log1p-normalized counts in adata.X, target labels in adata.obs, gene names in adata.var) and partition features into manageable subsets via the scalr.feature.feature_subsetting module (e.g. feature_subsetsize: 5000). For each subset, train a single-layer SequentialModel (scalr.nn.model.sequential_model) with SimpleModelTrainer, keeping the optimizer identical across subsets (e.g. SGD, lr 1.0e-3, weight_decay 0.1, CrossEntropyLoss, ~10 epochs) and initializing all weights to zero (weights_init_zero: True) so weight magnitudes directly reflect feature contribution. Collect the weight matrix from each subset model — these weights quantify each feature's contribution toward predicting specific classes — and aggregate them across subsets. Score every feature with the LinearScorer (scalr.feature.scoring.linear_scorer) and rank via the AbsMean selector (scalr.feature.selector.abs_mean), which computes the mean of absolute class-wise weights; retain the top-K features (e.g. k: 5000) to train the final model.

## Related tools

- **scaLR** (Provides the feature_subsetting, SequentialModel, SimpleModelTrainer, LinearScorer, and AbsMean modules that implement subset-wise feature extraction and top-K selection) — https://github.com/infocusp/scaLR
- **Python** (Implementation language of the scaLR pipeline; tested with Python 3.10.20) — https://github.com/infocusp/scaLR

## Examples

```
python pipeline.py --config /path/to/config.yaml -l -m  # with feature_selection: feature_subsetsize: 5000, LinearScorer scoring, AbsMean k: 5000
```

## Evaluation signals

- feature_scores.csv exists and contains an importance score for every input feature
- The top-K rows in feature_scores.csv correspond to the highest mean absolute class-wise weights from the aggregated subset weight matrices
- The number of selected features equals the configured k (e.g. 5000) and the subset training kept the optimizer identical across subsets with weights_init_zero: True
- Final model trained on the selected top-K features achieves precision/recall/f1/accuracy comparable to the all-features model (paper reports top 3,500 features performed on par)

## Limitations

- Memory usage is inversely proportional to execution time — reducing memory consumption may increase processing time.
- Performance may vary based on input data format; users must ensure proper preprocessing (log1p normalization in range 0-10) for optimal results.
- Specific hyperparameters (layers, epochs) for the DNN are referenced only via sensitivity analysis without reported values; results may be sensitive to these choices.
- Feature importance scoring via LinearScorer relies on linear model weights; the paper notes no feature extraction from comparison tools (scBalance, scvi, devCellPy, CellPLM), limiting cross-tool comparability.

## Evidence

- [readme] Then, the top-k features are selected to train the final model, using a feature score based on the model's coefficients/weights: [evidence span withheld — anchor status: external_doc]
- [readme] A model is trained on feature subsets in a batch-wise process, so all features and samples are utilized in the feature selection process.: [evidence span withheld — anchor status: external_doc]
