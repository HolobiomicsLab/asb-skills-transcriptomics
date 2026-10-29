---
name: classification-metrics-evaluation
description: Use when a supervised cell type or cell state classifier has been trained
  on labeled scRNA-seq data and its predictive quality must be quantified on a test
  set, especially when comparing pipelines on accuracy alongside resource metrics.
when_to_use_negative:
- The task is unsupervised cluster annotation (marker-gene or reference-mapping based),
  where precision/recall against ground-truth labels is not defined in the same way
- Only a validation set is available and no truly held-out test split exists, risking
  optimistic metrics from early-stopping selection
- The goal is feature ranking/biomarker discovery rather than classification performance
  measurement
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: scaLR
  role: Provides the eval_and_analysis_pipeline stage that computes precision, recall,
    f1-score, and accuracy on the test dataset and generates visualizations such as
    ROC curves
  repo: https://github.com/infocusp/scaLR
- name: Python
  role: Language in which scaLR and the compared pipelines' custom benchmark scripts
    are implemented and executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf243
    title: 'scaLR: a low-resource deep neural network-based platform for single cell
      analysis and biomarker discovery'
  source_content_hash: sha256:b147fd5a5eae528c40ce1e5d792771f7ed240dc894342116eaa81add3fadd3df
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# classification-metrics-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluate a trained cell type/state classifier on a held-out test set by computing precision, recall, f1-score, and accuracy per class, as scaLR does for its DNN annotation model and benchmark comparisons on scRNA-seq datasets like PBMCs-BS.

## When to use

Use when a supervised cell type or cell state classifier has been trained on labeled scRNA-seq data and its predictive quality must be quantified on a test set, especially when comparing pipelines on accuracy alongside resource metrics.

## When NOT to use

- The task is unsupervised cluster annotation (marker-gene or reference-mapping based), where precision/recall against ground-truth labels is not defined in the same way
- Only a validation set is available and no truly held-out test split exists, risking optimistic metrics from early-stopping selection
- The goal is feature ranking/biomarker discovery rather than classification performance measurement

## Inputs

- Trained model checkpoint from scaLR final training stage
- Test-split AnnData (.h5ad) with `adata.obs` target column (e.g., cell_type or disease) and normalized `adata.X` (log1p, range 0-10)
- Predictions for each compared pipeline on the same test set

## Outputs

- Precision, recall, f1-score, and accuracy scores for the test set
- ROC AUC curve plots of class annotation
- Benchmark comparison table (e.g., pbmcs_bs_benchmark_comparison.csv) with accuracy, memory usage, and execution time per pipeline

## How to apply

After final model training (e.g., scaLR's SequentialModel trained on top-K features with EarlyStopping and ModelCheckpoint callbacks), run the evaluation and analysis stage on the held-out test split using a SimpleDataLoader (e.g., batch_size 15000) with a model checkpoint. Compute precision, recall, f1-score, and accuracy scores on the predictions versus the `target` column of `adata.obs`. For multi-layer scaLR models, optionally complement metrics with SHAP analysis to inspect feature contributions. In benchmark settings (e.g., the PBMCs-BS comparison against scVI-tools, CellTypist, ACTINN, devCellPy, scBalance, cellPLM), apply the identical metric computation to every pipeline's test-set predictions so accuracy, memory usage, and execution time can be compiled into a single comparison CSV.

## Related tools

- **scaLR** (Provides the eval_and_analysis_pipeline stage that computes precision, recall, f1-score, and accuracy on the test dataset and generates visualizations such as ROC curves) — https://github.com/infocusp/scaLR
- **Python** (Language in which scaLR and the compared pipelines' custom benchmark scripts are implemented and executed)

## Examples

```
python pipeline.py --config /path/to/config.yaml -l -m
```

## Evaluation signals

- All four metrics (precision, recall, f1-score, accuracy) are computed for every class on the held-out test split, not the training or validation split
- Benchmark scaLR accuracy against the compared pipelines (scVI-tools, SingleCellNet, CellTypist variants, ACTINN, devCellPy, scBalance, cellPLM) and confirm it is comparable while training time and resources are lower
- Identical metric computation and resource logging applied to all pipelines on the same VM (12GB GPU / 128GB RAM / 8 CPU) so the comparison is fair
- Recorded caveats propagate: e.g., scVI-tools failing on all 22,858 features due to >128GB RAM demand is noted alongside the metrics

## Limitations

- Benchmark result tables (Tables 2-4) were referenced but not present in the extracted text, so exact metric values are not verifiable from the article text alone
- scaLR's memory usage is inversely proportional to execution time, so resource metrics trade off and must be interpreted together
- Feature importance cannot be extracted from several comparison tools (scBalance, scVI-tools, devCellPy, cellPLM), limiting comparability beyond classification metrics
- Comparison tool versions and scaLR software version numbers are not stated

## Evidence

- [readme] README lists evaluation metrics and ROC: [evidence span withheld — anchor status: external_doc]
- [discussion] Benchmark comparison workflow validation: [evidence span withheld — anchor status: not_found]
