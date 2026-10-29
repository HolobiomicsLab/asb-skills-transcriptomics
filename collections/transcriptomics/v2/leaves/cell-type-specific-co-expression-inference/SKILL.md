---
name: cell-type-specific-co-expression-inference
description: Use when raw UMI-count scRNA-seq data need cell-type-specific co-expression
  networks, when correlation-based methods give inflated false positives due to sequencing
  depth confounding, or when differential co-expression between disease groups is
  sought.
when_to_use_negative:
- When aggregating cells across multiple cell types — CS-CORE is cell-type-specific
  and applying it to mixed populations confounds co-expression with varying cell-type
  compositions (Simpson's paradox risk).
- When the input is already a normalized/transformed expression matrix rather than
  raw UMI counts, since the measurement model is defined on Poisson UMI counts.
- When the target is bulk-sample co-expression inference — the article describes integrating
  bulk with single-cell data as a future opportunity, not an implemented capability.
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: CS-CORE / CSCORE (R package)
  role: 'Core estimator: IRLS-based inference of cell-type-specific co-expressions
    and p-values from raw UMI counts; diff_CSCORE for differential co-expression;
    covariate adjustment via covariate_names.'
  repo: https://github.com/ChangSuBiostats/CS-CORE
- name: Seurat
  role: Loads and subsets the scRNA-seq Seurat object (e.g., pbmc[, pbmc$cell.type.coarse
    %in% 'B']) to the target cell type and condition.
- name: WGCNA
  role: Builds module structure from the thresholded CS-CORE co-expression matrix
    via adjacency.fromSimilarity (power = 1) and TOMsimilarity.
- name: dynamicTreeCut
  role: Cuts the WGCNA hierarchical clustering dendrogram with cutreeDynamic to define
    co-expressed gene modules (minClusterSize = 10).
- name: clusterProfiler
  role: Annotates co-expression modules with GO terms via enrichGO (OrgDb = org.Hs.eg.db,
    keyType = SYMBOL, pAdjustMethod = BH, pvalueCutoff = 0.05).
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-40503-7
    title: Cell-type-specific co-expression inference from single cell RNA-sequencing
      data
  source_content_hash: sha256:cabdeb93ca51aff744c8a4038928a9ce3f83dd4bd74138418e15a9ddb784fa9f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChangSuBiostats/CS-CORE
  tools:
  - CS-CORE
  - CS-CORE / CSCORE (R package)
  - CSCORE
  - Seurat
  - WGCNA
  - clusterProfiler
  - diff_CSCORE
  - dynamicTreeCut
  - p.adjust BH
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# cell-type-specific-co-expression-inference

## Summary

Infers and tests cell-type-specific gene co-expressions from single-cell RNA-seq data using CS-CORE, which models sequencing depth variations and measurement errors via a Poisson measurement model and an IRLS-based correlation estimator, enabling reproducible co-expression networks and differential co-expression analysis.

## When to use

Use when raw UMI-count scRNA-seq data need cell-type-specific co-expression networks, when correlation-based methods give inflated false positives due to sequencing depth confounding, or when differential co-expression between disease groups is sought.

## When NOT to use

- When aggregating cells across multiple cell types — CS-CORE is cell-type-specific and applying it to mixed populations confounds co-expression with varying cell-type compositions (Simpson's paradox risk).
- When the input is already a normalized/transformed expression matrix rather than raw UMI counts, since the measurement model is defined on Poisson UMI counts.
- When the target is bulk-sample co-expression inference — the article describes integrating bulk with single-cell data as a future opportunity, not an implemented capability.

## Inputs

- Raw UMI count matrix from scRNA-seq data (e.g., a Seurat object such as blish_covid.seu.rds)
- Cell-type labels / metadata for subsetting cells (e.g., cell.type.coarse)
- Group and sample/donor labels for differential co-expression analysis (e.g., Status, Donor)
- Selected gene list (e.g., top 5000 genes by mean expression)
- Optional covariate names (e.g., percent.mt, Sex)

## Outputs

- Cell-type-specific co-expression estimates (correlation matrix) with p-values
- BH-thresholded co-expression matrix with non-significant entries set to 0
- Co-expressed gene modules from WGCNA clustering
- GO enrichment tables per module (clusterProfiler enrichGO results)
- Differentially co-expressed gene pairs and modules from diff_CSCORE
- Compiled results table (e.g., ad_coexpression_results.csv)

## How to apply

Subset cells by cell type (e.g., B cells or CD14 monocytes from a Seurat object) to avoid confounding by cell-type composition, then select genes with meaningful expression (e.g., top 5000 by mean expression) and run CSCORE on raw UMI counts using the IRLS-based estimator to obtain co-expression estimates and p-values. Apply Benjamini-Hochberg adjustment and zero out entries with BH-adjusted p > 0.05 to threshold the co-expression matrix. Extract co-expressed gene modules with WGCNA: adjacency.fromSimilarity on absolute CS-CORE estimates with power = 1, TOMsimilarity, average-linkage hierarchical clustering, and dynamicTreeCut::cutreeDynamic (minClusterSize = 10), then annotate modules with clusterProfiler::enrichGO against org.Hs.eg.db. For group comparisons, run diff_CSCORE with group_label and sample_label and donor-level permutations (e.g., 100) to identify differentially co-expressed pairs; optionally adjust covariates (e.g., percent.mt, Sex) via CSCORE's covariate_names argument.

## Related tools

- **CS-CORE / CSCORE (R package)** (Core estimator: IRLS-based inference of cell-type-specific co-expressions and p-values from raw UMI counts; diff_CSCORE for differential co-expression; covariate adjustment via covariate_names.) — https://github.com/ChangSuBiostats/CS-CORE
- **Seurat** (Loads and subsets the scRNA-seq Seurat object (e.g., pbmc[, pbmc$cell.type.coarse %in% 'B']) to the target cell type and condition.)
- **WGCNA** (Builds module structure from the thresholded CS-CORE co-expression matrix via adjacency.fromSimilarity (power = 1) and TOMsimilarity.)
- **dynamicTreeCut** (Cuts the WGCNA hierarchical clustering dendrogram with cutreeDynamic to define co-expressed gene modules (minClusterSize = 10).)
- **clusterProfiler** (Annotates co-expression modules with GO terms via enrichGO (OrgDb = org.Hs.eg.db, keyType = SYMBOL, pAdjustMethod = BH, pvalueCutoff = 0.05).)

## Examples

```
library(CSCORE); library(Seurat); pbmc_B = pbmc[,pbmc$cell.type.coarse %in% 'B']; CSCORE_result <- CSCORE(pbmc_B_healthy, genes = genes_selected); CSCORE_coexp[p_matrix_BH > 0.05] <- 0
```

## Evaluation signals

- Significant CS-CORE co-expressions (BH-adjusted p < 0.05) reproduce in an independent dataset (e.g., Synapse syn21261143/syn22079621) at higher rates than rho-sctransform, rho-analytic PR, or SpQN across p-value cutoffs and cell types.
- Inferred co-expression networks are enriched for known TRRUST transcription factor-target pairs relative to competing methods.
- Co-expression modules yield at least one GO term with BH-adjusted p < 0.001 and at least 10 enriched GO terms for top modules.
- Estimates are accurate in simulation benchmarks without the inflated type-I errors and estimation biases observed for existing methods.
- Differential co-expression calls are made at the donor/sample level with permutation testing (e.g., 100 permutations), not cell-level pseudo-replication.

## Limitations

- The measurement model uses only a Poisson distribution; adaptation to other count distributions (e.g., negative binomial) is deferred to future work.
- CS-CORE assumes gene expressions from cells of the same cell type follow the same distribution, which may not hold across individuals with different genetic, demographic, and clinical characteristics.
- Extension to individualized co-expression networks and cross-genotype/condition comparisons is planned but not implemented in the published version.
- Existing correlation-based alternatives (rho-sctransform, rho-analytic PR) suffer from inflated type-I errors, estimation biases, and reduced power — do not substitute them naively.

## Evidence

- [abstract] We propose a statistical approach, CS-CORE, for estimating and testing cell-type-specific co-expressions: "We propose a statistical approach, CS-CORE, for estimating and testing cell-type-specific co-expressions"
- [abstract] CS-CORE models sequencing depth and measurement error: "explicitly models sequencing depth variations and measurement errors in scRNA-seq data"
- [methods] Run CSCORE on selected genes: "CSCORE_result <- CSCORE(pbmc_B_healthy, genes = genes_selected)"
- [methods] BH thresholding of co-expression matrix: "CSCORE_coexp[p_matrix_BH > 0.05] <- 0"
- [methods] WGCNA module extraction: "adj = WGCNA::adjacency.fromSimilarity(abs(CSCORE_coexp), power = 1)"
- [methods] Covariate adjustment in CSCORE: "covariate_names = c('percent.mt', 'Sex'))"
- [intro] IRLS estimator and independence test: "CS-CORE implements a fast and efficient iteratively re-weighted least squares approach for estimating the true correlations between underlying expression levels"
- [readme] README: CS-CORE is an R package on GitHub: "`CS-CORE` is a R package for cell-type-specific co-expression inference from single cell RNA-sequencing data."
- [discussion] Discussion: distributional assumption is Poisson only: "It can be useful to adapt CS-CORE to model xij∣zij using other distributions that model nonnegative integers such as the negative binomial distribution"
