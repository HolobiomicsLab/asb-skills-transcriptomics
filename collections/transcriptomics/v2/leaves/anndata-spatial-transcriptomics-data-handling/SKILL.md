---
name: anndata-spatial-transcriptomics-data-handling
description: Use when seq-based ST data (e.g. 10X Visium spots) and a single-cell
  reference must be loaded into anndata format before deconvolution or multi-slice
  spatial domain analysis.
when_to_use_negative:
- Input is single-cell resolution in-situ data (MERFISH, STARmap) where deconvolution
  of spot-level composition is not needed before Splane/Scube
- Data is not in or convertible to anndata format and no scanpy/anndata stack is available
- The analysis goal is nonlinear slice alignment, which Scube does not support
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: SPACEL
  role: Python toolkit providing the Spoint/Splane/Scube modules that consume anndata-formatted
    ST and scRNA-seq data
  repo: github.com/QuKunLab/SPACEL
- name: Spoint
  role: Deconvolves cell type composition per spot from anndata spatial data using
    the scRNA-seq reference
  repo: github.com/QuKunLab/SPACEL
- name: Splane
  role: Consumes cell type composition added to spot metadata (add_cell_type_composition)
    to identify spatial domains
  repo: github.com/QuKunLab/SPACEL
- name: Python
  role: Implementation language for the anndata/scanpy-based data loading and SPACEL
    workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-43220-3
    title: 'SPACEL: deep learning-based characterization of spatial transcriptome
      architectures'
  source_content_hash: sha256:cf361a846cd81f367b1a576d9bdf9eef0984d8fc9bc5c5698ca5002e9ea6fac0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - Python
  - SPACEL
  - Splane
  - Spoint
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# anndata-spatial-transcriptomics-data-handling

## Summary

Load and prepare 10X Visium spatial transcriptomics slices and scRNA-seq references as anndata objects so they can be consumed by SPACEL modules such as Spoint for cell type deconvolution and Splane for spatial domain identification.

## When to use

Use when seq-based ST data (e.g. 10X Visium spots) and a single-cell reference must be loaded into anndata format before deconvolution or multi-slice spatial domain analysis.

## When NOT to use

- Input is single-cell resolution in-situ data (MERFISH, STARmap) where deconvolution of spot-level composition is not needed before Splane/Scube
- Data is not in or convertible to anndata format and no scanpy/anndata stack is available
- The analysis goal is nonlinear slice alignment, which Scube does not support

## Inputs

- 10X Visium spatial transcriptomics slice datasets (e.g. 12 human DLPFC slices)
- scRNA-seq reference dataset (e.g. Allen Brain Map human brain) as an anndata object
- Spot-level spatial coordinate information from the ST slices
- Manually annotated cortical layer ground-truth labels per spot (for validation)

## Outputs

- anndata objects holding spatial count matrices, spatial coordinates, and spot metadata
- Per-spot cell type composition matrices produced by Spoint.deconv_spatial
- Cell-type-composition-enriched anndata objects ready for Splane input

## How to apply

Load each 10X Visium slice (e.g. the 12 human DLPFC slices) and the scRNA-seq reference (e.g. Allen Brain Map human brain dataset) into anndata objects, matching gene identifiers across the two matrices. Spoint then consumes these anndata objects: it embeds a multiple-layer perceptron with a probabilistic model trained on the single-cell reference (Spoint.base_model.SpointModel.train with PyTorch), and cell type composition per spot is produced via Spoint.base_model.SpointModel.deconv_spatial. For seq-based technologies like 10X Visium, this deconvolution step is mandatory before Splane, since Splane requires cell type composition (added via Splane.utils.add_cell_type_composition) as model input together with spatial coordinates. The DLPFC slices also carry manually annotated cortical layer labels in spot-level metadata, which serve as ground truth for downstream accuracy scoring.

## Related tools

- **SPACEL** (Python toolkit providing the Spoint/Splane/Scube modules that consume anndata-formatted ST and scRNA-seq data) — github.com/QuKunLab/SPACEL
- **Spoint** (Deconvolves cell type composition per spot from anndata spatial data using the scRNA-seq reference) — github.com/QuKunLab/SPACEL
- **Splane** (Consumes cell type composition added to spot metadata (add_cell_type_composition) to identify spatial domains) — github.com/QuKunLab/SPACEL
- **Python** (Implementation language for the anndata/scanpy-based data loading and SPACEL workflow)

## Examples

```
import anndata as ad; adata_st = ad.read_h5ad('DLPFC_slice1.h5ad'); adata_sc = ad.read_h5ad('allen_brain_ref.h5ad')
```

## Evaluation signals

- All slices and the reference load as anndata objects with matching gene identifiers between spatial and single-cell matrices
- SpointModel.deconv_spatial outputs a per-spot cell type composition table with 56 cell types for the DLPFC benchmark
- Predicted compositions against manually annotated cortical layers yield average accuracy score ~0.60 and excitatory L3/4 accuracy ~0.85
- Cell type composition is successfully attached to spot metadata before Splane via add_cell_type_composition

## Limitations

- Adding a new slice requires retraining the entire deep learning model from scratch, which may affect computational efficiency for large-scale datasets
- Splane depends on cell type composition input, so a deconvolution step (Spoint) is mandatory for seq-based ST datasets like 10X Visium
- The GPR model originally relied on MATLAB code and third-party GUI software, motivating a Python wrapper for accessibility

## Evidence

- [methods] Load the 12 human DLPFC 10X Visium slice datasets and the Allen Brain Map human brain scRNA-seq reference into anndata objects.: [evidence span withheld — anchor status: not_found]
- [discussion] one has to employ a cell type deconvolution algorithm such as Spoint to obtain the cell type composition information before Splane can be applied: "one has to employ a cell type deconvolution algorithm such as Spoint to obtain the cell type composition information before Splane can be applied"
- [methods] Spoint embedded a multiple-layer perceptron with a probabilistic model to deconvolute cell type composition for each spot on single ST slice.: "Spoint embedded a multiple-layer perceptron with a probabilistic model to deconvolute cell type composition for each spot on single ST slice."
- [readme] Spoint tutorial: Deconvolution of cell types compostion on human brain Visium dataset: "Spoint tutorial: Deconvolution of cell types compostion on human brain Visium dataset"
