---
name: cross-dataset-coexpression-reproducibility-assessment
description: Use when multiple methods infer cell-type-specific co-expression from
  scRNA-seq data and you must test which method's significant gene pairs replicate
  in an independent dataset.
when_to_use_negative:
- Only one scRNA-seq dataset is available with no independent replication dataset,
  so reproducibility cannot be computed.
- Cells are aggregated across cell types rather than subset — co-expression inference
  is then confounded (Simpson's paradox), invalidating any reproducibility comparison.
- 'Input is bulk RNA-seq data: inferred networks are confounded by varying cell-type
  compositions across samples and only give an aggregated view of gene regulations.'
tools:
- name: CSCORE (R package)
  role: Infers cell-type-specific co-expressions and p-values from raw UMI counts
    of each cell type using the ILS/IRLS-based estimator, producing the significant-pair
    sets whose reproducibility is scored.
  repo: https://github.com/ChangSuBiostats/CS-CORE
- name: CS-CORE
  role: The statistical approach (modeling sequencing depth variation and measurement
    error) whose co-expressions are benchmarked for reproducibility against competing
    estimators.
  repo: https://github.com/ChangSuBiostats/CS-CORE
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-40503-7
    title: Cell-type-specific co-expression inference from single cell RNA-sequencing
      data
  source_content_hash: sha256:4c9a3a911179af8d17063ff48cf7cf7e1a650f48af2a291df3dab9c567784efe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChangSuBiostats/CS-CORE
  tools:
  - CS-CORE
  - CSCORE
  - CSCORE (R package)
  - Seurat
  - clusterProfiler
  - diff_CSCORE
  - dynamicTreeCut
  - p.adjust BH
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# cross-dataset-coexpression-reproducibility-assessment

## Summary

A benchmarking skill for scRNA-seq co-expression inference: it quantifies whether significant gene co-expressions inferred by a method (e.g., CS-CORE) are recovered in independent datasets, against benchmarks such as rho-sctransform, rho-analytic PR, and SpQN. It was used to show CS-CORE co-expressions in Alzheimer's disease postmortem brain control cells are more reproducible than competitors.

## When to use

Use when multiple methods infer cell-type-specific co-expression from scRNA-seq data and you must test which method's significant gene pairs replicate in an independent dataset.

## When NOT to use

- Only one scRNA-seq dataset is available with no independent replication dataset, so reproducibility cannot be computed.
- Cells are aggregated across cell types rather than subset — co-expression inference is then confounded (Simpson's paradox), invalidating any reproducibility comparison.
- Input is bulk RNA-seq data: inferred networks are confounded by varying cell-type compositions across samples and only give an aggregated view of gene regulations.

## Inputs

- Raw UMI count matrix and cell-type annotations from the discovery scRNA-seq dataset (e.g., GSE157827 postmortem brain, control-subject cells per cell type)
- Independent reference scRNA-seq dataset(s) for replication (e.g., Synapse syn21261143 and syn22079621 prefrontal cortex datasets)
- Per-method co-expression estimate matrices with associated p-values (CS-CORE, rho-sctransform, rho-analytic PR, SpQN)

## Outputs

- Reproducibility statistics of significant co-expressed gene pairs per method, per cell type, and per p-value cutoff
- TRRUST transcription factor-target enrichment results per method
- Compiled comparison table (e.g., ad_coexpression_results.csv)

## How to apply

First, subset cells by cell type (e.g., the five major brain cell types from GSE157827 control subjects) to avoid Simpson's paradox, then run each estimator — CS-CORE (IRLS-based), rho-sctransform, rho-analytic PR, and SpQN — on the same raw UMI counts per cell type. Threshold every co-expression matrix by Benjamini-Hochberg-adjusting p-values and setting entries with adjusted p > 0.05 to zero. Compute reproducibility of the remaining significant co-expressed pairs against independent prefrontal cortex reference datasets (Synapse syn21261143 and syn22079621) across different p-value cutoffs and cell types. Complement reproducibility with enrichment of significant pairs for known TRRUST transcription factor-target pairs, since reproducibility alone does not confirm biological relevance. The method whose significant pairs replicate most consistently and are most TF-target enriched is judged superior; in ref. 17's data this was CS-CORE over rho-sctransform.

## Related tools

- **CSCORE (R package)** (Infers cell-type-specific co-expressions and p-values from raw UMI counts of each cell type using the ILS/IRLS-based estimator, producing the significant-pair sets whose reproducibility is scored.) — https://github.com/ChangSuBiostats/CS-CORE
- **CS-CORE** (The statistical approach (modeling sequencing depth variation and measurement error) whose co-expressions are benchmarked for reproducibility against competing estimators.) — https://github.com/ChangSuBiostats/CS-CORE

## Examples

```
CSCORE_result <- CSCORE(pbmc_B_healthy, genes = genes_selected); CSCORE_coexp[p_matrix_BH > 0.05] <- 0
```

## Evaluation signals

- CS-CORE-inferred significant co-expressed pairs show higher reproducibility in the independent dataset (syn21261143) than rho-sctransform pairs, consistently across different p-value cutoffs and across all five major brain cell types.
- All method matrices are thresholded identically: entries with BH-adjusted p > 0.05 set to zero before any reproducibility comparison.
- CS-CORE significant pairs show greater enrichment for known TRRUST transcription factor-target pairs than the benchmark methods.
- Per-cell-type subsetting is verified — no analysis pools cells across the five major brain cell types.
- Reproducibility is reported as a curve over p-value cutoffs, not at a single threshold, to demonstrate the signal is not cutoff-dependent.

## Limitations

- Reproducibility assessment requires an independent dataset of the same tissue/cell types; without one, only TF-target enrichment benchmarks remain available.
- Competing methods suffer from inflated type-I errors, so thresholding them at the same BH cutoff may still leak false positives into the comparison; consider multiple cutoffs.
- CS-CORE's measurement model uses only a Poisson distribution for UMI counts (negative-binomial adaptation deferred to future work), and it assumes cells of the same cell type follow the same distribution, which may not hold across individuals with different genetic, demographic and clinical characteristics.

## Evidence

- [intro] CS-CORE reproducibility superiority: "CS-CORE identified co-expressions that were more reproducible across independent data sets and more enriched with known transcription factor-target gene pairs than other methods"
- [methods] BH thresholding step: "p.adjust(CSCORE_p[upper.tri(CSCORE_p)], method = "BH")"
- [methods] Zeroing non-significant co-expressions: "CSCORE_coexp[p_matrix_BH > 0.05] <- 0"
- [intro] Existing methods confound UMI-count correlations: "gene co-expressions measured via correlations of UMI counts across cells can be seriously confounded by varying sequencing depths, resulting in inflated false positive findings"
- [readme] CS-CORE package description: "`CS-CORE` is a R package for cell-type-specific co-expression inference from single cell RNA-sequencing data."
