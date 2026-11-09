---
name: 3d-tissue-alignment-benchmarking
description: Use when consecutive 2D ST slices contain defects (rotation, flip, translation,
  cropping) and you need to reconstruct or evaluate a stacked 3D tissue architecture
  against a ground-truth reference.
when_to_use_negative:
- The task requires nonlinear alignment of ST slices — Scube does not support nonlinear
  alignment, unlike STAligner and PASTE
- The dataset is a seq-based ST technology (e.g., 10X Visium) without a prior cell
  type deconvolution step, since Splane requires cell type composition as input
- Slices lack substantial defects and only a simple alignment is needed; also adding
  a new slice requires retraining the entire deep learning model from scratch, hurting
  efficiency for incremental workflows
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: SPACEL
  role: Parent toolkit providing Splane and Scube for spatial domain identification
    and 3D slice alignment benchmarking
  repo: github.com/QuKunLab/SPACEL
- name: Splane
  role: GCN with adversarial learning that identifies uniform spatial domains across
    degraded slices, feeding Scube's alignment
  repo: github.com/QuKunLab/SPACEL
- name: Scube
  role: Globally transforms spatial coordinate systems of consecutive slices and stacks
    them for 3D architecture construction
  repo: github.com/QuKunLab/SPACEL
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:31e5b209c35e3269d78f566f13688752279fd7071c9f5fe273452e9fd12d9a74
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SPACEL
  - Scube
  - Splane
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# 3d-tissue-alignment-benchmarking

## Summary

Benchmark 3D alignment of consecutively perturbed ST slices (rotation, flip, crop) by stacking them with SPACEL's Scube and comparing against STAligner and PASTE using per-slice SSIM and Pearson correlation against the ground-truth 3D stack.

## When to use

Use when consecutive 2D ST slices contain defects (rotation, flip, translation, cropping) and you need to reconstruct or evaluate a stacked 3D tissue architecture against a ground-truth reference.

## When NOT to use

- The task requires nonlinear alignment of ST slices — Scube does not support nonlinear alignment, unlike STAligner and PASTE
- The dataset is a seq-based ST technology (e.g., 10X Visium) without a prior cell type deconvolution step, since Splane requires cell type composition as input
- Slices lack substantial defects and only a simple alignment is needed; also adding a new slice requires retraining the entire deep learning model from scratch, hurting efficiency for incremental workflows

## Inputs

- Consecutive ST slices from a 3D tissue stack (e.g., STARmap mouse brain dataset, ten consecutive 10 um slices) in AnnData format
- Cell type composition per spot (from Spoint deconvolution for seq-based ST data such as 10X Visium)
- Spatial coordinates of spots/cells per slice
- Ground-truth 3D tissue stack for SSIM/PCC comparison
- Perturbation parameters: rotation, flip, translation, and crop ratios 0 to 0.25

## Outputs

- Stacked 3D architecture of the tissue with globally aligned slice coordinate systems
- alignment_metrics.csv with per-slice SSIM and Pearson correlation (PCC) for Scube, STAligner, and PASTE

## How to apply

Load the ST 3D tissue stack (e.g., the simulated mouse brain STARmap dataset divided into ten consecutive 10 um slices) and simulate defects by applying random rotation, flip, and translation, then cropping slice edges at ratios from 0 to 0.25. Identify spatial domains across the degraded slices with Splane's graph convolutional network (Splane.base_model.SplaneModel.identify_spatial_domain), which takes cell type composition and spatial coordinates as input; spatial domains from Splane set the foundation for Scube's alignment. Globally transform the coordinate systems of consecutive slices with Scube.alignment.align and stack them into a 3D architecture. Run STAligner and PASTE on the same degraded slices as baselines (note PASTE is hyperparameter-sensitive at crop ratios >= 0.25), then compute SSIM and PCC per slice for each method against the ground-truth 3D stack and write results to alignment_metrics.csv. Correct application is judged by Scube reaching SSIM/PCC of roughly 0.96/0.97 at crop ratio 0.25, beating STAligner (~0.76/0.77) and PASTE (~0.72/0.75).

## Related tools

- **SPACEL** (Parent toolkit providing Splane and Scube for spatial domain identification and 3D slice alignment benchmarking) — github.com/QuKunLab/SPACEL
- **Splane** (GCN with adversarial learning that identifies uniform spatial domains across degraded slices, feeding Scube's alignment) — github.com/QuKunLab/SPACEL
- **Scube** (Globally transforms spatial coordinate systems of consecutive slices and stacks them for 3D architecture construction) — github.com/QuKunLab/SPACEL

## Examples

```
from SPACEL.Splane import SplaneModel; SplaneModel.identify_spatial_domain(adatas); from SPACEL.Scube import align; align(adatas)
```

## Evaluation signals

- At crop ratio 0.25, Scube alignment achieves average SSIM/PCC of approximately 0.96/0.97 on transformed slices
- Scube SSIM/PCC outperform STAligner (approximately 0.76/0.77) and PASTE (approximately 0.72/0.75) at crop ratio 0.25
- Per-slice SSIM and PCC values are written for every slice to alignment_metrics.csv
- Spatial domains identified by Splane are coherent across slices and provide a valid foundation for Scube alignment
- Baseline methods (STAligner, PASTE) are run on the identical degraded slices with optimized parameters for a fair comparison

## Limitations

- The current version of Scube does not support nonlinear alignment as well as STAligner and PASTE; nonlinear support is deferred to a future version
- PASTE's alignment results are sensitive to hyperparameters in datasets with significant defects (e.g., slices with crop ratios >= 0.25)
- STAligner does not utilize complete information on each spot of the whole slice, limiting its ability to balance global structure similarity and alignment accuracy
- When a new slice is added, the entire deep learning model needs to be retrained from scratch, affecting computational efficiency for large-scale datasets

## Evidence

- [methods] Scube achieved average SSIM/PCC of 0.96/0.97 for transformed slices: [evidence span withheld — anchor status: not_found]
- [abstract] Splane employs a graph convolutional network: "Splane employs a graph convolutional network approach and an adversarial learning algorithm to identify spatial domains"
- [abstract] Scube automatically transforms the spatial coordinate systems: "Scube automatically transforms the spatial coordinate systems of consecutive slices and stacks them together to construct a 3D architecture of the tissue"
- [discussion] Scube adopts a global optimization strategy: "Scube adopts a global optimization strategy for 3D alignment, taking into account the correspondences between all spots in adjacent slices"
- [discussion] PASTE sensitive to hyperparameters: "This design makes PASTE’s alignment results sensitive to hyperparameters in datasets with significant defects (e.g., slices with crop ratios ≥ 0.25)"
