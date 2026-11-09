---
name: gene-expression-profile-aggregation
description: Use when a clustered scRNA-seq AnnData object (scanpyobj.h5ad) must be
  reduced to per-cluster average expression profiles for cell-type annotation or to
  match a cell line for LINCS L1000 deconvolution.
when_to_use_negative:
- Input is bulk RNA-seq or a bulk expression matrix with no clustered scRNA-seq AnnData
  — there are no clusters over which to aggregate a GEP.
- The goal is per-single-cell annotation rather than cluster-level annotation; scMatch
  itself annotates single cells, but in scDrug the GEP aggregation stage specifically
  targets cluster-wise profiles.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDrug
  role: Wrapper pipeline; its single_cell_analysis.py computes the cluster-wise GEP
    file GEP.txt and orchestrates annotation (--GEP, --annotation flags)
  repo: https://github.com/ailabstw/scDrug
- name: Scanpy
  role: Provides the clustered AnnData (h5ad) whose per-cluster cells are averaged
    to form the GEP
- name: scMatch
  role: Annotates each cluster by comparing its GEP to 916 cell types in the truncated
    FANTOM5 reference dataset
  repo: https://github.com/asrhou/scMatch
- name: CIBERSORTx
  role: Consumes the single-cell GEPs and anti-log LINCS L1000 bulk GEPs of the selected
    cell line (Docker image cibersortx/fractions) to impute subpopulation fractions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:e5145dd2b2ffc120286df839bddd12f45ecec759ae81592baedbe11f15038649
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ailabstw/scDrug
  tools:
  - CIBERSORTx
  - Python
  - Scanpy
  - scDrug
  - scMatch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# gene-expression-profile-aggregation

## Summary

Computes cluster-wise gene expression profiles (GEPs) as the mean expression of each Scanpy cluster across all genes, then uses these GEPs for cluster cell-type annotation via scMatch against the FANTOM5 reference (916 cell types) and as the single-cell input to CIBERSORTx bulk deconvolution.

## When to use

Use when a clustered scRNA-seq AnnData object (scanpyobj.h5ad) must be reduced to per-cluster average expression profiles for cell-type annotation or to match a cell line for LINCS L1000 deconvolution.

## When NOT to use

- Input is bulk RNA-seq or a bulk expression matrix with no clustered scRNA-seq AnnData — there are no clusters over which to aggregate a GEP.
- The goal is per-single-cell annotation rather than cluster-level annotation; scMatch itself annotates single cells, but in scDrug the GEP aggregation stage specifically targets cluster-wise profiles.

## Inputs

- clustered Scanpy AnnData object (scanpyobj.h5ad) from Single-Cell Data Analysis
- truncated FANTOM5 reference dataset (decompressed from scMatch refDB/FANTOM5)
- bulk gene expression profiles of the eight supported cell lines (A375, A549, HCC515, HEPG2, HT29, MCF7, PC3, YAPC)
- LINCS L1000 bulk perturbation GEPs for the selected cell line

## Outputs

- GEP.txt — cluster-wise gene expression profile file
- annotated scanpyobj.h5ad with per-cluster cell type assignments
- umap_cell_type.png — UMAP colored by annotated cell type
- CIBERSORTx_Adjusted.txt and CIBERSORTx_Results.txt — per-sample fractions of each single-cell subpopulation

## How to apply

Load the clustered Scanpy AnnData object (scanpyobj.h5ad) and compute each cluster's gene expression profile as the mean expression of its cells across all genes, saving it as GEP.txt (invoked with the --GEP flag of single_cell_analysis.py). Run scMatch on the cluster-wise GEPs against the truncated FANTOM5 reference dataset (916 cell types) to assign a known cell type to each cluster, and record the annotation in the AnnData object before generating the umap_cell_type.png. When a reference cell line is needed for bulk deconvolution, compute the Pearson correlation coefficient between the single-cell GEP and the bulk GEP of each of the eight supported cell lines (A375, A549, HCC515, HEPG2, HT29, MCF7, PC3, YAPC) and select the highest-correlating one (or accept a user-specified cell type). Finally, extract the anti-logarithmic bulk GEPs of all LINCS L1000 perturbation experiments on the selected cell line and supply them together with the single-cell GEPs to CIBERSORTx Cell Fractions to impute subpopulation fractions per bulk sample.

## Related tools

- **scDrug** (Wrapper pipeline; its single_cell_analysis.py computes the cluster-wise GEP file GEP.txt and orchestrates annotation (--GEP, --annotation flags)) — https://github.com/ailabstw/scDrug
- **Scanpy** (Provides the clustered AnnData (h5ad) whose per-cluster cells are averaged to form the GEP)
- **scMatch** (Annotates each cluster by comparing its GEP to 916 cell types in the truncated FANTOM5 reference dataset) — https://github.com/asrhou/scMatch
- **CIBERSORTx** (Consumes the single-cell GEPs and anti-log LINCS L1000 bulk GEPs of the selected cell line (Docker image cibersortx/fractions) to impute subpopulation fractions)

## Examples

```
python3 single_cell_analysis.py --input INPUT --annotation --GEP
```

## Evaluation signals

- GEP.txt contains one column/row per cluster whose values are the mean expression of that cluster's cells across all genes
- Each cluster receives exactly one known cell type assigned by scMatch from the 916 FANTOM5 reference cell types
- umap_cell_type.png exists and is a readable image showing the cell-type-labeled UMAP
- The reference cell line chosen for deconvolution has the highest Pearson correlation coefficient between single-cell GEP and bulk GEP among the eight cell lines
- CIBERSORTx_Adjusted.txt is a tabular file with per-sample fraction values for each identified cell subpopulation

## Limitations

- CIBERSORTx Cell Fractions requires the cibersortx/fractions Docker image and a registered username and token; if docker-in-docker fails inside the scDrug container, CIBERSORTx must be run outside with a shared mounted folder.
- The FANTOM5 reference files must be decompressed before use, and annotation operates at cluster level, so mixed or unclustered populations get a single averaged identity.

## Evidence

- [methods] Cell-type annotation is downstream of cluster GEP computation: "functional enrichment by GSEAPY [22], [23] and cell-type annotation by scMatch [24]"
- [readme] README names the GEP.txt output of Single-Cell Data Analysis: "a UMAP `umap_cluster.png` and differentially expressed genes (DEGs) `cluster_DEGs.csv` of the clustering result, and a gene expression profile (GEP) file `GEP.txt`."
- [readme] README identifies scMatch as the annotation tool: "For cell type annotation, we used [scMatch: a single-cell gene expression profile annotation tool using reference datasets](https://github.com/asrhou/scMatch)."
- [other] scMatch mechanism: closest match in reference datasets: [evidence span withheld — anchor status: external_doc]
- [readme] README CLI flag that triggers GEP generation: "--GEP              generate Gene Expression Profile file."
