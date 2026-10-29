---
name: visium-count-matrix-normalization
description: Use when raw 10x Visium spot count matrices must be normalized prior
  to Cancer-Finder infer.py inference with an ST checkpoint, or when reproducing the
  paper's 82.00–97.37% per-slide spot-annotation accuracies.
when_to_use_negative:
- Input is scRNA-seq single-cell data rather than spatial spot-level Visium matrices
  — use the scRNA-seq checkpoint/preprocessing instead
- Analyzing hematologic tumor data, where Cancer-Finder is not recommended
- Non-Visium ST platforms whose softmax threshold and normalization assumptions differ
  from the 10x Visium defaults
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: Cancer-Finder
  role: Downstream consumer of the normalized matrices; runs malignant-spot inference
    via infer.py with the ST checkpoint and softmax threshold 0.5
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
- name: SCANPY
  role: 'Python preprocessing: pp.normalize_total + pp.log1p for log-transformed global-scaling
    normalization of spot counts'
- name: Seurat
  role: 'R alternative preprocessing: NormalizeData performs the equivalent log-transformed
    global-scaling normalization'
- name: PyTorch
  role: Framework underlying the Cancer-Finder checkpoint used for spot-level inference
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:c47b10a7f34fbdda143bea25d79a1e284b556a03e8130133c5f6dc2e62150c72
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - Cancer-Finder
  - PyTorch
  - SCANPY
  - Seurat
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# visium-count-matrix-normalization

## Summary

Preprocess raw 10x Visium spot-level count matrices with log-transformed global-scaling normalization (SCANPY pp.normalize_total + log1p, or Seurat NormalizeData) so they are compatible with Cancer-Finder's malignant-spot inference checkpoint.

## When to use

Use when raw 10x Visium spot count matrices must be normalized prior to Cancer-Finder infer.py inference with an ST checkpoint, or when reproducing the paper's 82.00–97.37% per-slide spot-annotation accuracies.

## When NOT to use

- Input is scRNA-seq single-cell data rather than spatial spot-level Visium matrices — use the scRNA-seq checkpoint/preprocessing instead
- Analyzing hematologic tumor data, where Cancer-Finder is not recommended
- Non-Visium ST platforms whose softmax threshold and normalization assumptions differ from the 10x Visium defaults

## Inputs

- 10x Visium spot-level gene expression count matrices (14 slides with pathologist-derived malignant=1 / non-malignant=0 spot labels)
- Cancer-Finder spatial-transcriptomics (ST) pre-trained checkpoint

## Outputs

- Normalized (library-size-scaled and log1p-transformed) spot expression matrices per slide
- Spot-level softmax scores and binarized malignant/non-malignant predictions from infer.py
- Per-slide accuracy values computed as (TP+TN)/N against pathologist annotations

## How to apply

Load each of the 10x Visium spot expression matrices (malignant spots labeled 1, non-malignant 0) and apply log-transformed global-scaling normalization: in SCANPY run sc.pp.normalize_total followed by sc.pp.log1p, or in R use Seurat's NormalizeData. This log-transformed global scaling matches the preprocessing Cancer-Finder expects and is applied identically per slide before inference. Feed each normalized matrix to Cancer-Finder's infer.py with the spatial-transcriptomics pre-trained checkpoint, using the default softmax threshold of 0.5 for 10x Visium data to binarize spot-level softmax scores into malignant (1) vs non-malignant (0) predictions. Rationale: consistent normalization aligns spot expression distributions with the ST training data, enabling the domain-generalization model to produce pathologist-consistent per-slide accuracies in the reported 82.00–97.37% range (Fig. 4b,c).

## Related tools

- **Cancer-Finder** (Downstream consumer of the normalized matrices; runs malignant-spot inference via infer.py with the ST checkpoint and softmax threshold 0.5) — https://github.com/Patchouli-M/SequencingCancerFinder
- **SCANPY** (Python preprocessing: pp.normalize_total + pp.log1p for log-transformed global-scaling normalization of spot counts)
- **Seurat** (R alternative preprocessing: NormalizeData performs the equivalent log-transformed global-scaling normalization)
- **PyTorch** (Framework underlying the Cancer-Finder checkpoint used for spot-level inference)

## Examples

```
import scanpy as sc; adata = sc.read_visium('slide1/outs'); sc.pp.normalize_total(adata); sc.pp.log1p(adata)
```

## Evaluation signals

- Each per-slide accuracy for cancer types included in ST training falls within the reported 82.00–97.37% interval, matching Fig. 4b,c
- Every slide's count matrix was scaled to a common target (normalize_total) before log1p — no raw counts reach infer.py
- Binarized predictions use exactly the default softmax threshold of 0.5 for 10x Visium data
- Accuracy per slide reproduces as (TP+TN)/N against the pathologist malignant=1 / non-malignant=0 spot labels
- Output files contain both raw softmax scores and the thresholded binary predictions for every spot

## Limitations

- Cancer-Finder overlooks spatial relationships among spots in ST data, a factor with the potential to enhance overall efficacy
- Prediction errors may be resolvable only by increasing sequencing depth (Supplementary Fig. 12)
- The model is robust only to a certain percentage of mislabeling in the training set (Supplementary Fig. 11)

## Evidence

- [other] Preprocess each count matrix with log-transformed global-scaling normalization: [evidence span withheld — anchor status: not_found]
- [discussion] ST accuracy range on trained tissues: "Cancer-Finder displays a high level of accuracy (82.00-97.37%) on ST data of trained tissues"
- [intro] Spatial extension objective: "we rapidly extend Cancer-Finder to annotate malignant spots in ST data and demonstrate its high prediction accuracy after training with a small training set"
- [abstract] Spatial-slide identification claim: "by replacing the single-cell training data with spatial transcriptomic datasets, Cancer-Finder can accurately identify malignant spots on spatial slides"
- [discussion] Spatial relationships limitation: "Cancer-Finder overlooks the spatial relationships among spots in ST data, a factor with the potential to enhance its overall efficacy"
