---
name: single-cell-cluster-gene-ranking
description: Use when you have an AnnData object with predicted cell-cluster labels
  from a scRNA-seq clustering method and need per-cluster marker genes to annotate
  cell types against a curated marker database like CellMark.
when_to_use_negative:
- The input lacks cluster labels — gene ranking requires pre-defined clusters; run
  clustering (e.g. scBGEDA) first
- The goal is quantifying gene influence on clusters via the model's learned latent
  representation (scBGEDA's built-in functional genomic analysis) rather than standard
  differential-expression ranking
- The dataset has no expectation of known cell-type composition, so CellMark-based
  annotation would be meaningless
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scBGEDA
  role: Provides the predicted cell-cluster labels whose per-cluster DEGs are ranked
    and annotated
  repo: https://github.com/wangyh082/scBGEDA
- name: scanpy
  role: Runs the Wilcoxon rank-sum test via rank_genes_groups on the AnnData object
    to produce per-cluster gene rankings
- name: Python
  role: Execution environment for loading the expression matrix, running rank_genes_groups,
    and writing the annotated CSV
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad075
    title: 'scBGEDA: deep single-cell clustering analysis via a dual denoising autoencoder
      with bipartite graph ensemble clustering'
  source_content_hash: sha256:a9746734734e5fa3f3c9edff78d7d960e57a7d82b6385d903a31ec1c955916eb
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
yes_rate: 0.6666666666666666
n_claims: 9
---

# single-cell-cluster-gene-ranking

## Summary

Ranks genes per cluster by differential expression using scanpy's Wilcoxon rank-sum test on an AnnData object built from predicted cluster labels (e.g. scBGEDA outputs), then matches top-ranked genes to curated marker databases to assign cell types.

## When to use

Use when you have an AnnData object with predicted cell-cluster labels from a scRNA-seq clustering method and need per-cluster marker genes to annotate cell types against a curated marker database like CellMark.

## When NOT to use

- The input lacks cluster labels — gene ranking requires pre-defined clusters; run clustering (e.g. scBGEDA) first
- The goal is quantifying gene influence on clusters via the model's learned latent representation (scBGEDA's built-in functional genomic analysis) rather than standard differential-expression ranking
- The dataset has no expectation of known cell-type composition, so CellMark-based annotation would be meaningless

## Inputs

- Gene expression matrix for a scRNA-seq dataset (e.g. QS_Heart) loaded into a scanpy AnnData object
- Predicted per-cell cluster labels from the upstream clustering method (e.g. scBGEDA consensus clustering)
- Curated cell-type marker gene database for comparison (CellMark)

## Outputs

- Per-cluster ranked DEG table (top 20 genes per cluster, Wilcoxon rank-sum, P<0.05)
- Annotated per-cluster marker gene results CSV (e.g. qs_heart_marker_genes.csv) with cell-type labels assigned via CellMark matching

## How to apply

Load the gene expression matrix together with the predicted cluster assignments (e.g. from scBGEDA's bipartite graph ensemble clustering) into a scanpy AnnData object. For each cluster, run sc.tl.rank_genes_groups with method='wilcoxon' to rank genes by differential expression, retaining the top 20 genes per cluster at P<0.05. Compile the per-cluster DEG table and compare nominated markers (e.g. fibroblasts: Gsn, Col3a1, Col1a2, Mmp2; endothelial cells: Fabp4, Egfl7, Flt1, Pecam1) against the CellMark curated marker database to assign cell-type labels. Write the annotated per-cluster marker results to a CSV file (e.g. qs_heart_marker_genes.csv); correct application requires that each cluster's cell-type annotation is supported by its top-20 Wilcoxon DEGs matching CellMark entries.

## Related tools

- **scBGEDA** (Provides the predicted cell-cluster labels whose per-cluster DEGs are ranked and annotated) — https://github.com/wangyh082/scBGEDA
- **scanpy** (Runs the Wilcoxon rank-sum test via rank_genes_groups on the AnnData object to produce per-cluster gene rankings)
- **Python** (Execution environment for loading the expression matrix, running rank_genes_groups, and writing the annotated CSV)

## Examples

```
sc.tl.rank_genes_groups(adata, groupby='cluster', method='wilcoxon')
```

## Evaluation signals

- Fibroblast cluster annotation is supported by top-20 Wilcoxon DEGs including Gsn, Col3a1, Col1a2 and Mmp2 matching CellMark entries
- Endothelial cluster annotation is supported by top-20 DEGs including Fabp4, Egfl7, Flt1 and Pecam1 matching CellMark entries
- All retained genes pass the P<0.05 threshold and the table contains exactly the top 20 genes per cluster
- Output file qs_heart_marker_genes.csv exists with per-cluster marker annotations consistent with the marker gene panel reported in Fig 7
- Every assigned cell-type label in the output table traces back to at least one CellMark-matched DEG

## Limitations

- Validation depends on the upstream cluster quality: scBGEDA's cluster labels were produced by its dual denoising autoencoder and bipartite graph ensemble clustering, so misclustering propagates into marker assignments
- Marker matching is restricted to genes present in the CellMark database; clusters whose markers are absent from CellMark cannot be annotated this way
- The Wilcoxon rank-sum test is a statistical ranking; top-20 lists at P<0.05 may include genes without known markers, leaving some clusters unlabeled

## Evidence

- [abstract] scBGEDA was able to identify cell-type specific marker genes and provide functional genomic analysis by quantifying the influence of genes on cell clusters: "scBGEDA was able to identify cell-type specific marker genes and provide functional genomic analysis by quantifying the influence of genes on cell clusters"
- [other] For each cluster, run the Wilcoxon rank-sum test via scanpy's rank_genes_groups: [evidence span withheld — anchor status: not_found]
- [other] Load the QS_Heart gene expression matrix together with the scBGEDA cluster assignments into a scanpy AnnData object: [evidence span withheld — anchor status: not_found]
- [supplementary] The real single-cell RNA-seq datasets and source codes for our analyses are freely available at (https://figshare.com/articles/software/scBGEDA/19657911).: "The real single-cell RNA-seq datasets and source codes for our analyses are freely available at (https://figshare.com/articles/software/scBGEDA/19657911)."
