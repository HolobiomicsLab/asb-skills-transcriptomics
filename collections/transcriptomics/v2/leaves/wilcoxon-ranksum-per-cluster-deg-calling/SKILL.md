---
name: wilcoxon-ranksum-per-cluster-deg-calling
description: Use when you have a gene expression matrix plus scBGEDA cluster assignments
  (e.g. the QS_Heart dataset) and need to nominate per-cluster marker genes for cell-type
  annotation via known marker databases.
when_to_use_negative:
- You do not yet have cluster labels — this skill annotates existing scBGEDA clusters,
  it does not perform the dual denoising autoencoder / bipartite graph ensemble clustering
  itself
- You need genome-wide differential expression statistics or effect-size ranking rather
  than a short top-N marker shortlist for cell-type annotation
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scanpy
  role: Provides sc.tl.rank_genes_groups (method='wilcoxon') used to rank genes by
    differential expression within each scBGEDA cluster; version 1.7.2 is pinned in
    the scBGEDA environment
- name: scBGEDA
  role: Supplies the input cluster assignments whose cell-type identity is resolved
    by the per-cluster DEG calling
  repo: https://github.com/wangyh082/scBGEDA
- name: Python
  role: Runtime for the AnnData/scanpy DEG workflow and for writing the marker gene
    CSV output
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad075
    title: 'scBGEDA: deep single-cell clustering analysis via a dual denoising autoencoder
      with bipartite graph ensemble clustering'
  source_content_hash: sha256:d87ec465cd20e4b8ea21ef57b39c77e3dee3fd9cc84503063b33e51c709738f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangyh082/scBGEDA
  tools:
  - Python
  - scBGEDA
  - scanpy
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# wilcoxon-ranksum-per-cluster-deg-calling

## Summary

Identifies cell-type marker genes from scBGEDA-predicted cluster labels by running the Wilcoxon rank-sum test per cluster in scanpy, retaining the top 20 DEGs per cluster at P<0.05 and matching them against the CellMark marker database.

## When to use

Use when you have a gene expression matrix plus scBGEDA cluster assignments (e.g. the QS_Heart dataset) and need to nominate per-cluster marker genes for cell-type annotation via known marker databases.

## When NOT to use

- You do not yet have cluster labels — this skill annotates existing scBGEDA clusters, it does not perform the dual denoising autoencoder / bipartite graph ensemble clustering itself
- You need genome-wide differential expression statistics or effect-size ranking rather than a short top-N marker shortlist for cell-type annotation

## Inputs

- Gene expression count matrix for the scRNA-seq dataset (e.g. QS_Heart), loadable into a scanpy AnnData object
- Per-cell cluster labels produced by scBGEDA's bipartite graph ensemble clustering
- CellMark curated cell-type marker gene database

## Outputs

- Per-cluster DEG table (top 20 Wilcoxon-ranked genes per cluster at P<0.05)
- Annotated per-cluster marker gene CSV (e.g. qs_heart_marker_genes.csv) with cell-type labels
- Set of matched marker genes per cell type (e.g. fibroblasts: Gsn, Col3a1, Col1a2, Mmp2; endothelial cells: Fabp4, Egfl7, Flt1, Pecam1)

## How to apply

Load the gene expression matrix together with the scBGEDA cluster assignments into a scanpy AnnData object (scanpy 1.7.2 is the version used in the scBGEDA environment). For each cluster, run sc.tl.rank_genes_groups with method='wilcoxon', retaining the top 20 genes per cluster at P<0.05. Compile the per-cluster DEG table and compare nominated markers against the CellMark curated database — on QS_Heart the fibroblast markers are Gsn, Col3a1, Col1a2, Mmp2 and endothelial markers are Fabp4, Egfl7, Flt1, Pecam1. Assign a cell-type label to a cluster only when its top-20 Wilcoxon DEGs match the CellMark entries, consistent with the marker panel reported in Fig 7, then write the annotated results to a per-cluster marker gene CSV (e.g. qs_heart_marker_genes.csv).

## Related tools

- **scanpy** (Provides sc.tl.rank_genes_groups (method='wilcoxon') used to rank genes by differential expression within each scBGEDA cluster; version 1.7.2 is pinned in the scBGEDA environment)
- **scBGEDA** (Supplies the input cluster assignments whose cell-type identity is resolved by the per-cluster DEG calling) — https://github.com/wangyh082/scBGEDA
- **Python** (Runtime for the AnnData/scanpy DEG workflow and for writing the marker gene CSV output)

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('qs_heart_with_scbgeda_labels.h5ad'); sc.tl.rank_genes_groups(adata, 'cluster', method='wilcoxon'); sc.get.rank_genes_groups_df(adata, group=None).to_csv('qs_heart_marker_genes.csv')
```

## Evaluation signals

- Every cluster in qs_heart_marker_genes.csv carries a cell-type annotation supported by its top-20 Wilcoxon DEGs
- Fibroblast-annotated clusters show Gsn, Col3a1, Col1a2 and Mmp2 among their top DEGs, and endothelial-annotated clusters show Fabp4, Egfl7, Flt1 and Pecam1, consistent with the CellMark entries
- All retained DEGs pass the P<0.05 threshold, and exactly the top 20 genes per cluster are kept
- Cluster assignments used as input match the scBGEDA output labels (from BC.m / BGEC.m ensemble clustering) rather than ground-truth labels

## Limitations

- Marker assignment depends on CellMark coverage — DEGs absent from the curated database cannot be mapped to a cell type
- The scBGEDA paper reports cell-type specific marker identification and functional genomic analysis at a high level; the QS_Heart fibroblast/endothelial marker panel (Fig 7) is the concrete validation anchor, so other tissues need their own marker panels
- Wilcoxon rank-sum results are sensitive to cluster quality — poor scBGEDA consensus partitions will propagate into misleading DEG calls

## Evidence

- [abstract] marker identification via scBGEDA: "scBGEDA was able to identify cell-type specific marker genes and provide functional genomic analysis by quantifying the influence of genes on cell clusters"
- [other] per-cluster Wilcoxon DEG workflow: [evidence span withheld — anchor status: not_found]
- [other] marker panel and validation: [evidence span withheld — anchor status: not_found]
- [readme] scanpy dependency: "The real single-cell RNA-seq datasets and source codes for our analyses are freely available at (https://figshare.com/articles/software/scBGEDA/19657911)."
