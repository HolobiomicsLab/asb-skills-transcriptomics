---
name: domain-generalization-model-evaluation
description: Use when validating a pre-trained scRNA-seq cancer classifier against
  an annotated pan-cancer benchmark (e.g., TISCH tissues) to confirm reported accuracy
  and per-tissue generalization.
when_to_use_negative:
- Input data are hematologic tumor samples — the article explicitly does not recommend
  Cancer-Finder for hematologic tumor data
- The model must generalize to spatial transcriptomics without retraining — the single-cell
  models must be replaced with spatially trained models to identify malignant spots
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cancer-Finder
  role: Domain generalization-based deep-learning classifier whose pre-trained checkpoint
    is evaluated for malignant cell prediction accuracy
  repo: Patchouli-M/SequencingCancerFinder
- name: PyTorch
  role: Deep-learning framework backing the checkpoint loading and inference (torch
    1.13.1, CUDA 11.6 per README)
- name: SCANPY
  role: Applied to preprocess the validation matrix via 'pp.normalize_total' and 'log1p'
    (log-transformed global scaling) before inference
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:cb70fd78eae62a92c6847bc2fdadd77f2846e818c70e64a69470a986e378c168
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - Cancer-Finder
  - PyTorch
  - SCANPY
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# domain-generalization-model-evaluation

## Summary

Evaluate a domain generalization-based malignant cell annotation model (Cancer-Finder) by reproducing its reported 95.16% average malignant/non-malignant prediction accuracy across 13 TISCH tissues in the 20% internal-validation set under 5-fold leave-cells-out cross-validation.

## When to use

Use when validating a pre-trained scRNA-seq cancer classifier against an annotated pan-cancer benchmark (e.g., TISCH tissues) to confirm reported accuracy and per-tissue generalization.

## When NOT to use

- Input data are hematologic tumor samples — the article explicitly does not recommend Cancer-Finder for hematologic tumor data
- The model must generalize to spatial transcriptomics without retraining — the single-cell models must be replaced with spatially trained models to identify malignant spots

## Inputs

- Pre-trained model checkpoint pickle (checkpoints/sc_pretrain_article.pkl)
- TISCH 13-tissue internal-validation single-cell expression matrix (AnnData; 328,230 cells, 65,646-cell 20% validation fold)
- TISCH per-cell malignancy annotations (ground truth labels)

## Outputs

- Per-cell malignant/non-malignant predictions from infer.py at softmax threshold 0.5
- Per-tissue accuracy values ((TP + TN) / N) for the 13 tissues
- Cell-count-weighted average accuracy across the 13 tissues (target: 95.16%)

## How to apply

Load the pre-trained single-cell checkpoint (checkpoints/sc_pretrain_article.pkl) and pass it to infer.py via the --ckp parameter. Load the 13-tissue TISCH internal-validation expression matrix (328,230 cells; 65,646 cells held out as the 20% validation fold from the 4:1 train/validation split) and normalize with log-transformed global scaling using SCANPY's 'pp.normalize_total' and 'log1p'. Run inference with a softmax threshold of 0.5 over c = 2 classes to assign malignant/non-malignant predictions per cell. Compare predictions against TISCH cell malignancy annotations and compute per-tissue accuracy as (TP + TN) / N, then aggregate into a cell-count-weighted average across the 13 tissues.

## Related tools

- **Cancer-Finder** (Domain generalization-based deep-learning classifier whose pre-trained checkpoint is evaluated for malignant cell prediction accuracy) — Patchouli-M/SequencingCancerFinder
- **PyTorch** (Deep-learning framework backing the checkpoint loading and inference (torch 1.13.1, CUDA 11.6 per README))
- **SCANPY** (Applied to preprocess the validation matrix via 'pp.normalize_total' and 'log1p' (log-transformed global scaling) before inference)

## Examples

```
python infer.py --ckp checkpoints/sc_pretrain_article.pkl --input tis_13_tissues_internal_validation.h5ad --threshold 0.5
```

## Evaluation signals

- Cell-count-weighted average accuracy across the 13 TISCH tissues reproduces the reported 95.16%
- Malignant-status prediction accuracy exceeds 90% in 11 of the 13 tissues
- Per-tissue accuracy computed as (TP + TN) / N against TISCH malignancy annotations matches within expected tolerance
- All 328,230 validation cells receive exactly one binary prediction (c = 2 classes) at softmax threshold 0.5
- Preprocessing trace confirms 'pp.normalize_total' followed by 'log1p' was applied before inference

## Limitations

- Cancer-Finder is not recommended for hematologic tumor data
- Cancer-Finder overlooks spatial relationships among spots in ST data
- The model is robust only to a certain percentage of mislabeling in the training set
- Prediction errors may be resolvable only by increasing sequencing depth

## Evidence

- [abstract] average accuracy of 95.16%: "we present Cancer-Finder, a domain generalization-based deep-learning algorithm that can rapidly identify malignant cells in single-cell data with an average accuracy of 95.16%"
- [discussion] golden standard accuracy 98.30%: "achieving an accuracy of 98.30% in golden standard datasets"
- [discussion] hematologic tumor caveat: "We do not therefore recommend Cancer-Finder for hematologic tumor data"
- [other] checkpoint usage: [evidence span withheld — anchor status: not_found]
- [readme] environment prerequisites: "System: Ubuntu 18.04
python: 3.9.16
CUDA: 11.6
torch: 1.13.1"
