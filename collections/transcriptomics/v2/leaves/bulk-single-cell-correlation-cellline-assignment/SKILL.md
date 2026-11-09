---
name: bulk-single-cell-correlation-cellline-assignment
description: Use when scRNA-seq clusters (from scanpyobj.h5ad with a cluster-wise
  GEP, GEP.txt) must be linked to a specific bulk cancer cell line before deconvolving
  LINCS L1000 perturbation samples for subpopulation fractions.
when_to_use_negative:
- When the goal is only cluster-level cell-type naming (e.g. immune/stromal labels)
  — use the scMatch/FANTOM5 annotation step instead of bulk cell-line correlation
- When no LINCS L1000 perturbation data are needed and no bulk deconvolution is required
  (e.g. GDSC/PRISM IC50 prediction via CaDRReS-Sc)
- When the user's bulk reference sample is not one of the eight supported LINCS cell
  lines and no custom reference is available
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDrug
  role: Wraps the cell-line selection (Pearson correlation of single-cell vs bulk
    GEPs) and CIBERSORTx execution inside the Treatment Selection workflow
  repo: https://github.com/ailabstw/scDrug
- name: CIBERSORTx
  role: Docker image cibersortx/fractions; imputes the fraction of each single-cell
    subpopulation in each LINCS L1000 bulk sample
- name: Scanpy
  role: Provides the clustered AnnData (scanpyobj.h5ad) whose cluster GEPs are correlated
    against bulk cell-line GEPs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:1d3841b77e844e67c4283c583aa27559062b96588d9fa013400a3aa02d10bea7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ailabstw/scDrug
  tools:
  - CIBERSORTx
  - Python
  - Scanpy
  - scDrug
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# bulk-single-cell-correlation-cellline-assignment

## Summary

Selects the LINCS L1000 reference cancer cell line (from A375, A549, HCC515, HEPG2, HT29, MCF7, PC3, YAPC) whose bulk GEP best matches the single-cell cluster GEPs via Pearson correlation, then runs CIBERSORTx Cell Fractions to impute each single-cell subpopulation's fraction in bulk perturbation samples.

## When to use

Use when scRNA-seq clusters (from scanpyobj.h5ad with a cluster-wise GEP, GEP.txt) must be linked to a specific bulk cancer cell line before deconvolving LINCS L1000 perturbation samples for subpopulation fractions.

## When NOT to use

- When the goal is only cluster-level cell-type naming (e.g. immune/stromal labels) — use the scMatch/FANTOM5 annotation step instead of bulk cell-line correlation
- When no LINCS L1000 perturbation data are needed and no bulk deconvolution is required (e.g. GDSC/PRISM IC50 prediction via CaDRReS-Sc)
- When the user's bulk reference sample is not one of the eight supported LINCS cell lines and no custom reference is available

## Inputs

- Clustered Scanpy AnnData object (scanpyobj.h5ad) from Single-Cell Data Analysis
- Cluster-wise gene expression profile file (GEP.txt)
- LINCS L1000 bulk GEP data for the eight supported cell lines (A375, A549, HCC515, HEPG2, HT29, MCF7, PC3, YAPC)
- CIBERSORTx registered username and token

## Outputs

- Selected reference cell line (highest Pearson correlation, or user-specified)
- CIBERSORTx_Adjusted.txt with per-sample fraction values for each identified cell subpopulation
- CIBERSORTx_Results.txt
- Per-sample imputed subpopulation fractions used for Treatment Selection

## How to apply

First compute each cluster's GEP as the mean expression of its cells across all genes (GEP.txt) from the clustered Scanpy AnnData object. Compute the Pearson correlation coefficient between the single-cell GEP and the bulk GEP of each of the eight supported cell lines (A375, A549, HCC515, HEPG2, HT29, MCF7, PC3, YAPC) and select the line with the highest correlation, unless the user explicitly specifies a cell type. Then extract the anti-logarithmic bulk GEPs of all LINCS L1000 perturbation experiments on the selected cell line and run CIBERSORTx Cell Fractions (Docker image cibersortx/fractions, with a registered username and token) to impute the fraction of each single-cell subpopulation per bulk sample. Save CIBERSORTx_Adjusted.txt and CIBERSORTx_Results.txt to the output directory. Rationale: matching bulk and single-cell transcriptomic profiles ensures the deconvolution reference reflects the actual tumor context, enabling treatment selection against specific subpopulations.

## Related tools

- **scDrug** (Wraps the cell-line selection (Pearson correlation of single-cell vs bulk GEPs) and CIBERSORTx execution inside the Treatment Selection workflow) — https://github.com/ailabstw/scDrug
- **CIBERSORTx** (Docker image cibersortx/fractions; imputes the fraction of each single-cell subpopulation in each LINCS L1000 bulk sample)
- **Scanpy** (Provides the clustered AnnData (scanpyobj.h5ad) whose cluster GEPs are correlated against bulk cell-line GEPs)

## Examples

```
docker pull cibersortx/fractions
```

## Evaluation signals

- The chosen cell line is one of the eight supported lines (A375, A549, HCC515, HEPG2, HT29, MCF7, PC3, YAPC) and has the maximum Pearson correlation to the single-cell GEP (or equals the user-specified cell type)
- CIBERSORTx_Adjusted.txt exists and is a readable tabular file with one fraction value per identified cell subpopulation per sample, fractions summing to ~1 per sample
- CIBERSORTx_Results.txt is present alongside CIBERSORTx_Adjusted.txt in the output directory
- The anti-logarithmic bulk GEPs extracted correspond to perturbation experiments performed on the selected cell line only
- Fractions are consumed downstream by Treatment Selection to list treatment combinations for given cell clusters

## Limitations

- CIBERSORTx Cell Fractions requires a registered username and token, and the cibersortx/fractions Docker image; if docker-in-docker fails inside the scDrug container, the CIBERSORTx container must be run outside with a shared mounted folder
- Bulk deconvolution is restricted to the eight supported LINCS L1000 cell lines unless a user-specified cell type is given
- Drug response prediction in scDrug is limited to experimentally screened compounds, constraining which perturbation signatures can be deconvolved

## Evidence

- [methods] CIBERSORTx usage for LINCS deconvolution: "predict potential combined treatment strategies using the LINCS L1000 data [17]"
- [readme] CIBERSORTx Docker image requirement: "In the Docker container `scDrug`, pull the Docker image `cibersortx/fractions` used in treatment selection."
- [readme] Docker-in-docker fallback: "If docker-in-docker cannot be operated on the user's computer, the user can pull and run the CIBERSORTx container outside the scDrug container as long as a shared folder is mounted"
- [readme] Cluster GEP context for deconvolution: "output a Scanpy Anndata object `scanpyobj.h5ad`, a UMAP `umap_cluster.png` and differentially expressed genes (DEGs) `cluster_DEGs.csv` of the clustering result, and a gene expression profile (GEP)"
