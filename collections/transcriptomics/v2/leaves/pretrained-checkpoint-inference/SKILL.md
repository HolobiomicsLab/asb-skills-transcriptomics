---
name: pretrained-checkpoint-inference
description: Use when you have an expression matrix (scRNA-seq or 10x Visium spots),
  a Cancer-Finder pre-trained checkpoint (e.g., checkpoints/sc_pretrain_article.pkl),
  and need reference-free malignant-cell/spot predictions.
when_to_use_negative:
- Input is hematologic tumor data — the authors explicitly do not recommend Cancer-Finder
  for hematologic tumor data
- Analysis requires modeling spatial context among spots — Cancer-Finder overlooks
  spatial relationships among spots in ST data
- The task is deconvolving spot cell composition rather than malignancy labeling —
  use methods like cell2location or CARD instead
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cancer-Finder
  role: Pre-trained domain generalization model providing checkpoints (sc_pretrain_article.pkl,
    ST checkpoint) and infer.py / 'infering' function for malignant cell/spot classification
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
- name: SCANPY
  role: Preprocessing of count matrices with 'pp.normalize_total' and 'log1p' before
    inference
- name: PyTorch
  role: Runtime for loading the .pkl checkpoint and executing the trained classification
    network
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:9012b5fb6e2af5f1e951b67ac9ff279ca6ecfe75e2629e535c9724ca511e7911
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - Cancer-Finder
  - PyTorch
  - SCANPY
  - Seurat
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# pretrained-checkpoint-inference

## Summary

Apply Cancer-Finder's pre-trained PyTorch checkpoints to log-normalized single-cell or spatial transcriptomics expression matrices via infer.py, using a softmax threshold of 0.5 to assign malignant/non-malignant labels per cell or spot without retraining.

## When to use

Use when you have an expression matrix (scRNA-seq or 10x Visium spots), a Cancer-Finder pre-trained checkpoint (e.g., checkpoints/sc_pretrain_article.pkl), and need reference-free malignant-cell/spot predictions.

## When NOT to use

- Input is hematologic tumor data — the authors explicitly do not recommend Cancer-Finder for hematologic tumor data
- Analysis requires modeling spatial context among spots — Cancer-Finder overlooks spatial relationships among spots in ST data
- The task is deconvolving spot cell composition rather than malignancy labeling — use methods like cell2location or CARD instead

## Inputs

- Pre-trained Cancer-Finder PyTorch checkpoint (checkpoints/sc_pretrain_article.pkl for scRNA-seq; ST checkpoint sized for 5000 spatial input genes)
- Gene-by-cell or gene-by-spot raw UMI count matrix (AnnData/h5ad; e.g., TISCH 13-tissue internal-validation set of 328,230 cells, or 14 10x Visium spot matrices)
- Reference malignant/non-malignant labels (TISCH cell malignancy annotations or pathologist spot annotations) for accuracy evaluation

## Outputs

- Per-cell or per-spot binary malignant/non-malignant predictions with spot-level softmax scores
- Per-tissue or per-slide accuracy values computed as (TP + TN) / N
- Cell-count-weighted average accuracy across tissues (expected ~95.16% on TISCH internal validation)

## How to apply

Load the appropriate pre-trained checkpoint (checkpoints/sc_pretrain_article.pkl for the single-cell model trained on 74 TISCH datasets; the ST checkpoint for spatial data) with infer.py via the --ckp parameter. Preprocess the count matrix with log-transformed global-scaling normalization using SCANPY's 'pp.normalize_total' followed by 'log1p' (equivalently Seurat's NormalizeData). Run inference with infer.py or the 'infering' function on the AnnData object, using the default softmax threshold of 0.5 for c = 2 classes to assign malignant/non-malignant predictions per cell or spot. Compare predictions against reference annotations and compute accuracy per tissue/slide as (TP + TN) / N. Rationale: the domain generalization model learns from multiple dataset distributions, so no reference or CNV inference is required at inference time.

## Related tools

- **Cancer-Finder** (Pre-trained domain generalization model providing checkpoints (sc_pretrain_article.pkl, ST checkpoint) and infer.py / 'infering' function for malignant cell/spot classification) — https://github.com/Patchouli-M/SequencingCancerFinder
- **SCANPY** (Preprocessing of count matrices with 'pp.normalize_total' and 'log1p' before inference)
- **PyTorch** (Runtime for loading the .pkl checkpoint and executing the trained classification network)

## Examples

```
python infer.py --ckp checkpoints/sc_pretrain_article.pkl --input internal_validation.h5ad
```

## Evaluation signals

- Weighted average accuracy across the 13 TISCH internal-validation tissues reproduces the reported 95.16%, with 11 of 13 tissues exceeding 90% accuracy
- Per-slide accuracy on Visium slides of spatially trained cancer types falls within the reported 82.00–97.37% range (Fig. 4b,c)
- Predicted per-cell malignant labels match TISCH annotations at (TP + TN) / N accuracy computed per tissue before aggregation
- Spot-level softmax scores binarize consistently at the default threshold of 0.5 (c = 2 classes), yielding exactly two prediction classes
- All inference inputs are log-transformed global-scaling normalized (SCANPY normalize_total + log1p) before passing to the model

## Limitations

- Cancer-Finder is not recommended for hematologic tumor data
- It overlooks the spatial relationships among spots in ST data, a factor that could otherwise enhance efficacy
- The model is robust only to a certain percentage of mislabeling in the training set (Supplementary Fig. 11)
- Prediction errors linked to low signal may require increased sequencing depth to resolve (Supplementary Fig. 12)

## Evidence

- [abstract] Domain generalization checkpoint accuracy: "we present Cancer-Finder, a domain generalization-based deep-learning algorithm that can rapidly identify malignant cells in single-cell data with an average accuracy of 95.16%"
- [abstract] Spatial extension: "by replacing the single-cell training data with spatial transcriptomic datasets, Cancer-Finder can accurately identify malignant spots on spatial slides"
- [methods] Inference procedure and threshold: [evidence span withheld — anchor status: not_found]
- [methods] Checkpoint parameter: [evidence span withheld — anchor status: not_found]
- [readme] Environment prerequisites: "System: Ubuntu 18.04 python: 3.9.16 CUDA: 11.6 torch: 1.13.1"
