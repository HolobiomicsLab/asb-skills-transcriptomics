---
name: anndata-single-cell-data-handling
description: Use when an analysis pipeline (e.g. gssnng score_cells.with_gene_sets)
  requires a Scanpy AnnData object stored as an h5ad file, with expression values
  in adata.X and cell group labels in adata.obs.
when_to_use_negative:
- Input data is a plain count matrix, CSV, or bulk expression table rather than a
  scverse-compatible AnnData object, since gssnng works with Scanpy AnnData objects
  stored as h5ad files
- The task requires group-level or sample-level gene set analysis rather than per-cell/per-group
  scoring on a single-cell nearest neighbor graph
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: gssnng
  role: Consumes the AnnData object and writes per-cell gene set scores into adata.obsm['gssnng_estimate'];
    installable via pip install gssnng or from the GitHub repo
  repo: https://github.com/IlyaLab/gssnng
- name: Scanpy
  role: Reads the h5ad AnnData object (e.g. sc.datasets.pbmc3k_processed()) and supplies
    the obs group labels used as the groupby parameter
- name: AnnData
  role: Data container format whose X matrix and obs/obsm slots carry expression,
    groups, and gssnng scoring output
- name: decoupleR
  role: Extracts activities as an AnnData object via dc.get_acts using the gssnng
    obsm key, following the Decoupler/Omnipath style API
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad150
    title: Single-cell gene set scoring with nearest neighbor graph smoothed data
      (gssnng)
  source_content_hash: sha256:b0956e4238abde9f71fb13e00a49713ea1d6a8782ca9598896aeee968aea6bef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IlyaLab/gssnng
  tools:
  - AnnData
  - Python
  - Scanpy
  - decoupleR
  - gssnng
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# anndata-single-cell-data-handling

## Summary

Load, pass, and manipulate single-cell data as Scanpy AnnData (h5ad) objects when running gene set scoring workflows such as gssnng, which operates directly on adata.X expression matrices and adata.obs group labels.

## When to use

Use when an analysis pipeline (e.g. gssnng score_cells.with_gene_sets) requires a Scanpy AnnData object stored as an h5ad file, with expression values in adata.X and cell group labels in adata.obs.

## When NOT to use

- Input data is a plain count matrix, CSV, or bulk expression table rather than a scverse-compatible AnnData object, since gssnng works with Scanpy AnnData objects stored as h5ad files
- The task requires group-level or sample-level gene set analysis rather than per-cell/per-group scoring on a single-cell nearest neighbor graph

## Inputs

- Scanpy AnnData object stored as an h5ad file (e.g. via scanpy.read_* or sc.datasets.pbmc3k_processed())
- Expression matrix in adata.X (counts or values consumed by smoothing and scoring)
- Categorical cell group labels in adata.obs (e.g. 'louvain' cluster annotations used as groupby)

## Outputs

- AnnData object with smoothed counts (created by the gssnng data smoothing notebook workflow)
- Per-cell gene set scores stored in adata.obsm['gssnng_estimate'] with one column per gene set
- Activities AnnData object extracted via dc.get_acts(adata, obsm_key='gssnng_estimate')

## How to apply

Read the dataset into an AnnData object using scanpy (e.g. q = sc.datasets.pbmc3k_processed() for the pbmc3k dataset, or scanpy.read_* for local h5ad files), taking expression values from adata.X. Ensure the obs column used for the groupby parameter (e.g. 'louvain' cluster labels) is categorical, since nearest neighbor graphs are constructed within each group. Pass this AnnData directly into gssnng's score_cells.with_gene_sets, where results appear in adata.obsm['gssnng_estimate'] with one column per gene set; activities can be extracted back into an AnnData object via dc.get_acts(adata, obsm_key='gssnng_estimate'). Because gssnng is part of the scverse ecosystem, AnnData is the required interchange format rather than raw count matrices or CSVs.

## Related tools

- **gssnng** (Consumes the AnnData object and writes per-cell gene set scores into adata.obsm['gssnng_estimate']; installable via pip install gssnng or from the GitHub repo) — https://github.com/IlyaLab/gssnng
- **Scanpy** (Reads the h5ad AnnData object (e.g. sc.datasets.pbmc3k_processed()) and supplies the obs group labels used as the groupby parameter)
- **AnnData** (Data container format whose X matrix and obs/obsm slots carry expression, groups, and gssnng scoring output)
- **decoupleR** (Extracts activities as an AnnData object via dc.get_acts using the gssnng obsm key, following the Decoupler/Omnipath style API)

## Examples

```
import scanpy as sc; from gssnng import score_cells; q = sc.datasets.pbmc3k_processed(); score_cells.with_gene_sets(adata=q, gene_set_file='cibersort_lm22.gmt', score_method='geneset_overlap', groupby='louvain', smooth_mode='off', recompute_neighbors=0, ranked=False)
```

## Evaluation signals

- adata.X is populated with the expected expression values after loading (e.g. the pbmc3k_processed matrix from sc.datasets)
- The groupby obs column (e.g. 'louvain') is categorical and its categories define the groups over which nearest neighbor graphs are rebuilt
- After scoring, adata.obsm['gssnng_estimate'] exists and contains one column per gene set from the input .gmt file
- dc.get_acts returns an AnnData object of activities derived from the 'gssnng_estimate' obsm key without shape errors
- Downstream tabulation (e.g. B-cell overlap fractions written to CSV) covers the expected group categories from adata.obs

## Limitations

- Sparsity and technical noise of single-cell expression measures in adata.X create difficulties for gene set scoring methods designed for bulk data unless smoothing is applied
- Nearest neighbor graphs are constructed based on user-defined groups (groupby), so poorly chosen or missing obs group labels change graph construction; recompute_neighbors=0 skips rebuilding neighbors within groups
- No discussion-section content was available in the source document, so long-term caveats beyond those stated in the abstract/methods are not documented

## Evidence

- [abstract] The gssnng software is available using the python package index (PyPI) and works with Scanpy AnnData objects.: "The gssnng software is available using the python package index (PyPI) and works with Scanpy AnnData objects."
- [methods] Read in an AnnData object using scanpy (an h5ad file).: "1. Read in an AnnData object using scanpy (an h5ad file)."
- [methods] Score cells, each gene set will show up as a column in adata.obsm['gssnng_estimate']: "3. Score cells, each gene set will show up as a column in adata.obsm['gssnng_estimate']"
- [readme] README: package works with Scanpy AnnData objects stored as h5ad files: "This package is part of the [scverse ecosystem](https://scverse.org/packages/#ecosystem) and works with Scanpy AnnData objects stored as h5ad files."
- [readme] README: adata parameter description: "**adata:**  AnnData object from scanpy.read_*<br/>"
