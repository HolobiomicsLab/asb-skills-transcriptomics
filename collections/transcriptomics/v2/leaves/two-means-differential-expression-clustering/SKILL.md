---
name: two-means-differential-expression-clustering
description: Use when comparing cell-cycle-corrected vs. original scRNA-Seq matrices
  and you need DE genes plus a signature-gene enrichment statistic (odds ratio, hypergeometric
  p, 95% CI) to judge whether recovered substructure is biologically meaningful.
when_to_use_negative:
- The goal is to detect more than two subpopulations — the paper instead uses hierarchical
  clustering with 5 clusters for the glioblastoma data and 4-means clustering for
  lung adenocarcinoma replicate separation
- The task is removing the cell-cycle effect itself rather than evaluating corrected
  data downstream — use ccRemover (cutoff=3, max_it=4, nboot=200, ntop=10) as a pre-processing
  step first
- The expression data has not been corrected or centered gene-by-gene on the log scale
  — clustering raw uncorrected data on the T-helper data yields a signature odds ratio
  below 1 and is not physiologically meaningful
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ccRemover
  role: Produces one of the corrected matrices (xhat with gene means re-added) that
    is clustered with 2-means and tested for DE genes; yields the reference OR of
    3.439
- name: scLVM
  role: Produces the competing corrected matrices (default K=1 and scree-plot K) subjected
    to the identical 2-means/t-test/enrichment workflow; reference OR = 2.382
- name: Python
  role: Used to run the scLVM correction script from the scLVM GitHub page before
    DE clustering
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/srep33892
    title: Identifying and removing the cell-cycle effect from single-cell RNA-Sequencing
      data
  source_content_hash: sha256:94cdcee65d3d109dc11d394c2d7fc4a61fb24c758945df2d3c40e685700005fb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - ccRemover
  - org.Mm.eg.db
  - scLVM
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# two-means-differential-expression-clustering

## Summary

A downstream evaluation procedure from Barron & Li (2016): split corrected (or original) single-cell expression matrices into two cell clusters via 2-means clustering, identify differentially expressed genes with a t-test at FDR 0.01, and quantify biological meaningfulness by hypergeometric enrichment of curated signature genes (e.g. the 122 TH2 signature genes).

## When to use

Use when comparing cell-cycle-corrected vs. original scRNA-Seq matrices and you need DE genes plus a signature-gene enrichment statistic (odds ratio, hypergeometric p, 95% CI) to judge whether recovered substructure is biologically meaningful.

## When NOT to use

- The goal is to detect more than two subpopulations — the paper instead uses hierarchical clustering with 5 clusters for the glioblastoma data and 4-means clustering for lung adenocarcinoma replicate separation
- The task is removing the cell-cycle effect itself rather than evaluating corrected data downstream — use ccRemover (cutoff=3, max_it=4, nboot=200, ntop=10) as a pre-processing step first
- The expression data has not been corrected or centered gene-by-gene on the log scale — clustering raw uncorrected data on the T-helper data yields a signature odds ratio below 1 and is not physiologically meaningful

## Inputs

- Gene-by-cell expression matrices (e.g. t.cell_data, 81 cells × 14,147 genes, genes as rows) — original, scLVM-corrected, and ccRemover-corrected versions to compare
- Curated signature-gene list (the 122 TH2 signature genes) with gene identifiers matching the matrix row names (mouse symbols)
- Clustering assignment of cells into two groups from 2-means clustering

## Outputs

- Per-method set of differentially expressed genes (t-test, FDR 0.01)
- Enrichment statistics table: odds ratio, hypergeometric p-value, 95% CI per corrected/original matrix (th2_enrichment_results.csv)
- Bimodality density plots of cell-cycle genes for original vs. corrected matrices

## How to apply

For each expression matrix to compare (original data, scLVM-corrected at K=1 and at the scree-plot K, and ccRemover-corrected with the stored gene means re-added), cluster cells with 2-means clustering and identify differentially expressed genes using a t-test with false discovery rate 0.01. Then compute hypergeometric over-representation of the 122 curated TH2 signature genes among the DE genes for each method, reporting odds ratios, hypergeometric p-values, and confidence intervals. Rationale: the same clustering algorithms and statistical tests must be applied to every corrected dataset so the methods are directly comparable; a baseline run on the original (uncorrected) data is included and, for the T-helper data, its odds ratio falls below 1, indicating the clustering is not physiologically meaningful. Expected reference results: ccRemover OR = 3.439 (lower 95% CI 2.297, hypergeometric p = 2.373 × 10−9) vs. scLVM OR = 2.382, demonstrating ccRemover better reveals TH2 substructure.

## Related tools

- **ccRemover** (Produces one of the corrected matrices (xhat with gene means re-added) that is clustered with 2-means and tested for DE genes; yields the reference OR of 3.439)
- **scLVM** (Produces the competing corrected matrices (default K=1 and scree-plot K) subjected to the identical 2-means/t-test/enrichment workflow; reference OR = 2.382)
- **Python** (Used to run the scLVM correction script from the scLVM GitHub page before DE clustering)

## Examples

```
km <- kmeans(t(expr), centers=2); p <- sapply(seq_len(nrow(expr)), function(i) t.test(expr[i,] ~ km$cluster)$p.value); de <- p.adjust(p,'BH') < 0.01; phyper(sum(de %in% th2), length(de), nrow(expr)-length(de), 122, lower.tail=FALSE)
```

## Evaluation signals

- Differential expression set defined by t-test with FDR 0.01 after 2-means clustering of cells — verify the BH-adjusted p-value threshold was actually applied
- ccRemover-corrected matrix should give TH2 signature odds ratio ≈ 3.439 (lower 95% CI 2.297, hypergeometric p = 2.373 × 10−9)
- scLVM-corrected matrix should give a lower odds ratio (reference ≈ 2.382); original (uncorrected) data should give OR < 1
- Identical clustering and testing pipeline must be run on all matrices so odds ratios are comparable across methods
- All enrichment statistics saved to th2_enrichment_results.csv, and bimodality density of cell-cycle genes plotted for original vs. corrected matrices

## Limitations

- The 2-means/t-test setup assumes a two-cluster hypothesis; other datasets in the paper required 4-means (lung adenocarcinoma) or hierarchical clustering with 5 clusters (glioblastoma) instead
- On uncorrected original T-helper data the TH2 odds ratio is less than 1, so DE clustering on uncorrected data can produce clusters that are not physiologically meaningful
- scLVM results depend on the choice of K (default K=1 vs. scree-plot value), which changes the DE/enrichment outcome
- ccRemover upstream depends on annotation databases that are always incomplete and inaccurate, and may miss the cell-cycle effect if it influences too few control genes

## Evidence

- [results] 2-means DE clustering with t-test FDR 0.01: "we applied 2-means clustering and use a t-test with false discovery rate 0.01 to identify differentially expressed genes"
- [results] same tests applied to each corrected dataset for comparability: "the same clustering algorithms and statistical tests are applied to each of them allowing us to compare the performance of the methods"
- [results] original-data baseline clusters are not physiologically meaningful: "On the original data, the odds ratio is less than 1, indicating that the clustering of cells is unlikely to be physiologically meaningful"
- [methods] T-helper dataset provenance for the clustering input: [evidence span withheld — anchor status: not_found]
- [abstract] cell-cycle correction as prerequisite pre-processing: "ccRemover preserves other biological signals of interest in the data and thus can serve as an important pre-processing step for many scRNA-Seq data analyses."
