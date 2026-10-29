---
name: spot-level-cell-type-visualization
description: Use when you have an optimal-transport label-transfer matrix mapping
  spatial spots (10x Visium coordinates) to reference scRNA cell types and need per-spot
  cell-type composition maps on the tissue slice.
when_to_use_negative:
- When the OT plan cannot be stored in memory — saving a dense matrix T needs substantial
  memory and capacity must be ascertained first
- When no annotated scRNA reference with cell-type labels is available to drive the
  label transfer
- When numeric deconvolution benchmarks are required — validation here is a qualitative
  expert check, not a quantitative criterion
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3512
tools:
- name: uniPort
  role: Provides the OT-plan label transfer (uniport.metrics.label_transfer / mapCluster)
    that computes spot-level cell-type proportions and the spatial_function.R helpers
    (stClusterPie) for visualization
  repo: https://github.com/caokai1073/uniport
- name: scanpy
  role: Loads/preprocesses the Visium slice and scRNA reference AnnData (normalize_total,
    log1p) feeding the label transfer
- name: Seurat
  role: Loads the 10x Visium breast cancer data via Load10X_Spatial with NormalizeData/ScaleData
    before deconvolution and pie rendering
- name: scatterpie
  role: R plotting package used to draw per-spot cell-type composition pies on the
    Visium slice coordinates
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-35094-8
    title: A unified computational framework for single-cell data integration with
      optimal transport
  source_content_hash: sha256:1ee44487afd945dac86ffc997db66594ca759fa2f324203bb2641407e371045f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/caokai1073/uniport
  tools:
  - Seurat
  - scanpy
  - scatterpie
  - scikit-learn
  - uniPort
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# spot-level-cell-type-visualization

## Summary

Render uniPort's OT-plan-based spot-level cell-type proportions onto spatial coordinates of a Visium slice as scatterpie plots, enabling visual deconvolution readout (e.g., cortical layers, TLS colocalization of T and B cells).

## When to use

Use when you have an optimal-transport label-transfer matrix mapping spatial spots (10x Visium coordinates) to reference scRNA cell types and need per-spot cell-type composition maps on the tissue slice.

## When NOT to use

- When the OT plan cannot be stored in memory — saving a dense matrix T needs substantial memory and capacity must be ascertained first
- When no annotated scRNA reference with cell-type labels is available to drive the label transfer
- When numeric deconvolution benchmarks are required — validation here is a qualitative expert check, not a quantitative criterion

## Inputs

- Global OT plan matrix from up.Run(save_OT=True) (e.g., OT_BRCA.txt)
- Preprocessed 10x Visium spot AnnData/Seurat object with spatial coordinates
- Annotated reference scRNA metadata (e.g., celltype_major from Whole_miniatlas_meta.csv or Allen Institute cortical cell taxonomy labels)
- spatial_function.R helper (mapCluster, stClusterPie) from the uniPort repo

## Outputs

- Spot-level cell-type proportion table per Visium spot
- scatterpie plot of per-spot cell-type composition overlaid on slice coordinates (e.g., BRCA_celltype_LT_T_i60k_md.png)
- Projected spot-level expression per deconvoluted cell type

## How to apply

After running up.Run(adatas=[st_adata, scrna_ref], adata_cm=adata_cm, save_OT=True) to obtain the global OT plan, apply label transfer (uniport.metrics.label_transfer in Python, or the R mapCluster function with cluster='celltype_major' and min_cut=0.25 from spatial_function.R) to compute the spot-level proportion of each cell type, balanced by reference cluster ratio. Then render each spot as a pie whose slices show the 28 deconvoluted cell types (mouse cortex) or major tumor/immune types (HER2+ BRCA) using scatterpie/stClusterPie on the Visium spot coordinates. Also project spot-level expression of each cell type to cross-check against marker genes (e.g., Olig1/Olig2 for oligodendrocytes, Dcn/Osr1 for fibroblast-like cells). Correctness is judged qualitatively: the pie map should reproduce known tissue architecture (cortical layers) and cell-type-specific marker patterns, or show TLS as T/B-cell colocalization.

## Related tools

- **uniPort** (Provides the OT-plan label transfer (uniport.metrics.label_transfer / mapCluster) that computes spot-level cell-type proportions and the spatial_function.R helpers (stClusterPie) for visualization) — https://github.com/caokai1073/uniport
- **scanpy** (Loads/preprocesses the Visium slice and scRNA reference AnnData (normalize_total, log1p) feeding the label transfer)
- **Seurat** (Loads the 10x Visium breast cancer data via Load10X_Spatial with NormalizeData/ScaleData before deconvolution and pie rendering)
- **scatterpie** (R plotting package used to draw per-spot cell-type composition pies on the Visium slice coordinates)

## Examples

```
adata, OT = up.Run(adatas=[adata1, adata2], adata_cm=adata_cm, save_OT=True); proportions = up.metrics.label_transfer(adata, OT, min_cut=0.25)
```

## Evaluation signals

- Spatial pie map reconstructs known cortical layer structure from the 28 deconvoluted cell types on the anterior mouse brain slice
- Each deconvoluted cell type spatially aligns with its marker-gene expression (Olig1/Olig2 oligodendrocytes, Atp1b2/Slc1a2 astrocytes, Dcn/Osr1 fibroblast-like cells)
- TLS appears as colocalization of T and B cells with matching expression intensity in the HER2+ BRCA slice
- Label transfer used min_cut=0.25 with balance scaling by reference cluster ratio; proportions per spot should sum to ~1
- Qualitative expert review confirming the deconvolution pattern is reproduced, since no numeric criterion is reported

## Limitations

- Storing the dense full OT plan T requires substantial memory; verify memory capacity before save_OT=True
- Validation is a qualitative expert check; no numeric acceptance criterion is reported
- Marker-based visual checks depend on expert knowledge of expected tissue architecture and marker specificity

## Evidence

- [abstract] uniPort provides a flexible label transfer framework to deconvolute heterogeneous spatial transcriptomic data using an optimal transport plan, instead of embedding latent space.: "uniPort provides a flexible label transfer framework to deconvolute heterogeneous spatial transcriptomic data using an optimal transport plan, instead of embedding latent space."
- [methods] mapCluster usage in BRCA workflow: "ot_map <- mapCluster(ot,meta = brca_cluster, cluster = 'celltype_major')"
- [methods] The 'spatial_function.R' helper script location in the uniPort repo: "The 'spatial_function.R' is stored `here <https://github.com/caokai1073/uniPort/tree/main/R%20process/spot/>`_."
- [discussion] Memory caveat for the dense OT matrix: "Note that storing a dense matrix T needs more memory. Therefore, computing capacity with respect to memory should be ascertained before using this option"
- [readme] save_OT parameter behavior from README: "**save_OT**: if True, output a global OT plan. Need more memory. Default: False"
