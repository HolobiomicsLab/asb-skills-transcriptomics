---
name: single-cell-feature-extraction
description: Use when you need to reduce a large scRNA-seq gene-expression matrix
  (anndata .h5ad) to its most informative top-K genes for cell-type or clinical-condition
  classification, especially on resource-limited hardware.
when_to_use_negative:
- You already have a small curated marker-gene panel and full-dimensional training
  is feasible on your hardware
- You need model-agnostic feature importance from non-linear models — use the shap_scorer
  path instead of the linear weight-based scorer
- The input data is not anndata .h5ad or has not been normalized (scaLR's performance
  varies with input data format and requires proper preprocessing)
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scaLR
  role: 'End-to-end pipeline providing the feature-extraction stage: feature_subsetting
    module, SequentialModel, SimpleModelTrainer, LinearScorer, and AbsMean selector'
  repo: https://github.com/infocusp/scaLR
- name: Python
  role: Implementation language; scaLR is a Python-based platform tested with Python
    3.10.20
  repo: https://github.com/infocusp/scaLR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:0acdbb899cb70e385033e5ecc284552dacfb144f015e0cf31529efec14c5646d
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# single-cell-feature-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs scaLR's feature-extraction stage: training a single-layer DNN per feature subset on scRNA-seq data and scoring features by mean absolute class-wise weights to select top-K genes for the final classification model. It is the mechanism scaLR uses to make biomarker discovery tractable on low-resource machines.

## When to use

Use when you need to reduce a large scRNA-seq gene-expression matrix (anndata .h5ad) to its most informative top-K genes for cell-type or clinical-condition classification, especially on resource-limited hardware.

## When NOT to use

- You already have a small curated marker-gene panel and full-dimensional training is feasible on your hardware
- You need model-agnostic feature importance from non-linear models — use the shap_scorer path instead of the linear weight-based scorer
- The input data is not anndata .h5ad or has not been normalized (scaLR's performance varies with input data format and requires proper preprocessing)

## Inputs

- Preprocessed single-cell gene expression matrix in anndata .h5ad format (adata.X: log1p-normalized counts, range 0-10; adata.obs: metadata with target column; adata.var: gene names)
- Feature-selection YAML config block (feature_subsetsize, SequentialModel layers, SimpleModelTrainer with SGD optimizer, epochs, LinearScorer scoring_config, AbsMean selector with k)

## Outputs

- feature_scores.csv containing an importance score for every input feature
- Ranked list of top-K features (e.g. k=5000) used to train the final classification model

## How to apply

Load the preprocessed expression matrix (adata.X with log1p-normalized counts in 0–10 range, target column in adata.obs) and partition features into subsets of a fixed size (e.g. feature_subsetsize: 5000) via scalr.feature.feature_subsetting. For each subset, train a single-layer SequentialModel (scalr.nn.model.sequential_model) with SimpleModelTrainer, keeping the optimizer identical across subsets (e.g. SGD, lr=1.0e-3, weight_decay=0.1) and initializing all weights to zero (weights_init_zero: True). Aggregate the weight matrices from all subset models, since these weights quantify each feature's contribution toward predicting specific classes, then score every feature by the mean of absolute weights across classes using the abs_mean selector (scalr.feature.selector.abs_mean) driven by the linear scorer (scalr.feature.scoring.linear_scorer). Finally, rank all features by importance score and retain the top-K (e.g. k: 5000); the resulting feature_scores.csv feeds final model training. The subset-wise design ensures all features and samples participate in feature selection while keeping memory usage bounded.

## Related tools

- **scaLR** (End-to-end pipeline providing the feature-extraction stage: feature_subsetting module, SequentialModel, SimpleModelTrainer, LinearScorer, and AbsMean selector) — https://github.com/infocusp/scaLR
- **Python** (Implementation language; scaLR is a Python-based platform tested with Python 3.10.20) — https://github.com/infocusp/scaLR

## Examples

```
python pipeline.py --config /path/to/config.yaml -l -m  # with feature_selection: feature_subsetsize: 5000, model SequentialModel layers [5000, 10], weights_init_zero: True, AbsMean k: 5000
```

## Evaluation signals

- feature_scores.csv exists and contains an importance score for every input feature
- The top-K rows in feature_scores.csv correspond to the highest mean absolute class-wise weights across all subset models
- The same optimizer (e.g. SGD lr=1.0e-3, weight_decay=0.1) and zero weight initialization are used identically across all feature subsets
- The selected top-K feature count matches the configured AbsMean k parameter (e.g. k: 5000)
- Downstream final model trained on the top-K features achieves precision/recall/f1/accuracy comparable to or better than full-feature training (paper reports top 3,500 features retain performance)

## Limitations

- Feature scores from the linear scorer depend on a single-layer network; for multi-layer models SHAP-based scoring is required instead
- Memory usage is inversely proportional to execution time, so reducing memory consumption may increase processing time
- Performance varies with the input data format, requiring users to ensure proper data preprocessing (e.g. log1p normalization in the 0-10 range) for optimal results
- Benchmark result tables with exact accuracy numbers for top-feature training were not fully reported in the extracted text

## Evidence

- [readme] A model is trained on feature subsets in a batch-wise process, so all features and samples are utilized in the feature selection process.: [evidence span withheld — anchor status: external_doc]
- [readme] features_selector:
            name: AbsMean
            params:
                k: 5000: "features_selector:
            name: AbsMean
            params:
                k: 5000"
