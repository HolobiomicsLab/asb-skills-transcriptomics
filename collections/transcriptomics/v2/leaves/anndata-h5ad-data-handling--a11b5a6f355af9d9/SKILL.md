---
name: anndata-h5ad-data-handling
description: Use when single-cell RNA-seq data arrives as an .h5ad file and must be
  loaded with Scanpy into an AnnData object before gene set scoring (e.g. gssnng score_method
  on a phagocyte scRNA-seq object with LPS-treated and untreated cells).
when_to_use_negative:
- The input is already a bulk RNA-seq count matrix or feature table rather than a
  Scanpy AnnData / h5ad single-cell object
- The task only needs decoupler/OmniPath model retrieval and does not involve local
  AnnData/h5ad objects
- The h5ad lacks the obs columns needed to define groupby groups for nearest neighbor
  graph construction
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Scanpy
  role: Reads the h5ad file into an AnnData object and provides datasets (e.g. sc.datasets.pbmc3k_processed)
    used as demo inputs
- name: AnnData
  role: Data container holding counts, obs annotations, and obsm slots where gssnng
    writes gene set scores
- name: gssnng
  role: Consumes the Scanpy AnnData object to perform nearest neighbor graph smoothed
    gene set scoring
  repo: https://github.com/IlyaLab/gssnng
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad150
    title: Single-cell gene set scoring with nearest neighbor graph smoothed data
      (gssnng)
  source_content_hash: sha256:ae9e52d2ac52ff9fe335e9a61955fbd39beeda7d81627a582660a694fe1c6a75
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IlyaLab/gssnng
  tools:
  - AnnData
  - Python
  - Scanpy
  - gssnng
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# anndata-h5ad-data-handling

## Summary

Load and manipulate single-cell expression data as Scanpy AnnData objects stored in h5ad files, as required by the gssnng gene set scoring workflow, so that cells can be scored per-cell or per-group on a nearest neighbor graph.

## When to use

Use when single-cell RNA-seq data arrives as an .h5ad file and must be loaded with Scanpy into an AnnData object before gene set scoring (e.g. gssnng score_method on a phagocyte scRNA-seq object with LPS-treated and untreated cells).

## When NOT to use

- The input is already a bulk RNA-seq count matrix or feature table rather than a Scanpy AnnData / h5ad single-cell object
- The task only needs decoupler/OmniPath model retrieval and does not involve local AnnData/h5ad objects
- The h5ad lacks the obs columns needed to define groupby groups for nearest neighbor graph construction

## Inputs

- Single-cell RNA-seq expression data stored as an .h5ad file
- Scanpy AnnData object with obs annotations (e.g. treatment labels, groupby categories)
- GMT gene set file to be scored against the AnnData cells (downstream)

## Outputs

- In-memory AnnData object ready for gssnng scoring
- Per-cell gene set scores in adata.obsm['gssnng_estimate'] (one column per gene set)

## How to apply

Read the h5ad file with scanpy (e.g. scanpy.read_h5ad) to obtain an AnnData object whose counts matrix, obs annotations (e.g. LPS-treated vs untreated labels), and neighborhood/groupby columns are accessible. Confirm the AnnData structure satisfies downstream tool requirements: gssnng works directly with Scanpy AnnData objects and writes gene set scores into adata.obsm['gssnng_estimate']. If nearest neighbor smoothing will be applied, retain the obs column used for the 'groupby' parameter (e.g. cluster or cluster*sample) since NNGs are constructed within user-defined groups. Optionally install gssnng from its GitHub repo via pip so the loaded AnnData can be passed straight to scores_cells.with_gene_sets with a .gmt gene set file.

## Related tools

- **Scanpy** (Reads the h5ad file into an AnnData object and provides datasets (e.g. sc.datasets.pbmc3k_processed) used as demo inputs)
- **AnnData** (Data container holding counts, obs annotations, and obsm slots where gssnng writes gene set scores)
- **gssnng** (Consumes the Scanpy AnnData object to perform nearest neighbor graph smoothed gene set scoring) — https://github.com/IlyaLab/gssnng

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('phagocytes_lps.h5ad')
```

## Evaluation signals

- scanpy.read_h5ad returns an AnnData object (not a DataFrame or raw matrix) whose .X, .obs, and .var are populated
- obs contains the category column needed for groupby (e.g. LPS-treated vs untreated cell labels), matching expected cell counts (e.g. 2,340 LPS-treated and 2,104 untreated cells in the LPS task)
- Passing the AnnData to gssnng's scores_cells.with_gene_sets runs without schema errors and adds adata.obsm['gssnng_estimate'] with one column per gene set
- Gene identifiers in the AnnData .var align with gene names in the supplied .gmt file

## Limitations

- gssnng requires the data to be a Scanpy AnnData object stored as h5ad; other single-cell formats must be converted first
- Gene set scoring quality is limited by scRNA-seq sparsity and technical noise unless a nearest neighbor smoothing transformation is applied
- h5ad loading alone does not rebuild the neighbor graph; recompute_neighbors>0 must be set when groupby groups change to avoid scoring across an invalid graph

## Evidence

- [abstract] The gssnng software is available using the python package index (PyPI) and works with Scanpy AnnData objects.: "The gssnng software is available using the python package index (PyPI) and works with Scanpy AnnData objects."
- [intro] This package is part of the scverse ecosystem and works with Scanpy AnnData objects stored as h5ad files.: [evidence span withheld — anchor status: not_found]
- [methods] Read in an AnnData object using scanpy (an h5ad file).: "Read in an AnnData object using scanpy (an h5ad file)."
- [methods] Score cells, each gene set will show up as a column in adata.obsm['gssnng_estimate']: "Score cells, each gene set will show up as a column in adata.obsm['gssnng_estimate']"
- [readme] Nearest neighbor graphs (NNG) are constructed based on user defined groups (see the 'groupby' parameter below).: "Nearest neighbor graphs (NNG) are constructed based on user defined groups (see the 'groupby' parameter below)."
