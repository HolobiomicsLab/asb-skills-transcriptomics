---
name: extra-domain-validation-checkpoint-selection
description: Use when training a multi-domain deep-learning model (e.g. Cancer-Finder
  on 13 TISCH tissue domains) and you need to choose the saved checkpoint by generalization
  to an unseen domain rather than internal-validation loss.
when_to_use_negative:
- Input is hematologic tumor data — the article explicitly does not recommend Cancer-Finder
  for hematologic tumor data
- The target domain is included in the training tissues, in which case internal-validation
  accuracy alone suffices and extra-domain checkpoint selection is redundant
- Training spatial transcriptomics spot models where spatial relationships among spots
  matter — Cancer-Finder overlooks these
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3170
tools:
- name: Cancer-Finder
  role: Domain-generalization model whose epoch checkpoints are scored and filtered
    by extra-domain accuracy
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
- name: PyTorch
  role: Framework used to build the network, compute L(θ) = β·R_var + R_avg (β = 1),
    and save/restore epoch checkpoints
- name: CopyKAT
  role: Source of the normalized breast cancer scRNA-seq data used as the extra-domain
    (unseen-tissue) validation set
  repo: https://github.com/navinlabcode/copykat
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:66a9bcfd5dc0459b6b85d7a773ad5204b85a0aaa8e2d9a1d46aab53395bf6b86
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - Cancer-Finder
  - CopyKAT
  - PyTorch
  - Seurat
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# Extra-domain validation checkpoint selection

## Summary

A model-selection strategy from Cancer-Finder training in which, after each epoch of domain-generalization risk-extrapolation optimization, the model is additionally evaluated on normalized breast cancer data from the original CopyKAT study (a tissue not in the 13-tissue training set), and only the checkpoint whose extra-domain accuracy is maximal is retained.

## When to use

Use when training a multi-domain deep-learning model (e.g. Cancer-Finder on 13 TISCH tissue domains) and you need to choose the saved checkpoint by generalization to an unseen domain rather than internal-validation loss.

## When NOT to use

- Input is hematologic tumor data — the article explicitly does not recommend Cancer-Finder for hematologic tumor data
- The target domain is included in the training tissues, in which case internal-validation accuracy alone suffices and extra-domain checkpoint selection is redundant
- Training spatial transcriptomics spot models where spatial relationships among spots matter — Cancer-Finder overlooks these

## Inputs

- Per-epoch PyTorch checkpoints of the Cancer-Finder network (two FC layers 4572->512 with dropout 0.5, 2-neuron softmax classification layer)
- 80/20 internal validation split of the 1:1-downsampled 328,230 cells from 13 TISCH tissues (log1p global-scaling normalized, 4572 HVG features)
- Extra-domain validation set: log-normalized breast cancer scRNA-seq data from the original CopyKAT study (tissue excluded from training)

## Outputs

- Single retained checkpoint with maximal extra-domain (CopyKAT breast cancer) accuracy
- Training curves of variance risk R_var, average risk R_avg, internal-validation accuracy, and extra-domain accuracy over 50-70 epochs

## How to apply

During the 50-70 training epochs of 5-fold leave-cells-out cross-validation, after each epoch compute per-domain cross-entropy risks R_e and the loss L(θ) = β·R_var(θ) + R_avg(θ) with β = 1 (variance + average risk). In parallel, evaluate each epoch's weights on two sets: the held-out internal validation split of the 328,230 training cells and an extra-domain set of normalized breast cancer scRNA-seq data from the original CopyKAT study (breast tissue excluded from the 13 training tissues). Discard all intermediate checkpoints, retaining only the single checkpoint whose extra-domain accuracy on the CopyKAT breast cancer data is maximal. Rationale: extra-domain performance directly measures the out-of-distribution generalization that domain generalization aims to maximize, so it is a better checkpoint criterion than internal validation, which can favor tissue-specific overfitting. Correct application is confirmed when the retained model reaches ~95.16% average internal-validation accuracy across the 13 tissues.

## Related tools

- **Cancer-Finder** (Domain-generalization model whose epoch checkpoints are scored and filtered by extra-domain accuracy) — https://github.com/Patchouli-M/SequencingCancerFinder
- **PyTorch** (Framework used to build the network, compute L(θ) = β·R_var + R_avg (β = 1), and save/restore epoch checkpoints)
- **CopyKAT** (Source of the normalized breast cancer scRNA-seq data used as the extra-domain (unseen-tissue) validation set) — https://github.com/navinlabcode/copykat

## Examples

```
best_ckpt = max(epoch_checkpoints, key=lambda ck: ck['extra_domain_acc'])  # CopyKAT breast-cancer accuracy, Cancer-Finder training (torch 1.13.1)
```

## Evaluation signals

- The retained checkpoint is the one with maximal accuracy on the CopyKAT breast cancer extra-domain set, not the lowest training loss epoch
- Retained model's average internal-validation accuracy across the 13 tissues is approximately 95.16%, matching the reported value
- Breast tissue is genuinely absent from the 13 training tissue domains (i.e. the extra-domain set is out-of-distribution)
- Only one checkpoint is kept per fold; intermediate epoch weights are discarded rather than the last epoch being used by default
- Training was run 50-70 epochs under 5-fold leave-cells-out cross-validation with β = 1 in the risk-extrapolation loss

## Limitations

- Cancer-Finder is not recommended for hematologic tumor data, so extra-domain selection on such data is out of scope
- The model is robust only to a certain percentage of mislabeling in the training set
- Prediction errors may be resolvable only by increasing sequencing depth

## Evidence

- [other] After each epoch, evaluate on the internal validation set and on normalized breast cancer data from the original CopyKAT study as extra-domain validation, retaining only the checkpoint whose extra-domain accuracy is maximal.: [evidence span withheld — anchor status: not_found]
- [abstract] Cancer-Finder identifies malignant cells with average accuracy of 95.16%.: "rapidly identify malignant cells in single-cell data with an average accuracy of 95.16%"
- [readme] Cancer-Finder is a domain generalization-based deep learning algorithm.: "we present Cancer-Finder, a domain generalization-based deep learning algorithm that can rapidly identify malignant cells/spots in single-cell and spatial transcriptomics data"
- [readme] Environment uses PyTorch 1.13.1 with CUDA 11.6.: "System: Ubuntu 18.04
python: 3.9.16
CUDA: 11.6
torch: 1.13.1"
