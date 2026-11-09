---
name: scdesign2-count-data-simulation
description: Use when benchmarking scRNA-seq dimensionality reduction or clustering
  methods requires simulated count matrices with reliable ground-truth cluster labels,
  e.g. building an extended evaluation collection alongside real datasets like Zheng,
  Koh, Kumar, or Trapnell.
when_to_use_negative:
- When ground-truth labels must reflect biological reality rather than a generative
  model — use real datasets whose labels were determined independently of the assay
  (e.g. Koh, Kumar, Trapnell)
- When evaluating imputation or normalization performance itself — simulation fidelity
  depends on scDesign2's fit and may not reproduce dropout-driven sparsity patterns
  relevant to such comparisons
tools:
- name: scDesign2
  role: Generates the 10 simulation datasets by fitting count distributions to Tabula
    Muris mouse-tissue data covering diverse cell types
- name: scLENS
  role: Primary DR method evaluated on the simulated datasets; also used in shuffled
    non-zero-value control runs to compute Delta-SIL
  repo: github.com/Mathbiomed/scLENS
- name: Scanpy
  role: One of the 11 comparison DR packages run with default settings on all simulated
    and real benchmark datasets
- name: Seurat
  role: One of the 11 comparison DR packages run with default settings on all simulated
    and real benchmark datasets
- name: Monocle3
  role: One of the 11 comparison DR packages run with default settings on all simulated
    and real benchmark datasets
- name: ParallelPCA
  role: One of the 11 comparison DR packages referenced in the paper and run on the
    benchmark datasets
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-47884-3
    title: 'scLENS: data-driven signal detection for unbiased scRNA-seq data analysis'
  source_content_hash: sha256:2660b7599fca059958d052d34eccbad1c057326a0ad9ff3daf71f4fd8065b863
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Monocle3
  - ParallelPCA
  - Scanpy
  - Seurat
  - scDesign2
  - scLENS
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# scDesign2 count-data simulation

## Summary

Generates realistic simulated scRNA-seq count datasets with known ground-truth cell-type labels using scDesign2 (version 1.0.0), fitting to Tabula Muris mouse-tissue reference data, so that dimensionality reduction methods can be benchmarked against labels determined independently of the scRNA-seq assay.

## When to use

Use when benchmarking scRNA-seq dimensionality reduction or clustering methods requires simulated count matrices with reliable ground-truth cluster labels, e.g. building an extended evaluation collection alongside real datasets like Zheng, Koh, Kumar, or Trapnell.

## When NOT to use

- When ground-truth labels must reflect biological reality rather than a generative model — use real datasets whose labels were determined independently of the assay (e.g. Koh, Kumar, Trapnell)
- When evaluating imputation or normalization performance itself — simulation fidelity depends on scDesign2's fit and may not reproduce dropout-driven sparsity patterns relevant to such comparisons

## Inputs

- Tabula Muris mouse-tissue reference scRNA-seq count data covering diverse cell types
- Optionally, pre-sorted blood cell-type datasets (e.g. Zheng data) for mixture-based simulations with weighted random sampling over target cluster sizes and ratios

## Outputs

- 10 simulated count-matrix datasets with ground-truth cell-type labels (from Tabula Muris fits via scDesign2 v1.0.0)
- Per-dataset SIL and ECS scores against ground-truth labels for each tested DR method, supporting average-ranking comparisons (e.g. Fig. 5a-style summaries)

## How to apply

Fit scDesign2 (version 1.0.0) to Tabula Muris mouse-tissue data to generate 10 simulation datasets spanning diverse cell types, so each simulated dataset carries ground-truth labels derived independently of the assay. Combine these with simulation controls such as the 13 Zheng-mixture datasets, which are assembled by mixing pre-sorted blood cell types via weighted random sampling based on target cluster sizes and ratios. Apply the same QC as for real data before benchmarking: filter cells with <200 expressed genes, genes expressed in <15 cells, and cells with >5% mitochondrial reads. Feed the simulated matrices into the DR methods under test (scLENS vs. Seurat, Scanpy, Monocle3, ParallelPCA, Randomly, ACTIONet, SHARP, scDHA, scVI, ZINB-WaVE, scTransform) with default settings, then derive clusters (hierarchical on 2D UMAP with 15 neighbors, min distance 0.01, cosine metric; or Leiden on an SNN graph with 20 neighbors, edge cutoff 1/15, cosine similarity) and score them against the simulated ground truth using silhouette (SIL) and element-centric similarity (ECS, damping factor d=0.975).

## Related tools

- **scDesign2** (Generates the 10 simulation datasets by fitting count distributions to Tabula Muris mouse-tissue data covering diverse cell types)
- **scLENS** (Primary DR method evaluated on the simulated datasets; also used in shuffled non-zero-value control runs to compute Delta-SIL) — github.com/Mathbiomed/scLENS
- **Scanpy** (One of the 11 comparison DR packages run with default settings on all simulated and real benchmark datasets)
- **Seurat** (One of the 11 comparison DR packages run with default settings on all simulated and real benchmark datasets)
- **Monocle3** (One of the 11 comparison DR packages run with default settings on all simulated and real benchmark datasets)
- **ParallelPCA** (One of the 11 comparison DR packages referenced in the paper and run on the benchmark datasets)

## Evaluation signals

- The 10 simulated datasets cover diverse cell types inherited from the Tabula Muris fit, with ground-truth labels available for every simulated cell
- Benchmark summary reproduces the expected structure: average ECS and SIL computed per package over all 39 extended datasets, with scLENS ranking first under both hierarchical and Leiden clustering variants
- QC filtering applied identically to simulated and real data (cells <200 expressed genes, genes expressed in <15 cells, cells >5% mitochondrial reads) before DR runs
- Delta-SIL control behaves as expected: shuffling non-zero values and re-running scLENS yields lower SIL on datasets carrying non-binary information, quantifying signal content
- Relative performance (scLENS ECS minus best competitor ECS) computed per dataset shows the reported increasing trend with data sparsity

## Limitations

- scDesign2 is version-pinned (1.0.0) in the benchmark workflow; other versions may produce distributions that shift DR-method rankings
- Simulation is limited to 10 Tabula Muris-derived datasets, so conclusions about high-sparsity behavior should be cross-checked against real datasets and the shuffled-value (Delta-SIL) control
- Benchmark validity depends on clustering cut heights/resolutions being selected to maximize ECS against ground truth, which couples scoring to the clustering procedure

## Evidence

- [other] Simulation dataset generation via scDesign2: [evidence span withheld — anchor status: not_found]
- [other] Benchmark comparison set: [evidence span withheld — anchor status: not_found]
- [other] Scoring metrics against ground truth: [evidence span withheld — anchor status: not_found]
- [other] Delta-SIL shuffled control: [evidence span withheld — anchor status: not_found]
- [abstract] scLENS benchmark superiority: "Our method outperforms 11 widely used dimensionality reduction tools and performs particularly well for challenging scRNA-seq datasets with high sparsity and variability"
