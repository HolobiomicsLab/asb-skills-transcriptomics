---
name: r-package-function-invocation
description: Use when an R package exposes a documented function signature (e.g.,
  RCC_clus(x, config_file)) and the task requires loading data, calling that function
  with a configuration, and using its output objects downstream.
when_to_use_negative:
- The data is not in R and no R package API exists for the analysis — prefer native
  tooling rather than forcing an R invocation wrapper.
- The task requires a method with no documented function signature or config schema
  (e.g., ad hoc scripts), since argument passing cannot be verified against documentation.
- Input is single-cell data requiring integration first — the article notes RCC should
  be combined with scanorama for integration of multiple datasets before clustering.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3170
tools:
- name: RCC
  role: Target R package whose documented functions (RCC_clus, clusterAnnotation,
    clusterAttr, geneMarkers, ssgsea, survival, trackingPlot) are invoked with positional
    arguments
  repo: https://github.com/MSCTR/RecursiveConsensusClustering
- name: R
  role: Runtime environment in which the package functions are loaded and called
- name: devtools
  role: Used to install the package from GitHub via devtools::install_github
- name: mclust
  role: Provides adjusted Rand index computation to validate clustering output of
    invoked functions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-67016-3
    title: Recursive Consensus Clustering for novel subtype discovery from transcriptome
      data
  source_content_hash: sha256:412661c4ff8309e4d5e07605e5474fba4214025888ff0d840667f670953498d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MSCTR/RecursiveConsensusClustering
  tools:
  - ConsensusClusterPlus
  - R
  - RCC
  - devtools
  - mclust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# r-package-function-invocation

## Summary

Invoke R package functions by passing required positional arguments (e.g., x and config_file) and capturing their returned objects, as exemplified by the RCC package's RCC_clus(x, config_file) entry point for recursive consensus clustering of TCGA pan-cancer transcriptome data.

## When to use

Use when an R package exposes a documented function signature (e.g., RCC_clus(x, config_file)) and the task requires loading data, calling that function with a configuration, and using its output objects downstream.

## When NOT to use

- The data is not in R and no R package API exists for the analysis — prefer native tooling rather than forcing an R invocation wrapper.
- The task requires a method with no documented function signature or config schema (e.g., ad hoc scripts), since argument passing cannot be verified against documentation.
- Input is single-cell data requiring integration first — the article notes RCC should be combined with scanorama for integration of multiple datasets before clustering.

## Inputs

- R expression matrix object (e.g., TCGA pan-cancer transcriptome data) assigned to x
- config_file controlling RCC parameters (parallel processes, pItem/pFeature, repeat count)
- infoFile / attribute metadata files for downstream functions like clusterAttr and survival
- Installed R package and dependencies (clv, fields, matrixStats, data.table, cluster, clue, circlize, gdata, pforeach, scran, ComplexHeatmap)

## Outputs

- Returned RCC result object with cluster assignments (input to clusterAnnotation, clusterAttr, ssgsea, survival)
- Clustering output files written to OutputDir
- Derived tables/plots such as ARI-vs-runtime comparison CSVs and tracking plots

## How to apply

Load the input data object and any auxiliary files (e.g., an expression matrix and a config_file) into the R session before calling the function. Call the documented entry point positionally, e.g. RCC_clus(x, config_file), where x is the expression dataset and config_file controls parameters such as number of parallel processes, pItem/pFeature subsampling fractions, and repeat count (the parallel ablation used 8 processes with 100 repeats vs a single process with default pItem/pFeature and 800 repeats). Install the package and its dependencies first — devtools::install_github('MSCTR/RecursiveConsensusClustering') plus CRAN packages (clv, fields, matrixStats, data.table, cluster, clue, circlize, gdata, pforeach) and BiocManager packages (scran, ComplexHeatmap). Capture the returned object and pass it to follow-on functions from the same package (clusterAnnotation, clusterAttr, geneMarkers, ssgsea, survival, trackingPlot), each with their documented argument lists. Verify correctness by checking that returned cluster assignments can be scored against a reference partition (e.g., adjusted Rand index via mclust) and that outputs land in the specified OutputDir.

## Related tools

- **RCC** (Target R package whose documented functions (RCC_clus, clusterAnnotation, clusterAttr, geneMarkers, ssgsea, survival, trackingPlot) are invoked with positional arguments) — https://github.com/MSCTR/RecursiveConsensusClustering
- **R** (Runtime environment in which the package functions are loaded and called)
- **devtools** (Used to install the package from GitHub via devtools::install_github)
- **mclust** (Provides adjusted Rand index computation to validate clustering output of invoked functions)

## Examples

```
install.packages(c("devtools","clv","fields","matrixStats","data.table","cluster","clue","circlize","gdata")); devtools::install_github("MSCTR/RecursiveConsensusClustering"); res <- RCC_clus(x, config_file)
```

## Evaluation signals

- RCC_clus returns cluster assignments for every sample in x without error, and downstream functions accept the returned object.
- Ablation comparison reproduces the expected direction: 8-process/100-repeat variant-parameter run achieves higher ARI and lower wall-clock time than the single-process/800-repeat default run (Table S2).
- ARI against the reference partition can be computed (e.g., with mclust) and reported alongside processing time in a CSV comparison table.
- Output files appear in the OutputDir argument passed to each invoked function, matching the documented function signatures.
- Package installation completes: devtools::install_github('MSCTR/RecursiveConsensusClustering') plus the listed CRAN and BiocManager dependencies succeed.

## Limitations

- Computational time of RCC is a stated limitation of the algorithm and depends on the underlying base algorithm (consensus clustering with k-means).
- Analysis of large integrated datasets with millions of cells is described as the next step, not yet demonstrated.
- Parallel configuration (8 processes, variant pItem/pFeature, 100 repeats) trades off per-run repeats (100 vs 800) against ARI and runtime.

## Evidence

- [methods] RCC_clus: "RCC_clus(x, config_file)"
- [readme] install: "devtools::install_github("MSCTR/RecursiveConsensusClustering")"
- [results] ARI and runtime finding: [evidence span withheld — anchor status: not_found]
- [discussion] runtime limitation: "One of the limitations of our algorithm is computational time."
- [abstract] package purpose: "We have developed Recursive Consensus Clustering (RCC), an unsupervised clustering algorithm for novel subtype discovery from both bulk and single-cell datasets."
