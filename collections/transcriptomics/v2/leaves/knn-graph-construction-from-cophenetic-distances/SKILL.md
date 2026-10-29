---
name: knn-graph-construction-from-cophenetic-distances
description: Use when you have a single-cell lineage tree (e.g., Cassiopeia Newick
  with cells as leaves) plus expression data and need a cell-neighborhood metric for
  phylogenetic autocorrelation with Hotspot.
when_to_use_negative:
- No reliable lineage/phylogeny exists for the cells — a transcriptional latent space
  (e.g., PCA) or spatial coordinates should define the cell metric instead
- The tree has not been pruned to the analyzed clone and unifurcations collapsed (collapse.singles),
  so cophenetic distances would be distorted or leaf names would not match expression
  barcodes
- The goal is signature-level enrichment scoring only (VISION-style Geary's C on user
  signatures) rather than de novo Hotspot gene module discovery
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0203
tools:
- name: Hotspot
  role: Creates the KNN graph from the tree/cophenetic-distance metric (create_knn_graph,
    n_neighbors=30) that underpins autocorrelation and module inference
  repo: https://github.com/YosefLab/Hotspot
- name: PhyloVision
  role: Pipeline wrapper that invokes Hotspot via runHotspot with model='normal',
    tree=TRUE on the collapsed phylogeny
  repo: https://github.com/YosefLab/VISION
- name: R (ape)
  role: collapse.singles function prunes unifurcations from the clone phylogeny before
    distance computation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100200
    title: Interactive, integrated analysis of single-cell transcriptomic and phylogenetic
      data with PhyloVision
  source_content_hash: sha256:1855484b2cd2d2815ada7590245e3b765749b7da89511e16016b3c18925c07b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/Hotspot
  tools:
  - Hotspot
  - PhyloVision
  - R
  - R (ape)
  - VISION
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# knn-graph-construction-from-cophenetic-distances

## Summary

Builds the cell-similarity graph that Hotspot uses for heritable gene module detection by computing a K-nearest-neighbor graph (n_neighbors=30, unweighted) from cophenetic distances on a Cassiopeia-reconstructed phylogeny, after pruning cells and collapsing unifurcations with collapse.singles (R ape).

## When to use

Use when you have a single-cell lineage tree (e.g., Cassiopeia Newick with cells as leaves) plus expression data and need a cell-neighborhood metric for phylogenetic autocorrelation with Hotspot.

## When NOT to use

- No reliable lineage/phylogeny exists for the cells — a transcriptional latent space (e.g., PCA) or spatial coordinates should define the cell metric instead
- The tree has not been pruned to the analyzed clone and unifurcations collapsed (collapse.singles), so cophenetic distances would be distorted or leaf names would not match expression barcodes
- The goal is signature-level enrichment scoring only (VISION-style Geary's C on user signatures) rather than de novo Hotspot gene module discovery

## Inputs

- Cassiopeia-reconstructed single-cell phylogeny (e.g., Newick/ete3.TreeNode) with cell barcodes as leaves
- Library-size-normalized expression matrix (cells x genes) for the clone
- Cell metadata linking barcodes to the clone (e.g., GSE161363 / GSM4905334)

## Outputs

- Binary K-nearest-neighbor cell graph (n_neighbors=30) over cophenetic distances, consumed by Hotspot
- Per-gene phylogenetic autocorrelation statistics (C, Z, Pval, FDR) computed on the graph
- Pairwise gene local-correlation matrix used for module clustering

## How to apply

First prepare the lineage: prune the clone's phylogeny to the analyzed cells (1,127 cells for the CP007 metastatic lung adenocarcinoma clone) and collapse unifurcations using the collapse.singles function in the R package ape, so leaves match cell barcodes in the library-size-normalized expression matrix. Compute pairwise cophenetic distances on the tree and feed them as the cell metric — in Hotspot this is done either via a tree object (ete3.TreeNode, tree=TRUE in the PhyloVision runHotspot wrapper with model='normal') or via a cells-by-cells distance matrix. Build the KNN graph with hs.create_knn_graph(weighted_graph=False, n_neighbors=30); the unweighted choice keeps neighbor relationships binary rather than distance-decayed. The n_neighbors=30 choice trades robustness of autocorrelation/local-correlation detection against resolution of fine-grained heritable patterns. This graph, not a PCA latent space, is what downstream compute_autocorrelations and compute_local_correlations use to find phylogenetically autocorrelated genes (FDR < 0.05) and group them into modules (min_gene_threshold=70).

## Related tools

- **Hotspot** (Creates the KNN graph from the tree/cophenetic-distance metric (create_knn_graph, n_neighbors=30) that underpins autocorrelation and module inference) — https://github.com/YosefLab/Hotspot
- **PhyloVision** (Pipeline wrapper that invokes Hotspot via runHotspot with model='normal', tree=TRUE on the collapsed phylogeny) — https://github.com/YosefLab/VISION
- **R (ape)** (collapse.singles function prunes unifurcations from the clone phylogeny before distance computation)

## Examples

```
hs.create_knn_graph(weighted_graph=False, n_neighbors=30)
```

## Evaluation signals

- Every cell barcode in the expression matrix appears exactly once as a leaf in the graph's cell set (1,127 cells for the CP007 clone)
- The graph is binary (weighted_graph=False) with exactly n_neighbors=30 neighbors per cell
- Cophenetic distances derive from the collapsed tree — running collapse.singles (ape) leaves no single-child internal nodes and leaf names match expression column labels
- Downstream compute_autocorrelations on this graph yields genes passing FDR < 0.05, and module clustering enforces min_gene_threshold=70, producing three non-overlapping modules for CP007
- The final module table (hotspot_gene_modules.csv) contains a per-cell module score for every cell in the clone

## Limitations

- Larger n_neighbors values make correlation/autocorrelation detection more robust but miss finer-grained, smaller-scale heritable patterns and increase runtime
- PhyloVision's R implementation complicates interoperability with Python tools like Cassiopeia
- The lineage analysis of the metastasis dataset lacks experimental validation of the generated hypotheses

## Evidence

- [methods] Build KNN graph from cophenetic distances on collapsed phylogeny: "collapsed using the collapse.singles function in the R package ape"
- [methods] PhyloVision invokes Hotspot with tree metric: [evidence span withheld — anchor status: not_found]
- [readme] README documents KNN graph call: [evidence span withheld — anchor status: external_doc]
- [readme] README on n_neighbors trade-off: [evidence span withheld — anchor status: external_doc]
- [readme] README on tree as lineage metric: [evidence span withheld — anchor status: not_found]
