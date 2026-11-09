---
name: marker-gene-database-matching
description: Use when you have cluster assignments from a scRNA-seq clustering method
  (e.g. scBGEDA) and need to assign biological cell-type identities by matching cluster
  DEGs to a curated marker gene database such as CellMark.
when_to_use_negative:
- The input is raw unclustered scRNA-seq data with no cluster labels — clustering
  (e.g. scBGEDA's preprocess.py → SDDA.py → BC.m pipeline) must come first
- The target cell types are not covered by the marker database, so DEG-to-database
  matching cannot validate or annotate the clusters
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scBGEDA
  role: Produces the predicted cluster labels whose per-cluster DEGs are matched to
    marker genes
  repo: https://github.com/wangyh082/scBGEDA
- name: scanpy
  role: AnnData container and rank_genes_groups (method='wilcoxon') differential expression
    testing
- name: Python
  role: Runtime for the scanpy-based marker gene workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad075
    title: 'scBGEDA: deep single-cell clustering analysis via a dual denoising autoencoder
      with bipartite graph ensemble clustering'
  source_content_hash: sha256:f2d86e8d6c784b629500abf482b7db01d5dc9a727c7106723e3b3bbacdb86284
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# marker-gene-database-matching

## Summary

Assign cell-type labels to scBGEDA clusters by running per-cluster differential expression (Wilcoxon rank-sum via scanpy rank_genes_groups, top 20 genes at P<0.05) and matching the nominated markers against the CellMark curated marker database, e.g. fibroblast markers Gsn, Col3a1, Col1a2, Mmp2 and endothelial markers Fabp4, Egfl7, Flt1, Pecam1.

## When to use

Use when you have cluster assignments from a scRNA-seq clustering method (e.g. scBGEDA) and need to assign biological cell-type identities by matching cluster DEGs to a curated marker gene database such as CellMark.

## When NOT to use

- The input is raw unclustered scRNA-seq data with no cluster labels — clustering (e.g. scBGEDA's preprocess.py → SDDA.py → BC.m pipeline) must come first
- The target cell types are not covered by the marker database, so DEG-to-database matching cannot validate or annotate the clusters

## Inputs

- Gene expression matrix (count/expression matrix, scanpy-readable) for a scRNA-seq dataset such as QS_Heart
- Per-cell cluster assignments from scBGEDA's bipartite graph ensemble clustering
- Curated marker gene database (CellMark)

## Outputs

- Per-cluster DEG table (top 20 Wilcoxon-ranked genes per cluster at P<0.05)
- Annotated per-cluster marker gene CSV (e.g. qs_heart_marker_genes.csv) with cluster-to-cell-type assignments

## How to apply

Load the gene expression matrix together with the cluster assignments (e.g. scBGEDA-predicted labels) into a scanpy AnnData object. For each cluster, run scanpy's rank_genes_groups with method='wilcoxon' to rank genes by differential expression, retaining the top 20 genes per cluster at P<0.05. Compile the per-cluster DEG table and compare the nominated markers against the CellMark database to assign cell-type labels (e.g. fibroblasts via Gsn, Col3a1, Col1a2, Mmp2; endothelial cells via Fabp4, Egfl7, Flt1, Pecam1). The rationale, following scBGEDA's demonstration that the method 'was able to identify cell-type specific marker genes', is that cluster-level DEGs that overlap published markers validate the clustering and enable functional annotation. Write the annotated per-cluster marker gene results to a CSV such as qs_heart_marker_genes.csv.

## Related tools

- **scBGEDA** (Produces the predicted cluster labels whose per-cluster DEGs are matched to marker genes) — https://github.com/wangyh082/scBGEDA
- **scanpy** (AnnData container and rank_genes_groups (method='wilcoxon') differential expression testing)
- **Python** (Runtime for the scanpy-based marker gene workflow)

## Examples

```
sc.tl.rank_genes_groups(adata, groupby='cluster', method='wilcoxon'); degs = sc.get.rank_genes_groups_df(adata, group=None).query('pvals_adj < 0.05')
```

## Evaluation signals

- Each annotated cluster's top-20 Wilcoxon DEGs overlap CellMark entries for its assigned cell type (e.g. fibroblast markers Gsn, Col3a1, Col1a2, Mmp2 all present)
- All retained DEGs pass the P<0.05 threshold from rank_genes_groups
- The output CSV contains one annotated marker-gene row set per cluster, consistent with the marker gene panel reported in Fig 7
- Known marker panels reproduce: endothelial clusters carry Fabp4, Egfl7, Flt1, Pecam1 among their top DEGs

## Limitations

- Validation depends on marker database coverage; genes absent from CellMark cannot contribute supporting evidence
- Only the top 20 genes per cluster at P<0.05 are retained, so weakly expressed markers may be missed
- Wilcoxon rank-sum on cluster labels assumes the input clusters are meaningful; noisy scBGEDA labels will propagate to misannotation

## Evidence

- [methods] scBGEDA was able to identify cell-type specific marker genes and provide functional genomic analysis: "scBGEDA was able to identify cell-type specific marker genes and provide functional genomic analysis"
- [supplementary] Fig 7 marker gene panel from source card: [evidence span withheld — anchor status: not_found]
