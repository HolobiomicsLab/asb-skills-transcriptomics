---
name: percent-spliced-reads-per-cluster-summary
description: Use when you have a droplet-filtered snRNA-seq count matrix (e.g., quantile-threshold-filtered)
  and need to test whether background RNA-driven debris clusters persist in downstream
  Seurat clustering.
when_to_use_negative:
- Droplets have already been filtered with an expression-distribution method such
  as DIEM or EmptyDrops, so debris clusters are expected to be removed and the diagnostic
  adds little
- Input is not droplet-level raw counts (e.g., already a normalized feature table
  or aggregated pseudobulk matrix)
- Data are PBMC scRNA-seq, where the paper reports the debris-score/percent-spliced
  correlation was less successful due to smaller differences between background and
  cell type profiles
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DIEM
  role: Supplies convert_to_seurat to carry the SCE object, droplet metadata, and
    debris score into Seurat after droplet filtering
  repo: https://github.com/marcalva/diem
- name: Seurat
  role: Performs log-normalization, variable gene selection (top 2,000), PCA (30 PCs),
    Louvain clustering, and UMAP embedding of the passing droplets
- name: Matrix
  role: Stores the raw counts as a column-wise sparse matrix that is subset to kept
    droplets
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-020-67513-5
    title: Enhancing droplet-based single-nucleus RNA-seq resolution using the semi-supervised
      machine learning classifier DIEM
  source_content_hash: sha256:d5278e9d539141e54b6bd24e49fcffd377862d053ad226255c19847d90f6bffb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marcalva/diem
  tools:
  - DIEM
  - Matrix
  - R
  - Seurat
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 18
---

# percent-spliced-reads-per-cluster-summary

## Summary

A validation/diagnostic step that clusters droplets with Seurat (log-normalization, top 2000 variable genes, 30 PCs, Louvain clustering, UMAP) and computes the median percent spliced reads per cluster to expose residual background/debris clusters that survive hard count-threshold filtering.

## When to use

Use when you have a droplet-filtered snRNA-seq count matrix (e.g., quantile-threshold-filtered) and need to test whether background RNA-driven debris clusters persist in downstream Seurat clustering.

## When NOT to use

- Droplets have already been filtered with an expression-distribution method such as DIEM or EmptyDrops, so debris clusters are expected to be removed and the diagnostic adds little
- Input is not droplet-level raw counts (e.g., already a normalized feature table or aggregated pseudobulk matrix)
- Data are PBMC scRNA-seq, where the paper reports the debris-score/percent-spliced correlation was less successful due to smaller differences between background and cell type profiles

## Inputs

- Sparse droplet count matrix (R Matrix package format, e.g., from 10X CellRanger output read via diem's read_10x) restricted to droplets passing a quantile-based hard count threshold
- diem SCE object droplet metadata including the debris score
- Per-droplet percent spliced reads annotation

## Outputs

- Per-cluster summary CSV with one row per Seurat cluster and a median percent-spliced-reads field
- Seurat object with Louvain cluster assignments and UMAP embedding

## How to apply

Extract the kept droplets from the sparse count matrix (R Matrix package) and build a Seurat object with the diem convenience function convert_to_seurat, passing min.features = 50 and carrying the diem droplet metadata (including the debris score) into the Seurat meta data. Restrict to droplets with at least 200 detected genes, apply log-normalization, and identify the top 2,000 variable genes per the standardized pipeline. Run PCA retaining 30 principal components, cluster with Louvain clustering on the PC embedding, and compute a UMAP for visualization. For each Seurat cluster compute the median percent spliced reads and assemble a per-cluster summary table; clusters with very high median percent spliced (e.g., the reported 88.5% in the fresh mouse brain snRNA-seq data) indicate residual ambient/extranuclear RNA contamination. Rationale: the paper shows hard count cutoffs can retain debris droplets that DIEM and EmptyDrops remove, and percent spliced reads serves as a proxy for extranuclear contamination.

## Related tools

- **DIEM** (Supplies convert_to_seurat to carry the SCE object, droplet metadata, and debris score into Seurat after droplet filtering) — https://github.com/marcalva/diem
- **Seurat** (Performs log-normalization, variable gene selection (top 2,000), PCA (30 PCs), Louvain clustering, and UMAP embedding of the passing droplets)
- **Matrix** (Stores the raw counts as a column-wise sparse matrix that is subset to kept droplets)

## Examples

```
seur <- convert_to_seurat(sce, min.features = 50); seur <- subset(seur, subset = nFeature_RNA >= 200); seur <- NormalizeData(seur) %>% FindVariableFeatures(nfeatures = 2000) %>% RunPCA(npcs = 30) %>% FindNeighbors() %>% FindClusters() %>% RunUMAP(dims = 1:30)
```

## Evaluation signals

- Output CSV exists and contains exactly one row per Seurat cluster with a median-percent-spliced-reads field
- The residual background/debris cluster's median percent spliced reads matches the reported value of 88.5% within a small numerical tolerance
- All droplets in the Seurat object satisfy the min.features = 50 conversion filter and the ≥200 detected genes restriction
- Variable feature count equals the specified 2,000 genes and PCA retains 30 principal components before Louvain clustering
- Debris cluster is identifiable by its extreme median percent spliced reads relative to putative cell-type clusters

## Limitations

- Debris scores and related contamination proxies may not be comparable across independent experiments because they are normalized relative to within-sample cluster means; when integrating multiple samples the distributions of extranuclear RNA may differ
- Correlation between contamination metrics and extranuclear RNA was less successful in PBMC single-cell RNA-seq data, possibly due to smaller differences between background and cell type profiles
- A hard count threshold can retain debris droplets (and lose nuclear droplets), so a clean result here still depends on the upstream filtering strategy

## Evidence

- [methods] quantile-based filtering retained a background cluster: [evidence span withheld — anchor status: not_found]
- [readme] convert_to_seurat: [evidence span withheld — anchor status: external_doc]
- [discussion] hard cutoff underperformed: "using a hard count threshold underperformed in comparison to EmptyDrops and DIEM. Both DIEM and EmptyDrops12 remove droplets based on their expression distribution."
- [abstract] spurious cell types: "snRNA-seq is commonly subject to contamination by high amounts of ambient RNA, which can lead to biased downstream analyses, such as identification of spurious cell types if overlooked."
