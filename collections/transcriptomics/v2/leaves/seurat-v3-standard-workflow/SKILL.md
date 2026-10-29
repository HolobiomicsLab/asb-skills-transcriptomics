---
name: seurat-v3-standard-workflow
description: Use when a sparse count matrix of single-nucleus droplets (e.g., 10X
  mouse brain snRNA-seq) has been pre-filtered by DIEM, EmptyDrops, or a quantile-based
  hard count cutoff and you need to check for residual background/debris clusters
  downstream.
when_to_use_negative:
- Droplets have not yet been filtered — apply DIEM (estimate_dbr_score, call_targets
  with thresh_score = 0.5) or an equivalent filter first; clustering unfiltered data
  re-introduces debris droplets
- Data are PBMC single-cell RNA-seq with little ambient RNA signal — the article notes
  debris score correlation with extranuclear contamination was 'less successful' in
  PBMC scRNA-seq, so debris-cluster detection is less reliable
- Input is already a processed/normalized Seurat object or feature table rather than
  raw droplet counts
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3316
tools:
- name: Seurat
  role: Performs normalization, variable gene detection, PCA, Louvain clustering,
    and UMAP on the filtered droplets; the filtered SCE object is converted via diem's
    convert_to_seurat
  repo: https://github.com/marcalva/diem
- name: diem
  role: Provides convert_to_seurat to carry filtered droplet counts and debris score
    metadata into Seurat, and upstream filtering functions (set_debris_test_set, filter_genes,
    estimate_dbr_score, call_targets)
  repo: https://github.com/marcalva/diem
- name: Matrix
  role: Stores the raw droplet counts as a column-wise sparse matrix loaded via read_10x
    before conversion to Seurat
- name: R
  role: Implementation language for the diem package and the Seurat clustering pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:ab7eb58d8bc8e48a70725801b3d11831be06649a9a7b9e6f6942c5be79140eb1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marcalva/diem
  tools:
  - DIEM
  - Matrix
  - R
  - Seurat
  - diem
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9545454545454546
n_claims: 22
---

# seurat-v3-standard-workflow

## Summary

A standardized Seurat v3 clustering pipeline (filtering, log-normalization, top 2,000 variable genes, 30-component PCA, Louvain clustering, UMAP) used to re-cluster DIEM- or quantile-filtered droplets and detect residual debris clusters via median percent spliced reads per cluster.

## When to use

Use when a sparse count matrix of single-nucleus droplets (e.g., 10X mouse brain snRNA-seq) has been pre-filtered by DIEM, EmptyDrops, or a quantile-based hard count cutoff and you need to check for residual background/debris clusters downstream.

## When NOT to use

- Droplets have not yet been filtered — apply DIEM (estimate_dbr_score, call_targets with thresh_score = 0.5) or an equivalent filter first; clustering unfiltered data re-introduces debris droplets
- Data are PBMC single-cell RNA-seq with little ambient RNA signal — the article notes debris score correlation with extranuclear contamination was 'less successful' in PBMC scRNA-seq, so debris-cluster detection is less reliable
- Input is already a processed/normalized Seurat object or feature table rather than raw droplet counts

## Inputs

- Sparse column-wise count matrix (R Matrix package format) of droplets retained by an upstream filter (DIEM, EmptyDrops, or quantile-based hard count threshold), e.g., from read_10x of 10X CellRanger output
- diem SCE object with droplet metadata including debris score
- Per-droplet percent spliced reads annotation for computing cluster medians

## Outputs

- Seurat object with PCA (30 PCs), Louvain cluster assignments, and UMAP embedding
- Per-cluster summary CSV containing one row per cluster with a median-percent-spliced-reads field
- Identification of the residual background/debris cluster (e.g., median percent spliced reads of 88.5%)

## How to apply

Convert the cleaned droplet counts into a Seurat object with the diem convenience function convert_to_seurat (min.features = 50), carrying the diem droplet metadata including the debris score into the Seurat meta data. Restrict to droplets with at least 200 detected genes, apply log-normalization of counts, and identify the top 2,000 variable genes as specified for the standardized pipeline. Run PCA retaining 30 principal components, cluster droplets with Louvain clustering on the PC embedding, and compute a UMAP embedding for visualization. For each Seurat cluster, compute the median percent spliced reads and assemble a per-cluster summary table; a cluster with very high median percent spliced reads (e.g., 88.5% in the mouse brain data) is the residual background/debris cluster, indicating the upstream filter failed to remove it.

## Related tools

- **Seurat** (Performs normalization, variable gene detection, PCA, Louvain clustering, and UMAP on the filtered droplets; the filtered SCE object is converted via diem's convert_to_seurat) — https://github.com/marcalva/diem
- **diem** (Provides convert_to_seurat to carry filtered droplet counts and debris score metadata into Seurat, and upstream filtering functions (set_debris_test_set, filter_genes, estimate_dbr_score, call_targets)) — https://github.com/marcalva/diem
- **Matrix** (Stores the raw droplet counts as a column-wise sparse matrix loaded via read_10x before conversion to Seurat)
- **R** (Implementation language for the diem package and the Seurat clustering pipeline)

## Examples

```
seur <- convert_to_seurat(sce, min.features = 50); seur <- subset(seur, nFeature_RNA >= 200); seur <- NormalizeData(seur); seur <- FindVariableFeatures(seur, nfeatures = 2000); seur <- RunPCA(seur, npcs = 30); seur <- FindNeighbors(seur, dims = 1:30); seur <- FindClusters(seur); seur <- RunUMAP(seur, dims = 1:30)
```

## Evaluation signals

- Output CSV exists and contains exactly one row per Seurat cluster with a median-percent-spliced-reads field
- The residual background cluster's median percent spliced reads matches the reported value of 88.5% within a small numerical tolerance
- Every passing droplet from the quantile-filtered input is represented in the Seurat object (no dropped barcodes; note diem v2.4.1 fixed a bug that dropped barcode IDs during normalization)
- Droplets retained have at least 200 detected genes and at least 50 features (min.features = 50 at convert_to_seurat)
- Top 2,000 variable genes and 30 principal components are used, matching the standardized pipeline specification

## Limitations

- Debris scores are normalized relative to within-sample cluster means and cannot be assumed to be on the same scale in independent experiments, complicating multi-sample integration
- Debris scoring relies on accurate estimation of debris-enriched genes; if few or no genes are increased in the background distribution, scores will be inaccurate
- This skill detects but does not remove debris — the source finding shows quantile-based hard count filtering still left a debris cluster (88.5% median spliced reads) that EmptyDrops and DIEM removed
- DIEM's clustering-based filtering was designed for snRNA-seq; performance on scRNA-seq was mixed, with less success on PBMC data

## Evidence

- [discussion] The debris score, however, cannot be assumed to exist on the same scale in independent experiments: [evidence span withheld — anchor status: not_found]
- [results] quantile-based filtering retained a background cluster with median percent spliced reads 88.5%: [evidence span withheld — anchor status: not_found]
- [discussion] DIEM clusters all droplets to identify debris cluster or putative cell type clusters: "DIEM first clusters all droplets generated from the experiment to identify droplets belonging either to the debris cluster or putative cell type clusters."
- [readme] convert_to_seurat in README workflow: "seur <- convert_to_seurat(sce)"
- [methods] sparse matrix storage: "The raw counts are stored in a column-wise sparse matrix using the implementation from the R package [Matrix]"
