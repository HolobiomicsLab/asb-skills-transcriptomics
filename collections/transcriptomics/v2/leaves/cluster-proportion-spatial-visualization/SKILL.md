---
name: cluster-proportion-spatial-visualization
description: Use when a spatial transcriptomics dataset (e.g., ST spots with x-delimited
  coordinate column names) has been deconvolved via an OT plan into per-spot cluster
  proportions, and you need spatial pie-chart and cluster-expression maps.
when_to_use_negative:
- Input is raw unprocessed ST/scRNA expression with no OT plan or per-spot cluster
  proportions yet — run uniPort.Run and mapCluster first
- Spatial data lacks spot coordinates (no x/y information to overlay pies on tissue
  space)
- Goal is joint embedding or gene imputation rather than spatially resolved cluster-composition
  visualization
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: uniPort
  role: Provides the spatial plotting helpers (stClusterPie, stClusterExp) in R process/spot/spatial_function.R,
    plus Run/mapCluster that generate the OT plan and proportions being visualized
  repo: https://github.com/caokai1073/uniPort
- name: scatterpie
  role: R plotting backend for rendering per-spot pie charts of deconvolved cluster
    proportions
- name: Seurat
  role: Reference ecosystem for the scRNA cell-type annotations and spot metadata
    handling feeding the visualization
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-35094-8
    title: A unified computational framework for single-cell data integration with
      optimal transport
  source_content_hash: sha256:a7d6899fb3514eceba6119065d30f834d46a054ff81d6048f51be7024e45643a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/caokai1073/uniPort
  tools:
  - Seurat
  - scatterpie
  - uniPort
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# cluster-proportion-spatial-visualization

## Summary

Visualize the per-spot cell-type/cancer-clone proportions obtained from uniPort's optimal-transport label transfer as pie charts overlaid on spatial transcriptomics coordinates, and render expression maps of selected clusters (e.g., Cancer clone A, Ductal) to reveal spatial heterogeneity such as the PDAC clone A vs clone B subtype split.

## When to use

Use when a spatial transcriptomics dataset (e.g., ST spots with x-delimited coordinate column names) has been deconvolved via an OT plan into per-spot cluster proportions, and you need spatial pie-chart and cluster-expression maps.

## When NOT to use

- Input is raw unprocessed ST/scRNA expression with no OT plan or per-spot cluster proportions yet — run uniPort.Run and mapCluster first
- Spatial data lacks spot coordinates (no x/y information to overlay pies on tissue space)
- Goal is joint embedding or gene imputation rather than spatially resolved cluster-composition visualization

## Inputs

- Global OT plan matrix from uniPort.Run(save_OT=True) linking ST spots to scRNA cells
- Per-spot cluster proportion table from mapCluster (min_cut=0.25, balance=True)
- Spot coordinate matrix parsed from 'x'-delimited ST column names (e.g., 428 PDAC batch-A spots)
- Cell-type/cluster annotation metadata (e.g., celltype_major) from the scRNA reference

## Outputs

- Spatial pie-chart plot of per-spot cluster proportions (stClusterPie, pie_scale=0.8)
- Cluster expression spatial maps (stClusterExp, cut=0.25) for clusters such as Cancer clone A and Ductal
- Deconvolved cluster set (e.g., 15 clusters) supporting clone A vs clone B composition comparisons

## How to apply

First run uniPort.Run on the ST and scRNA AnnData objects with save_OT=True to obtain the global optimal transport plan (default 30000 iterations), then apply mapCluster on the OT matrix with the cell_type annotation (min_cut=0.25, balance=True), scaling spot proportions by cluster ratios in the scRNA reference. Parse spot coordinates from the 'x'-delimited column names of the ST matrix (e.g., GSE111672 PDAC batch-A, 428 spots) and feed them together with the deconvolved proportions to the plotting helpers in uniPort's R 'spatial_function.R' (stClusterPie with pie_scale=0.8, stClusterExp with cut=0.25), which are built on scatterpie/Seurat conventions. Render one pie per spot whose slice sizes reflect cluster proportions (ductal, acinar, Cancer clone A, Cancer clone B, etc.), and draw separate expression maps for clusters of interest. Judge success by whether pies align with tissue anatomy, resolve the expected clusters (15 in the PDAC case), and expose composition differences between clone regions.

## Related tools

- **uniPort** (Provides the spatial plotting helpers (stClusterPie, stClusterExp) in R process/spot/spatial_function.R, plus Run/mapCluster that generate the OT plan and proportions being visualized) — https://github.com/caokai1073/uniPort
- **scatterpie** (R plotting backend for rendering per-spot pie charts of deconvolved cluster proportions)
- **Seurat** (Reference ecosystem for the scRNA cell-type annotations and spot metadata handling feeding the visualization)

## Examples

```
ot_map <- mapCluster(ot, meta = brca_cluster, cluster = 'celltype_major')
```

## Evaluation signals

- Pie charts align with parsed spot coordinates and tissue anatomy (428 PDAC batch-A spots)
- Per-spot pie slices correspond to the deconvolved cluster set (15 clusters including ductal, acinar, Cancer clone A/B)
- Proportions derive from an OT plan generated with save_OT=True and mapCluster with min_cut=0.25, balance=True
- Expression maps (stClusterExp, cut=0.25) for Cancer clone A and Ductal show spatially coherent regions matching the pie assignments
- Clone A vs clone B region composition differences are statistically supported (two-sided t-test, n=36 vs n=41 regions)

## Limitations

- Storing the dense OT plan (save_OT=True) required for proportion mapping needs substantial memory; verify capacity beforehand
- Demonstrated on only batch A of the PDAC spatial dataset (428 spots, 1926 single cells); generalization to other batches/samples is untested here
- Proportion resolution depends on min_cut=0.25 and balance=True settings; different thresholds may alter the visible clone split

## Evidence

- [methods] The 'spatial_function.R' is stored `here <https://github.com/caokai1073/uniPort/tree/main/R%20process/spot/>`_.: "The 'spatial_function.R' is stored `here <https://github.com/caokai1073/uniPort/tree/main/R%20process/spot/>`_."
- [abstract] uniPort provides a flexible label transfer framework to deconvolute heterogeneous spatial transcriptomic data using an optimal transport plan: "uniPort provides a flexible label transfer framework to deconvolute heterogeneous spatial transcriptomic data using an optimal transport plan, instead of embedding latent space."
- [methods] ot_map <- mapCluster(ot,meta = brca_cluster, cluster = 'celltype_major'): "ot_map <- mapCluster(ot,meta = brca_cluster, cluster = 'celltype_major')"
- [readme] **save_OT**: if True, output a global OT plan. Need more memory. Default: False: "**save_OT**: if True, output a global OT plan. Need more memory. Default: False"
