---
name: per-tissue-accuracy-evaluation
description: Use when validating a pre-trained malignant-cell classifier (e.g. Cancer-Finder
  checkpoints/sc_pretrain_article.pkl) against per-cell malignancy labels from TISCH
  across multiple tissues, needing per-tissue and weighted-average accuracy.
when_to_use_negative:
- Data are hematologic tumor samples — the article explicitly does not recommend Cancer-Finder
  for hematologic tumor data
- The task is copy-number/CNV inference or clonal structure characterization (use
  CopyKAT or SCEVAN instead), not direct malignant-state classification accuracy evaluation
- Input is spatial transcriptomics spot data without the spatially trained Cancer-Finder
  models — single-cell pre-trained checkpoints target scRNA-seq malignant cell annotation
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: Cancer-Finder
  role: Pre-trained domain generalization model whose checkpoint (sc_pretrain_article.pkl)
    is used with infer.py to predict malignant/non-malignant state per cell
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
- name: SCANPY
  role: Preprocessing of the TISCH validation expression matrix using pp.normalize_total
    and log1p (log-transformed global scaling) before inference
- name: PyTorch
  role: Deep-learning framework in which the Cancer-Finder checkpoint is loaded and
    inference is executed (torch 1.13.1, CUDA 11.6 per README)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:249224dce339e1e4c0fa9bc56092d4d82d3187a4ea319441853f77c270b55ef1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - Cancer-Finder
  - PyTorch
  - SCANPY
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# per-tissue-accuracy-evaluation

## Summary

Evaluate malignant-state classification accuracy of Cancer-Finder per tissue on the TISCH internal-validation set and aggregate into a cell-count-weighted average to reproduce the reported 95.16% accuracy across 13 tissues.

## When to use

Use when validating a pre-trained malignant-cell classifier (e.g. Cancer-Finder checkpoints/sc_pretrain_article.pkl) against per-cell malignancy labels from TISCH across multiple tissues, needing per-tissue and weighted-average accuracy.

## When NOT to use

- Data are hematologic tumor samples — the article explicitly does not recommend Cancer-Finder for hematologic tumor data
- The task is copy-number/CNV inference or clonal structure characterization (use CopyKAT or SCEVAN instead), not direct malignant-state classification accuracy evaluation
- Input is spatial transcriptomics spot data without the spatially trained Cancer-Finder models — single-cell pre-trained checkpoints target scRNA-seq malignant cell annotation

## Inputs

- Pre-trained Cancer-Finder model checkpoint (checkpoints/sc_pretrain_article.pkl)
- TISCH 13-tissue internal-validation single-cell expression matrix (328,230 cells; 20% validation fold of a 4:1 train/validation split)
- TISCH per-cell malignancy annotations (ground-truth malignant/non-malignant labels)

## Outputs

- Per-tissue accuracy values ((TP + TN) / N) for each of the 13 TISCH tissues
- Cell-count-weighted average accuracy across the 13 tissues (target: 95.16%)
- Per-cell malignant/non-malignant predictions from infer.py with softmax threshold 0.5

## How to apply

Load the pre-trained Cancer-Finder checkpoint (checkpoints/sc_pretrain_article.pkl) and the TISCH 13-tissue internal-validation expression matrix (328,230 cells; the 20% validation fold of the 4:1 train/validation split). Normalize with SCANPY's 'pp.normalize_total' followed by 'log1p' (log-transformed global scaling). Run inference via infer.py (or the 'infering' function on the AnnData object) with the default softmax threshold of 0.5 for 2 classes (malignant/non-malignant) to assign per-cell predictions. Compute per-tissue accuracy as (TP + TN) / N against the TISCH cell malignancy annotations, then aggregate the 13 per-tissue accuracies into a cell-count-weighted average. Correct application reproduces the reported 95.16% weighted average, with 11 of 13 tissues exceeding 90% accuracy.

## Related tools

- **Cancer-Finder** (Pre-trained domain generalization model whose checkpoint (sc_pretrain_article.pkl) is used with infer.py to predict malignant/non-malignant state per cell) — https://github.com/Patchouli-M/SequencingCancerFinder
- **SCANPY** (Preprocessing of the TISCH validation expression matrix using pp.normalize_total and log1p (log-transformed global scaling) before inference)
- **PyTorch** (Deep-learning framework in which the Cancer-Finder checkpoint is loaded and inference is executed (torch 1.13.1, CUDA 11.6 per README))

## Examples

```
python infer.py --ckp checkpoints/sc_pretrain_article.pkl --input tisch_13tissue_validation.h5ad --threshold 0.5
```

## Evaluation signals

- Cell-count-weighted average accuracy across the 13 TISCH tissues reproduces the reported 95.16%
- Per-tissue accuracy exceeds 90% in 11 of 13 tissues
- Each per-tissue accuracy is computed as (TP + TN) / N against TISCH malignancy annotations, not a proxy metric
- Input matrix contains 328,230 internal-validation cells (the 20% fold of the 4:1 split) before inference
- Predictions are generated with the default softmax threshold of 0.5 for c = 2 classes; changing the threshold invalidates comparison to the reported figure

## Limitations

- Cancer-Finder is not recommended for hematologic tumor data
- Cancer-Finder is robust only to a certain percentage of mislabeling in the training set
- Prediction errors on low-quality cells may be resolvable only by increasing sequencing depth

## Evidence

- [abstract] Average accuracy across 13 TISCH tissues: "rapidly identify malignant cells in single-cell data with an average accuracy of 95.16%"
- [intro] Accuracy and generalization scope: "We collected 74 human tumor microenvironment datasets from Tumor Immune Single Cell Hub (TISCH)27 as training sets"
- [discussion] Hematologic tumor limitation: "We do not therefore recommend Cancer-Finder for hematologic tumor data"
- [readme] Environment prerequisites for running inference: "System: Ubuntu 18.04
python: 3.9.16
CUDA: 11.6
torch: 1.13.1"
