---
name: domain-generalization-risk-extrapolation-training
description: Use when training a malignant-cell classifier on multi-tissue scRNA-seq
  data where the model must generalize to unseen tissue domains — i.e., when optimizing
  the combined variance-plus-average risk objective over tissue domains is needed.
when_to_use_negative:
- Hematologic tumor data — the article explicitly does not recommend Cancer-Finder
  for hematologic tumors.
- Spatial transcriptomics spot annotation with a model trained on single-cell data
  — ST spot calling requires retraining on spatial transcriptomic training datasets,
  and the model overlooks spatial relationships among spots.
- Tasks requiring clonal structure or CNV inference (e.g., CopyKAT-style subclonal
  copy-number profiles) — this skill only performs binary malignant/non-malignant
  classification.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
tools:
- name: Cancer-Finder
  role: The domain-generalization MLP being trained; official implementation in the
    SequencingCancerFinder repository
  repo: Patchouli-M/SequencingCancerFinder
- name: PyTorch
  role: Framework used to build the network (torch 1.13.1, CUDA 11.6) and implement
    the per-domain risk-extrapolation loss
- name: Seurat
  role: Provides the log-transformed global-scaling normalization (NormalizeData +
    log1p) applied to the TISCH training sets
- name: SCANPY
  role: Single-cell preprocessing environment supporting the HVG selection (4572 features)
    that feeds the model
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:225d0441b8e59f5501fa9ffdd14d8c832e1b35694ea04ba65bd11face3596887
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - Cancer-Finder
  - PyTorch
  - SCANPY
  - Seurat
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# domain-generalization-risk-extrapolation-training

## Summary

Trains Cancer-Finder's multi-layer perceptron to generalize across tissues by optimizing the risk-extrapolation objective L(θ) = β·R_var(θ) + R_avg(θ), which penalizes variance of per-domain (per-tissue) cross-entropy risks alongside their average. This is the mechanism that lets a classifier trained on 74 TISCH tumor microenvironment datasets achieve 95.16% average accuracy on held-out malignant/non-malignant cell calls.

## When to use

Use when training a malignant-cell classifier on multi-tissue scRNA-seq data where the model must generalize to unseen tissue domains — i.e., when optimizing the combined variance-plus-average risk objective over tissue domains is needed.

## When NOT to use

- Hematologic tumor data — the article explicitly does not recommend Cancer-Finder for hematologic tumors.
- Spatial transcriptomics spot annotation with a model trained on single-cell data — ST spot calling requires retraining on spatial transcriptomic training datasets, and the model overlooks spatial relationships among spots.
- Tasks requiring clonal structure or CNV inference (e.g., CopyKAT-style subclonal copy-number profiles) — this skill only performs binary malignant/non-malignant classification.

## Inputs

- 74 TISCH tumor microenvironment scRNA-seq datasets (14 tissue categories with malignant cells), log-transformed global-scaling normalized (Seurat NormalizeData + log1p)
- 1:1 downsampled malignant/non-malignant cells per tissue, 80/20 train/internal-validation split (328,230 cells, 13 tissues)
- Highly variable gene matrix: 4572 gene features per cell (scRNA-seq)
- Normalized breast cancer scRNA-seq data from the original CopyKAT study (extra-domain validation)

## Outputs

- PyTorch checkpoint of the Cancer-Finder MLP (4572 -> 512 -> 2 neurons, dropout 0.5) selected at maximal extra-domain breast cancer accuracy
- Softmax malignant/non-malignant probabilities per cell, thresholded at 0.5
- Training curves for per-domain cross-entropy risk R_e, variance risk R_var, and average risk R_avg over 50-70 epochs

## How to apply

Prepare the training matrix by loading the 74 TISCH tumor microenvironment datasets, applying log-transformed global-scaling normalization (Seurat NormalizeData followed by log1p), downsampling malignant and non-malignant cells 1:1 per tissue, splitting 80/20 into training/internal-validation sets (328,230 cells from 13 tissues), and selecting highly variable genes ranked by variance as inputs (4572 features for scRNA-seq). Build the network in PyTorch: two fully connected layers without activation functions (4572 -> 512-neuron bottleneck) separated by a dropout layer with probability 0.5, followed by a 2-neuron classification layer with softmax output thresholded at 0.5 for malignant calls. For each of the m tissue domains compute the cross-entropy risk R_e, then optimize L(θ) = β·R_var(θ) + R_avg(θ) with β = 1 over 50-70 epochs under 5-fold leave-cells-out cross-validation. After each epoch, evaluate on the internal validation set and on normalized breast cancer data from the original CopyKAT study as extra-domain validation, retaining only the checkpoint whose extra-domain accuracy is maximal. Rationale: adding the inter-domain risk variance term forces features that work uniformly across tissues rather than fitting per-tissue idiosyncrasies, which is what enables domain generalization to unseen cancer types.

## Related tools

- **Cancer-Finder** (The domain-generalization MLP being trained; official implementation in the SequencingCancerFinder repository) — Patchouli-M/SequencingCancerFinder
- **PyTorch** (Framework used to build the network (torch 1.13.1, CUDA 11.6) and implement the per-domain risk-extrapolation loss)
- **Seurat** (Provides the log-transformed global-scaling normalization (NormalizeData + log1p) applied to the TISCH training sets)
- **SCANPY** (Single-cell preprocessing environment supporting the HVG selection (4572 features) that feeds the model)

## Examples

```
conda create -n scf python=3.9.16
```

## Evaluation signals

- Average internal-validation accuracy across the 13 tissues reaches approximately 95.16%, the value reported for the article's pre-trained models.
- Variance risk R_var(θ) and average risk R_avg(θ) both decrease monotonically over the 50-70 epochs at β = 1.
- The retained checkpoint is the one with maximal extra-domain accuracy on the normalized CopyKAT breast cancer dataset, not the maximal internal-validation checkpoint.
- Training data are 1:1 balanced (malignant vs non-malignant) per tissue and split 80/20 into training/internal-validation sets.
- Model input dimensionality matches the 4572 highly variable gene features, and the classification layer has exactly 2 neurons with softmax outputs thresholded at 0.5.

## Limitations

- Cancer-Finder is not recommended for hematologic tumor data.
- The model is robust only to a certain percentage of mislabeling in the training set.
- In ST applications, Cancer-Finder overlooks the spatial relationships among spots, a factor that could otherwise enhance efficacy.
- Prediction errors may be resolvable only by increasing sequencing depth.

## Evidence

- [methods] Domain generalization objective: [evidence span withheld — anchor status: not_found]
- [methods] Training data and normalization: [evidence span withheld — anchor status: not_found]
- [abstract] Reported accuracy: "rapidly identify malignant cells in single-cell data with an average accuracy of 95.16%"
- [discussion] Hematologic exclusion: "We do not therefore recommend Cancer-Finder for hematologic tumor data"
- [readme] Environment prerequisites: "System: Ubuntu 18.04
python: 3.9.16
CUDA: 11.6
torch: 1.13.1"
