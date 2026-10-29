---
name: spatial-transcriptome-cell-type-deconvolution
description: Use when you need spot-level cell type proportions from seq-based ST
  data such as 10X Visium, especially as the cell-type-composition input Splane requires
  before spatial domain identification.
when_to_use_negative:
- Input is imaging-based ST data with single-cell resolution (e.g., MERFISH, STARmap)
  where spots already correspond to cells and deconvolution is unnecessary
- No matched scRNA-seq reference exists for the tissue, so the probabilistic model
  cannot be trained on reliable cell type labels
- The goal is spatial domain identification across slices without needing cell type
  proportions from seq-based ST (though Splane then requires this step for 10X Visium
  input)
edam_operation: http://edamontology.org/operation_3644
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SPACEL
  role: Toolkit providing the Spoint module used to train and apply the deconvolution
    model
  repo: github.com/QuKunLab/SPACEL
- name: Spoint
  role: Embeds a multiple-layer perceptron with a probabilistic model to deconvolute
    cell type composition for each spot on a single ST slice via SpointModel.train
    and deconv_spatial
  repo: github.com/QuKunLab/SPACEL
- name: Python
  role: Implementation language; SpointModel.train runs with PyTorch, optionally GPU-accelerated
provenance:
  source_task_ids:
  - task_005
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:2e3d51dbc35b75f628147de994ed33472be401ee51d00e3b4ff618c704dfec81
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
  - Splane
  - Spoint
  - Stereoscope
  - Tangram
  edam_operation: http://edamontology.org/operation_3644
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# spatial-transcriptome-cell-type-deconvolution

## Summary

Deconvolves per-spot cell type composition from sequencing-based spatial transcriptomics data (e.g., 10X Visium) using Spoint, an SPACEL module that embeds a multiple-layer perceptron with a probabilistic model trained on an scRNA-seq reference.

## When to use

Use when you need spot-level cell type proportions from seq-based ST data such as 10X Visium, especially as the cell-type-composition input Splane requires before spatial domain identification.

## When NOT to use

- Input is imaging-based ST data with single-cell resolution (e.g., MERFISH, STARmap) where spots already correspond to cells and deconvolution is unnecessary
- No matched scRNA-seq reference exists for the tissue, so the probabilistic model cannot be trained on reliable cell type labels
- The goal is spatial domain identification across slices without needing cell type proportions from seq-based ST (though Splane then requires this step for 10X Visium input)

## Inputs

- Spot-level spatial transcriptomics dataset (e.g., 10X Visium) as AnnData (.h5ad)
- Single-cell RNA-seq reference for the same tissue (e.g., Allen Brain Map human brain scRNA-seq)
- Optional ground-truth spot annotations (e.g., manually annotated DLPFC cortical layers) for validation

## Outputs

- Per-spot cell type composition/proportion matrix per ST slice
- Benchmark metrics table (PCC, SSIM, RMSE, JSD, accuracy score) against ground truth
- Cell-type-composition AnnData aggregated across slices for Splane input

## How to apply

Load the ST slice and a matched single-cell RNA-seq reference (e.g., Allen Brain Map human brain scRNA-seq for DLPFC Visium slices) into anndata objects. Initialize Spoint with an MLP-plus-probabilistic model on the scRNA-seq reference, train it with Spoint.base_model.SpointModel.train using PyTorch (GPU acceleration if available), then predict per-spot composition with SpointModel.deconv_spatial for each slice. For downstream Splane workflows, aggregate per-slice composition matrices with Splane.utils.add_cell_type_composition so all slices share a unified representation. Judge accuracy against annotated ground truth (e.g., manually annotated DLPFC cortical layers) via accuracy score (AS) per cell type and Wilcoxon rank-sum tests; on 32 simulated benchmarks Spoint should reach PCC/SSIM ≈ 0.73/0.69, RMSE/JSD ≈ 0.05/0.41, and AS ≈ 0.93, and for DLPFC excitatory L3/4 neurons AS ≈ 0.85 with average AS over 56 cell types ≈ 0.60.

## Related tools

- **SPACEL** (Toolkit providing the Spoint module used to train and apply the deconvolution model) — github.com/QuKunLab/SPACEL
- **Spoint** (Embeds a multiple-layer perceptron with a probabilistic model to deconvolute cell type composition for each spot on a single ST slice via SpointModel.train and deconv_spatial) — github.com/QuKunLab/SPACEL
- **Python** (Implementation language; SpointModel.train runs with PyTorch, optionally GPU-accelerated)

## Examples

```
from SPACEL.Spoint.base_model import SpointModel; SpointModel.train(sc_adata); SpointModel.deconv_spatial(st_adata)
```

## Evaluation signals

- On the 32 simulated benchmark datasets, Spoint should yield average PCC/SSIM ≈ 0.73/0.69 and RMSE/JSD ≈ 0.05/0.41, with accuracy score ≈ 0.93
- On 12 DLPFC Visium slices with the Allen Brain reference, excitatory L3/4 neuron accuracy score should reach 0.85 and average AS over all 56 cell types should reach 0.60
- Average Wilcoxon rank-sum P-value comparing predicted versus annotated layer scores should be ≈ 0.01
- Per-spot composition values should be valid proportions (non-negative, summing to ~1 per spot)
- Baselines (Cell2location, SpatialDWLS, RCTD, STRIDE, Tangram, etc.) should fall in the 0.24–0.82 AS range, confirming Spoint's reported superiority

## Limitations

- Splane depends on cell type composition input, so for seq-based ST datasets like 10X Visium a deconvolution step such as Spoint must be run first, adding a required dependency
- Deconvolution accuracy depends on the quality and match of the scRNA-seq reference; the reported AS of 0.85 (excitatory L3/4) and 0.60 average were obtained specifically with the Allen Brain Map human brain reference on DLPFC slices

## Evidence

- [abstract] Spoint embeds a multiple-layer perceptron with a probabilistic model to deconvolute cell type composition: "SPACEL comprises three modules: Spoint embeds a multiple-layer perceptron with a probabilistic model to deconvolute cell type composition"
- [discussion] one has to employ a cell type deconvolution algorithm such as Spoint: "one has to employ a cell type deconvolution algorithm such as Spoint to obtain the cell type composition information before Splane can be applied"
- [methods] Spoint.base_model.SpointModel.deconv_spatial: "Spoint embedded a multiple-layer perceptron with a probabilistic model to deconvolute cell type composition for each spot on single ST slice."
- [methods] Deconvolution of cell types compostion on human brain Visium dataset: "Deconvolution of cell types compostion on human brain Visium dataset <tutorials/Visium_human_DLPFC_Spoint>"
- [readme] Spoint deconvolutes cell type composition for each spot on single ST slices: "Spoint embedded a multiple-layer perceptron with a probabilistic model to deconvolute cell type composition for each spot on single ST slice."
