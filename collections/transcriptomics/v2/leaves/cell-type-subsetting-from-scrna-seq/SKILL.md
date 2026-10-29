---
name: cell-type-subsetting-from-scrna-seq
description: Use when a scRNA-seq Seurat object or raw UMI count matrix contains mixed
  cell types and the analysis target is cell-type-specific co-expression inference
  or null-data simulation for one cell type.
when_to_use_negative:
- The input is already restricted to a single cell type or is a bulk RNA-seq sample,
  where cell-type subsetting is meaningless and CS-CORE's cell-type-specific model
  assumptions do not apply as intended
- The goal is cross-cell-type or aggregated co-expression; CS-CORE assumes gene expressions
  from cells of the same cell type follow the same distribution
- Only normalized/corrected expression matrices (e.g., SCTransform outputs) are available
  — CS-CORE's Poisson measurement model links latent expression to observed UMI counts,
  so raw counts must be retained
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Loads the Seurat object and performs the cell-type subset, e.g. pbmc[,pbmc$cell.type.coarse
    %in% 'B']
- name: CSCORE (R package)
  role: 'Downstream consumer: cell-type-specific co-expression inference is run on
    the subset count matrix with selected genes'
  repo: github.com/ChangSuBiostats/CS-CORE
- name: CS-CORE
  role: Statistical method whose cell-type-specific co-expression test defines why
    subsetting to one cell type is required before inference
  repo: github.com/ChangSuBiostats/CS-CORE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-40503-7
    title: Cell-type-specific co-expression inference from single cell RNA-sequencing
      data
  source_content_hash: sha256:ea538680881e250d93476445460d8920b9dedf87a5e6dbed3143e0713ab43989
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CS-CORE
  - CSCORE (R package)
  - Seurat
  - stats
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# cell-type-subsetting-from-scrna-seq

## Summary

Extracts the cells of a single cell type (e.g., B cells or CD14 monocytes from the Wilk PBMC COVID Seurat object, or excitatory neurons from GSE157827) from a raw UMI count matrix so that downstream cell-type-specific co-expression inference with CS-CORE operates on a homogeneous cell population.

## When to use

Use when a scRNA-seq Seurat object or raw UMI count matrix contains mixed cell types and the analysis target is cell-type-specific co-expression inference or null-data simulation for one cell type.

## When NOT to use

- The input is already restricted to a single cell type or is a bulk RNA-seq sample, where cell-type subsetting is meaningless and CS-CORE's cell-type-specific model assumptions do not apply as intended
- The goal is cross-cell-type or aggregated co-expression; CS-CORE assumes gene expressions from cells of the same cell type follow the same distribution
- Only normalized/corrected expression matrices (e.g., SCTransform outputs) are available — CS-CORE's Poisson measurement model links latent expression to observed UMI counts, so raw counts must be retained

## Inputs

- Seurat object (.rds) with a cell-type metadata column (e.g., cell.type.coarse), such as blish_covid.seu.rds
- Raw UMI count matrix (genes x cells) from a single cell type-annotated scRNA-seq dataset (e.g., GSE157827 postmortem brain samples)
- Per-cell sequencing depths t_i and cell-type labels for the target cell population

## Outputs

- A UMI count matrix restricted to cells of one cell type (e.g., pbmc_B, pbmc_CD14, or excitatory neurons)
- A gene set selected from the subset (e.g., top 5000 genes by mean expression) suitable as CSCORE input
- Optionally, permuted constant-depth and varying-depth null count matrices (e.g., saved as null_count_matrices.rds) for type-I error benchmarking

## How to apply

Load the Seurat object (e.g., blish_covid.seu.rds from the single cell atlas of peripheral immune response to SARS-CoV-2 infection) with the Seurat R package, then subset the count matrix by an existing cell-type annotation column, e.g. pbmc$cell.type.coarse %in% 'B' for B cells or 'CD14 Monocytes'. For the GSE157827 postmortem-brain dataset used in CS-CORE's simulations, subset the raw UMI count matrix to excitatory-neuron cells before normalizing by per-cell sequencing depth t_i. Keep the raw (non-normalized) counts after subsetting, because CS-CORE's measurement model expects UMI counts and accounts for depth variation internally. After subsetting, optionally select genes with meaningful expression (the CS-CORE vignette takes the top 5000 genes by mean expression) and run CSCORE on the subset object. The same subset can serve as the basis for null-data generation: normalize y_ij = x_ij / t_i, permute each gene's values across cells, and resample x*_ij ~ Poisson(t_i * y*_ij^p) under equal or observed depths.

## Related tools

- **Seurat** (Loads the Seurat object and performs the cell-type subset, e.g. pbmc[,pbmc$cell.type.coarse %in% 'B'])
- **CSCORE (R package)** (Downstream consumer: cell-type-specific co-expression inference is run on the subset count matrix with selected genes) — github.com/ChangSuBiostats/CS-CORE
- **CS-CORE** (Statistical method whose cell-type-specific co-expression test defines why subsetting to one cell type is required before inference) — github.com/ChangSuBiostats/CS-CORE

## Examples

```
pbmc_B = pbmc[,pbmc$cell.type.coarse %in% 'B']
```

## Evaluation signals

- The subset count matrix contains only nonnegative integer UMI counts and its columns all carry the target cell-type label
- Each gene's marginal expression distribution is preserved when the subset is used for permutation-based null data generation
- Permutation of normalized expression y_ij = x_ij / t_i across cells yields near-zero co-expression between gene pairs, as required by the null hypothesis for CS-CORE type-I error benchmarking
- The subset object still runs CSCORE successfully on the selected genes (e.g., top 5000 by mean expression) without additional normalization steps
- Both null matrices (constant-depth and varying-depth Poisson resamples) match the dimensions of the original subset matrix

## Limitations

- CS-CORE assumes gene expressions from cells of the same cell type follow the same distribution, which may not hold across individuals with different genetic, demographic and clinical characteristics
- The measurement model uses only a Poisson distribution; adaptation to other count distributions such as the negative binomial is deferred to future work
- The skill depends on a pre-existing, reliable cell-type annotation column; it does not itself perform cell-type identification or clustering

## Evidence

- [methods] Subset cells to cell type of interest: [evidence span withheld — anchor status: not_found]
- [methods] Subset command: "pbmc_B = pbmc[,pbmc$cell.type.coarse %in% 'B']"
- [methods] Gene selection after subsetting: "genes_selected = names(sort.int(mean_exp, decreasing = T))[1:5000]"
- [readme] CS-CORE package purpose: "`CS-CORE` is a R package for cell-type-specific co-expression inference from single cell RNA-sequencing data."
- [intro] Depth confounding motivation: "gene co-expressions measured via correlations of UMI counts across cells can be seriously confounded by varying sequencing depths, resulting in inflated false positive findings"
