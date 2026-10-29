---
name: 3d-slice-alignment-and-stacking
description: Use when you have multiple consecutive ST slices (e.g., 33 MERFISH mouse
  primary motor cortex slices, Stereo-seq mouse embryo slices) and need to construct
  a 3D stack by aligning slice coordinate systems.
when_to_use_negative:
- 'You need nonlinear alignment of ST slices: the current version of Scube does not
  support nonlinear alignment, and no automatic method yet exists for ST data'
- You only have a single ST slice — 3D stacking requires multiple consecutive slices
- Seq-based ST data (e.g., 10X Visium) without a prior cell type deconvolution step,
  since Splane requires cell type composition as input
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SPACEL
  role: Deep-learning toolkit providing the 3D alignment workflow used by this skill
  repo: github.com/QuKunLab/SPACEL
- name: Scube
  role: Transforms spatial coordinate systems of consecutive slices and stacks them
    into a 3D architecture via global optimization (Scube.alignment.align)
  repo: github.com/QuKunLab/SPACEL
- name: Splane
  role: GCN with adversarial learning that identifies the spatial domains which set
    the foundation for Scube's slice alignment
  repo: github.com/QuKunLab/SPACEL
- name: STAligner
  role: Baseline alignment method for benchmarking Scube's stacked slices via SSIM/PCC
- name: PASTE
  role: Baseline alignment method for benchmarking; hyperparameter-sensitive on defect-laden
    slices
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:73b3e761d9eb89e0cdfd7d707257766e213832e8da27c48998bc9aaf6830167d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PASTE
  - Python
  - SPACEL
  - STAligner
  - Scube
  - Splane
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# 3d-slice-alignment-and-stacking

## Summary

Align consecutive spatial transcriptomics slices into a stacked 3D tissue architecture using Scube from SPACEL, which transforms the spatial coordinate systems of adjacent slices based on spatial domains identified by Splane, and benchmark the result against PASTE and STAligner using adjacent-slice SSIM and PCC.

## When to use

Use when you have multiple consecutive ST slices (e.g., 33 MERFISH mouse primary motor cortex slices, Stereo-seq mouse embryo slices) and need to construct a 3D stack by aligning slice coordinate systems.

## When NOT to use

- You need nonlinear alignment of ST slices: the current version of Scube does not support nonlinear alignment, and no automatic method yet exists for ST data
- You only have a single ST slice — 3D stacking requires multiple consecutive slices
- Seq-based ST data (e.g., 10X Visium) without a prior cell type deconvolution step, since Splane requires cell type composition as input

## Inputs

- Consecutive ST slices (e.g., 33 MERFISH mouse primary motor cortex slices, or Stereo-seq mouse embryo slices) with per-spot/per-cell spatial coordinates and expression
- Cell type composition per spot/cell (from Spoint deconvolution for seq-based ST such as 10X Visium, or direct composition for image-based ST like MERFISH)
- Spatial neighbor graphs per slice

## Outputs

- Transformed spatial coordinates per slice aligned into a stacked 3D tissue architecture
- Stacked slice visualizations (Scube.plot.plot_stacked_slices, Scube.plot.plot_3d)
- Benchmark table of pairwise adjacent-slice SSIM and PCC per method (e.g., ssim_pcc_metrics.csv: Scube 0.71/0.76 vs STAligner 0.61/0.65 vs PASTE 0.57/0.61)

## How to apply

First assemble per-cell cell type composition and spatial neighbor graphs with Splane.utils.add_cell_type_composition, then train the Splane graph convolutional network with adversarial learning (Splane.base_model.SplaneModel.train) and call Splane.base_model.SplaneModel.identify_spatial_domain to assign transcriptomically and spatially coherent spatial domains across all slices — these domains set the foundation for Scube's alignment. Run Scube.alignment.align to transform the coordinate systems of consecutive slices and stack them into a 3D tissue alignment, visualizing with Scube.plot.plot_stacked_slices and Scube.plot.plot_3d. Scube adopts a global optimization strategy considering correspondences between all spots in adjacent slices, which makes it robust on datasets with substantial defects (e.g., slices with crop ratios ≥ 0.25) where PASTE is hyperparameter-sensitive. As a decision point, repeat alignment with STAligner and PASTE baselines and compute pairwise SSIM and PCC between every pair of adjacent transformed slices; on the 33-slice MERFISH MOp dataset Scube reached SSIM/PCC of 0.71/0.76 versus STAligner 0.61/0.65 and PASTE 0.57/0.61.

## Related tools

- **SPACEL** (Deep-learning toolkit providing the 3D alignment workflow used by this skill) — github.com/QuKunLab/SPACEL
- **Scube** (Transforms spatial coordinate systems of consecutive slices and stacks them into a 3D architecture via global optimization (Scube.alignment.align)) — github.com/QuKunLab/SPACEL
- **Splane** (GCN with adversarial learning that identifies the spatial domains which set the foundation for Scube's slice alignment) — github.com/QuKunLab/SPACEL
- **STAligner** (Baseline alignment method for benchmarking Scube's stacked slices via SSIM/PCC)
- **PASTE** (Baseline alignment method for benchmarking; hyperparameter-sensitive on defect-laden slices)

## Examples

```
Scube.alignment.align(...)  # after Splane.base_model.SplaneModel.identify_spatial_domain; visualize with Scube.plot.plot_3d
```

## Evaluation signals

- Average adjacent-slice SSIM and PCC exceed baseline methods (Scube 0.71/0.76 vs STAligner 0.61/0.65 and PASTE 0.57/0.61 on the 33-slice MERFISH MOp dataset)
- Stacked 3D reconstruction shows anatomically coherent architecture rather than a twisted 3D structure as observed with STAligner and PASTE stacks
- Spatial domains assigned by SplaneModel.identify_spatial_domain are transcriptomically and spatially coherent across all slices (seven domains on the MERFISH MOp data)
- Per-pair SSIM/PCC computed between every pair of adjacent transformed slices is aggregated per method into a metrics table (ssim_pcc_metrics.csv)

## Limitations

- The current version of Scube does not support nonlinear alignment, unlike STAligner and PASTE; nonlinear support is deferred to a future version
- Adding a new slice to the dataset requires retraining the entire deep learning model from scratch, limiting computational efficiency for large-scale datasets
- Splane depends on cell type composition input, so seq-based ST datasets like 10X Visium require a prior deconvolution step (e.g., Spoint)
- PASTE's alignment results are sensitive to hyperparameters in datasets with significant defects (e.g., slices with crop ratios ≥ 0.25), which matters when choosing baselines

## Evidence

- [abstract] Scube automatically transforms the spatial coordinate systems of consecutive slices and stacks them together to construct a 3D architecture of the tissue: "Scube automatically transforms the spatial coordinate systems of consecutive slices and stacks them together to construct a 3D architecture of the tissue"
- [discussion] Scube adopts a global optimization strategy for 3D alignment, taking into account the correspondences between all spots in adjacent slices: "Scube adopts a global optimization strategy for 3D alignment, taking into account the correspondences between all spots in adjacent slices"
- [discussion] Spatial domains identified in Splane also set a foundation for Scube to achieve an accurate alignment of the consecutive slices: "Spatial domains identified in Splane also set a foundation for Scube to achieve an accurate alignment of the consecutive slices and hence a precise 3D architecture construction"
- [discussion] the current version of Scube does not support nonlinear alignment as well as STAligner and PASTE: "the current version of Scube does not support nonlinear alignment as well as STAligner and PASTE"
- [methods] Scube.alignment.align and Splane.base_model.SplaneModel.identify_spatial_domain are documented method entry points: "Scube.alignment.align"
- [readme] SPACEL consists of three modules including Scube which constructs 3D alignment of the tissue: "Scube automatically transforms the spatial coordinate systems of consecutive slices and stacks them together to construct a three-dimensional (3D) alignment of the tissue."
