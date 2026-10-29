---
name: single-core-compute-profiling
description: Use when comparing the computational scalability of imputation pipelines
  on large single-cell expression matrices (tens of thousands of cells) and needing
  a runtime-vs-cell-count profile under controlled single-core execution.
when_to_use_negative:
- The goal is imputation accuracy rather than computational benchmarking — accuracy
  comparisons require ground-truth validation, not timing runs
- Only a small dataset (under ~5000 cells) is available, so runtime scaling differences
  between methods cannot be resolved
- Parallel or multi-core execution is required — this skill's timings are only comparable
  under strict single-core restriction
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: ALRA
  role: Imputation method being benchmarked; run with default parameters (randomized
    SVD via rsvd, q=10 power iterations, k=48 for full Hrvatin data) on each cell
    subset
  repo: https://github.com/KlugerLab/ALRA
- name: rpca-mkl
  role: Optional MKL-based randomized SVD backend that speeds up ALRA; not a required
    package but can be enabled via the use.mkl flag
  repo: https://github.com/KlugerLab/rpca-mkl
- name: MAGIC
  role: Baseline imputation method benchmarked for runtime comparison on each subset
- name: DCA
  role: Baseline imputation method benchmarked for runtime comparison on each subset
- name: SAVER
  role: Baseline imputation method benchmarked with the 'do.fast' option set to true
    on each subset
- name: scImpute
  role: Baseline imputation method benchmarked for runtime comparison on each subset
- name: Seurat
  role: R toolkit in which ALRA is integrated (RunALRA); relevant if the benchmark
    matrix is handled as a Seurat object
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:b74555886630d98ca111ab24d704fe636b7d22ab27a9863ea13c44de09c3738e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KlugerLab/ALRA
  tools:
  - ALRA
  - DCA
  - MAGIC
  - SAVER
  - Seurat
  - rpca-mkl
  - scImpute
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# single-core-compute-profiling

## Summary

Benchmarks the wall-clock runtime of scRNA-seq imputation methods (ALRA, MAGIC, DCA, SAVER, scImpute) on random cell subsets of the Hrvatin et al. mouse visual cortex matrix, each restricted to a single core, to characterize how runtime scales with cell count.

## When to use

Use when comparing the computational scalability of imputation pipelines on large single-cell expression matrices (tens of thousands of cells) and needing a runtime-vs-cell-count profile under controlled single-core execution.

## When NOT to use

- The goal is imputation accuracy rather than computational benchmarking — accuracy comparisons require ground-truth validation, not timing runs
- Only a small dataset (under ~5000 cells) is available, so runtime scaling differences between methods cannot be resolved
- Parallel or multi-core execution is required — this skill's timings are only comparable under strict single-core restriction

## Inputs

- Hrvatin et al. (2018) mouse visual cortex scRNA-seq expression matrix (19155 genes x 65539 cells, filtered for genes with mean expression >= 0.00003 and non-zero expression in >= 4 cells)
- Random cell subsets of sizes 5000, 10000, 20000, 30000, 40000, and 50000 cells

## Outputs

- runtime_table.csv: wall-clock runtime for each of the five methods (ALRA, MAGIC, DCA, SAVER, scImpute) at each of the six subset sizes
- Per-run recorded wall-clock times from single-core execution

## How to apply

Load the Hrvatin et al. (2018) mouse visual cortex expression matrix filtered to 19155 genes and 65539 cells, excluding genes with mean expression below 0.00003 or non-zero expression in fewer than 4 cells. Sample random subsets of 5000, 10000, 20000, 30000, 40000, and 50000 cells. Run each method — ALRA with default parameters (k=48 for full Hrvatin data, randomized SVD via the rsvd package, q=10 power iterations), MAGIC, DCA, scImpute, and SAVER with its 'do.fast' option set to true — restricted to a single core, recording wall-clock time for each run. Assemble times into a runtime-vs-cell-count table (runtime_table.csv) so scaling behavior across methods can be compared directly. Restricting all methods to a single core is the key control that makes runtimes comparable across implementations.

## Related tools

- **ALRA** (Imputation method being benchmarked; run with default parameters (randomized SVD via rsvd, q=10 power iterations, k=48 for full Hrvatin data) on each cell subset) — https://github.com/KlugerLab/ALRA
- **rpca-mkl** (Optional MKL-based randomized SVD backend that speeds up ALRA; not a required package but can be enabled via the use.mkl flag) — https://github.com/KlugerLab/rpca-mkl
- **MAGIC** (Baseline imputation method benchmarked for runtime comparison on each subset)
- **DCA** (Baseline imputation method benchmarked for runtime comparison on each subset)
- **SAVER** (Baseline imputation method benchmarked with the 'do.fast' option set to true on each subset)
- **scImpute** (Baseline imputation method benchmarked for runtime comparison on each subset)
- **Seurat** (R toolkit in which ALRA is integrated (RunALRA); relevant if the benchmark matrix is handled as a Seurat object) — https://github.com/satijalab/seurat

## Examples

```
library(ALRA); result.completed <- alra(A_norm, k = 48); A_norm_completed <- result.completed[[3]]
```

## Evaluation signals

- runtime_table.csv contains recorded runtimes for all five methods at all six subset sizes (5000 through 50000 cells)
- ALRA and MAGIC take roughly 40 minutes on 50000 cells
- DCA takes about 4 hours on the 50000-cell subset
- scImpute and SAVER exceed 10 hours at 10000 cells and were not attempted on larger matrices
- All runs completed under single-core restriction so timings are directly comparable across methods

## Limitations

- scImpute and SAVER take over 10 hours on 10000 cells, so their runtime on larger subsets was not measured and must be extrapolated
- Timings reflect single-core execution; rpca-mkl (MKL-backed randomized SVD) can dramatically speed up ALRA but was not required for the default benchmark
- Runtime scaling is hardware- and implementation-dependent; the k=48 setting applies specifically to the full Hrvatin dataset

## Evidence

- [intro] SAVER and scImpute may be extremely slow on scRNA-seq datasets with over 100,000 cells: "SAVER and scImpute may be extremely slow on scRNA-seq datasets with over 100,000 cells"
- [readme] rsvd prerequisite for ALRA: "The only prerequisite for ALRA is installation of the randomized SVD package RSVD which can be installed as `install.packages('rsvd')`."
- [readme] ALRA algorithm overview: "ALRA first computes its rank-k approximation using randomized SVD. Next, each row (gene) is thresholded by the magnitude of the most negative value of that gene."
- [readme] rpca-mkl speedup flag: [evidence span withheld — anchor status: not_found]
- [intro] ALRA first step via SVD: "The first step of ALRA is to compute a low-rank approximation of the observed matrix using the singular vector decomposition (SVD)"
