---
name: class-balanced-cell-downsampling
description: Use when assembling a training corpus of annotated single-cell data from
  multiple tumor datasets (e.g., TISCH) where malignant and non-malignant cell counts
  differ per tissue and a balanced classifier must be trained.
when_to_use_negative:
- The input data are hematologic tumor datasets, for which Cancer-Finder (and hence
  its balanced training protocol) is explicitly not recommended
- The analysis target is a single-dataset classifier (e.g., CopyKAT CNV inference)
  where per-domain class balance across tissues is irrelevant
- Labels are unavailable or unreliable for malignant vs non-malignant cells, since
  1:1 downsampling requires trusted class labels per tissue
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cancer-Finder
  role: The domain-generalization classifier whose training set is class-balanced
    by this downsampling step
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
- name: Seurat
  role: Log-transformed global-scaling normalization ('NormalizeData' + log1p) applied
    before per-tissue downsampling
- name: SCANPY
  role: Single-cell preprocessing ecosystem supporting the balanced per-tissue subsampling
    workflow
- name: PyTorch
  role: Framework in which the balanced training corpus is used to optimize the variance
    + average risk loss
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:64119c1f3c097068783c86e1bb07e1fc1f8533399d2ad8925756dc5d03de789e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - Cancer-Finder
  - PyTorch
  - SCANPY
  - Seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# Class-balanced cell downsampling

## Summary

Equalizes malignant and non-malignant cell counts 1:1 per tissue before training Cancer-Finder, so that the domain-generalization loss is not skewed by class imbalance across the 74 TISCH tumor microenvironment datasets.

## When to use

Use when assembling a training corpus of annotated single-cell data from multiple tumor datasets (e.g., TISCH) where malignant and non-malignant cell counts differ per tissue and a balanced classifier must be trained.

## When NOT to use

- The input data are hematologic tumor datasets, for which Cancer-Finder (and hence its balanced training protocol) is explicitly not recommended
- The analysis target is a single-dataset classifier (e.g., CopyKAT CNV inference) where per-domain class balance across tissues is irrelevant
- Labels are unavailable or unreliable for malignant vs non-malignant cells, since 1:1 downsampling requires trusted class labels per tissue

## Inputs

- TISCH tumor microenvironment scRNA-seq datasets (74 datasets, 14 tissue categories with malignant cells)
- Cell-level malignant/non-malignant labels per dataset
- Gene expression matrices for Seurat/SCANPY normalization prior to subsampling

## Outputs

- Class-balanced (1:1 malignant:non-malignant) per-tissue cell sets
- 80/20 training and internal validation split totalling 328,230 cells from 13 tissues
- Balanced training corpus consumed by the domain-generalization training loop (R_var + R_avg, β = 1, 50–70 epochs)

## How to apply

Load the 74 TISCH tumor microenvironment datasets covering 14 tissue categories with malignant cells, and normalize each with log-transformed global-scaling normalization (Seurat 'NormalizeData' followed by log1p). For each tissue independently, downsample malignant and non-malignant cells to a 1:1 ratio so no tissue's class frequency dominates the cross-entropy risks R_e computed per domain. After downsampling, split cells 80/20 into training and internal validation sets, yielding 328,230 cells from 13 tissues used for the 5-fold leave-cells-out cross-validation of Cancer-Finder. The rationale is that balanced per-domain classes make the variance-plus-average risk loss L(θ) = β·R_var(θ) + R_avg(θ) (β = 1) optimize over comparable malignant/non-malignant evidence from every tissue domain.

## Related tools

- **Cancer-Finder** (The domain-generalization classifier whose training set is class-balanced by this downsampling step) — https://github.com/Patchouli-M/SequencingCancerFinder
- **Seurat** (Log-transformed global-scaling normalization ('NormalizeData' + log1p) applied before per-tissue downsampling)
- **SCANPY** (Single-cell preprocessing ecosystem supporting the balanced per-tissue subsampling workflow)
- **PyTorch** (Framework in which the balanced training corpus is used to optimize the variance + average risk loss)

## Evaluation signals

- After subsampling, the malignant:non-malignant cell ratio is exactly 1:1 within every retained tissue domain
- The balanced and split corpus contains 328,230 cells drawn from 13 tissues (one of the 14 tissue categories dropped during assembly)
- The 80/20 training/internal-validation split is preserved per tissue, not just globally
- Downstream training reaches an average internal-validation accuracy of approximately 95.16% across the 13 tissues
- Variance risk R_var and average risk R_avg curves over 50–70 epochs behave stably under 5-fold leave-cells-out cross-validation

## Limitations

- Balancing by downsampling discards cells, so tissues with few cells of one class may contribute little usable training data
- The protocol depends on correct malignant/non-malignant labels; the paper notes Cancer-Finder is robust only to a certain percentage of mislabeling in the training set
- Prediction errors tied to low sequencing depth may persist regardless of class balance

## Evidence

- [methods] apply log-transformed global-scaling normalization (Seurat 'NormalizeData' followed by log1p), downsample malignant and non-malignant cells 1:1 per tissue: [evidence span withheld — anchor status: not_found]
- [methods] split cells 80/20 into training and internal validation sets, yielding 328,230 cells from 13 tissues: [evidence span withheld — anchor status: not_found]
- [intro] We collected 74 human tumor microenvironment datasets from Tumor Immune Single Cell Hub (TISCH)27 as training sets: "We collected 74 human tumor microenvironment datasets from Tumor Immune Single Cell Hub (TISCH)27 as training sets"
- [discussion] it is robust to a certain percentage of mislabeling in training set (Supplementary Fig. 11): "it is robust to a certain percentage of mislabeling in training set (Supplementary Fig. 11)"
