---
name: softmax-thresholded-binary-classification
description: Use when running Cancer-Finder inference on 10x Visium spot matrices
  (or scRNA-seq cells) and you need to convert raw softmax probabilities into binary
  malignant/non-malignant labels for accuracy evaluation.
when_to_use_negative:
- Hematologic tumor data — the authors explicitly do not recommend Cancer-Finder for
  hematologic tumors
- Data that has not been normalized with log-transformed global scaling — raw UMI
  counts fed to the classifier violate the expected input distribution
- Analyses requiring spatial-neighborhood context — Cancer-Finder overlooks spatial
  relationships among spots, so tasks depending on spot adjacency are out of scope
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: Cancer-Finder
  role: Deep-learning classifier whose softmax outputs are thresholded to produce
    binary malignant/non-malignant calls via infer.py
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
- name: SCANPY
  role: 'Preprocessing: pp.normalize_total + log1p for log-transformed global-scaling
    normalization before inference'
- name: Seurat
  role: Alternative preprocessing via NormalizeData for the same global-scaling normalization
- name: PyTorch
  role: Framework underlying the Cancer-Finder checkpoint used for inference
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:d47fac5774cc03dc475e87305ab35d8eb16f93323d52e21e0fa6fb0c10fc8fdf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - Cancer-Finder
  - PyTorch
  - SCANPY
  - Seurat
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# softmax-thresholded-binary-classification

## Summary

Binarize Cancer-Finder's per-spot/per-cell malignant probabilities by applying a softmax score threshold (default 0.5 for 10x Visium ST data) via infer.py, converting continuous model output into malignant (1) vs non-malignant (0) calls.

## When to use

Use when running Cancer-Finder inference on 10x Visium spot matrices (or scRNA-seq cells) and you need to convert raw softmax probabilities into binary malignant/non-malignant labels for accuracy evaluation.

## When NOT to use

- Hematologic tumor data — the authors explicitly do not recommend Cancer-Finder for hematologic tumors
- Data that has not been normalized with log-transformed global scaling — raw UMI counts fed to the classifier violate the expected input distribution
- Analyses requiring spatial-neighborhood context — Cancer-Finder overlooks spatial relationships among spots, so tasks depending on spot adjacency are out of scope

## Inputs

- Preprocessed spot/cell expression matrices (10x Visium count matrices normalized via SCANPY pp.normalize_total + log1p or Seurat NormalizeData)
- Cancer-Finder pre-trained checkpoint (ST-trained for spatial transcriptomics, scRNA-seq-trained for single-cell)

## Outputs

- Spot-level (or cell-level) softmax probability scores
- Binary malignant (1) / non-malignant (0) predictions after thresholding at 0.5
- Per-slide accuracy values computed as (TP+TN)/N against pathologist annotations

## How to apply

Load the appropriate pre-trained Cancer-Finder checkpoint (ST-trained checkpoint for spatial data) and feed in log-transformed, global-scaling-normalized count matrices (SCANPY pp.normalize_total + log1p, or Seurat NormalizeData). Run infer.py, which outputs per-spot/per-cell softmax scores; apply the default threshold of 0.5 for 10x Visium data so scores >= 0.5 are labeled malignant (1) and < 0.5 non-malignant (0). Write both the continuous softmax scores and the binarized predictions so downstream evaluation can adjust the cutoff. Validate by comparing binarized predictions to pathologist-derived spot labels and computing per-slide accuracy as (TP+TN)/N; for Visium slides of trained cancers, per-slide accuracies of 82.00-97.37% (Fig. 4b,c) indicate correct application.

## Related tools

- **Cancer-Finder** (Deep-learning classifier whose softmax outputs are thresholded to produce binary malignant/non-malignant calls via infer.py) — https://github.com/Patchouli-M/SequencingCancerFinder
- **SCANPY** (Preprocessing: pp.normalize_total + log1p for log-transformed global-scaling normalization before inference)
- **Seurat** (Alternative preprocessing via NormalizeData for the same global-scaling normalization)
- **PyTorch** (Framework underlying the Cancer-Finder checkpoint used for inference)

## Examples

```
python infer.py --checkpoint ST_pretrained.pth --input visium_slide.h5ad --threshold 0.5 --output spot_predictions.csv
```

## Evaluation signals

- Each per-slide accuracy for cancer types included in ST training falls within the reported 82.00-97.37% interval (matching Fig. 4b,c)
- Binarized predictions agree with pathologist-derived labels (malignant=1, non-malignant=0), with accuracy = (TP+TN)/N computed per slide
- Softmax scores are written alongside binary predictions, allowing threshold sensitivity checks around the 0.5 default
- On golden-standard scRNA-seq datasets, accuracy should reach ~98.30%; on silver-standard datasets, similarity ~90.89%

## Limitations

- The 0.5 threshold is the default for 10x Visium data; other platforms or data types may require tuning, so softmax scores should be retained
- Cancer-Finder overlooks the spatial relationships among spots in ST data
- Prediction errors may be resolvable only by increasing sequencing depth (Supplementary Fig. 12)
- The model is robust only to a certain percentage of mislabeling in the training set (Supplementary Fig. 11)
- Not recommended for hematologic tumor data

## Evidence

- [methods] workflow quote (source card): [evidence span withheld — anchor status: not_found]
- [discussion] accuracy range finding: "Cancer-Finder displays a high level of accuracy (82.00-97.37%) on ST data of trained tissues"
- [abstract] spatial extension: "by replacing the single-cell training data with spatial transcriptomic datasets, Cancer-Finder can accurately identify malignant spots on spatial slides"
- [readme] README abstract: "we present Cancer-Finder, a domain generalization-based deep learning algorithm that can rapidly identify malignant cells/spots in single-cell and spatial transcriptomics data"
