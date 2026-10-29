---
name: single-cell-rna-reference-mapping
description: Use when an annotated scRNA-seq reference (e.g., Allen Institute cortical
  taxonomy or Whole_miniatlas_meta.csv BRCA annotations) must be projected onto a
  10x Visium slice to resolve per-spot cell-type composition and localize structures
  like TLS or cortical layers.
when_to_use_negative:
- Datasets are profiled from the same cells (vertical integration) or share no genes
  (diagonal integration) — use up.Run with mode='v' or mode='d' instead of the horizontal
  default
- When computing memory cannot hold the dense global OT matrix T — the paper warns
  capacity 'should be ascertained before using this option' (save_OT)
- Input is raw spatial data needing normalization/annotation itself without a matched
  annotated scRNA reference available
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: uniPort
  role: 'Core framework: coupled-VAE + Minibatch-UOT training (up.Run with save_OT=True)
    and OT-plan label transfer (uniport.metrics.label_transfer / mapCluster) to deconvolute
    the Visium slice'
  repo: https://github.com/caokai1073/uniport
- name: scanpy
  role: 'Preprocessing: normalize_total, log1p (offset 1), load_file handling of AnnData
    objects'
- name: Seurat
  role: Load and normalize/scale the 10x Visium spatial data (Load10X_Spatial, NormalizeData,
    ScaleData)
- name: scikit-learn
  role: MaxAbsScaler per-gene 0-1 scaling and kNN classifier (KNeighborsClassifier)
    for label-transfer evaluation
- name: scatterpie
  role: Render per-spot cell-type composition pie charts on Visium coordinates (stClusterPie
    spatial_function.R)
  repo: https://github.com/caokai1073/uniPort/tree/main/R%20process/spot/
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-35094-8
    title: A unified computational framework for single-cell data integration with
      optimal transport
  source_content_hash: sha256:92d3103a3ae6159b4a1752f7d767ee26464ab931cd15c07bd3b9be36f0767160
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# single-cell-rna-reference-mapping

## Summary

Map an annotated single-cell RNA-seq reference onto spatial transcriptomics (10x Visium) data via uniPort's coupled-VAE plus Minibatch unbalanced optimal transport, using the global OT plan — rather than the latent space — to transfer cell-type labels and deconvolute per-spot cell-type proportions.

## When to use

Use when an annotated scRNA-seq reference (e.g., Allen Institute cortical taxonomy or Whole_miniatlas_meta.csv BRCA annotations) must be projected onto a 10x Visium slice to resolve per-spot cell-type composition and localize structures like TLS or cortical layers.

## When NOT to use

- Datasets are profiled from the same cells (vertical integration) or share no genes (diagonal integration) — use up.Run with mode='v' or mode='d' instead of the horizontal default
- When computing memory cannot hold the dense global OT matrix T — the paper warns capacity 'should be ascertained before using this option' (save_OT)
- Input is raw spatial data needing normalization/annotation itself without a matched annotated scRNA reference available

## Inputs

- 10x Visium spatial slice AnnData (e.g., sagittal/anterior mouse brain or human breast cancer slice loaded via Seurat Load10X_Spatial or uniPort load_file)
- Annotated scRNA-seq reference (e.g., Allen Institute adult mouse cortical cell taxonomy; Whole_miniatlas_meta.csv celltype_major annotations)
- Common-HVG AnnData (adata_cm) with 2000 shared highly variable genes
- Dataset-specific HVG AnnData for each dataset

## Outputs

- Global OT plan matrix T (e.g., saved as OT_BRCA.txt) mapping spots to reference cells
- Spot-level cell-type proportion table / transferred labels (28 cortical types or celltype_major clusters)
- Integrated latent embeddings in adata.obs['latent']
- Spatial scatterpie plot of per-spot cell-type composition (e.g., BRCA_celltype_LT_T_i60k_md.png)

## How to apply

Preprocess both spatial and reference datasets with uniPort's load_file/filter_data/batch_scale: normalize total counts (scanpy.pp.normalize_total), log1p with offset 1, and MaxAbsScaler rescale each gene to 0-1 per dataset, selecting k=2000 highly variable common genes (adata_cm) plus kx=ky=2000 dataset-specific HVGs. Train the coupled-VAE with Minibatch-UOT via up.Run(adatas=[st_adata, scrna_ref], adata_cm=adata_cm, save_OT=True) in default mode='h' with the reference as the last dataset (iteration=30000, lambda_s=0.5, lambda_ot=1.0), which returns cell embeddings and the dense global OT plan matrix. Apply OT-plan-based label transfer (uniport.metrics.label_transfer, or the R mapCluster function with cluster='celltype_major' and min_cut=0.25, balancing by reference cluster ratio) to compute spot-level proportions of each cell type. Visualize with scatterpie/stClusterPie on Visium spot coordinates and confirm that spatial proportions align with marker-gene expression (e.g., Olig1/Olig2 for oligodendrocytes) or reproduce reported deconvolution patterns. The key rationale: uniPort deconvolutes using the optimal transport plan instead of the latent-space embedding, making the mapping robust to heterogeneous spatial data.

## Related tools

- **uniPort** (Core framework: coupled-VAE + Minibatch-UOT training (up.Run with save_OT=True) and OT-plan label transfer (uniport.metrics.label_transfer / mapCluster) to deconvolute the Visium slice) — https://github.com/caokai1073/uniport
- **scanpy** (Preprocessing: normalize_total, log1p (offset 1), load_file handling of AnnData objects)
- **Seurat** (Load and normalize/scale the 10x Visium spatial data (Load10X_Spatial, NormalizeData, ScaleData))
- **scikit-learn** (MaxAbsScaler per-gene 0-1 scaling and kNN classifier (KNeighborsClassifier) for label-transfer evaluation)
- **scatterpie** (Render per-spot cell-type composition pie charts on Visium coordinates (stClusterPie spatial_function.R)) — https://github.com/caokai1073/uniPort/tree/main/R%20process/spot/

## Examples

```
adata, OT = up.Run(adatas=[st_adata, scrna_ref], adata_cm=adata_cm, save_OT=True); labels = uniport.metrics.label_transfer(adata, scrna_ref, min_cut=0.25)
```

## Evaluation signals

- Transferred labels reconstruct known spatial structure: cortical layers of the mouse brain Visium slice appear well-organized and spatially coherent
- Each deconvoluted cell type's spot-level proportion spatially colocalizes with its marker genes (Olig1/Olig2 oligodendrocytes; Atp1b2/Slc1a2 astrocytes; Dcn/Osr1 fibroblast-like; HER2+ BRCA T/B cell colocalization for TLS)
- OT plan matrix T loads and has expected shape (spots x reference cells) without exceeding available memory
- Label transfer used min_cut=0.25 threshold and reference-cluster-ratio balancing, producing per-spot proportions that sum to a sensible composition
- Output deconvolution map qualitatively reproduces the reported pattern for the dataset (expert review, as no numeric criterion is reported)

## Limitations

- Storing the dense full OT plan T requires substantial memory; computing capacity must be ascertained before save_OT=True
- Validation of spatial deconvolution is qualitative (expert review) — no numeric acceptance criterion is reported for the BRCA TLS task
- Which hyperparameter combination from the searched grids (λ ∈ {0.5, 1.0, 5.0}, γ ∈ {0.5, 1.0}, λs ∈ {0.5, 1.0}) was selected per dataset is not stated
- Results depend on the annotated reference covering the cell types present; heterogeneity is handled via 2000 common HVGs plus 2000 dataset-specific HVGs per dataset

## Evidence

- [abstract] uniPort provides a flexible label transfer framework to deconvolute heterogeneous spatial transcriptomic data using an optimal transport plan: "uniPort provides a flexible label transfer framework to deconvolute heterogeneous spatial transcriptomic data using an optimal transport plan"
- [discussion] OT plan hyperparameters and training: "We identified k = 2000 highly variable common genes across cells of all datasets, and identified kx = ky = 2000 highly variable genes for each dataset"
- [discussion] Memory caveat for dense OT plan: "Note that storing a dense matrix T needs more memory. Therefore, computing capacity with respect to memory should be ascertained before using this option"
- [methods] mapCluster R invocation on BRCA data: "ot_map <- mapCluster(ot,meta = brca_cluster, cluster = 'celltype_major')"
- [readme] README example saving the global OT matrix: "# save global optimal transport matrix: adata, OT = up.Run(adatas=[adata1, adata2], adata_cm=adata_cm, save_OT=True)"
- [readme] README save_OT memory warning: "**save_OT**: if True, output a global OT plan. Need more memory. Default: False"
