---
name: subsampling-based-clustering-resolution-selection
description: Use when scRNA-seq count data must be clustered but the Louvain resolution
  is not known a priori and an objective, subsampling-based choice among resolutions
  0.4–1.4 is required (e.g., sub-clustering tumor cells in an h5ad AnnData object).
when_to_use_negative:
- The resolution is already known or user-specified — use the -r/--resolution flag
  (default 0.6) instead of --auto-resolution
- Input is not cell-level count data (e.g., already a feature/summary table or bulk
  expression matrix); subsampling-based Louvain robustness requires per-cell profiles
- The goal is downstream drug response prediction (IC50/AUC) rather than choosing
  a clustering resolution
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scDrug
  role: Hosts the Single-Cell Data Analysis pipeline whose --auto-resolution flag
    implements the subsampling-based resolution sweep and robustness scoring
  repo: https://github.com/ailabstw/scDrug
- name: Scanpy
  role: Performs the preprocessing (filtering, normalization to 10,000 counts, log
    transform, HVG annotation, scaling), PCA, and Louvain clustering whose assignments
    are scored across resolutions
- name: Harmony
  role: Adjusts the principal components to eliminate batch effects before graph building
    so that the resolution selection is not confounded by batch structure
- name: MAGIC
  role: Optionally imputes missing values in the expression matrix prior to PCA/clustering
    when imputation is needed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:e580615dedc8e235162bc1e6dd725a5deb5acfefed00c0611179b9e67267b764
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ailabstw/scDrug
  tools:
  - Harmony
  - MAGIC
  - Python
  - Scanpy
  - scDrug
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# subsampling-based-clustering-resolution-selection

## Summary

Automatically selects the Louvain clustering resolution for scRNA-seq data by scoring the robustness of cluster assignments across repeated 80% subsamples over a resolution grid of [0.4, 1.4] with 0.2 spacing, as implemented in scDrug's Single-Cell Data Analysis module (--auto-resolution).

## When to use

Use when scRNA-seq count data must be clustered but the Louvain resolution is not known a priori and an objective, subsampling-based choice among resolutions 0.4–1.4 is required (e.g., sub-clustering tumor cells in an h5ad AnnData object).

## When NOT to use

- The resolution is already known or user-specified — use the -r/--resolution flag (default 0.6) instead of --auto-resolution
- Input is not cell-level count data (e.g., already a feature/summary table or bulk expression matrix); subsampling-based Louvain robustness requires per-cell profiles
- The goal is downstream drug response prediction (IC50/AUC) rather than choosing a clustering resolution

## Inputs

- Raw scRNA-seq count matrix as a 10x-Genomics-formatted mtx directory or CSV file (single_cell_analysis.py -i)
- Previously produced Scanpy AnnData (h5ad) for sub-clustering, with --clusters specifying target clusters (e.g. '1,3,8,9')
- Optional metadata CSV with a batch column (e.g. 'PatientID') for Harmony batch correction

## Outputs

- scanpyobj.h5ad — Scanpy AnnData object with a categorical 'louvain' observation field and resolution scores
- umap_cluster.png — UMAP visualization of the clustering result
- cluster_DEGs.csv — differentially expressed genes per cluster

## How to apply

After Scanpy preprocessing (filter cells <200 genes and genes <3 cells, keep mitochondrial proportion <30%; normalize to 10,000 total counts, log-transform, annotate highly variable genes, scale), compute PCA and optionally adjust the top PCs with Harmony to eliminate batch effects. Build the neighborhood graph from the top 20 PCs, then sweep Louvain resolutions in the interval [0.4, 1.4] with 0.2 spacing; for each resolution, repeat clustering 5 times on random 80% subsets drawn without replacement and compute the robustness score as the average silhouette over a distance matrix defined as 1.0 minus the co-clustering frequency. Select the resolution with the highest robustness score and record the final Louvain assignments computed on the full dataset. Save the clustered AnnData (with cluster assignments and resolution scores) as scanpyobj.h5ad. As a reference point, on GSE156625 this procedure yielded 15 cell groups overall, and sub-clustering of the tumor subpopulation at the automatically determined resolution of 0.6 produced 14 sub-clusters after Harmony correction.

## Related tools

- **scDrug** (Hosts the Single-Cell Data Analysis pipeline whose --auto-resolution flag implements the subsampling-based resolution sweep and robustness scoring) — https://github.com/ailabstw/scDrug
- **Scanpy** (Performs the preprocessing (filtering, normalization to 10,000 counts, log transform, HVG annotation, scaling), PCA, and Louvain clustering whose assignments are scored across resolutions)
- **Harmony** (Adjusts the principal components to eliminate batch effects before graph building so that the resolution selection is not confounded by batch structure)
- **MAGIC** (Optionally imputes missing values in the expression matrix prior to PCA/clustering when imputation is needed)

## Examples

```
python3 single_cell_analysis.py -f h5ad --input scanpyobj.h5ad --clusters 1,3,8,9 --auto-resolution --cpus 4
```

## Evaluation signals

- scanpyobj.h5ad exists, loads as a valid AnnData object, and contains a 'louvain' categorical observation field
- The chosen resolution lies within the [0.4, 1.4] grid
- For each resolution in the grid, the robustness score is computed as the average silhouette over 5 repetitions on random 80% subsets, using a distance of 1.0 minus co-clustering frequency
- Neighborhood graph is built from the top 20 PCs, with Harmony batch correction applied when metadata/batch information is supplied
- Reproduction check: GSE156625 full-dataset Louvain clustering yields 15 cell groups, and tumor sub-clustering at the auto-determined resolution of 0.6 yields 14 sub-clusters

## Limitations

- The resolution sweep is restricted to [0.4, 1.4] with 0.2 spacing; structures requiring coarser or finer resolutions are outside the searchable grid
- Without --auto-resolution, single_cell_analysis.py defaults to resolution 0.6, which may not suit datasets with different granularity
- Robustness scores depend on the 5×80% subsampling scheme; batch effects must be corrected (e.g., Harmony on the top PCs) beforehand or the resolution choice may reflect batch structure rather than biology

## Evidence

- [methods] resolution values in the interval [0.4, 1.4] with 0.2 spacing: "resolution values in the interval [0.4, 1.4] with 0.2 spacing"
- [methods] Harmony PC adjustment before clustering: "adjusted the principal components (PCs) with the Harmony algorithm to eliminate batch effects if needed"
- [readme] auto-resolution CLI flag for automatic resolution determination: "--auto-resolution             automatically determine resolution for clustering"
- [readme] sub-clustering at automatically determined resolution under 4 CPUs: "python3 single_cell_analysis.py -f h5ad --input scanpyobj.h5ad --clusters CLUSTERS --auto-resolution --cpus 4"
