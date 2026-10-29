---
name: spatial-transcriptomic-deconvolution
description: Use when a spatial transcriptomics dataset (e.g. 10x Visium, ST microarray,
  STARmap spots) must be deconvoluted into cell-type proportions using an annotated
  scRNA-seq reference and the datasets share only a subset of genes.
when_to_use_negative:
- No annotated scRNA reference with cell-type labels is available, since the OT-plan
  label transfer maps spots to reference cell-type annotations
- Memory capacity cannot hold a dense OT plan matrix — storing the dense matrix T
  requires substantial memory and should be ascertained before using save_OT=True
- Datasets are paired profiles of the same cells (vertical integration), where mode='v'
  rather than the deconvolution-oriented mode='h' workflow applies
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
tools:
- name: uniPort
  role: 'Core method: trains coupled-VAE + Minibatch-UOT (up.Run with save_OT=True)
    and provides OT-plan-based label transfer (metrics.label_transfer / R mapCluster,
    min_cut=0.25, balance=True) for spot deconvolution'
  repo: https://github.com/caokai1073/uniport
- name: scanpy
  role: 'Preprocessing of ST and scRNA inputs: normalize_total, log1p (offset 1),
    HVG identification (k=2000 common, kx=ky=2000 dataset-specific)'
- name: Seurat
  role: Loading 10x Visium data (Load10X_Spatial) with NormalizeData/ScaleData and
    handling scRNA reference cluster annotations
- name: scikit-learn
  role: MaxAbsScaler rescales each gene to the 0-1 range within each dataset prior
    to integration
- name: scatterpie
  role: Renders spot-level cell-type composition as scatterpie plots on spatial coordinates
    (stClusterPie, pie_scale=0.8)
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-35094-8
    title: A unified computational framework for single-cell data integration with
      optimal transport
  source_content_hash: sha256:48bebed904d9f589e2fb147763aee06609a716fdf55ba204ad2dcdd63660c886
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# spatial-transcriptomic-deconvolution

## Summary

Deconvolve spatial transcriptomics spots into per-spot cell-type proportions by using uniPort's Minibatch-UOT optimal transport plan to transfer cell-type labels from an annotated scRNA reference, rather than relying on latent-space embedding.

## When to use

Use when a spatial transcriptomics dataset (e.g. 10x Visium, ST microarray, STARmap spots) must be deconvoluted into cell-type proportions using an annotated scRNA-seq reference and the datasets share only a subset of genes.

## When NOT to use

- No annotated scRNA reference with cell-type labels is available, since the OT-plan label transfer maps spots to reference cell-type annotations
- Memory capacity cannot hold a dense OT plan matrix — storing the dense matrix T requires substantial memory and should be ascertained before using save_OT=True
- Datasets are paired profiles of the same cells (vertical integration), where mode='v' rather than the deconvolution-oriented mode='h' workflow applies

## Inputs

- Spatial transcriptomics expression matrix (e.g. 10x Visium slice from cellranger 'st' folder, ST microarray matrix from GEO such as GSE111672 batch A, or STARmap spot data 189 spots x 882 genes)
- Annotated scRNA-seq reference (e.g. Allen Institute adult mouse cortical cell taxonomy reference; inDrop PDAC scRNA matrix; synthetic STARmap reference 14249 cells x 34041 genes)
- AnnData object of common highly variable genes (adata_cm, k=2000) shared across ST and scRNA datasets
- Spot coordinate information (e.g. parsed from 'x'-delimited column names in ST matrices)

## Outputs

- Global optimal transport plan matrix (dense OT matrix T, saved when save_OT=True)
- Per-spot cell-type proportion vectors (deconvolution result for each of the cell types, e.g. 28 cortical cell types)
- Integrated cell embeddings in a shared latent space
- Spatial pie-map plots of spot-level composition (e.g. BRCA_celltype_LT_T_i60k_md.png) and cell-type expression maps

## How to apply

Preprocess the ST data and scRNA reference with scanpy (normalize_total, log1p with offset 1, MaxAbsScaler rescaling each gene to 0-1 within each dataset), identifying k=2000 highly variable common genes (adata_cm) plus kx=ky=2000 dataset-specific HVGs. Train the coupled-VAE with Minibatch-UOT via up.Run(adatas=[st_adata, scrna_ref], adata_cm=adata_cm, save_OT=True) in default mode='h' with iteration=30000, so the reference is the last dataset and a global OT plan matrix is output. Transfer labels from the OT plan rows using uniport.metrics.label_transfer or the R mapCluster function (min_cut=0.25, balance=True, scaling spot proportions by the cluster ratio in the scRNA reference). Visualize spot-level proportions as scatterpie plots on parsed spot coordinates (stClusterPie, pie_scale=0.8) and project cell-type expression with stClusterExp (cut=0.25). Rationale: uniPort deconvolutes heterogeneous spatial data using the OT plan itself instead of embedding latent space, and leverages both common and dataset-specific genes to handle cross-platform heterogeneity (e.g. microarray ST vs scRNA, as in the PDAC batch-A GSE111672 task).

## Related tools

- **uniPort** (Core method: trains coupled-VAE + Minibatch-UOT (up.Run with save_OT=True) and provides OT-plan-based label transfer (metrics.label_transfer / R mapCluster, min_cut=0.25, balance=True) for spot deconvolution) — https://github.com/caokai1073/uniport
- **scanpy** (Preprocessing of ST and scRNA inputs: normalize_total, log1p (offset 1), HVG identification (k=2000 common, kx=ky=2000 dataset-specific))
- **Seurat** (Loading 10x Visium data (Load10X_Spatial) with NormalizeData/ScaleData and handling scRNA reference cluster annotations)
- **scikit-learn** (MaxAbsScaler rescales each gene to the 0-1 range within each dataset prior to integration)
- **scatterpie** (Renders spot-level cell-type composition as scatterpie plots on spatial coordinates (stClusterPie, pie_scale=0.8))

## Examples

```
adata, OT = up.Run(adatas=[st_adata, scrna_ref], adata_cm=adata_cm, save_OT=True)
```

## Evaluation signals

- Reproduced deconvolution metrics match published values within tolerance (STARmap benchmark: uniPort PCC ~0.449, RMSE ~0.157, JSD ~0.569 vs Tangram PCC 0.619 and SpaOTsc PCC 0.409)
- Spot-level pie-map shows spatially coherent structure matching known anatomy (e.g. cortical-layer-consistent cell-type distributions in the mouse brain Visium slice)
- Each deconvoluted cell type spatially aligns with its marker-gene expression (Olig1/Olig2 for oligodendrocytes, Atp1b2/Slc1a2 for astrocytes, Dcn/Osr1 for fibroblast-like cells)
- Deconvolution resolves the expected cluster structure (e.g. 15 PDAC clusters with Cancer clone A vs clone B split; significant composition differences by two-sided t-test, n=36 vs n=41 regions)
- Dense OT matrix loaded without exceeding memory when save_OT=True

## Limitations

- Storing the dense full OT plan T requires substantial memory; computing capacity with respect to memory should be ascertained before using this option
- Specific hyperparameter combination selected per dataset from the searched grids (λ ∈ {0.5, 1.0, 5.0}, γ ∈ {0.5, 1.0}, λs ∈ {0.5, 1.0}) is not stated
- On the synthetic STARmap benchmark uniPort underperformed Tangram (PCC 0.449 vs 0.619), though it beat SpaOTsc (PCC 0.409)
- Validation in tumor cases (e.g. HER2+ BRCA TLS detection) is qualitative/expert review; no numeric criterion is reported

## Evidence

- [abstract] OT-plan deconvolution rationale: "uniPort provides a flexible label transfer framework to deconvolute heterogeneous spatial transcriptomic data using an optimal transport plan, instead of embedding latent space."
- [discussion] Preprocessing and HVG parameters: "We identified k = 2000 highly variable common genes across cells of all datasets, and identified kx = ky = 2000 highly variable genes for each dataset"
- [discussion] Gene scaling step: "We normalized values of each gene to the range of 0-1 within each dataset by the MaxAbsScaler function"
- [methods] mapCluster usage on BRCA data: "ot_map <- mapCluster(ot,meta = brca_cluster, cluster = 'celltype_major')"
- [discussion] OT memory caveat: "Note that storing a dense matrix T needs more memory. Therefore, computing capacity with respect to memory should be ascertained before using this option"
- [readme] save_OT and ref_id defaults: "**save_OT**: if True, output a global OT plan. Need more memory. Default: False"
