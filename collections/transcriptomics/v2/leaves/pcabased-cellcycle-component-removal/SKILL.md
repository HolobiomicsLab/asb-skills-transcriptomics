---
name: pcabased-cellcycle-component-removal
description: Use when scRNA-Seq expression matrices show within-cell-type heterogeneity
  driven by cell-cycle activity that obscures clustering of cell types, and you need
  a correction before downstream clustering.
when_to_use_negative:
- Data has very low cell-cycle activity — for glioblastoma data, where cell-cycle
  activity is known to be at a very low level, the paper uses a smaller cutoff CΔ
  = 2, so the default cutoff=3 is inappropriate there
- The dominant unwanted effect is not the cell cycle (e.g. batch/technical replicate
  effects) — ccRemover is designed for removing the cell-cycle effect exclusively
  and does not remove other confounding effects
- The cell-cycle effect influences too few control genes (below ~8% of control genes),
  where it may not be captured by the principal components and will be missed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ccRemover
  role: R package that iteratively identifies cell-cycle principal components via
    bootstrap t-statistics and removes them from the expression matrix
  repo: https://github.com/cran/ccRemover
- name: gene_indexer
  role: Maps gene names to cell-cycle gene indices for human or mouse to build the
    if_cc indicator vector
- name: Biomart
  role: Source of gene lists annotated to the cell cycle used as the control annotation
    set
- name: org.Hs.eg.db
  role: Human gene annotation database supplying cell-cycle gene annotations
- name: scLVM
  role: Comparison latent-factor correction method (Python) used alongside ccRemover
    on the same datasets
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/srep33892
    title: Identifying and removing the cell-cycle effect from single-cell RNA-Sequencing
      data
  source_content_hash: sha256:eca96fbb5882400009c3090014777d97ba98da4b0e0a8f415685ad5d508c2a26
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cran/ccRemover
  tools:
  - Biomart
  - ccRemover
  - gene_indexer
  - org.Hs.eg.db
  - scLVM
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# pcabased-cellcycle-component-removal

## Summary

PCA-based identification and removal of cell-cycle principal components from single-cell RNA-Seq data using ccRemover, which bootstrap-tests whether each leading PC is enriched in cell-cycle annotated genes before removing it, preserving other biological signals.

## When to use

Use when scRNA-Seq expression matrices show within-cell-type heterogeneity driven by cell-cycle activity that obscures clustering of cell types, and you need a correction before downstream clustering.

## When NOT to use

- Data has very low cell-cycle activity — for glioblastoma data, where cell-cycle activity is known to be at a very low level, the paper uses a smaller cutoff CΔ = 2, so the default cutoff=3 is inappropriate there
- The dominant unwanted effect is not the cell cycle (e.g. batch/technical replicate effects) — ccRemover is designed for removing the cell-cycle effect exclusively and does not remove other confounding effects
- The cell-cycle effect influences too few control genes (below ~8% of control genes), where it may not be captured by the principal components and will be missed

## Inputs

- scRNA-Seq expression matrix (genes x cells) with raw measurements, e.g. 176 cells x 57,820 genes from GSE69405
- Vector of gene names or identifiers for the matrix rows
- Cell-cycle gene annotations (human or mouse) from gene_indexer, Biomart, org.Hs.eg.db, or Cyclebase
- Boolean if_cc indicator vector marking cell-cycle annotated genes

## Outputs

- Cell-cycle-corrected expression matrix with cell-cycle principal components removed (xhat)
- Cluster/cell-type cross-tabulation of downstream clustering on corrected vs original data (e.g. cluster_composition.csv)

## How to apply

Log-transform the count matrix (add 1, natural log) and center each gene to zero mean, having first removed genes with zero expression in over two thirds of cells (e.g. 57,820 down to 10,977 genes in the Kim et al. lung adenocarcinoma data). Retrieve the cell-cycle gene set (e.g. 757 human genes via gene_indexer with species='human', or Biomart/org.Hs.eg.db annotations) and build the if_cc boolean indicator vector; use all non-cell-cycle genes as control genes. Run ccRemover(dat, cutoff=3, max_it=4, nboot=200, ntop=10): it performs PCA on the control-gene matrix, bootstrap-tests each PC against the cutoff CΔ=3 for enrichment of cell-cycle genes, and iteratively removes significant PCs up to max_it=4 iterations. Then apply the downstream analysis (e.g. 4-means clustering on the corrected matrix) to both original and corrected data and compare, since ccRemover removes only the cell-cycle effect while retaining other biological components such as true cell-type structure.

## Related tools

- **ccRemover** (R package that iteratively identifies cell-cycle principal components via bootstrap t-statistics and removes them from the expression matrix) — https://github.com/cran/ccRemover
- **gene_indexer** (Maps gene names to cell-cycle gene indices for human or mouse to build the if_cc indicator vector)
- **Biomart** (Source of gene lists annotated to the cell cycle used as the control annotation set)
- **org.Hs.eg.db** (Human gene annotation database supplying cell-cycle gene annotations)
- **scLVM** (Comparison latent-factor correction method (Python) used alongside ccRemover on the same datasets)

## Examples

```
cc_cycle_genes <- gene_indexer(gene_names, species = "human", name_type = "symbols"); xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar = FALSE)
```

## Evaluation signals

- After correction, cells should cluster by cell type with cell-cycle stages mixed (e.g. ccRemover correctly identified the 2nd and 3rd PCs as cell-cycle effects on simulated data)
- Differential-expression odds ratios should become physiologically meaningful: on T helper data the TH2 signature odds ratio was < 1 on original data and improved after correction
- Known biological substructure must survive correction — technical replicates should NOT separate (in Kim et al. LUAD data, original and scLVM-corrected data keep LC.PT/LC.PT_RE non-separable, whereas a correction that makes cluster 3 ~80% LC.PT_RE and cluster 4 ~89% LC.PT signals over-removal or batch leakage)
- Gene filter invariant: after dropping genes with zero expression in >2/3 of cells, the gene count should match expectation (57,820 → 10,977 with 757 cell-cycle genes)
- No further cell-cycle effects should be flagged after removal (ccRemover found no further significant PCs beyond iteration 1 on the T helper data)

## Limitations

- Annotation databases are always incomplete and inaccurate, and ccRemover relies on a known set of cell-cycle genes
- ccRemover may miss the cell-cycle effect if it influences too few control genes, as it may not be captured by the principal components
- ccRemover removes only the cell-cycle effect, not other confounding effects such as batch/technical-replicate effects that may dominate after correction
- Cutoff CΔ must be tuned to the level of cell-cycle activity (default 3; lowered to 2 for glioblastoma data with low cell-cycle activity)
- No version numbers or explicit repository URL for ccRemover/scLVM are reported in the article

## Evidence

- [abstract] We present ccRemover, a new method that reliably identifies the cell-cycle effect and removes it.: "We present ccRemover, a new method that reliably identifies the cell-cycle effect and removes it."
- [results] gene filtering: "any genes which had zero expression for over two thirds of the cells were removed from the data, leaving 10,977 genes of which 757 were annotated to the cell cycle"
- [methods] ccRemover call: "xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE)"
- [discussion] control genes: "we recommend simply using all genes that are not annotated as cell-cycle genes as control genes"
- [results] pc identification: "ccRemover is able to correctly identify the second and third principal components as cell-cycle effects and removes them"
- [methods] low activity cutoff: "except for the glioblastoma data, where it is known that the cell-cycle activity is at a very low level47. We use a smaller cutoff value CΔ = 2"
- [discussion] scope limit: "ccRemover does not remove these effects as it is designed for removing the cell-cycle effect exclusively."
- [abstract] preprocessing role: "ccRemover preserves other biological signals of interest in the data and thus can serve as an important pre-processing step for many scRNA-Seq data analyses."
