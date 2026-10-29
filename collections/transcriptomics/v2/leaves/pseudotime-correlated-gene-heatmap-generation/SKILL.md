---
name: pseudotime-correlated-gene-heatmap-generation
description: Use when pseudotime trajectories are inferred for a CellexalVR cell selection
  (e.g. slingshot on four clusters) and a pseudotime-ordered heatmap of correlated
  marker genes with ten colour groups is required.
when_to_use_negative:
- The analysis goal is differential expression between discrete groups rather than
  ordering along a trajectory — the Wilcox-test heatmap pipeline is only applied after
  pseudotime inference
- The dataset has no meaningful developmental or continuous progression, so partitioning
  into four slingshot clusters and splitting into ten colour groups would produce
  an arbitrary ordering
- Data has not been converted to CellexalVR input files via as_cellexalvrR/export2cellexalvr,
  so backend R scripts cannot consume it
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellexalVR
  role: VR platform whose hand-traced path defines start/stop clusters and whose InputReader
    consumes backend results to build the pseudotime heatmap
  repo: https://github.com/sonejilab/cellexalvr
- name: cellexalvrR
  role: R package providing the in-session backend functions for slingshot pseudotime,
    Wilcox-test gene correlation, hierarchical clustering, and heatmap/report generation
  repo: https://github.com/sonejilab/cellexalvrR
- name: slingshot
  role: Trajectory inference on the four-cluster partition to compute pseudotime values
    used to order cells
- name: Seurat
  role: Pre-processing of raw GEO expression data (e.g. GSE81682) into a normalized
    matrix before conversion to a cellexalvrR object
- name: R
  role: Runtime for the cellexalvrR backend scripts invoked via Rscript.exe by CellexalVR
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:f3506b771c52e84b283fe99d0ec747a36fbac644f7ea108eef109d78484ace1b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sonejilab/cellexalvr
  tools:
  - CellexalVR
  - R
  - Seurat
  - cellexalvrR
  - slingshot
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# pseudotime-correlated-gene-heatmap-generation

## Summary

Generates a heatmap of genes correlated to slingshot-inferred pseudotime, as implemented in CellexalVR's in-session backend: selected cells are partitioned into four clusters, ordered by pseudotime, split into ten colour groups, and the top N marker genes (default 250) are hierarchically clustered and rendered as a heatmap.

## When to use

Use when pseudotime trajectories are inferred for a CellexalVR cell selection (e.g. slingshot on four clusters) and a pseudotime-ordered heatmap of correlated marker genes with ten colour groups is required.

## When NOT to use

- The analysis goal is differential expression between discrete groups rather than ordering along a trajectory — the Wilcox-test heatmap pipeline is only applied after pseudotime inference
- The dataset has no meaningful developmental or continuous progression, so partitioning into four slingshot clusters and splitting into ten colour groups would produce an arbitrary ordering
- Data has not been converted to CellexalVR input files via as_cellexalvrR/export2cellexalvr, so backend R scripts cannot consume it

## Inputs

- Normalized single-cell expression matrix (e.g. GSE81682 mouse HSPC data processed with Seurat)
- cellexalvrR object produced by as_cellexalvrR and exported with export2cellexalvr
- User-defined start and stop clusters from a hand-traced cell-selection path
- Four-cluster partition of the selected cells for slingshot

## Outputs

- pseudotime_heatmap.png with gene rows ordered by hierarchical clustering of top marker genes (default N=250)
- pseudotime_ordering.tsv with one row per analyzed cell and ten balanced colour-group labels
- Slingshot trajectory and pseudotime values for the selected cells

## How to apply

Define the trajectory by having the user hand-trace a path and identify start and stop clusters from that selection, then partition the selected cells into four clusters which are used for a slingshot analysis. Run slingshot on this four-cluster partition to compute the trajectory and order cells by the returned pseudotime values, then split the ordered cells into ten even colour groups for group-wise visualization in VR. Identify genes correlated to pseudotime using the Wilcox test (implemented in C++ for speed), take the top N marker genes (default 250), and perform hierarchical clustering on them to order the heatmap rows. Compute the reduction-anchored heatmap on-the-fly in 3D reductions via the cellexalvrR backend, which CellexalVR's InputReader consumes to build the heatmap object. Validate by checking that pseudotime_heatmap.png exists, gene rows are ordered by hierarchical clustering, and pseudotime_ordering.tsv has one row per analyzed cell with ten balanced colour-group labels.

## Related tools

- **CellexalVR** (VR platform whose hand-traced path defines start/stop clusters and whose InputReader consumes backend results to build the pseudotime heatmap) — https://github.com/sonejilab/cellexalvr
- **cellexalvrR** (R package providing the in-session backend functions for slingshot pseudotime, Wilcox-test gene correlation, hierarchical clustering, and heatmap/report generation) — https://github.com/sonejilab/cellexalvrR
- **slingshot** (Trajectory inference on the four-cluster partition to compute pseudotime values used to order cells)
- **Seurat** (Pre-processing of raw GEO expression data (e.g. GSE81682) into a normalized matrix before conversion to a cellexalvrR object)
- **R** (Runtime for the cellexalvrR backend scripts invoked via Rscript.exe by CellexalVR)

## Examples

```
install.packages("devtools"); devtools::install_github("sonejilab/cellexalvrR"); library(cellexalvrR)
```

## Evaluation signals

- pseudotime_heatmap.png exists and its gene rows are ordered according to hierarchical clustering of the top marker genes
- pseudotime_ordering.tsv has exactly one row per analyzed cell
- The ten colour-group labels in pseudotime_ordering.tsv are balanced (evenly sized groups)
- The selected cells were partitioned into four clusters matching the slingshot setting before trajectory computation
- Cells appear in pseudotime_ordering.tsv ordered by the slingshot-returned pseudotime values

## Limitations

- Backend calculations such as identifying differentially expressed genes take longer to complete depending on how many cells/groups are selected, so large selections slow heatmap generation
- Requires a VR-ready workstation (e.g. NVIDIA GTX1060 or higher, Windows 10) with R >= 4.0.0 and the cellexalvrR package installed; pandoc is needed for HTML reports
- Compatibility was only established with HTC Vive and Valve Index during development

## Evidence

- [discussion] Pseudotime trajectories are calculated by partitioning the selected cells into four clusters which are used for a slingshot analysis.: "Pseudotime trajectories are calculated by partitioning the selected cells into four clusters which are used for a slingshot analysis."
- [discussion] clustering is performed using hierarchical clustering for the top N marker genes (default is 250): "clustering is performed using hierarchical clustering for the top N marker genes (default is 250)"
- [discussion] Differentially expressed genes for heatmaps are identified using the Wilcox test (implemented in C++ for speed): "Differentially expressed genes for heatmaps are identified using the Wilcox test (implemented in C++ for speed)"
- [readme] cellexalvrR is an R package to prep single-cell expression data for exploration/use in our virtual realitiy data analysis platform CellexalVR, and also to provide back-end functions that perform in-session calculations.: [evidence span withheld — anchor status: external_doc]
- [readme] The InputReader script reads the data from the CellexalVR R package and calls the appropriate functions to create graphs and transcription factor networks: "The [InputReader](Assets/Prefabs/Environment/InputReader.prefab) gameobject holds the [InputReader](Assets/Scripts/AnalysisLogic/InputReader.cs) script, which reads the data from the CellexalVR R"
