---
name: anndata-cell-metadata-handling
description: Use when the input is a single-cell dataset in AnnData/.h5ad format and
  a downstream tool needs per-cell metadata columns (samples_col, clusters_col, conds_col)
  extracted or validated before proportion analysis.
when_to_use_negative:
- When the input is a raw count matrix or FASTQ before clustering — cell-type clustering
  and annotation must be completed first, since correct cell clustering is paramount
  for proportion analysis
- When the data is already a pre-computed proportion/count table per sample and cluster
  rather than per-cell AnnData
- When cell clusters are suspected to be defined by technical parameters (batch, sequencing
  machine, timepoint) rather than real cell types
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Scanpro
  role: Downstream consumer of the AnnData object; it reads cell metadata via samples_col,
    clusters_col, and conds_col keyword arguments
  repo: https://github.com/loosolab/scanpro
- name: Scanpy
  role: Python toolkit used to load and manipulate the AnnData object holding the
    single-cell data and its metadata
- name: AnnData
  role: File format/data structure that saves data matrices together with cell metadata
    in the .h5ad file
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-024-66381-7
    title: Scanpro is a tool for robust proportion analysis of single-cell resolution
      data
  source_content_hash: sha256:8e222854d808ca8cf405eca33257dd332ee0126b5e6747e98bba6906781ddf2a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/loosolab/scanpro
  tools:
  - AnnData
  - Python
  - Scanpro
  - Scanpy
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# anndata-cell-metadata-handling

## Summary

Loading a single-cell dataset as an AnnData/Scanpy object and extracting the per-cell sample, cluster, and condition metadata columns that downstream proportion analysis (Scanpro) requires. This is the prerequisite wiring step before any cell-type proportion analysis on .h5ad inputs.

## When to use

Use when the input is a single-cell dataset in AnnData/.h5ad format and a downstream tool needs per-cell metadata columns (samples_col, clusters_col, conds_col) extracted or validated before proportion analysis.

## When NOT to use

- When the input is a raw count matrix or FASTQ before clustering — cell-type clustering and annotation must be completed first, since correct cell clustering is paramount for proportion analysis
- When the data is already a pre-computed proportion/count table per sample and cluster rather than per-cell AnnData
- When cell clusters are suspected to be defined by technical parameters (batch, sequencing machine, timepoint) rather than real cell types

## Inputs

- Single-cell expression data in .h5ad / AnnData format (e.g. scanpro_tutorial.h5ad)
- Cell metadata in adata.obs with sample/replicate, cluster/cell-type, and condition columns
- Cluster assignments from prior single-cell analysis (e.g. Leiden clustering on a neighborhood graph)

## Outputs

- An AnnData object with validated obs columns suitable for scanpro(adata, samples_col=..., clusters_col=..., conds_col=...)
- Column name mapping for samples_col, clusters_col, and conds_col used by the downstream analysis call

## How to apply

Read the data with scanpy/anndata (e.g. ad.read_h5ad('scanpro_tutorial.h5ad')) so cell-by-gene matrices and cell metadata travel together in one AnnData object, which is the format Scanpro natively handles. Inspect adata.obs to locate the column holding sample/replicate identity, the column holding cluster (cell-type) labels, and the column holding the biological condition, since Scanpro consumes these by name via scanpro(adata, samples_col=..., clusters_col=..., conds_col=...). Decide upfront whether samples_col is populated: omitting it or setting it to None signals an unreplicated dataset, causing Scanpro to switch to its bootstrapping pseudo-replicate mode. Verify that clustering into cell types was already performed on the neighborhood graph, because proportion analysis is defined on those clusters; if a cluster is supported by fewer than ~2% of all cells (below the n_min threshold), flag or drop it before analysis, since benchmarking showed such clusters produce unstable results.

## Related tools

- **Scanpro** (Downstream consumer of the AnnData object; it reads cell metadata via samples_col, clusters_col, and conds_col keyword arguments) — https://github.com/loosolab/scanpro
- **Scanpy** (Python toolkit used to load and manipulate the AnnData object holding the single-cell data and its metadata)
- **AnnData** (File format/data structure that saves data matrices together with cell metadata in the .h5ad file)

## Examples

```
import anndata as ad; adata = ad.read_h5ad('scanpro_tutorial.h5ad'); from scanpro.scanpro import scanpro; out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds')
```

## Evaluation signals

- adata.obs contains columns that map cleanly onto samples_col, clusters_col, and conds_col with no missing values in the cells retained for analysis
- Each cell carries exactly one sample label, one cluster label, and one condition label; cluster labels match the prior Leiden clustering results
- Clusters supported by fewer than n_min cells (below ~2% of all cells) are identified for trimming before passing the object to scanpro
- Deciding samples_col=None vs a populated sample column is a deliberate choice: None triggers Scanpro's bootstrapping method on unreplicated data
- The invocation out = scanpro(adata, samples_col=..., clusters_col=..., conds_col=...) returns a results object accessible via out.results without column-name errors

## Limitations

- Clustering parameters such as Leiden resolution critically affect the number and size of resulting clusters, which challenges biologically meaningful proportion analysis
- Clusters might not necessarily reflect cell types but can be defined by technical parameters such as batch effects from different sequencing machines or collection timepoints
- The article recommends generating larger, less specific cell clusters with higher cell numbers rather than many highly specific clusters with small cell numbers
- If samples_col is omitted, Scanpro assumes the dataset is unreplicated and runs bootstrapping — results should be interpreted with caution and require additional experimental validation

## Evidence

- [intro] which saves data matrices into a format known as AnnData: "which saves data matrices into a format known as AnnData"
- [methods] run scanpro function on anndata object: "out = scanpro(adata, samples_col='sample', clusters_col='cluster', conds_col='conds')"
- [readme] bootstrapping trigger on missing samples_col: [evidence span withheld — anchor status: external_doc]
- [results] h5ad input example: "adata = ad.read_h5ad('scanpro_tutorial.h5ad')"
- [discussion] clustering prerequisite: "correct cell clustering and thus cell type identification is paramount for PA"
