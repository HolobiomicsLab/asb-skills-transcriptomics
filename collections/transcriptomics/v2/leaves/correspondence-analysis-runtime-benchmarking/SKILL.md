---
name: correspondence-analysis-runtime-benchmarking
description: Use when evaluating whether CA is computationally feasible as a PCA substitute
  in an scRNAseq pipeline, or when justifying a dimension-reduction method choice
  on wall-clock runtime grounds for datasets with tens of thousands of cells.
when_to_use_negative:
- Timing on dense or non-count data where CA's chi-squared residual decomposition
  is not the intended decomposition — the benchmark specifically targets single-cell
  count matrices
- Benchmarking the five CA adaptations (e.g. CA-FT) rather than standard CA versus
  glmPCA — those require the clustering-accuracy benchmarking workflow instead
- Datasets cannot be down-selected to a standardized feature count (e.g. 1500 features),
  so per-dataset runtimes are not comparable across methods
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: corral
  role: Runs standard CA via irlba-based partial SVD on the down-selected count matrices;
    timing target of the benchmark
  repo: github.com/laurenhsu1/corral
- name: glmPCA
  role: Comparison method timed on the same down-selected matrices to quantify the
    runtime gap versus CA
- name: R
  role: Language in which the benchmark and timing measurements are implemented
- name: Bioconductor
  role: Framework through which corral interfaces with SingleCellExperiment classes
    during the benchmark
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:6304991fb0da6addd5967eda944f98e32b4a0a52a0cdc0604704735cc0b81cda
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - corral
  - glmPCA
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# correspondence-analysis-runtime-benchmarking

## Summary

A wall-clock timing benchmark comparing standard CA (corral, irlba-based partial SVD) against glmPCA on down-selected scRNAseq count matrices (1500 features) across ten benchmark datasets, quantifying the runtime advantage of CA for large single-cell inputs.

## When to use

Use when evaluating whether CA is computationally feasible as a PCA substitute in an scRNAseq pipeline, or when justifying a dimension-reduction method choice on wall-clock runtime grounds for datasets with tens of thousands of cells.

## When NOT to use

- Timing on dense or non-count data where CA's chi-squared residual decomposition is not the intended decomposition — the benchmark specifically targets single-cell count matrices
- Benchmarking the five CA adaptations (e.g. CA-FT) rather than standard CA versus glmPCA — those require the clustering-accuracy benchmarking workflow instead
- Datasets cannot be down-selected to a standardized feature count (e.g. 1500 features), so per-dataset runtimes are not comparable across methods

## Inputs

- Ten benchmark scRNAseq count datasets (Zhengmix4eq, SCMixology, and related benchmarks) with original-publication cell labels
- Count matrices down-selected to 1500 features (e.g. 1500 features x ~22,000 cells)

## Outputs

- runtime_comparison.csv reporting runtime per method (corral CA vs glmPCA) per dataset
- Elapsed wall-clock times for corral and glmPCA on each down-selected matrix

## How to apply

Load the ten benchmark single-cell count datasets (Zhengmix4eq, SCMixology, and related benchmarks carrying original-publication cell labels) and down-select each to 1500 features so the timing test is standardized across matrices of differing gene counts. Run standard CA via the corral R/Bioconductor package, which uses an irlba-based partial SVD of the chi-squared (Pearson) residual matrix, and record elapsed wall-clock time per dataset. Run glmPCA on the identical down-selected matrices (e.g. 1500 features x ~22,000 cells) and record its wall-clock time under the same conditions. Compile per-method, per-dataset timings into a runtime_comparison.csv table. The reproduction is validated if the pattern matches the paper's report: corral CA completes in under a minute even on datasets with over 20,000 cells, while glmPCA requires over an hour on the equivalent input.

## Related tools

- **corral** (Runs standard CA via irlba-based partial SVD on the down-selected count matrices; timing target of the benchmark) — github.com/laurenhsu1/corral
- **glmPCA** (Comparison method timed on the same down-selected matrices to quantify the runtime gap versus CA)
- **R** (Language in which the benchmark and timing measurements are implemented)
- **Bioconductor** (Framework through which corral interfaces with SingleCellExperiment classes during the benchmark)

## Examples

```
library(corral); sce <- downselect(sce, nfeatures = 1500); system.time(sce <- corral(sce, whichmat = "counts"))
```

## Evaluation signals

- corral CA completes each dataset in under one minute, including datasets with over 20,000 cells
- glmPCA takes over one hour on the equivalent ~22,000-cell, 1500-feature input matrix
- runtime_comparison.csv contains one timing row per method per dataset, covering all ten benchmark datasets
- Feature counts are identical across the two methods per dataset (1500 features) so timings are directly comparable
- The runtime ordering (CA faster than glmPCA) is consistent across all datasets, not just the largest ones

## Limitations

- No tool or package versions reported for corral or dependencies, so exact runtime reproduction may vary with software versions
- Benchmark datasets such as Zhengmix and SCMixology are simpler, well-defined cell-cluster mixtures and by design simpler than data from complex tissues, so runtime on very large complex-tissue matrices is extrapolated
- No accession IDs or links for the SCMixology and Zhengmix benchmarking datasets are provided in the discussion, requiring retrieval from original publications

## Evidence

- [results] Standard CA (corral) consistently runs in under a minute even for datasets with over 20,000 cells, while glmPCA scales less favorably: [evidence span withheld — anchor status: not_found]
- [discussion] We implemented CA, CA-FT, and other variations that adjust for overdispersion of scRNAseq data in the R/Bioconductor package corral: "We implemented CA, CA-FT, and other variations that adjust for overdispersion of scRNAseq data in the R/Bioconductor package corral (including documentation, tutorials, vignettes)"
- [readme] corral is a package to perform correspondence analysis on count data, particularly for single-cell (e.g., RNA-seq).: "corral is a package to perform correspondence analysis on count data, particularly for single-cell (e.g., RNA-seq)."
- [intro] an R/Bioconductor package that interfaces directly with Bioconductor classes (including SingleCellExperiment): "We implemented standard and adapted CA for scRNAseq in corral, an R/Bioconductor package that interfaces directly with Bioconductor classes (including SingleCellExperiment)"
