---
name: protein-interaction-enrichment-testing
description: Use when you have gene sets (gene programs/modules) from scRNA-seq analysis
  and want to validate them functionally by testing enrichment of STRING PPI interactions,
  or when benchmarking methods like SSN vs ICA/NMF.
when_to_use_negative:
- Gene sets are already validated curated pathway signatures rather than de novo discovered
  programs — enrichment testing against STRING adds no benchmarking value
- The goal is cell-type annotation rather than unsupervised gene program discovery/functional
  validation — use the Miko scoring pipeline instead
- Single-cell data has not been feature-selected or clustered into gene modules —
  there are no defined gene sets to test for PPI enrichment
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scMiko
  role: R package supplying runSSN, pruneSSN, findNetworkFeatures, getICAGenes, getNMFGenes,
    consolidateNMF, and getAnnotationPathways used to generate and annotate the gene
    programs tested for PPI enrichment
  repo: https://github.com/NMikolajewicz/scMiko
- name: STRING
  role: Reference protein-protein interaction database against which gene programs
    are tested for interaction enrichment
- name: Seurat
  role: Framework on which scMiko is built; provides the single-cell object containers
    for expression data and program annotations
  repo: https://satijalab.org/seurat/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-022-04093-2
    title: Multi-level cellular and functional annotation of single-cell transcriptomes
      using scPipeline
  source_content_hash: sha256:fabf6757b6b97acf6eef80ad977a9a79bdc83e5c7588c847a77433382d3fd11f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMikolajewicz/scMiko
  tools:
  - STRING
  - Seurat
  - scMiko
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# protein-interaction-enrichment-testing

## Summary

Tests whether discovered single-cell gene programs (e.g., from SSN, ICA, or NMF) are enriched for known protein-protein interactions from STRING, providing a functional-validation metric that complements GO-term recovery when benchmarking gene program discovery methods.

## When to use

Use when you have gene sets (gene programs/modules) from scRNA-seq analysis and want to validate them functionally by testing enrichment of STRING PPI interactions, or when benchmarking methods like SSN vs ICA/NMF.

## When NOT to use

- Gene sets are already validated curated pathway signatures rather than de novo discovered programs — enrichment testing against STRING adds no benchmarking value
- The goal is cell-type annotation rather than unsupervised gene program discovery/functional validation — use the Miko scoring pipeline instead
- Single-cell data has not been feature-selected or clustered into gene modules — there are no defined gene sets to test for PPI enrichment

## Inputs

- Gene sets / gene programs (gene symbol lists) from SSN, ICA (getICAGenes), or NMF (getNMFGenes/consolidateNMF) workflows
- Seurat object with normalized scRNA-seq expression data
- GO annotation pathways (via getAnnotationPathways)
- STRING protein-protein interaction database

## Outputs

- Per-program STRING PPI enrichment statistics
- GO-term recovery scores (maximum column-wise Jaccard similarity between program gene sets and GO terms)
- Aggregated per-method comparison table (ssn_benchmark_results.csv)

## How to apply

First derive gene programs from the scRNA-seq object: select network features with findNetworkFeatures (method='expr', min_pct=0.5 or method='hvg', n_features=2000), run the SSN workflow (runSSN with scale_free=TRUE, data_type='pearson', pca_var_explained=0.9, optimize_resolution=TRUE, target_purity=0.8, step_size=0.05), and prune low-connectivity genes with pruneSSN at threshold=0.1. For each program from each method (SSN, SN with scale_free=FALSE, ICA via getICAGenes, NMF via getNMFGenes with consolidateNMF), retrieve pathway annotations with getAnnotationPathways and compute GO-term recovery as the maximum column-wise Jaccard similarity between program gene sets and GO terms. In parallel, test enrichment of STRING PPI interactions within each program's gene set. Aggregate PPI-enrichment and GO-recovery statistics per method into a comparison table (e.g., ssn_benchmark_results.csv) and evaluate whether programs from the candidate method (SSN) show superior GO-term recovery and STRING PPI enrichment relative to comparators, as reported for Fig. 5c-e.

## Related tools

- **scMiko** (R package supplying runSSN, pruneSSN, findNetworkFeatures, getICAGenes, getNMFGenes, consolidateNMF, and getAnnotationPathways used to generate and annotate the gene programs tested for PPI enrichment) — https://github.com/NMikolajewicz/scMiko
- **STRING** (Reference protein-protein interaction database against which gene programs are tested for interaction enrichment)
- **Seurat** (Framework on which scMiko is built; provides the single-cell object containers for expression data and program annotations) — https://satijalab.org/seurat/

## Examples

```
features_expr <- findNetworkFeatures(object = so.query, method = "expr", min_pct = 0.5)
```

## Evaluation signals

- SSN gene programs show significantly superior GO term recovery (maximum column-wise Jaccard similarity) compared to ICA and NMF programs, as in Fig. 5c
- SSN gene programs show significantly superior STRING PPI enrichment compared to ICA and NMF programs, as in Fig. 5d
- Per-method statistics are aggregated into a comparison table (ssn_benchmark_results.csv) covering every program from each method
- Programs have been pruned with pruneSSN at threshold=0.1 before enrichment testing, so low-connectivity genes are excluded
- Comparator SSN run without the scale-free transform (SN, scale_free=FALSE) is included as a negative control for the scale-free step

## Limitations

- Marker-based and functional annotations are only as good as the cell-type/gene set prescriptions of the source catalogs; no validation of source catalog curations is provided
- Cluster/gene program stability is limited in single-replicate datasets, making it difficult to distinguish technical artifacts from real biology
- Occasional co-similarities between marker/annotation sets from heterogeneous cell types may reflect marker curation inaccuracies or cross-cell-type process similarities and are left unresolved

## Evidence

- [discussion] gene programs identified by SSN yielded superior GO term recovery and enrichment of STRING PPIs compared to ICA and NMF methods: "gene programs identified by SSN yielded superior GO term recovery and enrichment of STRING PPIs compared to ICA and NMF methods"
- [discussion] SSN analysis as unsupervised gene set identification: "we describe SSN analysis as an approach to identify and functionally annotate gene sets in an unsupervised manner"
- [methods] pruneSSN implements low-connectivity pruning: "This is implemented using the pruneSSN function."
- [methods] expression-based network feature selection parameters: "features_expr <- findNetworkFeatures(object = so.query, method = "expr", min_pct = 0.5)"
