---
name: tf-target-enrichment-evaluation
description: Use when evaluating a co-expression inference method on scRNA-seq data
  (e.g., AD snRNA-seq or COVID PBMC) by checking that its significant co-expressed
  gene pairs capture known TRRUST transcription factor-target relationships more than
  benchmark methods.
when_to_use_negative:
- The goal is functional annotation of co-expression gene modules rather than benchmarking
  the co-expression inference method itself — use GO enrichment via clusterProfiler::enrichGO
  instead.
- Input co-expressions are inferred across mixed cell types without subsetting, violating
  the CS-CORE recommendation to subset by cell type to avoid Simpson's paradox.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0659
tools:
- name: CS-CORE
  role: Primary co-expression inference method whose significant gene pairs are tested
    for TRRUST transcription factor-target enrichment
  repo: github.com/ChangSuBiostats/CS-CORE
- name: rho-sctransform
  role: Benchmark estimator whose co-expressions are evaluated for TRRUST enrichment
    in comparison to CS-CORE
- name: Seurat
  role: Loads and subsets the scRNA-seq data by cell type before running the estimators
- name: p.adjust BH
  role: Benjamini-Hochberg multiple-testing correction used to threshold co-expression
    p-values at 0.05 before enrichment testing
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-40503-7
    title: Cell-type-specific co-expression inference from single cell RNA-sequencing
      data
  source_content_hash: sha256:acbc456b646ae305b11bef16741b8c5839cff3cb1ce796a8511facb113179bda
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CS-CORE
  - CSCORE
  - Seurat
  - clusterProfiler
  - diff_CSCORE
  - dynamicTreeCut
  - p.adjust BH
  - rho-sctransform
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# tf-target-enrichment-evaluation

## Summary

Benchmarks cell-type-specific co-expression inference by testing whether inferred co-expressed gene pairs are enriched for known transcription factor-target pairs from TRRUST, comparing CS-CORE against competitors (rho-sctransform, rho-analytic PR, SpQN). It serves as a biological-relevance validation that complements cross-dataset reproducibility.

## When to use

Use when evaluating a co-expression inference method on scRNA-seq data (e.g., AD snRNA-seq or COVID PBMC) by checking that its significant co-expressed gene pairs capture known TRRUST transcription factor-target relationships more than benchmark methods.

## When NOT to use

- The goal is functional annotation of co-expression gene modules rather than benchmarking the co-expression inference method itself — use GO enrichment via clusterProfiler::enrichGO instead.
- Input co-expressions are inferred across mixed cell types without subsetting, violating the CS-CORE recommendation to subset by cell type to avoid Simpson's paradox.

## Inputs

- Raw UMI count matrix from scRNA-seq cells of a single cell type (Seurat object subset, e.g., GSE157827 control brain cells or Wilk et al. PBMC COVID B cells)
- Co-expression estimate and p-value matrices from CS-CORE and competitor estimators (rho-sctransform, rho-analytic PR, SpQN)
- TRRUST transcription factor-target pair reference database

## Outputs

- TRRUST enrichment statistics for significant co-expressed gene pairs per method and cell type
- Comparison table of enrichment results across methods (contributing to ad_coexpression_results.csv)

## How to apply

Run each co-expression estimator (CS-CORE via CSCORE with the IRLS estimator, rho-sctransform, rho-analytic PR, SpQN) on the same raw UMI counts from cells of one cell type, e.g., control cells from GSE157827 across five major brain cell types. Threshold each inferred co-expression matrix by setting entries with Benjamini-Hochberg-adjusted p-values greater than 0.05 to zero, keeping only significant pairs. Test the significant gene pairs for enrichment of TRRUST-curated transcription factor-target pairs, and compare enrichment across methods. The method is judged superior when its significant co-expressions show greater TRRUST enrichment than the benchmark methods, alongside greater reproducibility in independent datasets (e.g., Synapse syn21261143 and syn22079621).

## Related tools

- **CS-CORE** (Primary co-expression inference method whose significant gene pairs are tested for TRRUST transcription factor-target enrichment) — github.com/ChangSuBiostats/CS-CORE
- **rho-sctransform** (Benchmark estimator whose co-expressions are evaluated for TRRUST enrichment in comparison to CS-CORE)
- **Seurat** (Loads and subsets the scRNA-seq data by cell type before running the estimators)
- **p.adjust BH** (Benjamini-Hochberg multiple-testing correction used to threshold co-expression p-values at 0.05 before enrichment testing)

## Examples

```
CSCORE_result <- CSCORE(pbmc_B_healthy, genes = genes_selected)
```

## Evaluation signals

- CS-CORE significant co-expressions (BH-adjusted p <= 0.05) show greater TRRUST transcription factor-target enrichment than rho-sctransform, rho-analytic PR, and SpQN across the five brain cell types.
- Co-expression matrices are correctly thresholded: all entries with BH-adjusted p > 0.05 are set to zero before enrichment testing.
- TRRUST enrichment is computed on the same cell-type-subsetted cells for all methods, ensuring a fair comparison.
- The enrichment result is corroborated by reproducibility in independent prefrontal cortex datasets (refs. 26/27, Synapse syn21261143 and syn22079621).

## Limitations

- TRRUST enrichment evaluates only known transcription factor-target relationships; co-expressions outside TF-target biology are not assessed by this criterion.
- Existing methods (including rho-sctransform and rho-analytic PR) suffered from inflated type-I errors, estimation biases, and reduced power, so their enrichment scores are not directly interpretable as true co-expression strength.
- CS-CORE assumes gene expressions from cells of the same cell type follow the same distribution, which may not hold across individuals with different genetic, demographic, and clinical characteristics.

## Evidence

- [intro] CS-CORE identified co-expressions that were more reproducible across independent data sets and more enriched with known transcription factor-target gene pairs than other methods: "CS-CORE identified co-expressions that were more reproducible across independent data sets and more enriched with known transcription factor-target gene pairs than other methods"
- [methods] BH p-value thresholding of co-expression matrix: "CSCORE_coexp[p_matrix_BH > 0.05] <- 0"
- [intro] Existing methods suffer inflated type-I errors and reduced power, motivating benchmark comparison: "we found that all the existing methods, including ρ-sctransform and ρ-analytic PR, suffer from inflated type-I errors, varying degrees of estimation biases, reduced power in detecting co-expressions"
- [readme] CS-CORE is an R package for cell-type-specific co-expression inference from scRNA-seq data: "CS-CORE` is a R package for cell-type-specific co-expression inference from single cell RNA-sequencing data."
