---
name: single-cell-rna-seq-normalization
description: Use when a scRNA-Seq expression matrix shows strong within-cell-type
  heterogeneity driven by cell-cycle genes (e.g. cells cluster by cell-cycle stage
  instead of cell type) and you need a corrected matrix before clustering.
when_to_use_negative:
- The dataset is known to have very low cell-cycle activity (e.g. the Patel et al.
  glioblastoma data), where the default CΔ cutoff of 3 is too strict and a lower cutoff
  (CΔ = 2) must be used instead of the default call
- The dominant unwanted variation is not the cell cycle (e.g. batch or other confounders)
  — ccRemover is designed for removing the cell-cycle effect exclusively and will
  not remove these other effects
- The goal is to remove any leading factor regardless of its annotation (the scLVM
  approach) rather than to test whether each factor is cell-cycle-driven; blind factor
  removal risks deleting the biological signal of interest
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ccRemover
  role: 'Core correction routine: bootstraps cell-cycle vs control gene loadings,
    flags PCs with bootstrap t-statistic above cutoff, and removes them'
  repo: cran/ccRemover
- name: scLVM
  role: Comparison method applied via the Python script from the scLVM GitHub page,
    with K=1 default and scree-plot-suggested K, to benchmark correction quality
- name: org.Mm.eg.db
  role: Mouse cell-cycle gene annotations used by gene_indexer to build the if_cc
    indicator for the T-helper dataset
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/srep33892
    title: Identifying and removing the cell-cycle effect from single-cell RNA-Sequencing
      data
  source_content_hash: sha256:8ae7ad8c4cae6264ff0072ab61af8fc03e257dc02165007dad42bdeb51cbde67
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cran/ccRemover
  tools:
  - Python
  - ccRemover
  - org.Mm.eg.db
  - scLVM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# single-cell-rna-seq-normalization

## Summary

A pre-processing workflow for single-cell RNA-Seq matrices that identifies leading principal components driven by annotated cell-cycle genes (using a bootstrap t-statistic) and removes them, while preserving other biological signals such as the TH2 substructure in differentiating T-helper cells.

## When to use

Use when a scRNA-Seq expression matrix shows strong within-cell-type heterogeneity driven by cell-cycle genes (e.g. cells cluster by cell-cycle stage instead of cell type) and you need a corrected matrix before clustering.

## When NOT to use

- The dataset is known to have very low cell-cycle activity (e.g. the Patel et al. glioblastoma data), where the default CΔ cutoff of 3 is too strict and a lower cutoff (CΔ = 2) must be used instead of the default call
- The dominant unwanted variation is not the cell cycle (e.g. batch or other confounders) — ccRemover is designed for removing the cell-cycle effect exclusively and will not remove these other effects
- The goal is to remove any leading factor regardless of its annotation (the scLVM approach) rather than to test whether each factor is cell-cycle-driven; blind factor removal risks deleting the biological signal of interest

## Inputs

- Log-scaled scRNA-Seq gene-by-cell expression matrix (e.g. t.cell_data: 81 cells x 14,147 genes, genes as rows) from the ccRemover package
- Vector of gene names (official symbols for 'mouse' or 'human')
- Cell-cycle gene annotations from org.Mm.eg.db / org.Hs.eg.db retrieved via gene_indexer

## Outputs

- Corrected expression matrix xhat with cell-cycle principal components removed (original dimensionality restored by re-adding stored gene means)
- if_cc TRUE/FALSE indicator vector marking annotated cell-cycle genes
- Per-iteration detection log of flagged components (e.g. PC1 flagged on iteration 1 with t.load.boot = 7.13, no further effects on iteration 2)
- Downstream comparison statistics (odds ratios, hypergeometric p-values, 95% CIs, e.g. th2_enrichment_results.csv)

## How to apply

Log-transform the counts and center the matrix gene-by-gene with rowMeans, storing the gene means so they can be re-added after correction (e.g. the 81-cell x 14,147-gene T-helper t.cell_data from Buettner/Mahata, genes as rows). Build the cell-cycle indicator with gene_indexer(gene_names, species='mouse'|'human', name_type='symbols') using org.Mm.eg.db / org.Hs.eg.db annotations, and pass dat <- list(x = centered_matrix, if_cc = if_cc) to ccRemover with the default parameters cutoff=3, max_it=4, nboot=200, ntop=10 (lower the CΔ cutoff to 2 for datasets with known low cell-cycle activity such as the Patel et al. glioblastoma data). ccRemover bootstraps differences between loadings of cell-cycle genes and control genes — using all non-annotated genes as controls — and removes any principal component whose bootstrap t-statistic exceeds cutoff, repeating up to max_it iterations. After correction, verify the biological signal survives by re-running the downstream analysis (e.g. 2-means clustering with a t-test at FDR 0.01 to call DE genes, then a hypergeometric enrichment test of a curated signature such as the 122 TH2 genes, expecting OR ≈ 3.439 for ccRemover versus 2.382 for scLVM and OR < 1 on uncorrected data).

## Related tools

- **ccRemover** (Core correction routine: bootstraps cell-cycle vs control gene loadings, flags PCs with bootstrap t-statistic above cutoff, and removes them) — cran/ccRemover
- **scLVM** (Comparison method applied via the Python script from the scLVM GitHub page, with K=1 default and scree-plot-suggested K, to benchmark correction quality)
- **org.Mm.eg.db** (Mouse cell-cycle gene annotations used by gene_indexer to build the if_cc indicator for the T-helper dataset)

## Examples

```
if_cc <- gene_indexer(gene_names, species = "mouse", name_type = "symbols"); xhat <- ccRemover(list(x = t_cell_data_cen, if_cc = if_cc), cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar = FALSE)
```

## Evaluation signals

- On the T-helper data, ccRemover must flag PC1 as a cell-cycle effect on the first iteration (t.load.boot = 7.13, above the cutoff of 3) and report no further effects on iteration 2
- Enrichment of the 122 TH2 signature genes among 2-means/t-test (FDR 0.01) DE genes should yield OR ≈ 3.439 (95% CI lower bound 2.297, hypergeometric p = 2.373e-9) for ccRemover versus 2.382 for scLVM, with OR < 1 on the original data
- The stored gene rowMeans must be re-added so the corrected matrix xhat has the same dimensions (81 x 14,147) and scale as the input
- Cell-cycle gene loadings should show reduced bimodality in the density plot of corrected versus original data
- After correction, cells should cluster by biological substructure with cell-cycle stages mixed, not by cell-cycle stage alone

## Limitations

- Relies on annotation databases that are always incomplete and inaccurate; a known set of cell-cycle genes is required upfront
- If the cell-cycle effect influences only a few control genes (below ~8% affected), it may not be captured by the principal components and can be missed
- ccRemover does not remove other confounding effects that may dominate after cell-cycle removal

## Evidence

- [abstract] ccRemover serves as a pre-processing step before downstream analyses: "ccRemover preserves other biological signals of interest in the data and thus can serve as an important pre-processing step for many scRNA-Seq data analyses."
- [methods] Recommended input transformation: "We recommend transforming the data to a log scale and centering it on a gene-by-gene basis."
- [methods] Concrete ccRemover call and parameters: "xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE)"
- [results] Detection behavior on the T-helper dataset: "When ccRemover is applied to the original data it identifies the first principal component to be a cell-cycle effect on the first iteration."
- [results] Enrichment benchmark on original data: "On the original data, the odds ratio is less than 1, indicating that the clustering of cells is unlikely to be physiologically meaningful"
- [discussion] Control-gene choice recommendation: "we recommend simply using all genes that are not annotated as cell-cycle genes as control genes"
