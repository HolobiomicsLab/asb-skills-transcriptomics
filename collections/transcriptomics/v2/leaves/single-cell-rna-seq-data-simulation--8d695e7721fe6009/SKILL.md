---
name: single-cell-rna-seq-data-simulation
description: Use when benchmarking scRNA-seq dimensionality reduction or clustering
  tools requires ground-truth labels plus controlled sparsity and CV of TGC, e.g.
  testing robustness to >90% zeros and dropout-driven variability.
when_to_use_negative:
- Real scRNA-seq data already has trusted expert labels and known sparsity — analyze
  it directly instead of simulating.
- The goal is biological discovery on the dataset itself rather than method benchmarking;
  simulation adds no information about the underlying biology.
- The data is not UMI/count-based scRNA-seq (e.g. bulk RNA-seq, spatial protein assays)
  that the scDesign2 count model was not trained to mimic.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scDesign2
  role: Count-model simulator (v1.0.0) trained on the reference immune cell dataset
    to generate ~60,000 synthetic cells across ten T-cell subtypes for the sparsity/TGC
    benchmark
- name: scLENS
  role: Downstream dimensionality reduction tool evaluated on the simulated datasets
    using its default pipeline (scaling factor 1, L2 normalization, RMT-based noise
    filtering, signal robustness test)
  repo: github.com/Mathbiomed/scLENS
- name: UMAP
  role: UMAP.jl (15 neighbors, min_dist 0.01, cosine metric) used to build 2D embeddings
    of the simulated data for hierarchical-clustering evaluation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:937ec71d824ed777ce6ed3312e5176d80bfbe0337444a3d44f69f9d11fc6ada2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ACTIONet
  - Monocle3
  - ParallelPCA
  - Randomly
  - SHARP
  - Scanpy
  - Seurat
  - UMAP
  - ZINB-WaVE
  - scDHA
  - scDesign2
  - scLENS
  - scTransform
  - scVI
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# single-cell-rna-seq-data-simulation

## Summary

Generate ground-truth-labeled synthetic scRNA-seq count data by training the scDesign2 (v1.0.0) simulator on a reference immune cell dataset, then subsample to build stratified benchmark cohorts spanning gradients of sparsity and CV of total gene counts (TGC). This enables quantitative DR-tool comparison (SIL/ECS) when no real data with trusted cell-type labels exists.

## When to use

Use when benchmarking scRNA-seq dimensionality reduction or clustering tools requires ground-truth labels plus controlled sparsity and CV of TGC, e.g. testing robustness to >90% zeros and dropout-driven variability.

## When NOT to use

- Real scRNA-seq data already has trusted expert labels and known sparsity — analyze it directly instead of simulating.
- The goal is biological discovery on the dataset itself rather than method benchmarking; simulation adds no information about the underlying biology.
- The data is not UMI/count-based scRNA-seq (e.g. bulk RNA-seq, spatial protein assays) that the scDesign2 count model was not trained to mimic.

## Inputs

- Reference immune cell scRNA-seq dataset (cell-by-gene count matrix) for training scDesign2
- Ground-truth cell-type labels for the ten T-cell subtypes in the reference data
- Target stratification plan (sparsity levels and CV of TGC values, e.g. CV ~0.3 and sparsity ~0.84)

## Outputs

- 13 simulated datasets of ~3,000 cells each with ground-truth T-cell subtype labels
- Benchmark score tables of average SIL and ECS (damping factor 0.975) per dataset-package pair
- Embeddings (e.g. 2D UMAP) and cluster assignments per tool per dataset

## How to apply

Train the scDesign2 simulator (version 1.0.0) on the reference immune cell scRNA-seq dataset and simulate approximately 60,000 cells spanning ten T-cell subtypes, so each synthetic cell carries a ground-truth subtype label. Subsample ~3,000 cells per dataset by weighted random sampling with probability weights based on cells' total gene counts, producing 13 datasets: five with similar CV of TGC (~0.3) and varying sparsity, and five with similar sparsity (~0.84) and varying CV of TGC. This stratification is what lets you isolate the effect of sparsity from the effect of gene-count variability — the two confounders that distort real-data benchmarks. Run each candidate tool (scLENS default pipeline with scaling factor 1, L2 normalization, RMT-based noise filtering, signal robustness test; baselines at default settings) on every simulated dataset, then score embeddings against the simulated labels with average silhouette (SIL) and element-centric similarity (ECS, damping factor 0.975). Simulated zeros are essential to the design because scRNA-seq zeros can exceed 90% and may be either true absence or dropout, so only a controlled simulator can separate signal loss from biology.

## Related tools

- **scDesign2** (Count-model simulator (v1.0.0) trained on the reference immune cell dataset to generate ~60,000 synthetic cells across ten T-cell subtypes for the sparsity/TGC benchmark)
- **scLENS** (Downstream dimensionality reduction tool evaluated on the simulated datasets using its default pipeline (scaling factor 1, L2 normalization, RMT-based noise filtering, signal robustness test)) — github.com/Mathbiomed/scLENS
- **UMAP** (UMAP.jl (15 neighbors, min_dist 0.01, cosine metric) used to build 2D embeddings of the simulated data for hierarchical-clustering evaluation)

## Examples

```
using scLENS; pre_df = scLENS.preprocess(scLENS.read_file("simulated_dataset.csv.gz")); emb = scLENS.sclens(pre_df, device_="gpu")
```

## Evaluation signals

- Cohort structure matches the design: 13 datasets, ~3,000 cells each, five with CV of TGC ~0.3 across varying sparsity and five with sparsity ~0.84 across varying CV of TGC.
- Every simulated cell carries one of the ten T-cell subtype ground-truth labels, and all subtypes are represented so SIL/ECS against ground truth are computable.
- Benchmark output reproduces the expected signature: scLENS attains the highest SIL/ECS among the 12 packages with the smallest decline as sparsity increases, while fixed-50-PC baselines degrade substantially.
- Per-dataset QC removes cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells before embedding.
- ECS is computed with damping factor 0.975 and SIL is averaged over clusters, matching the published Fig. 4a,c score tables.

## Limitations

- Simulated zeros conflate true mRNA absence with dropout; the paper notes zeros can exceed 90% and represent either actual absence or loss of information due to dropout events, so simulator assumptions may not capture real dropout exactly.
- Conclusions (e.g. scLENS's robustness to high sparsity and variability) are anchored to the reference immune cell dataset used to train scDesign2 and may not transfer to tissues or platforms unlike it.
- RMT-based downstream filtering on large simulated datasets with whole genes requires computing all eigenvalues of the cell-to-cell similarity matrix, demanding significant memory resources and a high-specification computer.

## Evidence

- [methods] Train the scDesign2 simulator (version 1.0.0) on the reference immune cell scRNA-seq dataset and simulate approximately 60,000 cells spanning ten T-cell subtypes.: [evidence span withheld — anchor status: not_found]
- [abstract] Our method outperforms 11 widely used dimensionality reduction tools and performs particularly well for challenging scRNA-seq datasets with high sparsity and variability: "Our method outperforms 11 widely used dimensionality reduction tools and performs particularly well for challenging scRNA-seq datasets with high sparsity and variability"
- [intro] These zeros can represent either the actual absence of mRNA or a loss of information due to dropout events: "These zeros can represent either the actual absence of mRNA or a loss of information due to dropout events"
- [readme] cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells are filtered out to ensure data quality: "cells with fewer than 200 expressed genes and genes expressed in fewer than 15 cells are filtered out to ensure data quality"
