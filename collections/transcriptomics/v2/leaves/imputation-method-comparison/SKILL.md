---
name: imputation-method-comparison
description: Use when comparing runtime/scaling of scRNA-seq imputation methods on
  large datasets (tens of thousands of cells), e.g. to decide which imputer is tractable
  for >10,000-cell matrices on a single core.
when_to_use_negative:
- When the goal is to evaluate imputation accuracy (e.g., preserving true biological
  zeros) rather than computational runtime — ALRA's thresholding-based zero preservation
  is a separate capability not measured here
- When the dataset is small (fewer than ~10,000 cells) — all five methods are tractable,
  so runtime benchmarking does not discriminate between them
- When the input is not a scRNA-seq expression matrix — DCA and MAGIC treat all zeros
  as 'missing data' and densify the matrix, which is irrelevant for non-count data
tools:
- name: ALRA
  role: Primary imputation method being benchmarked; run via R package with randomized
    SVD (rsvd, k=48, q=10), optionally with rpca-mkl acceleration via the use.mkl
    flag
  repo: github.com/KlugerLab/ALRA
- name: MAGIC
  role: Baseline imputation method timed for comparison; one of the two fastest methods
    (~40 min on 50,000 cells)
- name: DCA
  role: Baseline imputation method timed for comparison; takes ~4 h on 50,000 cells
- name: SAVER
  role: Baseline imputation method timed for comparison with 'do.fast' set to true;
    exceeds 10 h at 10,000 cells
- name: scImpute
  role: Baseline imputation method timed for comparison; exceeds 10 h at 10,000 cells
- name: rpca-mkl
  role: Optional MKL-accelerated randomized SVD backend that speeds up ALRA via ALRA's
    use.mkl flag
  repo: github.com/KlugerLab/rpca-mkl
- name: Seurat
  role: Single-cell R toolkit in which ALRA is integrated as RunALRA() (Seurat v3.0)
  repo: github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:b48339c5c791da3c16fec6aed5bd230edf816f72f80a3a5003b8724d86e2c849
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALRA
  - DCA
  - MAGIC
  - SAVER
  - Seurat
  - rpca-mkl
  - scImpute
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- imputation-method-benchmark-comparison
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# imputation-method-comparison

## Summary

Benchmarks the wall-clock runtime and scaling behavior of scRNA-seq imputation methods (ALRA, MAGIC, DCA, scImpute, SAVER) on progressively larger cell-count subsets, using the Hrvatin et al. mouse visual cortex dataset as the testbed.

## When to use

Use when comparing runtime/scaling of scRNA-seq imputation methods on large datasets (tens of thousands of cells), e.g. to decide which imputer is tractable for >10,000-cell matrices on a single core.

## When NOT to use

- When the goal is to evaluate imputation accuracy (e.g., preserving true biological zeros) rather than computational runtime — ALRA's thresholding-based zero preservation is a separate capability not measured here
- When the dataset is small (fewer than ~10,000 cells) — all five methods are tractable, so runtime benchmarking does not discriminate between them
- When the input is not a scRNA-seq expression matrix — DCA and MAGIC treat all zeros as 'missing data' and densify the matrix, which is irrelevant for non-count data

## Inputs

- Normalized Hrvatin et al. (2018) mouse visual cortex scRNA-seq expression matrix, cells as rows, genes as columns (19,155 genes x 65,539 cells after gene filtering)
- Gene filter: exclude genes with mean expression < 0.00003 or non-zero expression in fewer than 4 cells
- Random cell subsets of 5,000, 10,000, 20,000, 30,000, 40,000, and 50,000 cells

## Outputs

- Wall-clock runtimes for ALRA, MAGIC, DCA, scImpute, and SAVER at each subset size
- runtime_table.csv: runtime-vs-cell-count table across all five methods and six subset sizes

## How to apply

Load the Hrvatin et al. (2018) mouse visual cortex expression matrix filtered to 19,155 genes and 65,539 cells, excluding genes with mean expression < 0.00003 or non-zero expression in fewer than 4 cells. Sample random subsets of 5,000, 10,000, 20,000, 30,000, 40,000, and 50,000 cells. Run each method restricted to a single core with default parameters, recording wall-clock time: ALRA via randomized SVD from the rsvd package (k=48 on full Hrvatin data, q=10 power iterations, cells as rows and genes as columns), MAGIC and DCA with defaults, scImpute with defaults, and SAVER with its 'do.fast' option set to true. Assemble all runtimes into a runtime-vs-cell-count table saved as runtime_table.csv. Validate that the table reproduces the reported pattern: ALRA and MAGIC are fastest (~40 min on 50,000 cells), DCA takes ~4 h on 50,000 cells, while scImpute and SAVER exceed 10 h at 10,000 cells and are not attempted on larger matrices.

## Related tools

- **ALRA** (Primary imputation method being benchmarked; run via R package with randomized SVD (rsvd, k=48, q=10), optionally with rpca-mkl acceleration via the use.mkl flag) — github.com/KlugerLab/ALRA
- **MAGIC** (Baseline imputation method timed for comparison; one of the two fastest methods (~40 min on 50,000 cells))
- **DCA** (Baseline imputation method timed for comparison; takes ~4 h on 50,000 cells)
- **SAVER** (Baseline imputation method timed for comparison with 'do.fast' set to true; exceeds 10 h at 10,000 cells)
- **scImpute** (Baseline imputation method timed for comparison; exceeds 10 h at 10,000 cells)
- **rpca-mkl** (Optional MKL-accelerated randomized SVD backend that speeds up ALRA via ALRA's use.mkl flag) — github.com/KlugerLab/rpca-mkl
- **Seurat** (Single-cell R toolkit in which ALRA is integrated as RunALRA() (Seurat v3.0)) — github.com/satijalab/seurat

## Examples

```
library(ALRA); result.completed <- alra(A_norm)  # A_norm: normalized Hrvatin subset matrix, cells as rows
```

## Evaluation signals

- runtime_table.csv contains runtime entries for all five methods across all six subset sizes (5,000 to 50,000 cells)
- ALRA and MAGIC runtimes at 50,000 cells are roughly 40 minutes
- DCA runtime at 50,000 cells is approximately 4 hours
- scImpute and SAVER runtimes exceed 10 hours at 10,000 cells and no larger-subset runs exist for them
- All methods ran under identical constraints (single core, default parameters) so timings are comparable

## Limitations

- scImpute and SAVER are extremely slow on large datasets and were not run beyond 10,000 cells, so their scaling beyond that size is unmeasured
- rpca-mkl is under development, is not on CRAN, and its pre-compiled MKL libraries only support Linux and OS X (not Windows); the out-of-core functionality is untested
- ALRA expects a normalized matrix with cells as rows and genes as columns; k may need manual inspection since the automatic choice is based on singular-value spacings
- All timings reflect single-core execution; multi-core or MKL-accelerated runs (e.g., ALRA with use.mkl=TRUE) will shift absolute runtimes

## Evidence

- [intro] Speed motivation from the article's scope statement: "it should be able to process, in a reasonable time, large-scale scRNA-seq datasets that contain hundreds of thousands to millions of cells"
- [intro] Slow methods motivates runtime comparison: "SAVER and scImpute may be extremely slow on scRNA-seq datasets with over 100,000 cells"
- [readme] ALRA algorithmic core timed in the benchmark: "ALRA first computes its rank-k approximation using randomized SVD. Next, each row (gene) is thresholded by the magnitude of the most negative value of that gene"
- [readme] Input orientation requirement for ALRA runs: "Please be sure to pass ALRA a matrix where the cells are rows and genes are columns."
- [readme] rsvd prerequisite for the ALRA runtime used in the benchmark: "The only prerequisite for ALRA is installation of the randomized SVD package RSVD which can be installed as `install.packages('rsvd')`."
