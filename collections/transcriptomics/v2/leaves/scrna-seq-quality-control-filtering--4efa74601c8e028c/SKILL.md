---
name: scrna-seq-quality-control-filtering
description: Use when raw scRNA-seq count data (10x-formatted mtx directory or CSV)
  must be filtered for cells with <200 expressed genes, genes in <3 cells, and high
  mitochondrial fractions before normalization and clustering.
when_to_use_negative:
- Input is bulk RNA-seq or an already-aggregated feature table rather than a per-cell
  count matrix — the cell-level thresholds (200 genes/cell, mitochondrial <30%) do
  not apply
- The AnnData has already passed QC and normalization (e.g., re-running single_cell_analysis.py
  on scanpyobj.h5ad in sub-clustering mode should not re-filter a clean object)
- The task is drug response prediction on existing clusters (drug_response_prediction.py
  consumes scanpyobj.h5ad and skips QC)
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: scDrug
  role: Workflow container whose single_cell_analysis.py implements the filtering,
    normalization, HVG annotation, and scaling defaults
  repo: https://github.com/ailabstw/scDrug
- name: Scanpy
  role: Library performing data preprocessing (cell/gene filtering, normalization,
    log transformation, HVG selection, scaling)
- name: MAGIC
  role: Optional imputation of missing values after preprocessing, when needed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:26bc825f2202134fa83a7bdf4096c9ad7d2c2a991507ebf9da33da5c973e67dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ailabstw/scDrug
  tools:
  - MAGIC
  - Python
  - Scanpy
  - scDrug
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# scrna-seq-quality-control-filtering

## Summary

Quality-control and preprocessing step from the scDrug Single-Cell Data Analysis module: it removes low-quality cells and genes from raw scRNA-seq count matrices (10x mtx or CSV), then normalizes to 10,000 counts per cell with log transformation, highly variable gene annotation, and scaling to unit variance and zero mean, so downstream PCA, Harmony batch correction, and Louvain clustering are performed on a clean matrix.

## When to use

Use when raw scRNA-seq count data (10x-formatted mtx directory or CSV) must be filtered for cells with <200 expressed genes, genes in <3 cells, and high mitochondrial fractions before normalization and clustering.

## When NOT to use

- Input is bulk RNA-seq or an already-aggregated feature table rather than a per-cell count matrix — the cell-level thresholds (200 genes/cell, mitochondrial <30%) do not apply
- The AnnData has already passed QC and normalization (e.g., re-running single_cell_analysis.py on scanpyobj.h5ad in sub-clustering mode should not re-filter a clean object)
- The task is drug response prediction on existing clusters (drug_response_prediction.py consumes scanpyobj.h5ad and skips QC)

## Inputs

- Raw scRNA-seq count matrix as a 10x-Genomics-formatted mtx directory (matrix.mtx, genes.tsv, barcodes.tsv, optional --prefix)
- Raw scRNA-seq count matrix as a CSV file
- Previously produced Scanpy AnnData object (h5ad) when sub-clustering specified clusters

## Outputs

- QC-filtered, normalized (10,000 counts/cell), log-transformed, scaled AnnData object with highly variable gene annotations
- scanpyobj.h5ad clustered AnnData produced from the filtered matrix
- cluster_DEGs.csv (differentially expressed genes of the clustering result)

## How to apply

Load the count matrix into Scanpy and filter out cells with fewer than 200 expressed genes, drop genes expressed in fewer than three cells, and keep only cells whose mitochondrial gene proportion is below 30%. Normalize the remaining data to 10,000 total counts per cell, apply natural logarithmic transformation, annotate highly variable genes, and scale to unit variance and zero mean. If missing values must be imputed before downstream analysis, integrate MAGIC. These thresholds and steps are the scDrug pipeline defaults preceding PCA computation (optionally Harmony-corrected PCs) and Louvain clustering at a chosen or auto-determined resolution. The resulting processed matrix is carried in the Scanpy AnnData object saved as scanpyobj.h5ad.

## Related tools

- **scDrug** (Workflow container whose single_cell_analysis.py implements the filtering, normalization, HVG annotation, and scaling defaults) — https://github.com/ailabstw/scDrug
- **Scanpy** (Library performing data preprocessing (cell/gene filtering, normalization, log transformation, HVG selection, scaling))
- **MAGIC** (Optional imputation of missing values after preprocessing, when needed)

## Examples

```
python3 single_cell_analysis.py -i INPUT -f 10x -o OUTPUT
```

## Evaluation signals

- Every retained cell expresses at least 200 genes; cells with fewer are absent from the output object
- All retained cells have mitochondrial gene proportion below 30%
- Every retained gene is expressed in at least three cells
- Expression values in the output are normalized to 10,000 total counts per cell and natural-log transformed, with highly variable genes annotated and data scaled to unit variance and zero mean
- scanpyobj.h5ad loads as a valid AnnData object with a categorical 'louvain' observation field downstream

## Limitations

- Filtering thresholds (200 genes/cell, genes in >=3 cells, mitochondrial <30%) are pipeline defaults; datasets needing different stringency require manual adjustment
- MAGIC imputation is optional and only applied when the analysis needs it, not a mandatory step
- Poor-quality cells identified only by high mitochondrial percentage may not capture all artifacts, e.g. doublets or batch-specific effects (the latter are addressed later by Harmony, not by QC)

## Evidence

- [methods] Cell/gene quality filtering thresholds: "we filtered out cells with less than 200 genes expressed and genes expressed in less than three cells and kept the cells with a proportion of mitochondrial genes below 30 %"
- [methods] Normalization and transformation steps after filtering: "The remaining data undertook normalization to 10,000 total counts per cell, natural logarithmic transformation, highly variable gene annotation"
- [results] Rationale: removing low-expression genes and poor-quality cells: "we filtered out the genes and cells with low expression and poor-quality cells showing a high percentage of mitochondrial genes"
- [methods] Optional imputation integration: "Once the data imputation is needed, scDrug also integrates MAGIC [37] to impute missing values."
