---
name: scrna-seq-preprocessing-normalization-scaling
description: Use when raw single-cell gene expression count matrices (e.g. AnnData/scRNA-seq
  data) must be filtered, normalized, log-transformed and scaled prior to clustering
  methods such as scAce that require pre-processed input.
when_to_use_negative:
- Input is already normalized/log-transformed (e.g. a feature table or embeddings)
  — re-normalizing would double-transform the data
- Data has fewer than the minimum thresholds (genes detected in <3 cells or cells
  with <200 genes) such that filtering would remove most of the dataset rather than
  low-quality features
edam_operation: http://edamontology.org/operation_3563
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scanpy
  role: Provides sc.pp.filter_genes, filter_cells, normalize_per_cell, log1p and scale
    used for the preprocessing steps on AnnData
  repo: https://github.com/sldyns/scAce
- name: scAce
  role: Downstream consumer of the preprocessed AnnData; requires the stored scale_factor
    for its VAE/ZINB pre-training
  repo: https://github.com/sldyns/scAce
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad546
    title: 'scAce: an adaptive embedding and clustering method for single-cell gene
      expression data'
  source_content_hash: sha256:2dd23f458be6b79151fd650aaee015cc73d0837df1bc7afd96a6152978c22900
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sldyns/scAce
  tools:
  - Leiden
  - Python 3.8
  - scAce
  - scanpy
  edam_operation: http://edamontology.org/operation_3563
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# scrna-seq-preprocessing-normalization-scaling

## Summary

Standard scRNA-seq preprocessing pipeline used before scAce clustering: basic gene/cell filtering, total-count normalization with storage of scale factors, log1p transformation, and z-score scaling on a scanpy AnnData object.

## When to use

Use when raw single-cell gene expression count matrices (e.g. AnnData/scRNA-seq data) must be filtered, normalized, log-transformed and scaled prior to clustering methods such as scAce that require pre-processed input.

## When NOT to use

- Input is already normalized/log-transformed (e.g. a feature table or embeddings) — re-normalizing would double-transform the data
- Data has fewer than the minimum thresholds (genes detected in <3 cells or cells with <200 genes) such that filtering would remove most of the dataset rather than low-quality features

## Inputs

- Single-cell gene expression count matrix as a scanpy AnnData object (e.g. Mouse kidney scRNA-seq data from Adam et al. 2017)

## Outputs

- Filtered, total-count normalized, log1p-transformed and scaled AnnData object with adata.raw preserved and adata.obs['scale_factor'] populated, ready for scAce's VAE pre-training

## How to apply

Load the count matrix into a scanpy AnnData object, then apply basic filtering with sc.pp.filter_genes(min_cells=3) and sc.pp.filter_cells(min_genes=200). Copy the AnnData into adata.raw before normalization so raw counts are retained. Total-count normalize with sc.pp.normalize_per_cell, and store the per-cell scale factor as adata.obs['scale_factor'] = n_counts / median(n_counts), which scAce needs for its ZINB loss. Finish with sc.pp.log1p followed by sc.pp.scale to logarithmize and z-score the expression values before passing the AnnData to run_scace.

## Related tools

- **scanpy** (Provides sc.pp.filter_genes, filter_cells, normalize_per_cell, log1p and scale used for the preprocessing steps on AnnData) — https://github.com/sldyns/scAce
- **scAce** (Downstream consumer of the preprocessed AnnData; requires the stored scale_factor for its VAE/ZINB pre-training) — https://github.com/sldyns/scAce

## Examples

```
import scanpy as sc; sc.pp.filter_genes(adata, min_cells=3); sc.pp.filter_cells(adata, min_genes=200); sc.pp.normalize_per_cell(adata); adata.obs['scale_factor'] = adata.obs.n_counts / adata.obs.n_counts.median(); sc.pp.log1p(adata); sc.pp.scale(adata)
```

## Evaluation signals

- adata.raw exists and holds the pre-normalization counts
- adata.obs['scale_factor'] is present and equals n_counts / median(n_counts) per cell
- Genes expressed in fewer than 3 cells and cells expressing fewer than 200 genes have been removed
- Expression values in adata.X are scaled (approximately zero-mean, unit-variance per gene) after log1p

## Limitations

- The min_cells=3 / min_genes=200 thresholds are the defaults used in the scAce paper; other datasets may need tuning
- sc.pp.scale modifies adata.X in place, so raw counts must be preserved in adata.raw beforehand as the README shows

## Evidence

- [readme] Basic filtering: "# Basic filtering
sc.pp.filter_genes(adata, min_cells=3)
sc.pp.filter_cells(adata, min_genes=200)"
- [readme] Normalization and scale factor: "# Total-count normlize, logarithmize and scale the data  
sc.pp.normalize_per_cell(adata)
adata.obs['scale_factor'] = adata.obs.n_counts / adata.obs.n_counts.median()

sc.pp.log1p(adata)
sc.pp.scale(a"
- [methods] Article methods description of preprocessing: "Perform data pre-processing:"
- [readme] Preprocessing precedes VAE pre-training: "a pre-training step based on a variational autoencoder, a cluster initialization step to obtain initial cluster labels, and an adaptive cluster merging step"
