---
name: cross-dataset-overlapping-gene-selection
description: Use when a reference dataset (e.g. annotated HuBMAP data such as HBMP3-spleen-CC3)
  is being used to transfer labels to a query dataset (e.g. HBMP2-spleen-2) whose
  AnnData object has a different gene set, or when comparing clusterings across the
  two objects.
when_to_use_negative:
- Reference and query come from the same experiment or were processed together, so
  their gene sets already match and intersection only removes useful features.
- The datasets have minimal shared gene space (e.g. cross-species or scATAC gene-activity
  vs scRNA-seq matrices), where the intersection is too small to support a stable
  reference PCA space.
- The analysis goal is not reference-based label transfer or cross-dataset clustering
  comparison (e.g. single-dataset differential expression with diffxpy), where subsetting
  to an external dataset's genes would bias results.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: AnnData
  role: In-memory container whose var index is intersected and used to subset both
    reference and query matrices
- name: scanpy
  role: Preprocessing (filtering, CPM normalization, log1p, scaling) applied to the
    gene-intersected datasets
- name: scanpy Ingest
  role: 'Downstream consumer of the aligned gene space: projects the query into the
    reference PCA space to transfer labels'
- name: Cellar
  role: Interactive tool in which the reference-based alignment and semi-supervised
    clustering on the intersected features are performed
  repo: github.com/euxhenh/cellar
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:907fb039655ca9ffc9479250d3838c7e680c9f0848425368da2440647ecc8dae
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AnnData
  - Cellar
  - scanpy
  - scanpy Ingest
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Cross-dataset overlapping gene selection

## Summary

Restricts a reference and a query dataset loaded as AnnData objects to the genes they share, so that label transfer or clustering comparison operates on a common feature space. In the Cellar HBMP2-spleen-2 vs HBMP3-spleen-CC3 analysis, this intersection was the prerequisite for Ingest-based label transfer in the 40-component PCA space.

## When to use

Use when a reference dataset (e.g. annotated HuBMAP data such as HBMP3-spleen-CC3) is being used to transfer labels to a query dataset (e.g. HBMP2-spleen-2) whose AnnData object has a different gene set, or when comparing clusterings across the two objects.

## When NOT to use

- Reference and query come from the same experiment or were processed together, so their gene sets already match and intersection only removes useful features.
- The datasets have minimal shared gene space (e.g. cross-species or scATAC gene-activity vs scRNA-seq matrices), where the intersection is too small to support a stable reference PCA space.
- The analysis goal is not reference-based label transfer or cross-dataset clustering comparison (e.g. single-dataset differential expression with diffxpy), where subsetting to an external dataset's genes would bias results.

## Inputs

- Reference AnnData object (e.g. HBMP3-spleen-CC3 with human-expert cell-type annotations)
- Query AnnData object (e.g. HBMP2-spleen-2)
- Gene/var index tables of both datasets for computing the overlap

## Outputs

- Reference and query AnnData objects subset to shared overlapping genes
- Aligned gene/var index shared by both objects, ready for PCA/Ingest label transfer

## How to apply

Load both datasets as AnnData objects, then intersect their var (gene) indices and subset each AnnData to only the overlapping genes before any preprocessing or modeling. This keeps the reference and query in the same feature space, which is required because scanpy Ingest projects the query into a PCA latent space (here 40 components) fit on the reference. After intersection, apply the article's standard preprocessing to the aligned matrices: CPM normalization to total count 1e5, log1p transform, and scaling to unit variance and zero mean, optionally preceded by cell/gene quality filtering (e.g. drop cells with fewer than 50 or more than 3000 expressed genes). Rationale: unconstrained intersections at other levels (e.g. per-dataset top-HVG selections) would misalign features and break the neighbors graph used for Leiden clustering and Ingest label mapping.

## Related tools

- **AnnData** (In-memory container whose var index is intersected and used to subset both reference and query matrices)
- **scanpy** (Preprocessing (filtering, CPM normalization, log1p, scaling) applied to the gene-intersected datasets)
- **scanpy Ingest** (Downstream consumer of the aligned gene space: projects the query into the reference PCA space to transfer labels)
- **Cellar** (Interactive tool in which the reference-based alignment and semi-supervised clustering on the intersected features are performed) — github.com/euxhenh/cellar

## Examples

```
shared = adata_ref.var_names.intersection(adata_query.var_names); adata_ref = adata_ref[:, shared]; adata_query = adata_query[:, shared]; sc.tl.ingest(adata_query, adata_ref, obs='cell_type', embedding_method='pca', n_pcs=40)
```

## Evaluation signals

- After subsetting, both AnnData objects have identical, sorted var indices; the number of overlapping genes is logged and > 0.
- The reference PCA is fit after intersection, and Ingest projects the query onto exactly 40 components of that reference space.
- Label transfer, unconstrained Leiden (resolution 1), and semi-supervised Leiden ARI scores against human-expert annotations land in the reported regime (Ingest 0.39 > unconstrained Leiden 0.27; frozen-cluster Leiden 0.66), matching the ordering of Supplementary Fig. 2.
- Downstream CPM/log1p/scaling is applied to the intersected matrix, not to a matrix that still contains non-shared genes.
- Feature-alignment invariant: any gene present in the query's processed matrix must also be present in the reference matrix used to fit the PCA.

## Limitations

- The intersection discards genes unique to one dataset, which can remove informative features if the shared gene space is small.
- The paper does not state software versions for Cellar, Python, Dash, or scanpy, nor accession IDs for the HuBMAP reference datasets, so exact reproduction of the gene intersection depends on which genome annotation (e.g. GENCODE version) the user resolves.
- Large datasets rely on AnnData memory-mapping, and faiss approximate neighbors (15 neighbors) introduces approximation into downstream graphs built on the intersected features.

## Evidence

- [methods] Reference and query must share genes before label transfer: "Label transfer between HBMP2-spleen-2 and HBMP3-spleen-CC3 was performed using scanpy’s Ingest"
- [methods] Standard preprocessing thresholds on the aligned matrix: "we filtered cells with less than 50 or more than 3000 expressed genes"
- [discussion] AnnData is the data container for large reference datasets: "using the Annotated Data object17 in memory-mapping mode which allows the analysis of large datasets"
- [discussion] Annotated HuBMAP datasets serve as transfer references: "These annotated datasets (mostly from HuBMAP) can serve as a reference for transferring labels to other datasets."
