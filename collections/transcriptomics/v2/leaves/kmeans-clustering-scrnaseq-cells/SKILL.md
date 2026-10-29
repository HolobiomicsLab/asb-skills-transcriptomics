---
name: kmeans-clustering-scrnaseq-cells
description: Use when scRNA-Seq cells (e.g. 176 LUAD cells including 77 LC.PT/LC.PT_RE
  technical-replicate cells) must be partitioned into k clusters to test separability
  of known labels across original, scLVM-corrected, and ccRemover-corrected data.
when_to_use_negative:
- When the data are known to have very low cell-cycle activity (e.g. the Patel et
  al. glioblastoma data, where a smaller cutoff C_delta = 2 was used) — clustering
  alone will not distinguish which correction best reveals structure
- When the biological question is hierarchical/tumor-subtype assignment rather than
  flat partitioning — the article used hierarchical clustering split into five clusters
  for the five glioblastomas instead of k-means
- When the goal is cell-cycle effect identification itself — k-means is a downstream
  evaluation step; ccRemover/scLVM are the correction methods under test
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ccRemover
  role: Upstream cell-cycle correction whose effect on k-means cluster separability
    is being evaluated; run as ccRemover(dat, cutoff=3, max_it=4, nboot=200, ntop=10)
  repo: cran/ccRemover
- name: scLVM
  role: Comparison latent-factor correction (applied via the Python script) whose
    corrected matrix is clustered with the same k-means procedure as a baseline
- name: gene_indexer
  role: Annotates the 757 cell-cycle genes (757 of 10,977 filtered genes) via species='human'
    to build the if_cc indicator needed before correction and clustering
- name: org.Hs.eg.db
  role: Human gene-annotation backend used by gene_indexer to supply the cell-cycle
    gene set to ccRemover prior to clustering
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/srep33892
    title: Identifying and removing the cell-cycle effect from single-cell RNA-Sequencing
      data
  source_content_hash: sha256:88c56b436db32373a3e77b0f9556805ab923e00b34c1eeb2e3a6db074a417607
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# kmeans-clustering-scrnaseq-cells

## Summary

Apply k-means clustering (e.g. 4-means on the 176-cell Kim et al. lung adenocarcinoma scRNA-Seq data, or 2-means for differential-expression testing) to corrected and uncorrected single-cell expression matrices to test whether known cell-type or technical-replicate labels separate, and cross-tabulate cluster assignments against those labels.

## When to use

Use when scRNA-Seq cells (e.g. 176 LUAD cells including 77 LC.PT/LC.PT_RE technical-replicate cells) must be partitioned into k clusters to test separability of known labels across original, scLVM-corrected, and ccRemover-corrected data.

## When NOT to use

- When the data are known to have very low cell-cycle activity (e.g. the Patel et al. glioblastoma data, where a smaller cutoff C_delta = 2 was used) — clustering alone will not distinguish which correction best reveals structure
- When the biological question is hierarchical/tumor-subtype assignment rather than flat partitioning — the article used hierarchical clustering split into five clusters for the five glioblastomas instead of k-means
- When the goal is cell-cycle effect identification itself — k-means is a downstream evaluation step; ccRemover/scLVM are the correction methods under test

## Inputs

- Raw scRNA-Seq expression matrix (cells x genes, e.g. 176 cells x 57,820 genes from GSE69405)
- Filter/transformed gene-centered log-scale matrix (cells x 10,977 genes) for each correction condition (original, scLVM, ccRemover)
- Integer number of clusters k (e.g. 4 for technical-replicate separation testing)
- Table of known cell-type / replicate labels per cell (e.g. LC.PT, LC.PT_RE)

## Outputs

- Cluster assignment vector assigning each of the 176 cells to one of k clusters
- Cross-tabulation of cluster assignments against cell-type labels (cluster_composition.csv) with per-cluster majority-label percentages
- For the 2-means variant: sets of differentially expressed genes identified by t-test at FDR 0.01 and downstream hypergeometric odds ratios

## How to apply

Preprocess the raw matrix by removing genes with zero expression in over two thirds of cells (57,820 → 10,977 genes for the Kim et al. LUAD data), log-transforming by adding 1 and taking the natural log, then centering each gene to zero mean. Choose k from the biological question: k=4 for the LUAD task testing whether the 77 cells of the first cell type split by technical replicate set, or k=2 when the goal is to identify differentially expressed genes via a t-test at FDR 0.01. Run the same clustering algorithm and statistical tests on each corrected matrix (original, scLVM-corrected, ccRemover-corrected) so performance can be compared across corrections. Finally cross-tabulate cluster assignments against known cell-type labels (e.g. save cluster_composition.csv) and inspect per-cluster majority-label fractions — after ccRemover correction, cluster 3 is ~80% LC.PT_RE cells and cluster 4 is ~89% LC.PT cells, a replicate separation not visible on original or scLVM-corrected data.

## Related tools

- **ccRemover** (Upstream cell-cycle correction whose effect on k-means cluster separability is being evaluated; run as ccRemover(dat, cutoff=3, max_it=4, nboot=200, ntop=10)) — cran/ccRemover
- **scLVM** (Comparison latent-factor correction (applied via the Python script) whose corrected matrix is clustered with the same k-means procedure as a baseline)
- **gene_indexer** (Annotates the 757 cell-cycle genes (757 of 10,977 filtered genes) via species='human' to build the if_cc indicator needed before correction and clustering)
- **org.Hs.eg.db** (Human gene-annotation backend used by gene_indexer to supply the cell-cycle gene set to ccRemover prior to clustering)

## Examples

```
xhat <- ccRemover(dat, cutoff=3, max_it=4, nboot=200, ntop=10, bar=FALSE); km <- kmeans(xhat, centers=4)
```

## Evaluation signals

- Cluster membership sums to the full cell count (176 cells partitioned across k=4 clusters with none dropped)
- After ccRemover correction, cluster 3 is ~80% LC.PT_RE cells and cluster 4 is ~89% LC.PT cells, while on original and scLVM-corrected data the technical replicates remain non-separable
- The identical clustering algorithm and statistical tests are applied to every corrected matrix so results are directly comparable
- Per-cluster majority-label fraction computed from the cluster x label cross-tabulation (cluster_composition.csv) matches expected compositions
- For the 2-means variant, DE gene identification holds the t-test false discovery rate at 0.01 before computing odds ratios

## Limitations

- k-means on corrected data can separate technical-replicate batch effects that ccRemover does not remove — it is designed to remove the cell-cycle effect exclusively, so other confounders may dominate after correction
- scLVM can mistakenly remove biological rather than cell-cycle effects (e.g. with K=1 on simulated data it removed the cell-type effect; with K=3 it removed both, leaving only noise), which corrupts downstream cluster compositions
- The choice of k is question-dependent and somewhat arbitrary (k=4 to probe two technical-replicate sets; k=2 for DE testing), and clustering results depend on the completeness of the cell-cycle gene annotation

## Evidence

- [results] 4-means clustering to test technical replicate separation: "we consider using 4-means clustering, in order to see whether the 77 cells of the first cell type can be clustered accordingly to the two sets of technical replicates"
- [results] 2-means clustering with t-test FDR 0.01 for DE genes: "we applied 2-means clustering and use a t-test with false discovery rate 0.01 to identify differentially expressed genes"
- [results] Same clustering applied across all correction conditions: "the same clustering algorithms and statistical tests are applied to each of them allowing us to compare the performance of the methods"
- [methods] Log-transform and gene-centering as clustering prerequisite: "We recommend transforming the data to a log scale and centering it on a gene-by-gene basis."
