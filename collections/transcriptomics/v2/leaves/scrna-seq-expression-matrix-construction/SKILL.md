---
name: scrna-seq-expression-matrix-construction
description: Use when raw scRNA-seq count data must be turned into an R gene expression
  matrix as input to SSCC/sscClust subsampling-based clustering of large-scale datasets
  (e.g. 68k PBMC-scale).
when_to_use_negative:
- Input is already a processed feature table or dimension-reduced embedding rather
  than raw count data needing an expression matrix.
- The dataset is small enough that direct clustering of all cells is feasible, making
  the subsampling-classification machinery unnecessary overhead.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: sscClust
  role: R package whose data-preparation utilities build the gene expression matrix
    and implement subsampling, Spearman-correlation feature projection, clustering,
    and classification
  repo: https://github.com/Japrin/sscClust
- name: SSCC
  role: The Spearman subsampling-clustering-classification framework that consumes
    the constructed expression matrix and reduces complexity from O(n^2) to O(n)
  repo: https://github.com/Japrin/sscClust
- name: R
  role: Runtime environment; an optimized-BLAS-linked build (ATLAS, MKL, Microsoft
    R Open) is recommended because dataset-scale computation becomes expensive
  repo: https://github.com/Japrin/sscClust
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2018.10.003
    title: 'SSCC: A Novel Computational Framework for Rapid and Accurate Clustering
      Large-Scale Single Cell RNA-Seq Data'
  source_content_hash: sha256:8841127abf222726f36e9f3b9feb77aa71d671f9d105dae8370bae7038ff211a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Japrin/sscClust
  tools:
  - R
  - SSCC
  - sscClust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# scrna-seq-expression-matrix-construction

## Summary

Load single-cell RNA-seq count data into R and build the gene expression matrix with the sscClust package's data-preparation utilities, forming the entry step (Step 1) of the SSCC subsampling-clustering-classification pipeline that later works in a Spearman-correlation feature space.

## When to use

Use when raw scRNA-seq count data must be turned into an R gene expression matrix as input to SSCC/sscClust subsampling-based clustering of large-scale datasets (e.g. 68k PBMC-scale).

## When NOT to use

- Input is already a processed feature table or dimension-reduced embedding rather than raw count data needing an expression matrix.
- The dataset is small enough that direct clustering of all cells is feasible, making the subsampling-classification machinery unnecessary overhead.

## Inputs

- single-cell RNA-seq count data (e.g. a SingleCellExperiment object such as sce.Pollen shipped with sscClust)
- gene x cell expression matrix to be built in R via sscClust data-preparation utilities

## Outputs

- R gene expression matrix ready for subsampling and Spearman-correlation feature construction
- downstream cluster assignment table (e.g. sscc_clusters.csv) for all cells

## How to apply

Load the single-cell RNA-seq count matrix into R and construct the gene expression matrix using sscClust's data-preparation utilities, which also cover variable-gene identification and dimension reduction before clustering. Use an R build linked to an optimized BLAS library (ATLAS, MKL; Microsoft R Open on Windows), since the README warns computation 'will become expensive when the dataset become large', and install the required sscVis package first. The resulting matrix is then randomly subsampled (e.g. sub.frac = 0.8) so that subsampled cells are clustered in a Spearman-correlation feature space while unsampled cells are later classified by nearest subsampled neighbors, reducing complexity from O(n^2) to O(n). The decision point after matrix construction is whether to run clustering on all cells (subsampling = F) or on a subsample plus classification (subsampling = T).

## Related tools

- **sscClust** (R package whose data-preparation utilities build the gene expression matrix and implement subsampling, Spearman-correlation feature projection, clustering, and classification) — https://github.com/Japrin/sscClust
- **SSCC** (The Spearman subsampling-clustering-classification framework that consumes the constructed expression matrix and reduces complexity from O(n^2) to O(n)) — https://github.com/Japrin/sscClust
- **R** (Runtime environment; an optimized-BLAS-linked build (ATLAS, MKL, Microsoft R Open) is recommended because dataset-scale computation becomes expensive) — https://github.com/Japrin/sscClust

## Examples

```
devtools::install_github("Japrin/sscVis"); devtools::install_github("Japrin/sscClust"); data("sce.Pollen"); sce.sub <- ssc.run(sce.Pollen, subsampling=T, sub.frac = 0.8, k.batch=11)
```

## Evaluation signals

- The built expression matrix contains all cells and genes from the source count data with no cells dropped before the subsampling step.
- Downstream clustering runs in the constructed Spearman-correlation feature space, not on raw gene expression, distinguishing SFCC/SSCC from plain SCC.
- Every cell receives a final cluster label: subsampled cells via base clustering, unsampled cells via nearest-neighbor classification, combined in the output assignment table.
- Runtime scales linearly (O(n)) on large datasets, consistent with the reported 50-fold acceleration over SC3 at ~66% memory usage.
- Package installs cleanly, including the required sscVis dependency, on an optimized-BLAS R build.

## Limitations

- The article section provides no explicit parameter settings or implementation details for how Spearman correlations are computed.
- No dataset accessions or identifiers are given for the real scRNA-seq evaluation datasets in this section.
- Computation becomes expensive on large datasets unless R is linked to an optimized BLAS library; sscVis must be installed alongside sscClust.

## Evidence

- [abstract] An R implementation of SSCC is available at https://github.com/Japrin/sscClust: "An R implementation of SSCC is available at https://github.com/Japrin/sscClust"
- [discussion] with Spearman correlations as the feature engineering and projecting methods, we formulate a framework named as SSCC: "with Spearman correlations as the feature engineering and projecting methods, we formulate a framework named as SSCC"
- [readme] package implement mutilple functionalities which are basic procedures in single cell RNAseq data analysis, including variable genes identification, dimension reduction: "package implement mutilple functionalities which are basic procedures in single cell RNAseq data analysis, including variable genes identification, dimension reduction"
- [readme] Calculation will become expensive when the dataset become large, so we stongly recommand use R which linked to optimized BLAS library: "Calculation will become expensive when the dataset become large, so we stongly recommand use R which linked to optimized BLAS library"
