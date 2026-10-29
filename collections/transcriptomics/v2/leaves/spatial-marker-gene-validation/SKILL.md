---
name: spatial-marker-gene-validation
description: Use when an OT-plan or label-transfer deconvolution of a spatial slice
  (e.g., 10x Visium) yields per-spot cell-type proportions and you must confirm biological
  plausibility against layer anatomy and marker-gene expression before accepting the
  result.
when_to_use_negative:
- Input is raw counts or an unannotated reference — marker-gene alignment checks require
  a completed label transfer with cell-type assignments, not preprocessed data
- Tissue lacks known spatial anatomy or established marker genes, so the qualitative
  layer-structure and marker-alignment checks cannot be evaluated
- The dense OT plan could not be produced (save_OT=False or insufficient memory),
  since the validation validates deconvolution derived from the global OT matrix
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: uniPort
  role: Supplies the OT-plan-based label transfer (uniport.metrics.label_transfer)
    whose deconvolution output (28 cortical cell-type proportions per spot) is being
    validated; scatterpie rendering helpers in R process/spot/ spatial_function.R
  repo: https://github.com/caokai1073/uniPort
- name: scanpy
  role: Preprocesses the Visium slice (normalize_total, log1p) and holds spot coordinates/expression
    used for marker-gene spatial projection
- name: Seurat
  role: Referenced comparative platform for spatial label transfer workflows whose
    outputs this validation paradigm parallels
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-35094-8
    title: A unified computational framework for single-cell data integration with
      optimal transport
  source_content_hash: sha256:363f1ce910df50062f7bf149b1d40373bf8baa0cfa57c4eea0430af8cc07636b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/caokai1073/uniPort
  tools:
  - Seurat
  - scanpy
  - scikit-learn
  - uniPort
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# spatial-marker-gene-validation

## Summary

Validates spatial transcriptomic deconvolution by checking that per-spot transferred cell-type proportions reconstruct known anatomical structure and spatially align with canonical marker genes (e.g., Olig1/Olig2 for oligodendrocytes, Atp1b2/Slc1a2 for astrocytes, Dcn/Osr1 for fibroblast-like cells) on the Visium slice.

## When to use

Use when an OT-plan or label-transfer deconvolution of a spatial slice (e.g., 10x Visium) yields per-spot cell-type proportions and you must confirm biological plausibility against layer anatomy and marker-gene expression before accepting the result.

## When NOT to use

- Input is raw counts or an unannotated reference — marker-gene alignment checks require a completed label transfer with cell-type assignments, not preprocessed data
- Tissue lacks known spatial anatomy or established marker genes, so the qualitative layer-structure and marker-alignment checks cannot be evaluated
- The dense OT plan could not be produced (save_OT=False or insufficient memory), since the validation validates deconvolution derived from the global OT matrix

## Inputs

- Per-spot cell-type proportion table (28 Allen Institute cortical cell types) from OT-plan label transfer on the anterior mouse brain 10x Visium slice
- Visium spot spatial coordinates (e.g., 2696 spots) from the preprocessed st_adata AnnData object
- Normalized/log1p spot-level gene expression for marker genes (Olig1, Olig2, Atp1b2, Slc1a2, Dcn, Osr1)
- Reference scRNA cell-type annotations from the Allen Institute adult mouse cortical cell taxonomy dataset

## Outputs

- Scatterpie plot of per-spot cell-type composition overlaid on the Visium slice coordinates
- Projected spot-level spatial maps of each of the 28 deconvoluted cell types
- Qualitative validation verdict: cortical-layer-consistent distributions and per-type marker-gene spatial alignment

## How to apply

After label transfer (e.g., uniport.metrics.label_transfer with min_cut=0.25 on the global OT plan from up.Run with save_OT=True), render per-spot cell-type composition as scatterpie plots on the Visium slice coordinates and project spot-level expression of each deconvoluted cell type. First check that the pie-map reconstructs the known cortical-layer structure of the anterior adult mouse brain slice — layer-consistent, spatially coherent cell-type distributions indicate correct deconvolution. Then, for each of the 28 deconvoluted cortical cell types, overlay its projected spot-level abundance with the expression pattern of its canonical markers (Olig1/Olig2 for oligodendrocytes, Atp1b2/Slc1a2 for astrocytes, Dcn/Osr1 for fibroblast-like cells). Accept the deconvolution only if each cell type's spatial map aligns with its marker-gene expression (qualitative expert check), which also implicitly confirms the dense OT matrix loaded within memory.

## Related tools

- **uniPort** (Supplies the OT-plan-based label transfer (uniport.metrics.label_transfer) whose deconvolution output (28 cortical cell-type proportions per spot) is being validated; scatterpie rendering helpers in R process/spot/ spatial_function.R) — https://github.com/caokai1073/uniPort
- **scanpy** (Preprocesses the Visium slice (normalize_total, log1p) and holds spot coordinates/expression used for marker-gene spatial projection)
- **Seurat** (Referenced comparative platform for spatial label transfer workflows whose outputs this validation paradigm parallels)

## Examples

```
adata, OT = up.Run(adatas=[st_adata, scrna_ref], adata_cm=adata_cm, save_OT=True); label_transfer(ot, st_adata, scrna_ref, min_cut=0.25)  # then scatterpie on spot coords vs Olig1/Olig2, Atp1b2/Slc1a2, Dcn/Osr1
```

## Evaluation signals

- The scatterpie map reconstructs the well-structured cortical layers of the anterior adult mouse brain slice with layer-consistent cell-type distributions
- Each of the 28 deconvoluted cell types spatially aligns with its marker genes: Olig1/Olig2 (oligodendrocytes), Atp1b2/Slc1a2 (astrocytes), Dcn/Osr1 (fibroblast-like cells)
- Per-spot proportions derive from the OT plan with min_cut=0.25 and reference-cluster-ratio balancing, on k=2000 common HVGs and kx=ky=2000 dataset-specific HVGs
- The dense OT matrix T loaded within memory (save_OT=True), a precondition for the label transfer being validated
- No cell type shows a spatially incoherent or anatomically implausible map (qualitative expert check)

## Limitations

- Validation is a qualitative expert check, not a quantitative metric; there is no numerical acceptance threshold
- Storing the dense global OT plan T requires substantial memory and computing capacity should be ascertained beforehand
- Which hyperparameter combination (λ ∈ {0.5, 1.0, 5.0}, γ ∈ {0.5, 1.0}, λs ∈ {0.5, 1.0}) was selected for a given dataset is not stated, complicating exact reproduction

## Evidence

- [abstract] uniPort provides a label transfer framework to deconvolute spatial transcriptomic data using an optimal transport plan: "uniPort provides a flexible label transfer framework to deconvolute heterogeneous spatial transcriptomic data using an optimal transport plan, instead of embedding latent space."
- [discussion] We normalized total counts of each cell using the scanpy.pp.normalize_total function in the scanpy package: "We normalized total counts of each cell using the scanpy.pp.normalize_total function in the scanpy package"
- [readme] save_OT outputs a global OT plan but needs more memory: "save_OT: if True, output a global OT plan. Need more memory. Default: False"
- [methods] The spatial scatterpie helper is stored in the uniPort repository: "The 'spatial_function.R' is stored `here <https://github.com/caokai1073/uniPort/tree/main/R%20process/spot/>`_."
- [discussion] Storing the dense OT plan matrix requires verifying memory capacity: "Note that storing a dense matrix T needs more memory. Therefore, computing capacity with respect to memory should be ascertained before using this option"
