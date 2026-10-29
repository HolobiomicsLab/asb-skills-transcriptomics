---
name: r-single-cell-workflow-scripting
description: Use when large-scale scRNA-seq count data must be clustered in R and
  full-data clustering is computationally prohibitive, or when clustering must be
  run on subsampled cells and remaining cells classified.
when_to_use_negative:
- The dataset is small enough that full-data clustering (subsampling=F) is tractable
  and subsampling bias is unnecessary
- The task requires raw gene-expression-space clustering rather than clustering in
  the Spearman-correlation projected feature space
- The input is already a clustering result or NMI table and no R clustering workflow
  needs to be executed
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: sscClust
  role: R package implementing the SSCC subsampling-clustering-classification pipeline
    (ssc.run) with Spearman-correlation feature projection
  repo: https://github.com/Japrin/sscClust
- name: sscVis
  role: Required dependency package that must be installed before sscClust
  repo: https://github.com/Japrin/sscClust
- name: R
  role: Execution environment; recommended to link against optimized BLAS (ATLAS,
    MKL) for performance
- name: SC3
  role: Base clustering algorithm that can be plugged into the SSCC subsampling framework
- name: SIMLR
  role: Benchmarked base clustering algorithm usable within the workflow
  repo: https://github.com/BatzoglouLabSU/SIMLR
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2018.10.003
    title: 'SSCC: A Novel Computational Framework for Rapid and Accurate Clustering
      Large-Scale Single Cell RNA-Seq Data'
  source_content_hash: sha256:d41761475dfd7708a8611368c4e87e1e8e7af29ecef0059e01096b2723c424a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Japrin/sscClust
  tools:
  - CIDR
  - R
  - SC3
  - SIMLR
  - SSCC
  - Seurat
  - ZIFA
  - sscClust
  - sscVis
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# r-single-cell-workflow-scripting

## Summary

Script scRNA-seq clustering workflows in R using the sscClust package, which implements the SSCC subsampling-clustering-classification framework with Spearman-correlation feature projection to cluster large-scale single-cell data at O(n) complexity.

## When to use

Use when large-scale scRNA-seq count data must be clustered in R and full-data clustering is computationally prohibitive, or when clustering must be run on subsampled cells and remaining cells classified.

## When NOT to use

- The dataset is small enough that full-data clustering (subsampling=F) is tractable and subsampling bias is unnecessary
- The task requires raw gene-expression-space clustering rather than clustering in the Spearman-correlation projected feature space
- The input is already a clustering result or NMI table and no R clustering workflow needs to be executed

## Inputs

- scRNA-seq gene expression/count matrix in R (SingleCellExperiment object, e.g. sce.Pollen shipped with sscClust)
- Subsampling fraction parameter (e.g. sub.frac=0.8, or 0.1 and 0.5 for consistency tests)
- Target cluster number / k.batch parameter (e.g. k.batch=11)

## Outputs

- Cluster assignment table for all cells (subsampled + classified), e.g. sscc_clusters.csv
- NMI consistency table across subsampling rates, e.g. nmi_consistency_table.csv
- ssc.run output SingleCellExperiment with clustering results on reduced data

## How to apply

Load the raw count data in R and build a gene expression matrix (SingleCellExperiment object, e.g. the packaged sce.Pollen) using sscClust's data-preparation utilities, after installing sscVis and sscClust via devtools::install_github. Decide whether to cluster all cells (subsampling=F) or subsample (subsampling=T, sub.frac e.g. 0.8), then invoke the ssc.run pipeline, which identifies variable genes, performs dimension reduction, and clusters the reduced data. For the SSCC variant, project cells into a Spearman-correlation feature space before clustering the subsample, then classify unsampled cells by nearest subsampled cells, reducing complexity from O(n^2) to O(n). R should be linked to an optimized BLAS library (ATLAS, MKL) since computation is expensive on large datasets. Quantify consistency between different subsampling fractions by computing NMI between cluster labels (e.g., 10% vs 50% subsamples) across algorithms such as SC3, Seurat, SIMLR, ZIFA, and CIDR.

## Related tools

- **sscClust** (R package implementing the SSCC subsampling-clustering-classification pipeline (ssc.run) with Spearman-correlation feature projection) — https://github.com/Japrin/sscClust
- **sscVis** (Required dependency package that must be installed before sscClust) — https://github.com/Japrin/sscClust
- **R** (Execution environment; recommended to link against optimized BLAS (ATLAS, MKL) for performance)
- **SC3** (Base clustering algorithm that can be plugged into the SSCC subsampling framework)
- **SIMLR** (Benchmarked base clustering algorithm usable within the workflow) — https://github.com/BatzoglouLabSU/SIMLR

## Examples

```
devtools::install_github("Japrin/sscVis"); devtools::install_github("Japrin/sscClust"); data("sce.Pollen"); sce.sub <- ssc.run(sce.Pollen, subsampling=T, sub.frac=0.8, k.batch=11)
```

## Evaluation signals

- ssc.run completes on a packaged example (e.g. sce.Pollen) and returns clustering labels for every cell, including classified unsampled cells
- Switching subsampling=T with sub.frac=0.8 runs without error and yields labels for cells outside the subsample
- NMI consistency between 10% and 50% subsampled clusterings exceeds ~0.5 for SSCC (vs much smaller values for plain SCC on Kolodziejczyk)
- Runtime and memory should improve relative to full SC3 clustering: reported 50-fold acceleration, 66% memory usage, and 20% accuracy improvement

## Limitations

- Plain SCC subsampling introduces biases that decrease clustering accuracy and robustness; SSCC's Spearman feature construction mitigates but does not eliminate dependence on the subsample
- Computation becomes expensive as datasets grow; R not linked to optimized BLAS degrades performance
- The paper's evaluation section lacks dataset accessions, tool version numbers, and explicit SSCC parameter/implementation details (e.g. how Spearman correlations are computed)

## Evidence

- [abstract] An R implementation of SSCC is available at https://github.com/Japrin/sscClust: "An R implementation of SSCC is available at https://github.com/Japrin/sscClust"
- [readme] ssc.run with subsampling: [evidence span withheld — anchor status: external_doc]
- [readme] BLAS recommendation: "Calculation will become expensive when the dataset become large, so we stongly recommand use R which linked to optimized BLAS library"
- [intro] SSCC complexity reduction: "SSCC can reduce the computational complexity from O(n2) to O(n) while maintaining high clustering"
- [discussion] SCC subsampling bias: "the SCC strategy suffers from biases introduced by subsampling which may greatly decrease the clustering accuracy and robustness"
