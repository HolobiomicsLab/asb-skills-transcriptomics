---
name: cluster-wise-cell-type-annotation
description: Use when a clustered scRNA-seq object (scanpyobj.h5ad) needs cell type
  labels per cluster, or when a bulk deconvolution reference cell line must be chosen
  from the eight supported LINCS lines.
when_to_use_negative:
- Input is raw unclustered scRNA-seq data — clustering must precede annotation since
  scMatch scores cluster-average GEPs, not individual cells
- The tissue of interest has no plausible match among the 916 FANTOM5 reference cell
  types or the eight supported LINCS cell lines, and no custom reference is supplied
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scDrug
  role: 'Wrapper pipeline: single_cell_analysis.py --annotation computes cluster GEPs,
    runs scMatch, and writes the cell-type UMAP'
  repo: https://github.com/ailabstw/scDrug
- name: scMatch
  role: Annotates each cluster by matching its average GEP against the FANTOM5 reference
    dataset
  repo: https://github.com/asrhou/scMatch
- name: Scanpy
  role: Holds the clustered AnnData object from which cluster-wise mean expression
    profiles are computed
- name: CIBERSORTx
  role: Cell Fractions module (cibersortx/fractions Docker image) imputes the fraction
    of each single-cell subpopulation in each LINCS bulk sample
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:980427a8984a2e4a85243101207608af69b98170e3c04af162c813d5e529fb35
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# Cluster-wise cell type annotation

## Summary

Assigns known cell type labels to each Louvain cluster of a Scanpy AnnData object by scoring cluster-average gene expression profiles (GEPs) against the FANTOM5 reference with scMatch, then selects a LINCS L1000 cell line (e.g. A375, MCF7) via Pearson correlation for downstream CIBERSORTx deconvolution.

## When to use

Use when a clustered scRNA-seq object (scanpyobj.h5ad) needs cell type labels per cluster, or when a bulk deconvolution reference cell line must be chosen from the eight supported LINCS lines.

## When NOT to use

- Input is raw unclustered scRNA-seq data — clustering must precede annotation since scMatch scores cluster-average GEPs, not individual cells
- The tissue of interest has no plausible match among the 916 FANTOM5 reference cell types or the eight supported LINCS cell lines, and no custom reference is supplied

## Inputs

- Clustered Scanpy AnnData object (scanpyobj.h5ad) from Single-Cell Data Analysis
- Truncated FANTOM5 reference dataset (916 cell types) for scMatch
- Anti-logarithmic bulk GEPs of LINCS L1000 perturbation experiments for one of eight cell lines (A375, A549, HCC515, HEPG2, HT29, MCF7, PC3, YAPC)
- CIBERSORTx registered username and token

## Outputs

- Cluster-wise gene expression profile file (GEP.txt)
- Annotated AnnData with per-cluster cell type labels
- Cell-type UMAP (umap_cell_type.png)
- CIBERSORTx output files (CIBERSORTx_Adjusted.txt, CIBERSORTx_Results.txt) with per-sample subpopulation fractions

## How to apply

Load the clustered Scanpy AnnData (scanpyobj.h5ad) and compute each cluster's GEP as the mean expression of its cells across all genes, saving it as GEP.txt. Run scMatch (with --annotation on single_cell_analysis.py) comparing the cluster-wise GEPs to the truncated FANTOM5 reference dataset (916 cell types) and write the top-scoring cell type into the AnnData; the rationale is that clusters, not single cells, carry enough averaged signal for robust matching. Generate umap_cell_type.png colored by the assigned annotation. For bulk deconvolution, compute the Pearson correlation between the single-cell GEP and the bulk GEP of each of the eight supported cell lines (A375, A549, HCC515, HEPG2, HT29, MCF7, PC3, YAPC) and select the highest-correlating line, unless the user supplies one. Finally, extract anti-logarithmic bulk GEPs of all LINCS L1000 perturbation experiments on that cell line and run CIBERSORTx Cell Fractions (Docker image cibersortx/fractions with a registered username and token) to impute per-sample fractions of each single-cell subpopulation.

## Related tools

- **scDrug** (Wrapper pipeline: single_cell_analysis.py --annotation computes cluster GEPs, runs scMatch, and writes the cell-type UMAP) — https://github.com/ailabstw/scDrug
- **scMatch** (Annotates each cluster by matching its average GEP against the FANTOM5 reference dataset) — https://github.com/asrhou/scMatch
- **Scanpy** (Holds the clustered AnnData object from which cluster-wise mean expression profiles are computed)
- **CIBERSORTx** (Cell Fractions module (cibersortx/fractions Docker image) imputes the fraction of each single-cell subpopulation in each LINCS bulk sample)

## Examples

```
python3 single_cell_analysis.py -f h5ad --input scanpyobj.h5ad --annotation --GEP --cpus 4
```

## Evaluation signals

- umap_cell_type.png exists and opens as a readable image with one label per cluster
- Every cluster in the AnnData received a non-empty cell type annotation from the FANTOM5 reference
- GEP.txt contains one mean-expression profile (column) per Louvain cluster across all genes
- CIBERSORTx_Adjusted.txt is a tabular file with per-sample fraction values for each identified cell subpopulation, and fractions are interpretable as proportions
- Selected reference cell line has the highest Pearson correlation between single-cell GEP and bulk GEP among the eight supported lines (or matches the user-specified cell type)

## Limitations

- Bulk deconvolution is restricted to the eight supported cell lines (A375, A549, HCC515, HEPG2, HT29, MCF7, PC3, YAPC); other tissues require a user-specified reference
- CIBERSORTx requires registration (username and token) and Docker-in-Docker may not work on all machines, in which case the cibersortx/fractions container must be run outside the scDrug container with a shared mount

## Evidence

- [methods] Cluster-wise GEP against FANTOM5: "functional enrichment by GSEAPY [22], [23] and cell-type annotation by scMatch [24]"
- [readme] Annotation flag in CLI: "--annotation          perform cell type annotation"
- [readme] CIBERSORTx Docker image in scDrug: "In the Docker container `scDrug`, pull the Docker image `cibersortx/fractions` used in treatment selection."
- [readme] scMatch output UMAP: "provided additional UMAPs showing batch effects and cell types (`umap_batch.png` and `umap_cell_type.png`)"
- [readme] scMatch design rationale: [evidence span withheld — anchor status: external_doc]
