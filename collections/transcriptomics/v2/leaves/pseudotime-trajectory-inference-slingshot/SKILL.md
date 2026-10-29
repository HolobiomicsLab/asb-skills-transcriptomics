---
name: pseudotime-trajectory-inference-slingshot
description: Use when a single-cell RNA-seq dataset (e.g. GSE81682 mouse HSPC from
  Nestorowa et al., 2016) must be ordered along a pseudotime trajectory with slingshot
  and visualized as group-wise colour groups in CellexalVR.
when_to_use_negative:
- The analysis goal is differential expression between predefined groups rather than
  trajectory ordering along a hand-traced path
- The dataset has no biologically meaningful start/stop clusters, so slingshot pseudotime
  would be uninterpretable
- Input is not single-cell expression data (e.g. bulk RNA-seq or already-computed
  pseudotime values)
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: slingshot
  role: Infers the pseudotime trajectory on the four-cluster partition of selected
    cells; cells are ordered by returned pseudotime values
- name: cellexalvrR
  role: R backend that converts data via as_cellexalvrR/export2cellexalvr and performs
    in-session slingshot pseudotime and heatmap calculations
  repo: https://github.com/sonejilab/cellexalvrR
- name: Seurat
  role: Processes raw GEO expression data (e.g. GSE81682) to a normalized matrix before
    conversion to a cellexalvrR object
- name: CellexalVR
  role: VR front end where the user hand-traces the start/stop path and visualizes
    pseudotime groups on 3D reductions
  repo: https://github.com/sonejilab/cellexalvr
- name: R
  role: Language in which the backend pseudotime and clustering scripts run (productively
    used under R 4.0.2 and 4.1.0)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:7076e7f68006461be6ecb663e84a256a75538541337aac1498c5942d958e940d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sonejilab/cellexalvrR
  tools:
  - CellexalVR
  - R
  - Seurat
  - cellexalvrR
  - slingshot
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# pseudotime-trajectory-inference-slingshot

## Summary

Runs slingshot-based pseudotime trajectory inference on selected cells partitioned into four clusters, orders cells by pseudotime, splits them into ten colour groups, and generates a hierarchically clustered heatmap of correlated genes — the in-session trajectory workflow of CellexalVR's cellexalvrR backend.

## When to use

Use when a single-cell RNA-seq dataset (e.g. GSE81682 mouse HSPC from Nestorowa et al., 2016) must be ordered along a pseudotime trajectory with slingshot and visualized as group-wise colour groups in CellexalVR.

## When NOT to use

- The analysis goal is differential expression between predefined groups rather than trajectory ordering along a hand-traced path
- The dataset has no biologically meaningful start/stop clusters, so slingshot pseudotime would be uninterpretable
- Input is not single-cell expression data (e.g. bulk RNA-seq or already-computed pseudotime values)

## Inputs

- Normalized single-cell expression matrix (e.g. Seurat-processed GSE81682 mouse HSPC data)
- CellexalVR input files exported via export2cellexalvr from a cellexalvrR object
- User-selected cell set with a hand-traced path defining start and stop clusters
- Cluster partition of the selected cells (four clusters)

## Outputs

- Pseudotime-ordered cell list with ten balanced colour-group labels (pseudotime_ordering.tsv)
- Heatmap of genes correlated to pseudotime with hierarchical clustering of top marker genes (pseudotime_heatmap.png)

## How to apply

Process the raw GEO data (e.g. GSE81682) with Seurat to obtain a normalized expression matrix, convert it with as_cellexalvrR, and export via export2cellexalvr so backend R scripts can consume it. Partition the selected cells into four clusters — the fixed parameter CellexalVR uses for slingshot analysis — and run slingshot to compute the trajectory. Order cells by returned pseudotime values and split them into ten even colour groups for group-wise visualization. Identify genes correlated to pseudotime (Wilcox-test pipeline) and build a heatmap of the top N marker genes (default 250) with hierarchical clustering. Hand-trace a start/stop path in the 3D reduction so start and stop clusters are defined before inference.

## Related tools

- **slingshot** (Infers the pseudotime trajectory on the four-cluster partition of selected cells; cells are ordered by returned pseudotime values)
- **cellexalvrR** (R backend that converts data via as_cellexalvrR/export2cellexalvr and performs in-session slingshot pseudotime and heatmap calculations) — https://github.com/sonejilab/cellexalvrR
- **Seurat** (Processes raw GEO expression data (e.g. GSE81682) to a normalized matrix before conversion to a cellexalvrR object)
- **CellexalVR** (VR front end where the user hand-traces the start/stop path and visualizes pseudotime groups on 3D reductions) — https://github.com/sonejilab/cellexalvr
- **R** (Language in which the backend pseudotime and clustering scripts run (productively used under R 4.0.2 and 4.1.0))

## Examples

```
devtools::install_github("sonejilab/cellexalvrR"); library(cellexalvrR)
```

## Evaluation signals

- pseudotime_heatmap.png exists and its gene rows are ordered by hierarchical clustering
- pseudotime_ordering.tsv has one row per analyzed cell with ten balanced colour-group labels
- Selected cells were partitioned into exactly four clusters before slingshot was run
- Heatmap uses the top N marker genes with N defaulting to 250 (Wilcox-test-based pipeline)
- CellexalVR generates an HTML report of session actions after the run completes

## Limitations

- Backend calculations such as identifying differentially expressed genes will take longer to complete depending on how many cells/groups are selected
- A VR-ready workstation with a suitable graphics card (e.g. NVIDIA GTX1060 or higher) running Windows 10 is required
- Only HTC Vive and Valve Index were used during development; compatibility for other VR systems was not yet added

## Evidence

- [discussion] Pseudotime trajectories are calculated by partitioning the selected cells into four clusters which are used for a slingshot analysis.: "Pseudotime trajectories are calculated by partitioning the selected cells into four clusters which are used for a slingshot analysis."
- [discussion] Hierarchical clustering of top N marker genes (default 250): "clustering is performed using hierarchical clustering for the top N marker genes (default is 250)"
- [discussion] Wilcox-test heatmap pipeline: "Differentially expressed genes for heatmaps are identified using the Wilcox test (implemented in C++ for speed)"
- [readme] cellexalvrR backend role: [evidence span withheld — anchor status: external_doc]
