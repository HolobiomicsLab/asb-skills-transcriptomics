---
name: scrna-seq-subset-sampling
description: Use when benchmarking the wall-clock runtime of scRNA-seq imputation
  methods (ALRA, MAGIC, DCA, scImpute, SAVER) as a function of cell count, requiring
  random subsets of a large filtered expression matrix.
when_to_use_negative:
- The goal is imputation accuracy rather than runtime scaling — subsets would confound
  accuracy comparisons that should use the full annotated dataset
- The input matrix is already small (e.g., under ~10,000 cells), where subsetting
  adds no meaningful runtime-scaling information
- Genes have not yet been filtered (mean expression >= 0.00003 and non-zero in >=
  4 cells checks not applied), so runtime reflects unfiltered noise genes
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: ALRA
  role: Fastest benchmarked imputation method (with MAGIC); run on each cell subset
    with randomized SVD (rsvd package, k=48 for full Hrvatin data, q=10 power iterations)
    on a single core to record wall-clock time
  repo: https://github.com/KlugerLab/ALRA
- name: MAGIC
  role: Baseline imputation method timed on each subset for the runtime-vs-cell-count
    comparison
- name: DCA
  role: Baseline imputation method timed on each subset; took about 4 h on 50,000
    cells
- name: SAVER
  role: Baseline imputation method timed with its 'do.fast' option set to true; exceeded
    10 h at 10,000 cells so was not attempted on larger matrices
- name: scImpute
  role: Baseline imputation method timed on each subset; exceeded 10 h at 10,000 cells
    so was not attempted on larger matrices
- name: rsvd
  role: 'R package providing the randomized SVD used by ALRA on each sampled subset
    (prerequisite: install.packages(''rsvd''))'
  repo: https://github.com/KlugerLab/ALRA
- name: rpca-mkl
  role: Optional MKL-accelerated randomized SVD backend usable via ALRA's use.mkl
    flag for speedups during benchmarking
  repo: https://github.com/KlugerLab/rpca-mkl
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:47fe1245578561c7249e73c4b7bcc9f57b8eee28db9508567f5e1f91f3108cd4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KlugerLab/ALRA
  tools:
  - ALRA
  - DCA
  - MAGIC
  - SAVER
  - rpca-mkl
  - rsvd
  - scImpute
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# scrna-seq-subset-sampling

## Summary

Sampling random cell subsets (e.g., 5,000–50,000 cells) from a large filtered scRNA-seq expression matrix, such as the Hrvatin et al. mouse visual cortex dataset, to benchmark how imputation method runtime scales with cell count.

## When to use

Use when benchmarking the wall-clock runtime of scRNA-seq imputation methods (ALRA, MAGIC, DCA, scImpute, SAVER) as a function of cell count, requiring random subsets of a large filtered expression matrix.

## When NOT to use

- The goal is imputation accuracy rather than runtime scaling — subsets would confound accuracy comparisons that should use the full annotated dataset
- The input matrix is already small (e.g., under ~10,000 cells), where subsetting adds no meaningful runtime-scaling information
- Genes have not yet been filtered (mean expression >= 0.00003 and non-zero in >= 4 cells checks not applied), so runtime reflects unfiltered noise genes

## Inputs

- Filtered Hrvatin et al. (2018) mouse visual cortex scRNA-seq expression matrix (19155 genes x 65539 cells; genes with mean expression < 0.00003 or non-zero expression in fewer than 4 cells excluded)
- Subset size specification (e.g., 5000, 10000, 20000, 30000, 40000, 50000 cells)

## Outputs

- Six expression-matrix subsets with 5000-50000 cells sampled from the full filtered matrix
- runtime_table.csv: runtime-vs-cell-count table of wall-clock times for ALRA, MAGIC, DCA, scImpute, and SAVER on each subset

## How to apply

Start from the filtered Hrvatin et al. (2018) mouse visual cortex expression matrix (19,155 genes × 65,539 cells, after excluding genes with mean expression < 0.00003 or non-zero expression in fewer than 4 cells). Draw random subsets of 5,000, 10,000, 20,000, 30,000, 40,000, and 50,000 cells from this matrix so that all methods are evaluated on identical inputs at each size. Run each imputation method (ALRA with k=48, randomized SVD via the rsvd package and q=10 power iterations; MAGIC, DCA, scImpute, and SAVER with do.fast=TRUE at default parameters) restricted to a single core, recording wall-clock time per subset. Assemble the runtimes into a runtime-vs-cell-count table; the rationale is that only subset sampling at matched sizes makes the scaling comparison fair across methods whose cost grows very differently with cell number.

## Related tools

- **ALRA** (Fastest benchmarked imputation method (with MAGIC); run on each cell subset with randomized SVD (rsvd package, k=48 for full Hrvatin data, q=10 power iterations) on a single core to record wall-clock time) — https://github.com/KlugerLab/ALRA
- **MAGIC** (Baseline imputation method timed on each subset for the runtime-vs-cell-count comparison)
- **DCA** (Baseline imputation method timed on each subset; took about 4 h on 50,000 cells)
- **SAVER** (Baseline imputation method timed with its 'do.fast' option set to true; exceeded 10 h at 10,000 cells so was not attempted on larger matrices)
- **scImpute** (Baseline imputation method timed on each subset; exceeded 10 h at 10,000 cells so was not attempted on larger matrices)
- **rsvd** (R package providing the randomized SVD used by ALRA on each sampled subset (prerequisite: install.packages('rsvd'))) — https://github.com/KlugerLab/ALRA
- **rpca-mkl** (Optional MKL-accelerated randomized SVD backend usable via ALRA's use.mkl flag for speedups during benchmarking) — https://github.com/KlugerLab/rpca-mkl

## Examples

```
library(ALRA); result.completed <- alra(A_norm, k=48)
```

## Evaluation signals

- runtime_table.csv contains wall-clock times for all five methods (ALRA, MAGIC, DCA, scImpute, SAVER) across all six subset sizes (5000–50000 cells)
- ALRA and MAGIC take roughly 40 minutes on the 50,000-cell Hrvatin subset
- DCA takes about 4 hours on the 50,000-cell subset
- scImpute and SAVER exceed 10 hours on the 10,000-cell subset and are not attempted on larger matrices
- Each sampled subset is drawn from the same filtered 19,155-gene matrix so methods are compared on identical inputs at each size

## Limitations

- scImpute and SAVER are extremely slow on datasets with over 100,000 cells; runtimes over 10 h at 10,000 cells preclude attempting larger subsets
- DCA and MAGIC treat all zeros as missing data and output a matrix in which every gene is expressed in every cell, unlike ALRA, SAVER, and scImpute which preserve biological zeros — runtime comparison alone does not reflect output quality differences
- ALRA's default rsvd-based path differs in speed from the rpca-mkl (MKL) accelerated path; single-core timing must hold the backend fixed

## Evidence

- [intro] SAVER and scImpute may be extremely slow on scRNA-seq datasets with over 100,000 cells: "SAVER and scImpute may be extremely slow on scRNA-seq datasets with over 100,000 cells"
- [intro] Some methods such as DCA and MAGIC treat all zeros as missing data and output a matrix in which every gene is expressed in every cell: "Some methods such as DCA and MAGIC treat all zeros as “missing data” and output a matrix in which every gene is expressed in every cell"
- [intro] The first step of ALRA is to compute a low-rank approximation of the observed matrix using the singular vector decomposition (SVD): "The first step of ALRA is to compute a low-rank approximation of the observed matrix using the singular vector decomposition (SVD)"
- [readme] The only prerequisite for ALRA is installation of the randomized SVD package RSVD which can be installed as install.packages('rsvd'): "The only prerequisite for ALRA is installation of the randomized SVD package RSVD which can be installed as `install.packages('rsvd')`."
- [readme] Please be sure to pass ALRA a matrix where the cells are rows and genes are columns.: "Please be sure to pass ALRA a matrix where the cells are rows and genes are columns."
