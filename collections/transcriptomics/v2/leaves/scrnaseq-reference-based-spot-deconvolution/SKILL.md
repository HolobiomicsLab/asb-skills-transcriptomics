---
name: scrnaseq-reference-based-spot-deconvolution
description: Use when a seq-based ST dataset (e.g., 10X Visium spots) needs per-spot
  cell type proportions estimated from a matched scRNA-seq reference, or when cell-type
  composition must be obtained before Splane spatial-domain identification.
when_to_use_negative:
- Input is single-cell-resolution in-situ ST data where deconvolution of spot mixtures
  is unnecessary (Spoint targets spot-based seq technologies such as 10X Visium)
- The goal is spatial-domain identification or 3D alignment directly — use Splane
  or Scube instead; Spoint is only the prerequisite deconvolution step for seq-based
  ST data
- No annotated scRNA-seq reference exists for the tissue, since Spoint is reference-based
  and depends on the single-cell reference for training
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3678
- http://edamontology.org/topic_3170
tools:
- name: SPACEL
  role: Parent Python toolkit providing the Spoint deconvolution module
  repo: github.com/QuKunLab/SPACEL
- name: Spoint
  role: Embeds a multiple-layer perceptron with a probabilistic model on the scRNA-seq
    reference and deconvolutes cell type composition per spot via SpointModel.train
    and deconv_spatial
  repo: github.com/QuKunLab/SPACEL
- name: PyTorch
  role: Deep-learning backend used to train the Spoint model (GPU acceleration if
    available)
- name: Python
  role: Implementation language for loading AnnData, running Spoint, and computing
    benchmark metrics
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:02437531cff9a42296cd7e06a269fa0ae30c7fe6b41557231e7603b4f5d3456e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cell2location
  - DSTG
  - DestVI
  - PyTorch
  - Python
  - RCTD
  - SPACEL
  - SPOTlight
  - STRIDE
  - Seurat
  - SpatialDWLS
  - Spoint
  - Stereoscope
  - Tangram
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3678
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# scrnaseq-reference-based-spot-deconvolution

## Summary

Deconvolute cell-type composition per spot on single spatial-transcriptomics slices by embedding a multiple-layer perceptron with a probabilistic model (Spoint, the SPACEL deconvolution module) trained on a scRNA-seq reference, enabling downstream Splane spatial-domain identification on seq-based ST data such as 10X Visium.

## When to use

Use when a seq-based ST dataset (e.g., 10X Visium spots) needs per-spot cell type proportions estimated from a matched scRNA-seq reference, or when cell-type composition must be obtained before Splane spatial-domain identification.

## When NOT to use

- Input is single-cell-resolution in-situ ST data where deconvolution of spot mixtures is unnecessary (Spoint targets spot-based seq technologies such as 10X Visium)
- The goal is spatial-domain identification or 3D alignment directly — use Splane or Scube instead; Spoint is only the prerequisite deconvolution step for seq-based ST data
- No annotated scRNA-seq reference exists for the tissue, since Spoint is reference-based and depends on the single-cell reference for training

## Inputs

- Single ST slice (e.g., 10X Visium) as AnnData with spot-level gene expression and spatial coordinates
- scRNA-seq reference dataset with cell-type annotations (e.g., Allen Brain Map human brain scRNA-seq) as AnnData
- Ground-truth spot-level cell-type compositions (for simulated benchmark datasets)
- Manually annotated spatial labels (e.g., cortical layers on DLPFC Visium slices) for accuracy scoring

## Outputs

- Per-spot cell-type composition matrix (proportions per cell type per spot)
- Accuracy score (AS) per cell type and average AS across cell types and slices
- PCC, SSIM, RMSE, and JSD metrics versus ground truth on simulated datasets
- Summary results table (e.g., spoint_dlpfc_deconvolution_results.csv or deconvolution_benchmark_metrics.csv)

## How to apply

Load the ST slice(s) and the scRNA-seq reference (e.g., Allen Brain Map human brain scRNA-seq for DLPFC Visium data) into AnnData objects. Initialize Spoint with init_model, which embeds a multiple-layer perceptron with a probabilistic model on the single-cell reference, then train SpointModel via Spoint.base_model.SpointModel.train using PyTorch (GPU acceleration if available). Deconvolve per-spot cell-type composition with SpointModel.deconv_spatial for each spot on the ST slice. Validate against ground truth where available: on the 32 simulated benchmarks Spoint should reach PCC/SSIM ≈ 0.73/0.69, RMSE/JSD ≈ 0.05/0.41, and AS ≈ 0.93; on the 12 DLPFC Visium slices with manually annotated cortical layers, the accuracy score for excitatory L3/4 neurons should reach 0.85 and the average AS over all 56 cell types should reach 0.60. Write aggregated per-cell-type accuracy metrics to a results table (e.g., CSV) for comparison against baselines.

## Related tools

- **SPACEL** (Parent Python toolkit providing the Spoint deconvolution module) — github.com/QuKunLab/SPACEL
- **Spoint** (Embeds a multiple-layer perceptron with a probabilistic model on the scRNA-seq reference and deconvolutes cell type composition per spot via SpointModel.train and deconv_spatial) — github.com/QuKunLab/SPACEL
- **PyTorch** (Deep-learning backend used to train the Spoint model (GPU acceleration if available))
- **Python** (Implementation language for loading AnnData, running Spoint, and computing benchmark metrics)

## Examples

```
from SPACEL.Spoint.base_model import SpointModel; sp = SpointModel(adata_sc, init_model='MLP'); sp.train(); sp.deconv_spatial(adata_st)
```

## Evaluation signals

- On the 32 simulated benchmark datasets, Spoint yields average PCC/SSIM ≈ 0.73/0.69, RMSE/JSD ≈ 0.05/0.41, and AS ≈ 0.93, with all baselines (Cell2location, SpatialDWLS, RCTD, STRIDE, Stereoscope, Tangram, DestVI, Seurat, SPOTlight, DSTG) in the 0.24–0.82 AS range
- On the 12 DLPFC Visium slices, the accuracy score for excitatory L3/4 neurons reaches 0.85 versus 0.29–0.64 for other methods
- Average AS over all 56 cell types across the 12 DLPFC slices reaches 0.60 versus 0.30–0.48 for other deconvolution methods
- Average Wilcoxon rank-sum P-value for predicted versus annotated layer scores is 0.01 on the DLPFC benchmark
- Predicted per-spot proportions form a spots-by-cell-types matrix whose dominant cell type at each spot can be directly compared to ground-truth dominant cell type to derive AS

## Limitations

- Splane depends on cell type composition as input, so a deconvolution step such as Spoint is mandatory for seq-based ST datasets like 10X Visium — the skill alone does not identify spatial domains
- Benchmark comparisons rely on simulated ground truth or manually annotated cortical layers; accuracy may differ on tissues without reliable references or annotations
- Adding a new slice requires retraining the entire deep learning model from scratch, which may affect computational efficiency for large-scale datasets

## Evidence

- [abstract] Spoint embeds a multiple-layer perceptron with a probabilistic model to deconvolute cell type composition for each spot in a single ST slice: "Spoint embeds a multiple-layer perceptron with a probabilistic model to deconvolute cell type composition for each spot in a single ST slice"
- [intro] Spoint predicts the cell type composition of spots obtained by seq-based ST technologies: "Spoint predicts the cell type composition of spots obtained by seq-based ST technologies such as 10X Visium."
- [discussion] one has to employ a cell type deconvolution algorithm such as Spoint before Splane can be applied: "one has to employ a cell type deconvolution algorithm such as Spoint to obtain the cell type composition information before Splane can be applied"
- [methods] Spoint.base_model.SpointModel.deconv_spatial method signature: "Spoint.base_model.SpointModel.deconv_spatial"
- [methods] Spoint deconvolutes cell type composition for each spot on single ST slice: "Spoint embedded a multiple-layer perceptron with a probabilistic model to deconvolute cell type composition for each spot on single ST slice."
- [readme] SPACEL module description in README: "Spoint embedded a multiple-layer perceptron with a probabilistic model to deconvolute cell type composition for each spot on single ST slice."
