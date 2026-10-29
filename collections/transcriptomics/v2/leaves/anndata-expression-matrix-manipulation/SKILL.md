---
name: anndata-expression-matrix-manipulation
description: Use when single-cell RNA-seq data must be read, subset, smoothed, or
  scored within the scverse ecosystem — e.g. loading an h5ad or pbmc3k_processed()
  object, computing per-cell gene set scores that land in adata.obsm, or extracting
  activities with dc.get_acts.
when_to_use_negative:
- Input is already a bulk expression matrix — gssnng smoothing via the nearest neighbor
  graph of cells is designed for sparse, noisy single-cell data
- Input is raw unstructured count data not stored as (or convertible to) a Scanpy
  AnnData object — the workflow requires h5ad-compatible scverse tooling
- Analysis targets a single homogeneous group with no adata.obs grouping column —
  the NNG is constructed per user-defined groupby group, which must exist
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Scanpy
  role: Reads and manipulates the AnnData object (scanpy.read of h5ad files, sc.datasets.pbmc3k_processed());
    provides adata.X expression values
  repo: https://github.com/IlyaLab/gssnng
- name: AnnData
  role: Data container holding .X expression matrix, .obs cell metadata for groupby,
    and .obsm['gssnng_estimate'] score output
  repo: https://github.com/IlyaLab/gssnng
- name: gssnng
  role: Performs gene set scoring on the nearest neighbor graph, writing each gene
    set score as a column in adata.obsm['gssnng_estimate']
  repo: https://github.com/IlyaLab/gssnng
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad150
    title: Single-cell gene set scoring with nearest neighbor graph smoothed data
      (gssnng)
  source_content_hash: sha256:a5a812b97c7321469b12ed4a6bc455637b95c7dcf4895e131e4b42463fed0aeb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IlyaLab/gssnng
  tools:
  - AnnData
  - Python
  - Scanpy
  - gssnng
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# anndata-expression-matrix-manipulation

## Summary

Manipulate single-cell expression data as Scanpy AnnData objects (h5ad files), reading expression from adata.X, grouping cells via adata.obs columns, and storing per-cell gene set scores in adata.obsm['gssnng_estimate']. This is the data-handling backbone of the gssnng nearest-neighbor-smoothing gene set scoring workflow.

## When to use

Use when single-cell RNA-seq data must be read, subset, smoothed, or scored within the scverse ecosystem — e.g. loading an h5ad or pbmc3k_processed() object, computing per-cell gene set scores that land in adata.obsm, or extracting activities with dc.get_acts.

## When NOT to use

- Input is already a bulk expression matrix — gssnng smoothing via the nearest neighbor graph of cells is designed for sparse, noisy single-cell data
- Input is raw unstructured count data not stored as (or convertible to) a Scanpy AnnData object — the workflow requires h5ad-compatible scverse tooling
- Analysis targets a single homogeneous group with no adata.obs grouping column — the NNG is constructed per user-defined groupby group, which must exist

## Inputs

- Scanpy AnnData object (h5ad file) containing cells to be scored
- Expression matrix accessible as adata.X
- adata.obs column label, list, or dict used as the 'groupby' parameter
- .gmt gene set file (MSigDB nomenclature, e.g. cibersort_lm22.gmt)
- method_params dict for method-specific options (e.g. singscore normalization, ssGSEA omega=0.75, rbo_depth)

## Outputs

- AnnData object with smoothed counts (mini-pseudobulk expression profiles per cell)
- Per-cell gene set scores as columns in adata.obsm['gssnng_estimate']
- Activities extracted as an AnnData object via dc.get_acts(adata, obsm_key='gssnng_estimate')
- Overlap score table (one row per cell, one column per gene set, e.g. geneset_overlap_scores.csv)

## How to apply

Read the AnnData object with scanpy (e.g. sc.datasets.pbmc3k_processed() or scanpy.read of an h5ad file) and take expression values from adata.X. Use adata.obs columns (str, list, or dict) as the 'groupby' key so that nearest neighbor graphs are constructed within user-defined groups such as cluster or cluster*sample. After running scoring functions such as geneset_overlap, singscore, ssGSEA, mean_z, or robust_std, each gene set appears as a column in adata.obsm['gssnng_estimate']. To convert scored activities back into a manipulable AnnData object, call dc.get_acts(adata, obsm_key='gssnng_estimate'), optionally selecting genes with expression above a threshold before computing per-cell, per-geneset counts or fractions.

## Related tools

- **Scanpy** (Reads and manipulates the AnnData object (scanpy.read of h5ad files, sc.datasets.pbmc3k_processed()); provides adata.X expression values) — https://github.com/IlyaLab/gssnng
- **AnnData** (Data container holding .X expression matrix, .obs cell metadata for groupby, and .obsm['gssnng_estimate'] score output) — https://github.com/IlyaLab/gssnng
- **gssnng** (Performs gene set scoring on the nearest neighbor graph, writing each gene set score as a column in adata.obsm['gssnng_estimate']) — https://github.com/IlyaLab/gssnng

## Examples

```
import scanpy as sc; q = sc.datasets.pbmc3k_processed()
```

## Evaluation signals

- adata.obsm['gssnng_estimate'] contains one column per gene set parsed from the .gmt file
- Fraction-mode geneset_overlap values fall within [0, 1] for every cell
- dc.get_acts(adata, obsm_key='gssnng_estimate') returns a valid AnnData object of activities
- The AnnData object round-trips through scanpy.read on an h5ad file without schema loss (X, obs, obsm keys present)
- Per-group neighbor recomputation (recompute_neighbors > 0) yields neighborhoods within each groupby category, e.g. cluster*sample

## Limitations

- Neighbors must be recomputed within each group (recompute_neighbors) or the graph may span group boundaries
- smooth_mode choice ('adjacency' weights all neighbors equally, 'connectivity' weights close neighbors more) materially changes smoothed values
- Some scoring methods require ranked data (singscore, ssGSEA, rank_biased_overlap) while others recommend unranked counts (robust_std, mean_z)
- Sparsity and technical noise of single-cell measures motivate the smoothing step; scoring unsmoothed adata.X directly reduces score quality

## Evidence

- [readme] gssnng works with Scanpy AnnData objects stored as h5ad files: [evidence span withheld — anchor status: not_found]
- [methods] Score columns land in adata.obsm['gssnng_estimate']: "Score cells, each gene set will show up as a column in adata.obsm['gssnng_estimate']"
- [methods] Extract activities back as an AnnData object: "acts_gss = dc.get_acts(adata, obsm_key='gssnng_estimate')"
- [methods] pbmc3k AnnData loading pattern: "q = sc.datasets.pbmc3k_processed()"
- [readme] groupby parameter semantics for AnnData obs: "either a column label in adata.obs, and all categories taken, or a dict specifies one group."
