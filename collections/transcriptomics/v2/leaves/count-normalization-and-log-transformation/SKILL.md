---
name: count-normalization-and-log-transformation
description: Use when starting from a raw scRNA-seq count matrix (10x mtx directory
  or CSV) that must be depth-normalized and log-transformed before clustering or differential
  expression in a Scanpy-based pipeline.
when_to_use_negative:
- Input is already normalized (e.g. a log-transformed expression matrix or CPM/TPM
  table) rather than raw counts — re-normalizing would double-correct depth.
- The task is bulk transcriptomic or drug response prediction (IC50/AUC modeling via
  CaDRReS-Sc or PRISM data) rather than scRNA-seq count preprocessing.
- Quality control is the only goal and the counts will be analyzed with a different
  normalization convention (e.g. scran size factors) inconsistent with the 10,000-count
  log1p scheme.
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scDrug
  role: workflow that performs scRNA-seq data preprocessing (normalization and log
    transformation) as the first pipeline step
  repo: https://github.com/ailabstw/scDrug
- name: Scanpy
  role: Python library executing the count normalization to 10,000 counts per cell
    and natural logarithmic transformation
  repo: https://github.com/ailabstw/scDrug
- name: MAGIC
  role: optional imputation of missing values applied after preprocessing when needed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2022.11.055
    title: 'scDrug: From single-cell RNA-seq to drug response prediction'
  source_content_hash: sha256:5df497b4c781ed97518f7be1f91fbab1b093d9f28e6ab45fceb5f04b3c6f7944
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# count-normalization-and-log-transformation

## Summary

Normalizes raw scRNA-seq count matrices to 10,000 total counts per cell followed by natural logarithmic transformation, as implemented in the scDrug Single-Cell Data Analysis module via Scanpy. This step standardizes cell depth before highly variable gene annotation, scaling, and downstream Louvain clustering.

## When to use

Use when starting from a raw scRNA-seq count matrix (10x mtx directory or CSV) that must be depth-normalized and log-transformed before clustering or differential expression in a Scanpy-based pipeline.

## When NOT to use

- Input is already normalized (e.g. a log-transformed expression matrix or CPM/TPM table) rather than raw counts — re-normalizing would double-correct depth.
- The task is bulk transcriptomic or drug response prediction (IC50/AUC modeling via CaDRReS-Sc or PRISM data) rather than scRNA-seq count preprocessing.
- Quality control is the only goal and the counts will be analyzed with a different normalization convention (e.g. scran size factors) inconsistent with the 10,000-count log1p scheme.

## Inputs

- raw scRNA-seq count matrix as a 10x-Genomics-formatted mtx directory (matrix.mtx, genes.tsv, barcodes.tsv)
- raw count matrix as a CSV file
- existing Scanpy AnnData object (scanpyobj.h5ad) when re-running or sub-clustering

## Outputs

- depth-normalized (10,000 counts/cell) and natural-log-transformed expression matrix within a Scanpy AnnData object
- highly variable gene annotations on the normalized data
- scaled (unit variance, zero mean) expression matrix feeding PCA and clustering
- scanpyobj.h5ad AnnData file containing downstream cluster assignments

## How to apply

First apply quality filtering on the raw counts: drop cells with fewer than 200 expressed genes, drop genes expressed in fewer than three cells, and keep only cells whose mitochondrial gene proportion is below 30%. Then, in Scanpy, normalize each remaining cell to 10,000 total counts and apply a natural logarithmic (log1p) transformation. Annotate highly variable genes on the normalized data and scale to unit variance and zero mean; impute with MAGIC only if imputation is needed. The normalized, scaled matrix feeds PCA (top PCs optionally Harmony-corrected) and the neighborhood graph used for Louvain clustering, so skipping or mis-tuning normalization directly distorts cluster structure. Input can be a 10x-Genomics-formatted mtx directory or a CSV file passed to the scDrug single_cell_analysis.py entry point.

## Related tools

- **scDrug** (workflow that performs scRNA-seq data preprocessing (normalization and log transformation) as the first pipeline step) — https://github.com/ailabstw/scDrug
- **Scanpy** (Python library executing the count normalization to 10,000 counts per cell and natural logarithmic transformation) — https://github.com/ailabstw/scDrug
- **MAGIC** (optional imputation of missing values applied after preprocessing when needed)

## Examples

```
python3 single_cell_analysis.py --input INPUT --format 10x
```

## Evaluation signals

- Per-cell count sums after library-size normalization equal 10,000 before log transformation.
- No cell in the filtered matrix retains a mitochondrial gene proportion at or above 30%, and no gene is expressed in fewer than three cells.
- The resulting AnnData (scanpyobj.h5ad) loads with a valid 'louvain' categorical observation field, confirming normalization did not break the downstream clustering.
- Downstream Louvain clustering on the top 20 PCs yields a chosen resolution within the [0.4, 1.4] auto-resolution search interval (e.g. 0.6 for GSE156625 tumor sub-clustering), indicating the normalized features support stable clustering.
- Expression values are on the natural log scale (log1p of normalized counts), with highly variable genes annotated and scaled to unit variance and zero mean.

## Limitations

- Normalization parameters (10,000 counts/cell, natural log) are fixed by the scDrug pipeline; datasets needing alternative conventions must modify the pipeline.
- Imputation via MAGIC is optional and only performed 'when needed'; imputing unnecessarily may distort downstream clustering.
- Filtering thresholds (200 genes/cell, 3 cells/gene, <30% mitochondrial reads) are tuned for the hepatocellular carcinoma GSE156625-style inputs and may over- or under-filter other tissues.

## Evidence

- [methods] we filtered out cells with less than 200 genes expressed and genes expressed in less than three cells and kept the cells with a proportion of mitochondrial genes below 30 %: "we filtered out cells with less than 200 genes expressed and genes expressed in less than three cells and kept the cells with a proportion of mitochondrial genes below 30 %"
- [methods] The remaining data undertook normalization to 10,000 total counts per cell, natural logarithmic transformation, highly variable gene annotation: "The remaining data undertook normalization to 10,000 total counts per cell, natural logarithmic transformation, highly variable gene annotation"
- [methods] Once the data imputation is needed, scDrug also integrates MAGIC [37] to impute missing values.: "Once the data imputation is needed, scDrug also integrates MAGIC [37] to impute missing values."
- [readme] path to input 10x directory or CSV file: "-i INPUT, --input INPUT
                        path to input 10x directory or CSV file"
