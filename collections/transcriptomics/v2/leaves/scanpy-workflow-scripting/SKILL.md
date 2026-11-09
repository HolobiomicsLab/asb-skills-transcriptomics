---
name: scanpy-workflow-scripting
description: Use when scoring gene sets on single-cell RNA-seq data stored as Scanpy
  AnnData (h5ad), when per-cell or per-group scores are needed in adata.obsm, or when
  smoothing sparse scRNA-seq counts via a nearest-neighbor graph before scoring.
when_to_use_negative:
- Data are bulk RNA-seq — the smoothing step targets sparsity and technical noise
  specific to single-cell expression measures
- No cell-grouping structure (e.g. clusters or samples) exists to define nearest-neighbor
  graphs via groupby, and per-cell scores without neighborhood smoothing are acceptable
- Gene sets are not available as .gmt files or via the Decoupler/Omnipath API
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: gssnng
  role: Performs gene set scoring on the nearest-neighbor graph of cells; installed
    via pip and operates on Scanpy AnnData objects
  repo: github.com/IlyaLab/gssnng
- name: Scanpy
  role: Reads/writes the h5ad AnnData objects that gssnng consumes and produces (scanpy.read,
    sc.datasets.pbmc3k_processed)
- name: AnnData
  role: Container format holding counts, obs group labels, and obsm score/smoothed-count
    matrices
- name: decoupleR
  role: Provides the Decoupler/Omnipath style API for fetching gene set models and
    extracting activities with dc.get_acts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad150
    title: Single-cell gene set scoring with nearest neighbor graph smoothed data
      (gssnng)
  source_content_hash: sha256:ac7ed4dce4b6da69d3089bcd0b7901fd60993d5e8c4abca3a846586d0d581d97
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AnnData
  - Python
  - Scanpy
  - decoupleR
  - gssnng
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# scanpy-workflow-scripting

## Summary

Script Scanpy/AnnData workflows for single-cell gene set scoring with gssnng: load h5ad AnnData objects, build per-group nearest-neighbor graphs, store smoothed counts and per-cell gene set scores in obsm, and parallelize across groupby groups.

## When to use

Use when scoring gene sets on single-cell RNA-seq data stored as Scanpy AnnData (h5ad), when per-cell or per-group scores are needed in adata.obsm, or when smoothing sparse scRNA-seq counts via a nearest-neighbor graph before scoring.

## When NOT to use

- Data are bulk RNA-seq — the smoothing step targets sparsity and technical noise specific to single-cell expression measures
- No cell-grouping structure (e.g. clusters or samples) exists to define nearest-neighbor graphs via groupby, and per-cell scores without neighborhood smoothing are acceptable
- Gene sets are not available as .gmt files or via the Decoupler/Omnipath API

## Inputs

- Scanpy AnnData object stored as h5ad (adata.X containing expression counts)
- Gene set file in GMT format (one gene set per line, MSigDB nomenclature, e.g. cibersort_lm22.gmt)
- obs column label, list, or dict for the groupby parameter defining nearest-neighbor graph groups

## Outputs

- adata.obsm['gssnng_estimate'] with one column per gene set of per-cell scores
- AnnData object with smoothed counts (obsm['X_smooth'] in per-group objects, concatenated via anndata.concat)
- AnnData of extracted activities via dc.get_acts for visualization and statistical analysis

## How to apply

Read the AnnData object with scanpy (scanpy.read or sc.datasets, e.g. sc.datasets.pbmc3k_processed()), then call gssnng's scores_cells.with_gene_sets passing gene_set_file (.gmt), groupby (an obs column, list, or dict of one group), smooth_mode ('adjacency', 'connectivity', or 'off'), and recompute_neighbors (0 to skip, >0 to specify N, e.g. 11). The nearest-neighbor graph is constructed within each user-defined group (e.g. cluster or cluster*sample) so smoothing creates mini-pseudobulk profiles; groups can be processed in parallel with the cores parameter. Scores are written as columns in adata.obsm['gssnng_estimate'], and per-group smoothed AnnData objects can be combined with anndata.concat and written back to h5ad. For Decoupler/Omnipath-style inputs, fetch the model via the decoupler API, optionally filter out genes negatively associated with the pathway, then extract activities with dc.get_acts(adata, obsm_key='gssnng_estimate').

## Related tools

- **gssnng** (Performs gene set scoring on the nearest-neighbor graph of cells; installed via pip and operates on Scanpy AnnData objects) — github.com/IlyaLab/gssnng
- **Scanpy** (Reads/writes the h5ad AnnData objects that gssnng consumes and produces (scanpy.read, sc.datasets.pbmc3k_processed))
- **AnnData** (Container format holding counts, obs group labels, and obsm score/smoothed-count matrices)
- **decoupleR** (Provides the Decoupler/Omnipath style API for fetching gene set models and extracting activities with dc.get_acts)

## Examples

```
python3 -m pip install git+https://github.com/IlyaLab/gssnng
```

## Evaluation signals

- Every gene set in the .gmt file appears as a column in adata.obsm['gssnng_estimate']
- Nearest-neighbor graphs are constructed within each groupby group, not across groups (recompute_neighbors set to 0 or a chosen N such as 11)
- smooth_mode='off' leaves the original X unchanged; 'adjacency' weights all neighbors equally (all 1s) while 'connectivity' weights closer neighbors more
- Smoothed matrices are scipy sparse CSR and each cell's row reflects a neighbor-averaged mini-pseudobulk profile
- Downstream dc.get_acts successfully extracts activities as an AnnData object from obsm_key='gssnng_estimate'

## Limitations

- scoring methods that require ranked data (singscore, ssGSEA, rank_biased_overlap) depend on the ranked parameter being set correctly
- Some scoring methods have method-specific params that must be passed as a method_params dict (e.g. singscore normalization, rbo_depth, ssGSEA omega=0.75)
- recomputing neighbors within every group (recompute_neighbors>0) adds cost, but skipping it may mis-specify graph degree per group

## Evidence

- [abstract] The gssnng software is available using the python package index (PyPI) and works with Scanpy AnnData objects.: "The gssnng software is available using the python package index (PyPI) and works with Scanpy AnnData objects."
- [methods] Read in an AnnData object using scanpy (an h5ad file): "1. Read in an AnnData object using scanpy (an h5ad file)."
- [methods] Score cells, each gene set will show up as a column in adata.obsm['gssnng_estimate']: "3. Score cells, each gene set will show up as a column in adata.obsm['gssnng_estimate']"
- [readme] Nearest neighbor graphs (NNG) are constructed based on user defined groups (see the 'groupby' parameter below). The defined groups can be processed in parallel, speeding up the calculations.: "Nearest neighbor graphs (NNG) are constructed based on user defined groups (see the 'groupby' parameter below)."
- [readme] smooth_mode dictates adjacency (all 1s) vs connectivity weighting: "`adjacency` weights all neighbors equally, `connectivity` weights close neighbors more"
