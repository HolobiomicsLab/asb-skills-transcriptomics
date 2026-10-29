---
name: anndata-h5ad-io
description: Use when the input scRNA-seq count matrix is stored as an h5ad-formatted
  HDF5 file (or must be persisted as one) before denoising, normalization, or downstream
  Scanpy/DCA processing.
when_to_use_negative:
- Input is a raw CSV/TSV count matrix that DCA accepts directly via its command line
  (dca matrix.csv results) and no h5ad persistence is needed
- The task is visualization or differential analysis on already-normalized data rather
  than count-matrix I/O for a denoising workflow
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: anndata
  role: Reader/writer for h5ad-formatted HDF5 files carrying the count matrix and
    cell/gene labels into and out of the DCA workflow
  repo: https://github.com/theislab/anndata
- name: Scanpy
  role: Downstream API whose AnnData-compatible methods (e.g., filter_genes_dispersion,
    normalize_total) consume the objects read via anndata
- name: DCA
  role: Denoising method whose workflow takes the count matrix loaded from the h5ad
    file and produces the denoised NB-mean matrix
  repo: https://github.com/theislab/dca
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-07931-2
    title: Single-cell RNA-seq denoising using a deep count autoencoder
  source_content_hash: sha256:c607023b25a62c5431cbca6f612973e685ee362a4d25ad0ba389a64fa1633dd6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/anndata
  tools:
  - DCA
  - Keras
  - Scanpy
  - TensorFlow
  - anndata
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# anndata-h5ad-io

## Summary

Read and write single-cell count matrices and annotations in the h5ad-formatted HDF5 container via the anndata Python package, the I/O format supported by the DCA denoising workflow and the Scanpy ecosystem.

## When to use

Use when the input scRNA-seq count matrix is stored as an h5ad-formatted HDF5 file (or must be persisted as one) before denoising, normalization, or downstream Scanpy/DCA processing.

## When NOT to use

- Input is a raw CSV/TSV count matrix that DCA accepts directly via its command line (dca matrix.csv results) and no h5ad persistence is needed
- The task is visualization or differential analysis on already-normalized data rather than count-matrix I/O for a denoising workflow

## Inputs

- h5ad-formatted HDF5 file containing a raw scRNA-seq count matrix (cell x gene) with cell and gene labels
- TSV/CSV raw count matrices with genes in rows and cells in columns (alternative ingestible format)

## Outputs

- AnnData object held in memory with X (counts), obs (cells), and var (genes)
- h5ad-formatted HDF5 file with processed count matrix and annotations

## How to apply

Open the raw scRNA-seq count matrix from the h5ad-formatted HDF5 file using the anndata reader so the cell x gene count matrix, cell labels, and gene labels travel together in one AnnData object. Because DCA's Scanpy integration and its denoised outputs (e.g., mean.tsv written as a gene x cell matrix) share this container model, loading through anndata keeps dimensions consistent between the raw counts and the denoised replacement matrix. After any processing (such as removing cells and genes with zero expression or selecting the top 1000 highly variable genes via Scanpy's filter_genes_dispersion), persist results back with the AnnData writer so the h5ad file remains the single source of truth for downstream steps. Verify that the on-disk object preserves the input matrix dimensions and label alignment after each read/write round trip.

## Related tools

- **anndata** (Reader/writer for h5ad-formatted HDF5 files carrying the count matrix and cell/gene labels into and out of the DCA workflow) — https://github.com/theislab/anndata
- **Scanpy** (Downstream API whose AnnData-compatible methods (e.g., filter_genes_dispersion, normalize_total) consume the objects read via anndata)
- **DCA** (Denoising method whose workflow takes the count matrix loaded from the h5ad file and produces the denoised NB-mean matrix) — https://github.com/theislab/dca

## Examples

```
adata = anndata.read_h5ad('raw_counts.h5ad')
```

## Evaluation signals

- The h5ad file loads into an AnnData object without errors and X has the expected cell x gene dimensions matching the source count matrix
- Cell and gene labels from the original matrix align exactly with obs and var after the read step
- Round-trip write/read of the h5ad preserves count values (integer raw counts) and label ordering
- Downstream DCA output (mean.tsv) has the same dimensions as the input file, except zero-expression genes or cells that were excluded

## Limitations

- DCA's output h5ad/TSV excludes cells and genes with zero expression, so dimensions can legitimately differ from the raw input matrix
- Default library size normalization applied by DCA follows Scanpy's normalize_total, so naive comparison of raw counts vs. mean.tsv values without normalization context can mislead

## Evidence

- [discussion] h5ad file support in anndata: "it supports h5ad-formatted HDF5 files (https://github.com/theislab/anndata)"
- [discussion] Scanpy API compatibility: "the Python API is compatible with the Scanpy33 package."
- [readme] dimension preservation of DCA output relative to input: [evidence span withheld — anchor status: external_doc]
