---
name: organoid-cell-type-accuracy-evaluation
description: Use when quantifying how accurately a supervised single-cell classifier
  (SIMS/TabNet) transfers cell-type labels across iPSC cell lines in cortical organoid
  scRNA-seq datasets, or when reproducing reported accuracy and radial glia misclassification
  rates from Figures 6C-D, 6E and Table S11.
when_to_use_negative:
- Training and prediction data come from different species — the authors explicitly
  limited SIMS to samples within the same species because neuronal subtypes diverge
  significantly between species
- The target dataset lacks matching gene features or was not processed with the same
  normalization pipeline (filtering, normalize_total, log1p, scale) as the training
  data
- No pre-existing annotations are available for the reference dataset, since SIMS
  is supervised and relies on prior expert annotations
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: SIMS
  role: Scalable, interpretable deep-learning pipeline (scsims package) used to train
    on 11A organoid annotations and predict cell-type labels on PGP1 and GM8330 organoids
  repo: https://github.com/braingeneers/SIMS
- name: TabNet
  role: Underlying model architecture for tabular scRNA-seq data, configured with
    n_a=64 and n_d=64 and class-frequency loss weighting
  repo: https://github.com/braingeneers/SIMS
- name: PyTorch Lightning
  role: Trainer framework supplying EarlyStopping (val_loss, patience=50), LearningRateMonitor,
    and a 100-epoch training loop
  repo: https://github.com/braingeneers/SIMS
- name: Gruffi
  role: Identifies stressed cells in the organoid datasets that are removed in the
    sensitivity evaluation
  repo: https://github.com/jn-goe/gruffi
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.xgen.2024.100581
    title: 'SIMS: A deep-learning label transfer tool for single-cell RNA sequencing
      analysis'
  source_content_hash: sha256:be6da9e31d389ef3855bb2aa7d5b948a2ac5e6420bfba672919b612e635e7fd6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/jlehrer1/sims
  repo_url: https://github.com/braingeneers/SIMS
  tools:
  - Gruffi
  - PyTorch Lightning
  - SIMS
  - TabNet
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# organoid-cell-type-accuracy-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates SIMS (TabNet-based) cross-cell-line label transfer on human cortical organoids by training on 11A-derived organoid annotations and scoring predictions on Velasco PGP1 and GM8330 organoids, quantifying overall accuracy and lineage-specific misclassification (radial glia called immature projection neurons).

## When to use

Use when quantifying how accurately a supervised single-cell classifier (SIMS/TabNet) transfers cell-type labels across iPSC cell lines in cortical organoid scRNA-seq datasets, or when reproducing reported accuracy and radial glia misclassification rates from Figures 6C-D, 6E and Table S11.

## When NOT to use

- Training and prediction data come from different species — the authors explicitly limited SIMS to samples within the same species because neuronal subtypes diverge significantly between species
- The target dataset lacks matching gene features or was not processed with the same normalization pipeline (filtering, normalize_total, log1p, scale) as the training data
- No pre-existing annotations are available for the reference dataset, since SIMS is supervised and relies on prior expert annotations

## Inputs

- Velasco PGP1 organoid AnnData (.h5ad) expression matrix (cell x gene, log1p-normalized)
- Velasco GM8330 organoid AnnData (.h5ad) expression matrix
- 11A-derived organoid reference annotations with 'class_label' label column
- Gruffi-annotated stressed-cell labels per dataset (for the stress-removal sensitivity run)

## Outputs

- Per-cell predicted cell-type labels for PGP1 and GM8330 organoids
- Overall accuracy and Macro F1 per target cell line (expected ~71.3% / Macro F1 0.564 for PGP1; ~67.4% / Macro F1 0.570 for GM8330)
- Radial glia misclassification rate (fraction predicted as immature projection neurons; expected ~82% PGP1, ~42% GM8330)
- sims_label_transfer_metrics.csv with per-dataset rows

## How to apply

Load the Velasco PGP1 and GM8330 organoid expression matrices as AnnData (.h5ad) together with the 11A-derived reference annotations using the 'class_label' column, then instantiate SIMS with labels_key='class_label'. Configure the TabNet model with n_a=64, n_d=64, and weights=sims.weights to weight the loss inversely by label frequency so rare cell types are learned. Set up a PyTorch Lightning trainer with EarlyStopping on val_loss (patience=50), a LearningRateMonitor (logging_interval='epoch'), optionally a Weights & Biases logger, and num_epochs=100, then train on the 11A reference and run predict on PGP1 and GM8330 cells. Repeat the full evaluation after removing Gruffi-annotated stressed cells from each dataset to assess robustness to stress artifacts. Compute overall accuracy and Macro F1 plus the fraction of radial glia predicted as immature projection neurons, writing per-dataset rows to sims_label_transfer_metrics.csv.

## Related tools

- **SIMS** (Scalable, interpretable deep-learning pipeline (scsims package) used to train on 11A organoid annotations and predict cell-type labels on PGP1 and GM8330 organoids) — https://github.com/braingeneers/SIMS
- **TabNet** (Underlying model architecture for tabular scRNA-seq data, configured with n_a=64 and n_d=64 and class-frequency loss weighting) — https://github.com/braingeneers/SIMS
- **PyTorch Lightning** (Trainer framework supplying EarlyStopping (val_loss, patience=50), LearningRateMonitor, and a 100-epoch training loop) — https://github.com/braingeneers/SIMS
- **Gruffi** (Identifies stressed cells in the organoid datasets that are removed in the sensitivity evaluation) — https://github.com/jn-goe/gruffi

## Examples

```
from scsims import SIMS; sims = SIMS(adata=adata_11A, labels_key='class_label'); sims.setup_model(n_a=64, n_d=64, weights=sims.weights); sims.setup_trainer(callbacks=[EarlyStopping(monitor='val_loss', patience=50)], max_epochs=100); sims.train(); preds = sims.predict(pgp1_adata, top_k=1)
```

## Evaluation signals

- Overall accuracy on PGP1 organoids is approximately 71.3% (Macro F1 0.564) and on GM8330 approximately 67.4% (Macro F1 0.570), matching Figures 6C-D and Table S11 within a reasonable tolerance
- Radial glia misclassification as immature projection neurons is approximately 82% for PGP1 and 42% for GM8330, matching Figure 6E
- sims_label_transfer_metrics.csv exists and contains the required per-dataset rows (PGP1 and GM8330) with accuracy and misclassification columns
- Re-running the evaluation after Gruffi stressed-cell removal still yields metrics in the reported range, confirming robustness to stress artifacts
- Prediction output follows the SIMS predict() schema with pred_0 / prob_0 (top_k=1) columns per cell

## Limitations

- SIMS has only been validated for label transfer within the same species; trans-species prediction is untested because neuronal subtypes diverge significantly between species
- As a supervised model, SIMS depends on pre-existing annotations; technical factors such as sequencing technology or incorrect normalization can corrupt differential-expression-derived labels
- The accuracy gap between cell lines (82% vs 42% radial glia misclassification) shows cell-line-specific biases in transferred labels

## Evidence

- [results] Cross-cell-line accuracy reported: [evidence span withheld — anchor status: not_found]
- [readme] Prediction output schema: [evidence span withheld — anchor status: external_doc]
- [readme] Preprocessing requirement: [evidence span withheld — anchor status: external_doc]
