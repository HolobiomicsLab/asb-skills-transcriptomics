---
name: single-cell-benchmark-dataset-handling
description: 'Use when a new or adapted scRNAseq method needs objective evaluation
  against known cell identities: benchmark datasets with published ground-truth labels
  and ARI-based cluster recovery scoring.'
when_to_use_negative:
- The analysis target is complex-tissue data rather than method benchmarking — the
  SCMixology and Zhengmix datasets comprise well-defined cell clusters and are by
  design simpler than data from complex tissues
- The cell-type labels are not from a published, high-quality source — unvalidated
  labels give an overly narrow benchmark standard since ARI is limited by the original
  cell type assignments
- The method's advantage is distinguishing sub-populations or rare cell types, which
  ARI against a reference label set may penalize rather than reward
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2229
tools:
- name: corral
  role: R/Bioconductor package used to run CA on benchmark counts and produce embeddings
    for NNGraph clustering; interfaces with SingleCellExperiment
  repo: github.com/laurenhsu1/corral
- name: R
  role: Language in which the benchmark loading, clustering, and ARI computation are
    executed
- name: Bioconductor
  role: Framework providing SingleCellExperiment classes for handling benchmark datasets
    and interfacing with corral
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:53f25b04e4c1a595b1934288f980708c9c1f2de5934d717c2498a5c13c00f26e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - corral
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# single-cell-benchmark-dataset-handling

## Summary

Loading and handling published single-cell RNA-seq benchmarking datasets with ground-truth cell-type labels (SCMixology from CellBench and Zhengmix from DuoClustering2018) for objective evaluation of dimension reduction and clustering methods via Adjusted Rand Index against annotated cell types.

## When to use

Use when a new or adapted scRNAseq method needs objective evaluation against known cell identities: benchmark datasets with published ground-truth labels and ARI-based cluster recovery scoring.

## When NOT to use

- The analysis target is complex-tissue data rather than method benchmarking — the SCMixology and Zhengmix datasets comprise well-defined cell clusters and are by design simpler than data from complex tissues
- The cell-type labels are not from a published, high-quality source — unvalidated labels give an overly narrow benchmark standard since ARI is limited by the original cell type assignments
- The method's advantage is distinguishing sub-populations or rare cell types, which ARI against a reference label set may penalize rather than reward

## Inputs

- Six benchmark scRNAseq datasets with ground-truth labels: three SCMixology cell-mixture datasets (CellBench) and three Zhengmix datasets (DuoClustering2018)
- Raw counts matrices (gene x cell) for each benchmark dataset
- Published cell-type annotation vectors (ground-truth labels) for each dataset

## Outputs

- Adjusted Rand Index values per dataset per method
- Comparison table of ARI values across datasets (ari_comparison.csv)
- Cluster assignments for cells from NNGraph clustering of reduced-dimension embeddings

## How to apply

Select benchmark datasets with well-defined, published cell-type annotations — in the corral paper these are the three SCMixology cell-mixture datasets from CellBench and the three Zhengmix datasets from DuoClustering2018, all with ground-truth labels from the original publications. Load each dataset together with its annotated cell-type labels as a SingleCellExperiment, since corral interfaces directly with Bioconductor single-cell classes. Run the method under test (e.g., CA via corral, or PCA on log(x+1)-transformed logcounts) on the raw counts matrix of each dataset, then cluster cells in the resulting reduced-dimension embeddings using NNGraph clustering. Compute the Adjusted Rand Index of each method's cluster assignments against the ground-truth labels for every dataset and compile the values into a single comparison table (e.g., ari_comparison.csv) to judge whether the method matches or exceeds the baseline across all comparisons.

## Related tools

- **corral** (R/Bioconductor package used to run CA on benchmark counts and produce embeddings for NNGraph clustering; interfaces with SingleCellExperiment) — github.com/laurenhsu1/corral
- **R** (Language in which the benchmark loading, clustering, and ARI computation are executed)
- **Bioconductor** (Framework providing SingleCellExperiment classes for handling benchmark datasets and interfacing with corral)

## Examples

```
BiocManager::install("laurenhsu1/corral", dependencies = TRUE)
```

## Evaluation signals

- ARI is computed against annotated cell types for every one of the six benchmark datasets, not a subset
- CA result is compared with a PCA-on-logcounts baseline run on the same dataset and same clustering procedure (NNGraph)
- A method is judged successful when it matches or exceeds the baseline ARI in every comparison (as CA did against PCA in the corral paper)
- Output table (ari_comparison.csv) contains ARI rows for all six datasets: three SCMixology and three Zhengmix
- Cluster count parameter is noted as context-dependent — number of 'true' clusters is unknown a priori and should be justified per dataset

## Limitations

- Ground-truth labels come from original dataset publications and were not independently validated, so ARI performance is limited by the original cell type assignments
- High-quality ground-truth benchmark datasets are limited to simple pseudo-cell mixtures or pools of distinct cell types and do not reflect real-world tissue complexity
- ARI evaluation may penalize methods that better distinguish sub-populations or rare cell types relative to reference labels
- Optimal number of clusters is unknown a priori and depends on the research question and context

## Evidence

- [discussion] ground-truth labels from original publications, ARI-based scoring: "the datasets we analyzed did not have independently validated cell type annotations, so performance is limited by the original cell type assignments"
- [discussion] Benchmark datasets are simple mixtures by design: "Except for SCMixology and Zhengmix (both comprising well-defined cell clusters and by design simpler than data from complex tissues)"
- [discussion] ARI can penalize rare-type resolution: [evidence span withheld — anchor status: not_found]
- [intro] corral interfaces with Bioconductor single-cell classes: "We implemented standard and adapted CA for scRNAseq in corral, an R/Bioconductor package that interfaces directly with Bioconductor classes (including SingleCellExperiment)"
- [readme] corral installation via Bioconductor: "Or it can be installed with `BiocManager::install("laurenhsu1/corral", dependencies = TRUE)`"
