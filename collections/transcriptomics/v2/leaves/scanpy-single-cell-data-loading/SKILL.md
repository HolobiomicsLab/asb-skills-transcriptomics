---
name: scanpy-single-cell-data-loading
description: Use when a scRNA-seq analysis pipeline starts from raw count/expression
  data that must be read into an AnnData object (e.g. an h5ad file via scanpy.read_*)
  before scoring, smoothing, or visualization can proceed.
when_to_use_negative:
- Input is already an in-memory AnnData object from a prior scanpy step — reloading
  is unnecessary
- Data is bulk RNA-seq or a plain feature/count table not organized as per-cell observations;
  gssnng-style AnnData workflows expect per-cell matrices
- Data is stored in a non-scverse format the pipeline cannot convert to h5ad/AnnData
  without extra restructuring
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Scanpy
  role: Reads the h5ad file / bundled processed dataset (e.g. sc.datasets.pbmc3k_processed)
    into an AnnData object for downstream scoring
  repo: https://github.com/IlyaLab/gssnng
- name: AnnData
  role: Container format holding adata.X expression values and adata.obs metadata
    consumed by gssnng
- name: gssnng
  role: Downstream consumer of the loaded AnnData; works with Scanpy AnnData objects
    stored as h5ad files
  repo: https://github.com/IlyaLab/gssnng
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad150
    title: Single-cell gene set scoring with nearest neighbor graph smoothed data
      (gssnng)
  source_content_hash: sha256:486ef5e43edbf926790c0b59647dd902ed9177fb48882b162a8f923419ee977c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IlyaLab/gssnng
  tools:
  - AnnData
  - Python
  - Scanpy
  - gssnng
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# scanpy-single-cell-data-loading

## Summary

Load single-cell RNA-seq data into memory as a Scanpy AnnData object (typically an .h5ad file, or a bundled dataset such as sc.datasets.pbmc3k_processed) so downstream tools in the scverse ecosystem, like gssnng gene set scoring, can operate on adata.X and adata.obs.

## When to use

Use when a scRNA-seq analysis pipeline starts from raw count/expression data that must be read into an AnnData object (e.g. an h5ad file via scanpy.read_*) before scoring, smoothing, or visualization can proceed.

## When NOT to use

- Input is already an in-memory AnnData object from a prior scanpy step — reloading is unnecessary
- Data is bulk RNA-seq or a plain feature/count table not organized as per-cell observations; gssnng-style AnnData workflows expect per-cell matrices
- Data is stored in a non-scverse format the pipeline cannot convert to h5ad/AnnData without extra restructuring

## Inputs

- AnnData object file in h5ad format (scverse ecosystem)
- Expression matrix accessible as adata.X from the loaded AnnData
- Optionally adata.obs with a group label column (e.g. cluster, sample) used by downstream 'groupby' parameters

## Outputs

- In-memory scanpy AnnData object ready for gene set scoring or smoothing

## How to apply

Read the single-cell dataset with Scanpy — either from an h5ad file on disk using scanpy.read_* or from a bundled processed dataset such as sc.datasets.pbmc3k_processed(), whose expression matrix is taken from adata.X. Confirm the object is a valid AnnData from the scverse ecosystem, since tools like gssnng 'work with Scanpy AnnData objects stored as h5ad files'. If downstream scoring requires per-cell groupings (e.g. cluster or cluster*sample labels for the 'groupby' parameter of scores_cells.with_gene_sets), verify the relevant column exists in adata.obs. Optionally subset or preprocess the matrix before passing the AnnData to scoring functions; for overlap-style diagnostics, expression values in adata.X are compared against a user-given threshold. The loaded AnnData is then handed directly to the downstream tool (e.g. scores_cells or a smoothing notebook), which returns scores in adata.obsm['gssnng_estimate'].

## Related tools

- **Scanpy** (Reads the h5ad file / bundled processed dataset (e.g. sc.datasets.pbmc3k_processed) into an AnnData object for downstream scoring) — https://github.com/IlyaLab/gssnng
- **AnnData** (Container format holding adata.X expression values and adata.obs metadata consumed by gssnng)
- **gssnng** (Downstream consumer of the loaded AnnData; works with Scanpy AnnData objects stored as h5ad files) — https://github.com/IlyaLab/gssnng

## Examples

```
import scanpy as sc; q = sc.datasets.pbmc3k_processed()
```

## Evaluation signals

- Loaded object is a scanpy AnnData instance with a populated expression matrix at adata.X (as in q = sc.datasets.pbmc3k_processed())
- The object can be passed directly to gssnng scores_cells / smoothing functions without format errors, confirming scverse AnnData compatibility
- adata.obs contains any group labels required by downstream 'groupby' parameters (e.g. cluster, sample)
- Genes in adata.X cover the genes referenced in downstream gene set files (e.g. cibersort_lm22.gmt) so overlap/scoring is non-degenerate

## Limitations

- Scanpy loading only covers ingestion into AnnData; it does not itself smooth or score — gssnng must be applied afterwards on the nearest neighbor graph
- The article provides no discussion-section guidance on edge cases, and sparsity/technical noise of single-cell expression remains a problem that downstream smoothing (not loading) addresses

## Evidence

- [methods] Read in an AnnData object using scanpy: "1. Read in an AnnData object using scanpy (an h5ad file)."
- [methods] pbmc3k dataset load: "q = sc.datasets.pbmc3k_processed()"
- [readme] AnnData compatibility claim: "This package is part of the [scverse ecosystem](https://scverse.org/packages/#ecosystem) and works with Scanpy AnnData objects stored as h5ad files."
- [abstract] sparsity motivates downstream smoothing: "the sparsity and technical noise of single-cell expression measures create difficulties for these methods"
