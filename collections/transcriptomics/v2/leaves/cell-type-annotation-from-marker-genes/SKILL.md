---
name: cell-type-annotation-from-marker-genes
description: Use when scRNA-seq cluster assignments produced by an unsupervised method
  (e.g. scBGEDA) lack cell-type labels and a curated marker database such as CellMark
  can be matched against per-cluster DEGs.
when_to_use_negative:
- The input data has no cluster labels and clustering itself is the goal — scBGEDA's
  pipeline (preprocess.py, SDDA.py, BC.m) produces the consensus clusters that this
  annotation step consumes.
- The clusters of interest show no overlap with any known markers in the curated database
  — assignment would be unsupported guesswork.
- Input is bulk RNA-seq or a precomputed feature table without per-cell data — per-cluster
  Wilcoxon rank-sum testing on cells is not applicable.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scanpy
  role: Provides rank_genes_groups (method='wilcoxon') to compute per-cluster DEGs
    from the AnnData object for marker nomination
- name: scBGEDA
  role: Produces the predicted cluster labels whose biological identity the DEG-marker
    matching assigns
  repo: https://github.com/wangyh082/scBGEDA
- name: Python
  role: Runtime environment for the scanpy annotation workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad075
    title: 'scBGEDA: deep single-cell clustering analysis via a dual denoising autoencoder
      with bipartite graph ensemble clustering'
  source_content_hash: sha256:bbaac55ace5d45c110d3b331ea681ad1aca9f6c66b41658914dfbe0fb6ab81a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangyh082/scBGEDA
  tools:
  - Python
  - scBGEDA
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# cell-type-annotation-from-marker-genes

## Summary

Assigns biological cell-type labels to unsupervised scBGEDA clusters by running per-cluster differential expression (Wilcoxon rank-sum, top 20 genes at P<0.05 in scanpy) and matching the nominated marker genes to the CellMark curated marker database (e.g. fibroblast markers Gsn, Col3a1, Col1a2, Mmp2 on the QS_Heart dataset).

## When to use

Use when scRNA-seq cluster assignments produced by an unsupervised method (e.g. scBGEDA) lack cell-type labels and a curated marker database such as CellMark can be matched against per-cluster DEGs.

## When NOT to use

- The input data has no cluster labels and clustering itself is the goal — scBGEDA's pipeline (preprocess.py, SDDA.py, BC.m) produces the consensus clusters that this annotation step consumes.
- The clusters of interest show no overlap with any known markers in the curated database — assignment would be unsupported guesswork.
- Input is bulk RNA-seq or a precomputed feature table without per-cell data — per-cluster Wilcoxon rank-sum testing on cells is not applicable.

## Inputs

- Gene expression matrix from a scRNA-seq dataset (e.g. QS_Heart)
- scBGEDA cluster assignments (predicted cluster labels per cell)
- CellMark curated marker-gene database

## Outputs

- Per-cluster DEG table (top 20 Wilcoxon genes per cluster at P<0.05)
- qs_heart_marker_genes.csv with cell-type annotations per cluster

## How to apply

Load the gene expression matrix together with the scBGEDA cluster assignments into a scanpy AnnData object. For each cluster, run sc.tl.rank_genes_groups with method='wilcoxon', retaining the top 20 genes per cluster at P<0.05. Compile the per-cluster DEG table and compare the nominated markers (fibroblasts: Gsn, Col3a1, Col1a2, Mmp2; endothelial cells: Fabp4, Egfl7, Flt1, Pecam1) against the CellMark curated marker database to assign cell-type labels. Write the annotated per-cluster marker gene results to qs_heart_marker_genes.csv. The rationale is that scBGEDA quantifies the influence of genes on cell clusters, so its consensus clusters should show enrichment of published, cell-type-specific markers if the clustering is biologically meaningful.

## Related tools

- **scanpy** (Provides rank_genes_groups (method='wilcoxon') to compute per-cluster DEGs from the AnnData object for marker nomination)
- **scBGEDA** (Produces the predicted cluster labels whose biological identity the DEG-marker matching assigns) — https://github.com/wangyh082/scBGEDA
- **Python** (Runtime environment for the scanpy annotation workflow)

## Examples

```
sc.tl.rank_genes_groups(adata, groupby='scBGEDA_cluster', method='wilcoxon')
```

## Evaluation signals

- Fibroblast and endothelial cluster annotations in the output table must be supported by their top-20 Wilcoxon DEGs matching the CellMark entries.
- Identified fibroblast DEGs include Gsn, Col3a1, Col1a2 and Mmp2, consistent with the marker gene panel reported in Fig 7.
- Endothelial DEGs include Fabp4, Egfl7, Flt1 and Pecam1.
- Each retained gene passes the P<0.05 threshold and only the top 20 genes per cluster are used.
- Output CSV (qs_heart_marker_genes.csv) contains an annotated per-cluster marker gene table.

## Limitations

- Annotation quality depends on the coverage of the CellMark database; clusters whose dominant genes are absent from it cannot be labeled.
- The article evaluates scBGEDA primarily with clustering metrics (ARI/NMI median values from rand_index.m and computeNMI.m); marker-based annotation is a downstream interpretation, not an optimized output of the tool itself.
- Requires prior knowledge of the expected cell-type marker panel (e.g. the Fig 7 fibroblast/endothelial panel) to nominate candidate markers for matching.

## Evidence

- [abstract] scBGEDA was able to identify cell-type specific marker genes and provide functional genomic analysis by quantifying the influence of genes on cell clusters: "scBGEDA was able to identify cell-type specific marker genes and provide functional genomic analysis by quantifying the influence of genes on cell clusters"
- [other] For each cluster, run the Wilcoxon rank-sum test via scanpy's rank_genes_groups (method='wilcoxon') to rank genes by differential expression, retaining the top 20 genes per cluster at P<0.05.: [evidence span withheld — anchor status: not_found]
- [other] Most of the DEGs identified by scBGEDA on QS_Heart match published marker genes in CellMark: Gsn, Col3a1, Col1a2 and Mmp2 are fibroblast markers.: [evidence span withheld — anchor status: not_found]
- [readme] The real single-cell RNA-seq datasets and source codes for our analyses are freely available at (https://figshare.com/articles/software/scBGEDA/19657911).: "The real single-cell RNA-seq datasets and source codes for our analyses are freely available at (https://figshare.com/articles/software/scBGEDA/19657911)."
