---
name: gene-program-discovery-network-analysis
description: Use when the goal is unsupervised discovery of gene programs/modules
  from normalized scRNA-seq data, especially when benchmarking against ICA/NMF or
  seeking GO-term/PPI-enriched gene modules.
when_to_use_negative:
- The analysis goal is cell-type annotation or cluster resolution selection rather
  than gene module discovery — use the Miko scoring or specificity-based resolution
  workflows instead
- Data is raw counts without normalization; the SSN pearson-correlation input expects
  preprocessed expression
- A supervised or pre-defined gene set scoring task is intended — use mikoScore/sigScore
  rather than unsupervised SSN discovery
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scMiko
  role: R package providing runSSN, pruneSSN, findNetworkFeatures, getICAGenes, getNMFGenes,
    consolidateNMF, and getAnnotationPathways for the gene program discovery workflow
  repo: https://github.com/NMikolajewicz/scMiko
- name: Seurat
  role: Framework on which scMiko is built; supplies the Seurat object input for SSN
    analysis
  repo: https://satijalab.org/seurat/
- name: scPipeline
  role: Modular Rmarkdown scripts that generate dashboard reports implementing the
    SSN gene program discovery module
  repo: https://github.com/NMikolajewicz/scPipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-022-04093-2
    title: Multi-level cellular and functional annotation of single-cell transcriptomes
      using scPipeline
  source_content_hash: sha256:c2602cbb19a16df4a999eb57833cd1f1e6628a421916b8d9a3f0c4516e95cbbf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMikolajewicz/scMiko
  tools:
  - Seurat
  - scMiko
  - scPipeline
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# gene-program-discovery-network-analysis

## Summary

Discovers unsupervised gene programs from scRNA-seq data by building a scale-free shared nearest neighbor (SSN) network over selected network features, pruning low-connectivity genes, and functionally annotating modules — a graph-based alternative to factorization methods like ICA and NMF.

## When to use

Use when the goal is unsupervised discovery of gene programs/modules from normalized scRNA-seq data, especially when benchmarking against ICA/NMF or seeking GO-term/PPI-enriched gene modules.

## When NOT to use

- The analysis goal is cell-type annotation or cluster resolution selection rather than gene module discovery — use the Miko scoring or specificity-based resolution workflows instead
- Data is raw counts without normalization; the SSN pearson-correlation input expects preprocessed expression
- A supervised or pre-defined gene set scoring task is intended — use mikoScore/sigScore rather than unsupervised SSN discovery

## Inputs

- Processed Seurat object (e.g. so_tyser2021_220621.rds) with normalized scRNA-seq expression
- Feature sets: expression-based (min_pct=0.5), HVG (n_features=2000), and deviance-based features
- GO annotation pathway database (via getAnnotationPathways)
- STRING protein-protein interaction database

## Outputs

- Pruned SSN gene program modules (gene sets per program)
- GO-term recovery statistics (maximum column-wise Jaccard similarity per program)
- STRING PPI enrichment statistics per program
- Aggregated per-method comparison table (ssn_benchmark_results.csv)

## How to apply

First select network features with findNetworkFeatures using the expression-based criterion (method='expr', min_pct=0.5), alongside HVG and deviance-based feature sets (n_features=2000). Run the SSN workflow with runSSN (scale_free=TRUE, data_type='pearson', pca_var_explained=0.9, optimize_resolution=TRUE, target_purity=0.8, step_size=0.05), then apply pruneSSN at prune.threshold=0.1 to remove low-connectivity genes and obtain hub-like gene modules. For benchmarking, run comparators on the same features: an SN network without the scale-free transform (scale_free=FALSE), and ICA/NMF factorizations via getICAGenes/getNMFGenes with consolidateNMF for NMF factor consolidation. For every program, retrieve GO annotations with getAnnotationPathways and score recovery as the maximum column-wise Jaccard similarity between program gene sets and GO terms, plus STRING PPI enrichment. The rationale is that the scale-free transform produces superior GO-term recovery and STRING PPI enrichment relative to ICA and NMF (Fig. 5c-e).

## Related tools

- **scMiko** (R package providing runSSN, pruneSSN, findNetworkFeatures, getICAGenes, getNMFGenes, consolidateNMF, and getAnnotationPathways for the gene program discovery workflow) — https://github.com/NMikolajewicz/scMiko
- **Seurat** (Framework on which scMiko is built; supplies the Seurat object input for SSN analysis) — https://satijalab.org/seurat/
- **scPipeline** (Modular Rmarkdown scripts that generate dashboard reports implementing the SSN gene program discovery module) — https://github.com/NMikolajewicz/scPipeline

## Examples

```
features_expr <- findNetworkFeatures(object = so.query, method = "expr", min_pct = 0.5); ssn.res <- runSSN(object = so.query, features = features_expr, scale_free = TRUE, data_type = "pearson", pca_var_explained = 0.9, optimize_resolution = TRUE, target_purity = 0.8, step_size = 0.05); ssn.res <- pruneSSN(object = ssn.res, prune.threshold = 0.1)
```

## Evaluation signals

- SSN programs achieve significantly higher GO-term recovery (max column-wise Jaccard similarity) than ICA and NMF programs, as reported in Fig. 5c
- SSN programs show superior STRING PPI enrichment relative to ICA and NMF (Fig. 5d-e)
- pruneSSN at threshold=0.1 yields hub-like gene modules — check that low-connectivity genes are removed before annotation
- Aggregated per-method statistics populate ssn_benchmark_results.csv with recovery and PPI-enrichment columns for SSN, SN, ICA, and NMF
- Scaling the scale-free transform off (SN, scale_free=FALSE) should degrade performance relative to SSN, confirming the transform's contribution

## Limitations

- SSN performance was validated against ICA and NMF comparators on reference scRNA-seq datasets; other gene program methods (e.g. WGCNA, topic models) were not benchmarked in this comparison
- Feature selection via findNetworkFeatures (min_pct=0.5, n_features=2000) restricts discovery to pre-filtered genes; programs containing rare genes may be missed
- The scMiko package was developed and tested in Windows, with MacOS X and Linux support expected but not validated

## Evidence

- [results] SSN gene programs showed significantly superior GO term recovery and STRING PPI enrichment compared to ICA and NMF: "gene programs identified by SSN yielded superior GO term recovery and enrichment of STRING PPIs compared to ICA and NMF methods"
- [methods] pruneSSN implements low-connectivity gene pruning: "This is implemented using the pruneSSN function."
- [methods] findNetworkFeatures expression-based feature selection: "features_expr <- findNetworkFeatures(object = so.query, method = "expr", min_pct = 0.5)"
- [intro] SSN introduced as gene program discovery method: "introduce scale-free shared nearest neighbor network (SSN) analysis as a gene program discovery method"
- [discussion] SSN analysis described as unsupervised gene set identification: "we describe SSN analysis as an approach to identify and functionally annotate gene sets in an unsupervised manner"
- [readme] scMiko provides SSN functions: "scMiko is an **R package** developed for single-cell RNA-seq analysis. It was designed using the [Seurat](https://satijalab.org/seurat/) framework"
