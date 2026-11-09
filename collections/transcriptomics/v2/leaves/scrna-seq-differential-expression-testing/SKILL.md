---
name: scrna-seq-differential-expression-testing
description: Use when scRNA-seq expression data comes with predicted cluster labels
  (e.g. from scBGEDA on datasets like QS_Heart) and the goal is to assign cell-type
  identities by testing differential expression per cluster against published marker
  genes.
when_to_use_negative:
- 'Input cells have no cluster labels yet — clustering (e.g. scBGEDA''s three-module
  pipeline: preprocess.py, SDDA.py, BC.m) must be run first before marker testing
  is meaningful.'
- The data is bulk RNA-seq or a precomputed feature table rather than single-cell
  expression with per-cell labels — the Wilcoxon rank-sum test in scanpy requires
  per-cell expression grouped by cluster.
- No curated marker reference (e.g. CellMark) is available for the tissue or organism,
  since cell-type annotation depends on matching DEGs to published markers.
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scanpy
  role: Provides rank_genes_groups (method='wilcoxon') to compute per-cluster Wilcoxon
    rank-sum differential expression and top-20 gene rankings; part of the scBGEDA
    environment (scanpy 1.7.2)
- name: scBGEDA
  role: Produces the predicted cluster labels whose differential expression is tested;
    its marker-gene module supports cell-type identification by quantifying the influence
    of genes on cell clusters
  repo: https://github.com/wangyh082/scBGEDA
- name: CellMark
  role: Curated marker database used to match nominated per-cluster DEGs (e.g. Gsn,
    Col3a1, Col1a2, Mmp2 for fibroblasts) to published cell-type marker genes and
    assign labels
- name: Python
  role: Execution environment for the scanpy-based marker identification workflow
    and CSV export
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad075
    title: 'scBGEDA: deep single-cell clustering analysis via a dual denoising autoencoder
      with bipartite graph ensemble clustering'
  source_content_hash: sha256:e498b67a4a29c785a5e31e2eca3a25962ae86c19e5933be44f03976e89541174
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangyh082/scBGEDA
  tools:
  - CellMark
  - Python
  - scBGEDA
  - scanpy
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# scrna-seq-differential-expression-testing

## Summary

Identify cell-type marker genes from scRNA-seq cluster assignments by running the Wilcoxon rank-sum test per cluster in scanpy (rank_genes_groups, method='wilcoxon'), retaining the top 20 DEGs per cluster at P<0.05, and annotating clusters by matching nominated markers to the CellMark curated marker database.

## When to use

Use when scRNA-seq expression data comes with predicted cluster labels (e.g. from scBGEDA on datasets like QS_Heart) and the goal is to assign cell-type identities by testing differential expression per cluster against published marker genes.

## When NOT to use

- Input cells have no cluster labels yet — clustering (e.g. scBGEDA's three-module pipeline: preprocess.py, SDDA.py, BC.m) must be run first before marker testing is meaningful.
- The data is bulk RNA-seq or a precomputed feature table rather than single-cell expression with per-cell labels — the Wilcoxon rank-sum test in scanpy requires per-cell expression grouped by cluster.
- No curated marker reference (e.g. CellMark) is available for the tissue or organism, since cell-type annotation depends on matching DEGs to published markers.

## Inputs

- Gene expression count matrix (e.g. the QS_Heart scRNA-seq dataset)
- Per-cell cluster assignments (e.g. labels predicted by scBGEDA)
- Curated marker gene database (CellMark) with published cell-type marker panels

## Outputs

- Per-cluster DEG table of top-20 Wilcoxon-ranked genes at P<0.05
- Annotated per-cluster marker gene results CSV (qs_heart_marker_genes.csv) with cell-type labels (e.g. fibroblasts, endothelial cells)

## How to apply

Load the gene expression matrix together with the cluster assignments (e.g. scBGEDA cluster labels for QS_Heart) into a scanpy AnnData object. For each cluster, run the Wilcoxon rank-sum test via scanpy's rank_genes_groups (method='wilcoxon'), ranking genes by differential expression and retaining the top 20 genes per cluster at P<0.05. Compile the per-cluster DEG table and compare nominated markers — for QS_Heart, fibroblast markers Gsn, Col3a1, Col1a2, Mmp2 and endothelial markers Fabp4, Egfl7, Flt1, Pecam1 — against the CellMark curated marker database to assign cell-type labels. Write the annotated per-cluster marker gene results to a CSV (e.g. qs_heart_marker_genes.csv). Rationale: scBGEDA identifies clusters but needs downstream marker quantification to translate them into biological cell types, and validated DEGs matching CellMark entries confirm the clustering is biologically meaningful.

## Related tools

- **scanpy** (Provides rank_genes_groups (method='wilcoxon') to compute per-cluster Wilcoxon rank-sum differential expression and top-20 gene rankings; part of the scBGEDA environment (scanpy 1.7.2))
- **scBGEDA** (Produces the predicted cluster labels whose differential expression is tested; its marker-gene module supports cell-type identification by quantifying the influence of genes on cell clusters) — https://github.com/wangyh082/scBGEDA
- **CellMark** (Curated marker database used to match nominated per-cluster DEGs (e.g. Gsn, Col3a1, Col1a2, Mmp2 for fibroblasts) to published cell-type marker genes and assign labels)
- **Python** (Execution environment for the scanpy-based marker identification workflow and CSV export)

## Examples

```
python -c "import scanpy as sc; adata = sc.read_h5ad('qs_heart_with_scbgeda_clusters.h5ad'); sc.tl.rank_genes_groups(adata, 'scBGEDA_cluster', method='wilcoxon'); adata.uns['rank_genes_groups'].to_csv('qs_heart_marker_genes.csv')"
```

## Evaluation signals

- Each cluster's output contains at most 20 ranked genes and all retained DEGs pass P<0.05 under the Wilcoxon rank-sum test.
- Fibroblast-annotated clusters have their top-20 Wilcoxon DEGs matching the CellMark entries Gsn, Col3a1, Col1a2 and Mmp2, consistent with the marker panel reported in Fig 7.
- Endothelial-annotated clusters have their top-20 Wilcoxon DEGs matching the CellMark entries Fabp4, Egfl7, Flt1 and Pecam1.
- The output file qs_heart_marker_genes.csv exists, is parseable, and every cell-type annotation is supported by at least the nominated marker genes present in the cluster's DEG table.
- The AnnData object used for testing contains the same gene-by-cell dimensions as the input QS_Heart matrix, with cluster labels merged without cell loss.

## Limitations

- Marker matching depends on the coverage and curation of CellMark — DEGs not represented in the database cannot yield a cell-type label even if biologically real.
- Validation relies on known marker panels (fibroblast: Gsn, Col3a1, Col1a2, Mmp2; endothelial: Fabp4, Egfl7, Flt1, Pecam1); clusters outside these panels may remain unannotated.
- The Wilcoxon rank-sum result is only as good as the upstream scBGEDA cluster labels; misclustered cells will distort the DEG rankings.

## Evidence

- [other] For each cluster, run the Wilcoxon rank-sum test via scanpy's rank_genes_groups (method='wilcoxon') to rank genes by differential expression, retaining the top 20 genes per cluster at P<0.05.: [evidence span withheld — anchor status: not_found]
- [abstract] Marker genes identified from clusters: "scBGEDA was able to identify cell-type specific marker genes and provide functional genomic analysis by quantifying the influence of genes on cell clusters"
- [other] Marker gene panel validation: [evidence span withheld — anchor status: not_found]
