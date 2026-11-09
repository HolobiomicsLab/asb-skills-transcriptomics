---
name: weighted-random-cell-subsampling
description: Use when benchmarking clustering or dimensionality reduction tools needs
  datasets with known cell-type composition, by subsampling pre-sorted cells (e.g.,
  Zheng mixtures) at controlled cluster sizes and ratios.
when_to_use_negative:
- The goal is to evaluate method performance on real datasets with biologically authentic
  cluster proportions rather than controlled mixtures
- The reference cell types lack independent, assay-independent labels, so ground truth
  for ECS/SIL computation would be unreliable
- You need simulation of new expression profiles rather than resampling of measured
  cells (use a simulator like scDesign2 instead)
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2229
tools:
- name: scLENS
  role: Downstream DR method evaluated on the weighted-sampled Zheng-mixture benchmark
    datasets
  repo: https://github.com/Mathbiomed/scLENS
- name: scDesign2
  role: Complementary simulator (v1.0.0) generating the 10 simulation datasets from
    Tabula Muris mouse-tissue data to cover diverse cell types
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:c7dfff10cefced7f4e7df5f2ae34bc73481b1ac6f8f9b19c452ae77e77af0621
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mathbiomed/scLENS
  tools:
  - Monocle3
  - ParallelPCA
  - Scanpy
  - Seurat
  - scDesign2
  - scLENS
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# weighted-random-cell-subsampling

## Summary

Constructs simulated scRNA-seq mixture datasets by mixing pre-sorted cells from a reference dataset (e.g., Zheng blood cell types) via weighted random sampling according to target cluster sizes and ratios, enabling ground-truth-controlled benchmarking of dimensionality reduction methods.

## When to use

Use when benchmarking clustering or dimensionality reduction tools needs datasets with known cell-type composition, by subsampling pre-sorted cells (e.g., Zheng mixtures) at controlled cluster sizes and ratios.

## When NOT to use

- The goal is to evaluate method performance on real datasets with biologically authentic cluster proportions rather than controlled mixtures
- The reference cell types lack independent, assay-independent labels, so ground truth for ECS/SIL computation would be unreliable
- You need simulation of new expression profiles rather than resampling of measured cells (use a simulator like scDesign2 instead)

## Inputs

- Pre-sorted reference scRNA-seq dataset with known cell-type labels (e.g., Zheng blood cell types)
- Target cluster sizes and mixing ratios per simulated dataset
- Random seed / sampling specification for reproducibility

## Outputs

- 13 Zheng-mixture simulated count matrices with exact ground-truth cluster labels
- Extended benchmark collection (39 datasets total: 16 real + 10 scDesign2 simulations + 13 Zheng mixtures) for ECS/SIL evaluation

## How to apply

Start from a reference dataset with independently determined cell labels, such as the pre-sorted blood cell types from the Zheng data. For each of the target simulated datasets (the paper assembled 13 Zheng-mixture datasets), specify the desired cluster sizes and ratios, then draw cells from each pre-sorted cell type by weighted random sampling according to those targets, pooling the sampled cells into a mixture count matrix. Combine these mixtures with other labeled datasets (Koh, Kumar, Trapnell, whose labels were determined independently of the scRNA-seq assay) and simulation datasets (e.g., 10 scDesign2 simulations from Tabula Muris generated with scDesign2 v1.0.0) to form the extended benchmark collection. Apply the same QC to all datasets — filter cells with <200 expressed genes, genes expressed in <15 cells, and cells with >5% mitochondrial reads — before running the DR methods. The rationale is that weighted subsampling preserves realistic single-cell expression distributions while giving exact ground-truth labels for computing ECS (damping factor d=0.975) and silhouette (SIL) scores.

## Related tools

- **scLENS** (Downstream DR method evaluated on the weighted-sampled Zheng-mixture benchmark datasets) — https://github.com/Mathbiomed/scLENS
- **scDesign2** (Complementary simulator (v1.0.0) generating the 10 simulation datasets from Tabula Muris mouse-tissue data to cover diverse cell types)

## Examples

```
python sample_cells.py --reference zheng_sorted.h5ad --targets mixture_config.csv --method weighted --seed 42 --output zheng_mixtures/
```

## Evaluation signals

- Each of the 13 mixture datasets contains exactly the pre-specified number of cells per cluster, matching the target sizes and ratios
- Ground-truth labels of the mixtures are exact (sampled from pre-sorted cells), yielding unambiguous ECS (d=0.975) and SIL references
- After QC (<200 genes/cell removed, genes in <15 cells removed, >5% mitochondrial-read cells removed), the benchmark totals 39 datasets (16 real + 10 simulations + 13 mixtures)
- Downstream clustering on the mixtures can be validated by verifying that scLENS ranks first in ECS (hierarchical and graph-based) and SIL, consistent with Fig. 5a

## Limitations

- Subsampled mixtures inherit the expression distributions of the source assay, so they may not capture cross-dataset batch effects or simulation-level sparsity extremes; pair with scDesign2 simulations for diversity
- Weighted random sampling with low target cluster sizes increases sampling variance and can make ECS/SIL sensitive to the random seed
- Because labels come from pre-sorted cells, mixture benchmarks cannot test discovery of novel cell states absent from the sorted reference

## Evidence

- [other] Assemble the 13 Zheng-mixture datasets by mixing pre-sorted blood cell types from the Zheng data using weighted random sampling based on target cluster sizes and ratios: [evidence span withheld — anchor status: not_found]
- [other] labels independent of assay: [evidence span withheld — anchor status: not_found]
- [other] QC thresholds: [evidence span withheld — anchor status: not_found]
- [readme] README QC of preprocess function: "cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells are filtered out to ensure data quality."
