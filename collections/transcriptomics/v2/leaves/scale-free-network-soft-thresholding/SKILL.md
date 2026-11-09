---
name: scale-free-network-soft-thresholding
description: Use when you need unsupervised gene program/module discovery from scRNA-seq
  data and want to benchmark against or replace factorization methods like ICA or
  NMF, starting from an annotated Seurat object.
when_to_use_negative:
- The input is already a gene-module/factor list from another method and only functional
  annotation is needed — use getAnnotationPathways scoring directly rather than re-running
  SSN.
- Cluster stability cannot be assessed (e.g. single-replicate datasets), where the
  paper notes the related resolution-optimization methods are limited by cluster stability
  and reproducibility.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scMiko
  role: R package providing findNetworkFeatures, runSSN (scale-free SSN with data_type='pearson'
    and resolution optimization), pruneSSN, getICAGenes, getNMFGenes, consolidateNMF,
    and getAnnotationPathways
  repo: https://github.com/NMikolajewicz/scMiko
- name: Seurat
  role: Framework the scMiko package and SSN workflow are built on; supplies the Seurat
    object input
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-022-04093-2
    title: Multi-level cellular and functional annotation of single-cell transcriptomes
      using scPipeline
  source_content_hash: sha256:aa6a48a9c281d50b7ced303f6b818a0c4feb7feec5c1b8d3ab69f82742489489
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMikolajewicz/scMiko
  tools:
  - Seurat
  - scMiko
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# scale-free-network-soft-thresholding

## Summary

Unsupervised gene program discovery via a scale-free transformed shared nearest neighbor (SSN) network: Pearson co-expression graph built on selected network features, driven toward a scale-free topology, clustered into gene modules, and pruned of low-connectivity genes. Implemented in scMiko (runSSN/pruneSSN) and shown to recover more GO terms and STRING PPI enrichment than ICA and NMF.

## When to use

Use when you need unsupervised gene program/module discovery from scRNA-seq data and want to benchmark against or replace factorization methods like ICA or NMF, starting from an annotated Seurat object.

## When NOT to use

- The input is already a gene-module/factor list from another method and only functional annotation is needed — use getAnnotationPathways scoring directly rather than re-running SSN.
- Cluster stability cannot be assessed (e.g. single-replicate datasets), where the paper notes the related resolution-optimization methods are limited by cluster stability and reproducibility.

## Inputs

- Seurat object of scRNA-seq expression data (e.g. so.query, as used for the Tyser 2021 human gastrulation dataset)
- Selected network feature sets (expression-based with min_pct=0.5, HVGs, or deviance-based, n_features=2000)
- GO annotation pathway database for functional recovery scoring
- STRING PPI interaction database for protein-interaction enrichment tests

## Outputs

- Hub-pruned SSN gene programs (gene modules) from runSSN/pruneSSN
- Comparator gene programs from SN (scale_free=FALSE), ICA, and consolidated NMF
- Per-method GO-term recovery (max column-wise Jaccard similarity) and STRING PPI enrichment statistics, aggregated into ssn_benchmark_results.csv

## How to apply

Select network input features with findNetworkFeatures using the expression-based criterion (method='expr', min_pct=0.5) or HVG/deviance sets (n_features=2000), then run the SSN workflow with runSSN(scale_free=TRUE, data_type='pearson', pca_var_explained=0.9, optimize_resolution=TRUE, target_purity=0.8, step_size=0.05) so the shared nearest neighbor graph is soft-thresholded toward a scale-free topology while cluster resolution is optimized by the specificity-based criterion. Prune low-connectivity genes with pruneSSN (prune.threshold=0.1) to retain hub-like gene modules. Build the scale-free-free comparator (SN, scale_free=FALSE) and factorization comparators (getICAGenes, getNMFGenes with consolidateNMF) on the same features. For each program, retrieve annotations with getAnnotationPathways, score GO-term recovery as the maximum column-wise Jaccard similarity between program gene sets and GO terms, and test STRING PPI enrichment; aggregate into ssn_benchmark_results.csv.

## Related tools

- **scMiko** (R package providing findNetworkFeatures, runSSN (scale-free SSN with data_type='pearson' and resolution optimization), pruneSSN, getICAGenes, getNMFGenes, consolidateNMF, and getAnnotationPathways) — https://github.com/NMikolajewicz/scMiko
- **Seurat** (Framework the scMiko package and SSN workflow are built on; supplies the Seurat object input)

## Examples

```
ssn.res <- runSSN(object = so.query, features = features_expr, scale_free = TRUE, data_type = 'pearson', pca_var_explained = 0.9, optimize_resolution = TRUE, target_purity = 0.8, step_size = 0.05); ssn.res <- pruneSSN(object = ssn.res, prune.threshold = 0.1)
```

## Evaluation signals

- SSN programs achieve superior GO-term recovery (max column-wise Jaccard similarity vs. GO terms) relative to ICA and NMF programs, as reported for Fig. 5c-e
- SSN programs achieve superior STRING PPI enrichment relative to ICA and NMF
- Scale-free SSN outperforms the untransformed shared-nearest-neighbor (SN, scale_free=FALSE) control, confirming the scale-free transform is the operative step
- pruneSSN with prune.threshold=0.1 yields hub-like modules — programs should retain high-connectivity genes after pruning
- Benchmark comparison table ssn_benchmark_results.csv is populated with per-method recovery and PPI-enrichment statistics

## Limitations

- Performance depends on feature selection; SSN is run on expression-based network features (min_pct=0.5) or 2000-feature HVG/deviance sets, so poorly chosen feature sets will degrade program recovery
- The paper notes a limitation of the related specificity-based resolution criterion relates to stability and reproducibility of clusters, especially in single-replicate datasets

## Evidence

- [discussion] SSN analysis as an approach to identify and functionally annotate gene sets in an unsupervised manner: "we describe SSN analysis as an approach to identify and functionally annotate gene sets in an unsupervised manner"
- [discussion] SSN superiority over ICA and NMF: "gene programs identified by SSN yielded superior GO term recovery and enrichment of STRING PPIs compared to ICA and NMF methods"
- [methods] pruneSSN pruning step: "This is implemented using the pruneSSN function."
- [methods] findNetworkFeatures feature selection: "features_expr <- findNetworkFeatures(object = so.query, method = "expr", min_pct = 0.5)"
- [readme] scMiko package description: "scMiko is an **R package** developed for single-cell RNA-seq analysis. It was designed using the [Seurat](https://satijalab.org/seurat/) framework"
