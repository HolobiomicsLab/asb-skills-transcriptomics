---
name: hotspot-gene-module-identification
description: Use when you have CRISPR-lineage-traced single-cell expression data plus
  a Cassiopeia-reconstructed phylogeny and want de novo, signature-free detection
  of gene programs whose variation is heritable across the lineage.
when_to_use_negative:
- When no lineage/phylogeny is available — Hotspot would need a spatial or transcriptional
  (PCA) metric and would identify spatially varying or transcriptional modules, not
  heritable ones
- When the goal is guided signature scoring rather than de novo module discovery —
  use createGeneSignature with user-provided sigData instead
- When clone size is far below the min_gene_threshold=70 genes per module or the tree
  is unresolved (unifurcations not collapsed), making module grouping unstable
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Hotspot
  role: Computes phylogenetic autocorrelations, local gene-gene correlations, and
    groups genes into de novo modules using the phylogeny as the cell-similarity metric
  repo: https://github.com/YosefLab/Hotspot
- name: PhyloVision
  role: Pipeline wrapper that orchestrates Hotspot module identification on the phylogeny
    and interprets modules via hypergeometric signature enrichment
  repo: https://github.com/YosefLab/VISION
- name: VISION
  role: Provides the signature-scoring procedure used to compute per-cell module scores
    and the front-end for the interactive web report
  repo: https://github.com/YosefLab/VISION
- name: Cassiopeia
  role: Source of the reconstructed single-cell phylogenies used as Hotspot's latent
    lineage space
- name: R ape
  role: collapse.singles function used to prune unifurcations from the phylogeny before
    analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100200
    title: Interactive, integrated analysis of single-cell transcriptomic and phylogenetic
      data with PhyloVision
  source_content_hash: sha256:7f965d523a74cb9ef161316b27b1d94474d81a7777124fbb989050a094b4d917
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/Hotspot
  tools:
  - Cassiopeia
  - Hotspot
  - PhyloVision
  - R
  - R ape
  - VISION
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# hotspot-gene-module-identification

## Summary

De novo identification of heritable gene modules by applying Hotspot's phylogenetic autocorrelation analysis within the PhyloVision pipeline, using a reconstructed single-cell phylogeny (rather than PCA space) as the cell-similarity metric so that genes whose expression is concordant with lineage structure are grouped into modules.

## When to use

Use when you have CRISPR-lineage-traced single-cell expression data plus a Cassiopeia-reconstructed phylogeny and want de novo, signature-free detection of gene programs whose variation is heritable across the lineage.

## When NOT to use

- When no lineage/phylogeny is available — Hotspot would need a spatial or transcriptional (PCA) metric and would identify spatially varying or transcriptional modules, not heritable ones
- When the goal is guided signature scoring rather than de novo module discovery — use createGeneSignature with user-provided sigData instead
- When clone size is far below the min_gene_threshold=70 genes per module or the tree is unresolved (unifurcations not collapsed), making module grouping unstable

## Inputs

- Library-size-normalized gene expression matrix (cells x genes) for a single clone (e.g., 1,127 cells from the CP007 metastatic lung adenocarcinoma clone)
- Cassiopeia-reconstructed single-cell phylogeny (Newick tree with cell barcodes as leaf names, collapsed with ape::collapse.singles)
- Phylogenetic cophenetic distances between cells
- Optional: MSigDB gene signatures for post-hoc module enrichment

## Outputs

- Table of de novo gene modules (module identities and member genes) for phylogenetically autocorrelated genes passing FDR 0.05
- Per-cell module scores for every cell in the clone (hotspot_gene_modules.csv)
- Autocorrelation statistics (Geary's C, Z, P-value, FDR) per gene

## How to apply

Load the library-size-normalized expression matrix for the clone and the phylogeny, pruning cells and collapsing unifurcations with the collapse.singles function in the R ape package. Build a K-nearest-neighbor graph from cophenetic distances on the tree with n_neighbors=30, then invoke Hotspot via runHotspot with model='normal' and tree=TRUE. Use compute_autocorrelations to select up to number_top_genes=1000 genes significantly autocorrelated with the phylogenetic KNN graph, retaining genes passing FDR 0.05. Group the retained genes by clustering the pairwise autocorrelation matrix from compute_local_correlations, enforcing min_gene_threshold=70. Compute per-cell module scores using the signature-scoring procedure in VISION, and optionally interpret modules against MSigDB signatures via hypergeometric enrichment. Applied to the 1,127-cell metastatic lung adenocarcinoma clone (CP007), this yielded three non-overlapping gene modules.

## Related tools

- **Hotspot** (Computes phylogenetic autocorrelations, local gene-gene correlations, and groups genes into de novo modules using the phylogeny as the cell-similarity metric) — https://github.com/YosefLab/Hotspot
- **PhyloVision** (Pipeline wrapper that orchestrates Hotspot module identification on the phylogeny and interprets modules via hypergeometric signature enrichment) — https://github.com/YosefLab/VISION
- **VISION** (Provides the signature-scoring procedure used to compute per-cell module scores and the front-end for the interactive web report) — https://github.com/YosefLab/VISION
- **Cassiopeia** (Source of the reconstructed single-cell phylogenies used as Hotspot's latent lineage space)
- **R ape** (collapse.singles function used to prune unifurcations from the phylogeny before analysis)

## Examples

```
hs.create_knn_graph(weighted_graph=False, n_neighbors=30); hs_results = hs.compute_autocorrelations()
```

## Evaluation signals

- Output table hotspot_gene_modules.csv exists and lists modules each containing at least min_gene_threshold=70 member genes
- All module member genes are drawn from genes passing the FDR 0.05 phylogenetic autocorrelation threshold
- The table includes a per-cell module score for every cell in the 1,127-cell clone
- Modules are non-overlapping gene sets; on the CP007 clone the expected result is three modules
- Phylogeny used is the pruned tree after collapse.singles, with leaf names matching the expression matrix column barcodes

## Limitations

- R implementation of PhyloVision may create challenges for using separate Python software such as Cassiopeia
- Larger neighborhood sizes (n_neighbors=30) trade robustness for missing fine-grained, smaller-scale correlation patterns and increase run-time
- Analysis of the metastasis dataset lacks experimental validation to support the generated hypotheses
- Choice of cell metric determines module type — with a lineage metric these are heritable modules, not spatial or transcriptional ones

## Evidence

- [results] gene modules (i.e., not guided by pre-defined signatures) that are learned from the intersection of phylogeny and expression data using our Hotspot algorithm (DeTomaso and Yosef, 2021): "gene modules (i.e., not guided by pre-defined signatures) that are learned from the intersection of phylogeny and expression data using our Hotspot algorithm"
- [methods] Hotspot was invoked using the runHotspot command with the following parameters: model = “normal”, tree=TRUE: "Hotspot was invoked using the runHotspot command with the following parameters: model = “normal”, tree=TRUE"
- [methods] collapsed using the collapse.singles function in the R package ape: "collapsed using the collapse.singles function in the R package ape"
- [results] we computed module scores for every cell using the signature-scoring procedure in VISION: "we computed module scores for every cell using the signature-scoring procedure in VISION"
- [readme] n_neighbors: the number of neighbors per cell to use. Larger neighborhood sizes can result in more robust detection of correlations and autocorrelations at a cost of missing more fine-grained, smaller-scale patterns: [evidence span withheld — anchor status: external_doc]
