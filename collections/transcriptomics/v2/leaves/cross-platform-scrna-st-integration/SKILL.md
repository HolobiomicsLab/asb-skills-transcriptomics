---
name: cross-platform-scrna-st-integration
description: Use when a paired scRNA-seq reference and spatial transcriptomics spots
  (e.g. ST, Visium, MERFISH) need cell-type deconvolution/label transfer, especially
  when datasets share only partially overlapping genes or are heterogeneous (e.g.
  PDAC ST + inDrop scRNA).
when_to_use_negative:
- When the ST and scRNA data derive from the same cells (paired data) and a vertical-integration
  approach is needed — use uniPort's mode='v' scenario instead of spatial deconvolution
  via OT
- When only common genes exist and dataset-specific gene leverage is unnecessary,
  the coupled-VAE/UOT machinery with adata_cm is overkill; horizontal integration
  with common genes alone suffices
- 'When memory is constrained: storing the dense global OT plan T ''needs more memory'',
  so save_OT=True is the wrong choice for very large spot-by-cell problems'
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: uniPort
  role: Performs coupled-VAE + Minibatch-UOT integration of ST and scRNA data and
    outputs the OT plan used for label transfer/deconvolution
  repo: https://github.com/caokai1073/uniPort
- name: scanpy
  role: 'Preprocessing: normalize_total, log1p, highly variable gene identification,
    and scaling of ST/scRNA matrices'
- name: Seurat
  role: Mentioned as a prior single-cell integration platform (CCA-based) contrasted
    with uniPort's OT approach
- name: scatterpie
  role: R visualization of deconvolved spot-level cluster proportions via stClusterPie
- name: data.table
  role: R parsing of the OT matrix and cell-type metadata for mapCluster
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-35094-8
    title: A unified computational framework for single-cell data integration with
      optimal transport
  source_content_hash: sha256:3edd2ab824beac34eafd34e2da341e45e425bf18b61086b5a2774620bb80dcb1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/caokai1073/uniPort
  tools:
  - Seurat
  - data.table
  - scanpy
  - scatterpie
  - uniPort
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cross-platform-scrna-st-integration

## Summary

Integrates scRNA-seq reference data with spatial transcriptomics (ST) data via uniPort's coupled-VAE plus Minibatch unbalanced optimal transport (Minibatch-UOT), then deconvolves spot composition using the global OT plan rather than the shared latent space. This enables label transfer and cell-type (or cancer-clone) proportion estimation on heterogeneous spot data.

## When to use

Use when a paired scRNA-seq reference and spatial transcriptomics spots (e.g. ST, Visium, MERFISH) need cell-type deconvolution/label transfer, especially when datasets share only partially overlapping genes or are heterogeneous (e.g. PDAC ST + inDrop scRNA).

## When NOT to use

- When the ST and scRNA data derive from the same cells (paired data) and a vertical-integration approach is needed — use uniPort's mode='v' scenario instead of spatial deconvolution via OT
- When only common genes exist and dataset-specific gene leverage is unnecessary, the coupled-VAE/UOT machinery with adata_cm is overkill; horizontal integration with common genes alone suffices
- When memory is constrained: storing the dense global OT plan T 'needs more memory', so save_OT=True is the wrong choice for very large spot-by-cell problems

## Inputs

- ST/spatial expression matrix with spot coordinates (e.g. GEO GSE111672 PDAC batch-A ST, 428 spots; coordinates parsed from 'x'-delimited column names)
- Paired scRNA-seq reference matrix with cell-type labels in metadata (e.g. inDrop scRNA, 1926 single cells)
- Preprocessed AnnData objects: dataset-specific HVG matrices (adata1, adata2) and common HVG matrix (adata_cm)

## Outputs

- Integrated cell embeddings in adata.obs['latent']
- Global optimal transport plan matrix (returned when save_OT=True)
- Per-spot cell-type/cluster proportions from mapCluster (e.g. 15 clusters including ductal, acinar, Cancer clone A and B)
- Pie-chart spot proportion maps (stClusterPie) and cluster expression maps (stClusterExp)

## How to apply

Preprocess both datasets in scanpy with normalize_total and log1p, identify k=2000 highly variable common genes plus dataset-specific HVGs, and scale each gene to 0-1 with MaxAbsScaler. Call uniport.Run on the ST and scRNA AnnData objects with save_OT=True (default 30000 training iterations) to obtain integrated cell embeddings and the global OT plan; use mode='h' when common genes exist and supply adata_cm for the common HVG matrix. For spatial deconvolution, pass the OT matrix to mapCluster with the cell_type annotation (e.g. min_cut=0.25, balance=True) and scale spot proportions by the cluster ratio in the scRNA reference. Visualize spot-level proportions with stClusterPie (e.g. pie_scale=0.8) using parsed spot coordinates, and plot cluster expression maps with stClusterExp (cut=0.25). The rationale is that the OT plan provides a flexible label transfer mechanism that works directly on optimal transport rather than embedding latent space, handling dataset-specific genes and heterogeneity.

## Related tools

- **uniPort** (Performs coupled-VAE + Minibatch-UOT integration of ST and scRNA data and outputs the OT plan used for label transfer/deconvolution) — https://github.com/caokai1073/uniPort
- **scanpy** (Preprocessing: normalize_total, log1p, highly variable gene identification, and scaling of ST/scRNA matrices)
- **Seurat** (Mentioned as a prior single-cell integration platform (CCA-based) contrasted with uniPort's OT approach)
- **scatterpie** (R visualization of deconvolved spot-level cluster proportions via stClusterPie)
- **data.table** (R parsing of the OT matrix and cell-type metadata for mapCluster)

## Examples

```
import uniport as up; adata, OT = up.Run(adatas=[adata_st, adata_scrna], adata_cm=adata_cm, save_OT=True)
```

## Evaluation signals

- Deconvolution resolves the expected cluster structure (e.g. 15 clusters in PDAC with a clear Cancer clone A vs Cancer clone B split)
- Cluster-composition differences between clone regions are statistically significant via a two-sided t-test (n=36 vs n=41 regions), with clone A showing higher hematogenous cell (RBC, T cell, NK) proportions
- The OT plan output is non-degenerate: mapCluster with min_cut=0.25 yields non-empty cluster assignments when scaled by scRNA cluster ratios
- Training converges within the default 30000 iterations, and save_OT=True returns both embeddings and the global OT matrix without memory failure
- Spot proportion maps align with parsed spot coordinates and expression maps (stClusterExp cut=0.25) show spatially coherent Cancer clone A and Ductal patterns

## Limitations

- Storing the dense full OT plan T requires substantial memory; computing capacity should be ascertained before using save_OT=True
- Which hyperparameter combination from the searched grids (lambda in {0.5,1.0,5.0}, gamma in {0.5,1.0}, lambda_s in {0.5,1.0}) is optimal per dataset is not stated; selection required manual search
- Validation used only batch A of the PDAC spatial dataset with scRNA from the same sample; generalization to unpaired samples is not demonstrated here

## Evidence

- [abstract] label transfer framework: "uniPort provides a flexible label transfer framework to deconvolute heterogeneous spatial transcriptomic data using an optimal transport plan, instead of embedding latent space."
- [discussion] preprocessing: "We normalized total counts of each cell using the scanpy.pp.normalize_total function in the scanpy package"
- [readme] save_OT parameter: "save_OT: if True, output a global OT plan. Need more memory. Default: False"
- [methods] mapCluster invocation: "ot_map <- mapCluster(ot,meta = brca_cluster, cluster = 'celltype_major')"
- [discussion] PDAC dataset scope: "The spatial dataset of batch A was adopted together with scRNA-seq generated from the same sample"
