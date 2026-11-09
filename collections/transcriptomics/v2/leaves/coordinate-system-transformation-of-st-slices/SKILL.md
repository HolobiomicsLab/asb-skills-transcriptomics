---
name: coordinate-system-transformation-of-st-slices
description: Use when consecutive 2D ST slices (e.g., STARmap, MERFISH, Stereo-seq)
  must be aligned into a 3D stack, especially when slices carry defects such as rotation,
  flipping, translation, or crop ratios up to 0.25.
when_to_use_negative:
- Nonlinear alignment is required — the current version of Scube only performs global
  (non-nonlinear) transformation, unlike STAligner and PASTE
- Input is seq-based ST data (e.g., 10X Visium) without a prior Spoint cell type deconvolution
  step, since Splane/Scube depend on cell type composition input
- Single-slice analysis with no consecutive slices to stack — spatial domain identification
  or deconvolution skills are the appropriate choice instead
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Scube
  role: Performs the global transformation of spatial coordinate systems of consecutive
    slices and 3D stacking via Scube.alignment.align
  repo: github.com/QuKunLab/SPACEL
- name: Splane
  role: Identifies spatial domains (via Splane.base_model.SplaneModel.identify_spatial_domain)
    that serve as the foundation for Scube's slice alignment
  repo: github.com/QuKunLab/SPACEL
- name: Spoint
  role: Deconvolutes cell type composition required by Splane when processing seq-based
    ST datasets such as 10X Visium
  repo: github.com/QuKunLab/SPACEL
- name: STAligner
  role: Baseline comparison method for 3D alignment benchmarking on degraded slices
- name: PASTE
  role: Baseline comparison method for 3D alignment benchmarking on degraded slices
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:5192e13b9d7e34d7c256c0da0969e6c893a8abd5f32b5abc9158a3841100cd4f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PASTE
  - SPACEL
  - STAligner
  - Scube
  - Splane
  - Spoint
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# coordinate-system-transformation-of-st-slices

## Summary

Aligns consecutive spatial transcriptomics slices by globally transforming each slice's spatial coordinate system and stacking them into a 3D tissue architecture using SPACEL's Scube module, robust to rotation, flip, translation, and cropping defects.

## When to use

Use when consecutive 2D ST slices (e.g., STARmap, MERFISH, Stereo-seq) must be aligned into a 3D stack, especially when slices carry defects such as rotation, flipping, translation, or crop ratios up to 0.25.

## When NOT to use

- Nonlinear alignment is required — the current version of Scube only performs global (non-nonlinear) transformation, unlike STAligner and PASTE
- Input is seq-based ST data (e.g., 10X Visium) without a prior Spoint cell type deconvolution step, since Splane/Scube depend on cell type composition input
- Single-slice analysis with no consecutive slices to stack — spatial domain identification or deconvolution skills are the appropriate choice instead

## Inputs

- Consecutive ST slices with spatial coordinates (e.g., STARmap mouse brain, human brain MERFISH, mouse embryo Stereo-seq)
- Cell type composition per spot (from Spoint deconvolution for seq-based data such as 10X Visium)
- Spatial domain labels identified by Splane

## Outputs

- Globally transformed spatial coordinate systems per slice
- Stacked 3D architecture of the tissue
- Per-slice alignment quality metrics (SSIM, PCC) against a ground-truth stack, e.g., alignment_metrics.csv

## How to apply

First identify spatial domains across slices with Splane's graph convolutional network (Splane.base_model.SplaneModel.identify_spatial_domain), which takes cell type composition and spatial coordinates as input; these domains set the foundation for accurate alignment. Then call Scube.alignment.align, which globally transforms the spatial coordinate systems of consecutive slices and stacks them to reconstruct the 3D architecture. Scube uses a global optimization strategy that considers correspondences between all spots in adjacent slices, which makes it robust to substantial defects like partial overlap (crop ratios >= 0.25). Benchmark against STAligner and PASTE by computing per-slice SSIM and PCC against the ground-truth 3D stack (e.g., writing results to alignment_metrics.csv).

## Related tools

- **Scube** (Performs the global transformation of spatial coordinate systems of consecutive slices and 3D stacking via Scube.alignment.align) — github.com/QuKunLab/SPACEL
- **Splane** (Identifies spatial domains (via Splane.base_model.SplaneModel.identify_spatial_domain) that serve as the foundation for Scube's slice alignment) — github.com/QuKunLab/SPACEL
- **Spoint** (Deconvolutes cell type composition required by Splane when processing seq-based ST datasets such as 10X Visium) — github.com/QuKunLab/SPACEL
- **STAligner** (Baseline comparison method for 3D alignment benchmarking on degraded slices)
- **PASTE** (Baseline comparison method for 3D alignment benchmarking on degraded slices)

## Examples

```
from SPACEL import Scube; Scube.alignment.align(adata_list)  # globally transform consecutive slices and stack into 3D
```

## Evaluation signals

- At crop ratio 0.25 on simulated STARmap mouse brain slices, Scube alignment should reach average SSIM/PCC of approximately 0.96/0.97 for transformed slices
- Scube SSIM/PCC should exceed STAligner (~0.76/0.77) and PASTE (~0.72/0.75) on the same degraded slices
- Transformed consecutive slices should produce a coherent stacked 3D architecture consistent with the ground-truth tissue stack
- Alignment results should remain stable at substantial defects (crop ratios >= 0.25), where PASTE is reported to be hyperparameter-sensitive

## Limitations

- The current version of Scube does not support nonlinear alignment, unlike STAligner and PASTE; nonlinear support is deferred to a future version
- Adding a new slice requires retraining the entire deep learning model from scratch, limiting computational efficiency for large-scale datasets
- Splane requires cell type composition as input, so seq-based datasets like 10X Visium need a prior deconvolution step (e.g., Spoint)

## Evidence

- [abstract] Scube automatically transforms the spatial coordinate systems of consecutive slices and stacks them together: "Scube automatically transforms the spatial coordinate systems of consecutive slices and stacks them together"
- [discussion] Scube adopts a global optimization strategy for 3D alignment, taking into account the correspondences between all spots in adjacent slices: "Scube adopts a global optimization strategy for 3D alignment, taking into account the correspondences between all spots in adjacent slices"
- [discussion] Spatial domains identified in Splane also set a foundation for Scube to achieve an accurate alignment: "Spatial domains identified in Splane also set a foundation for Scube to achieve an accurate alignment of the consecutive slices and hence a precise 3D architecture construction"
- [discussion] PASTE's alignment results are sensitive to hyperparameters in datasets with significant defects: "This design makes PASTE’s alignment results sensitive to hyperparameters in datasets with significant defects (e.g., slices with crop ratios ≥ 0.25)"
- [readme] Scube module description in the SPACEL README: "Scube automatically transforms the spatial coordinate systems of consecutive slices and stacks them together to construct a three-dimensional (3D) alignment of the tissue."
