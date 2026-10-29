---
name: scrnaseq-count-matrix-simulation
description: Use when you need a simulated scRNA-seq benchmark dataset with known
  cell-type labels and controllable cluster sizes (e.g., a rare population <5% of
  cells) to evaluate clustering accuracy and robustness.
when_to_use_negative:
- The goal is to benchmark on real experimental data where biological variability,
  not controllable cluster sizes, is the test condition.
- The input is already a labeled synthetic count matrix — re-simulation adds no information
  and risks duplicating the benchmark.
- The analysis requires unnormalized continuous expression values rather than integer
  count matrices, since scDesign2 simulates raw counts.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDesign2
  role: R package used to fit gene-expression marginals and gene-gene correlations
    per cell type from the reference PBMC data and simulate the synthetic count matrix
- name: scanpy
  role: Python library used to load the reference counts into an AnnData object and
    apply basic gene filtering (min_cells=3) before parameter fitting
- name: scAce
  role: Downstream clustering method that the simulated dataset with ground-truth
    labels is used to benchmark
  repo: https://github.com/sldyns/scAce
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad546
    title: 'scAce: an adaptive embedding and clustering method for single-cell gene
      expression data'
  source_content_hash: sha256:2b821a3c579f3acb3c9c85840a7b43bd6cd1edaf7254750d04288b4940d26898
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sldyns/scAce
  tools:
  - Python 3.8
  - scAce
  - scDesign2
  - scanpy
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# scrnaseq-count-matrix-simulation

## Summary

Generate synthetic scRNA-seq count matrices with known ground-truth cell-type labels by fitting the scDesign2 generative model to a real reference dataset, enabling controlled benchmarking of clustering methods such as scAce.

## When to use

Use when you need a simulated scRNA-seq benchmark dataset with known cell-type labels and controllable cluster sizes (e.g., a rare population <5% of cells) to evaluate clustering accuracy and robustness.

## When NOT to use

- The goal is to benchmark on real experimental data where biological variability, not controllable cluster sizes, is the test condition.
- The input is already a labeled synthetic count matrix — re-simulation adds no information and risks duplicating the benchmark.
- The analysis requires unnormalized continuous expression values rather than integer count matrices, since scDesign2 simulates raw counts.

## Inputs

- Public scRNA-seq reference count matrix (e.g., 10x Genomics PBMC data, Zheng et al. 2017) loadable into a scanpy AnnData object
- Per-cell-type ground-truth labels for the reference dataset (for scDesign2 per-type parameter fitting)
- Target simulation configuration: number of cell types and per-type cell counts (e.g., 5 types at 600/200/200/100/50)

## Outputs

- Simulated gene x cell count matrix (16,653 genes x 1,150 cells in the scAce benchmark) as a CSV file, e.g. scdesign2_pbmc_simulated_counts.csv
- Ground-truth cell-type label column matching the simulated per-type sizes

## How to apply

Load a public reference count matrix (e.g., the 10x Genomics PBMC dataset from Zheng et al. 2017) into a scanpy AnnData object and apply basic gene filtering (sc.pp.filter_genes with min_cells=3) to retain informative genes before parameter fitting. Fit the scDesign2 model per cell type to learn gene-expression marginal parameters and gene-gene correlations from the real data. Simulate a synthetic count matrix with the desired number of cell types and per-type cell sizes — for the scAce benchmark, 16,653 genes and 1150 cells across five cell types sized 600/200/200/100/50 so the rarest type is <5% of cells. Write the simulated count matrix together with the ground-truth cell-type label column (e.g., to scdesign2_pbmc_simulated_counts.csv) for downstream benchmarking.

## Related tools

- **scDesign2** (R package used to fit gene-expression marginals and gene-gene correlations per cell type from the reference PBMC data and simulate the synthetic count matrix)
- **scanpy** (Python library used to load the reference counts into an AnnData object and apply basic gene filtering (min_cells=3) before parameter fitting)
- **scAce** (Downstream clustering method that the simulated dataset with ground-truth labels is used to benchmark) — https://github.com/sldyns/scAce

## Examples

```
import scanpy as sc; adata = sc.AnnData(pbmc_counts); sc.pp.filter_genes(adata, min_cells=3); # then fit scDesign2 per cell type and simulate 5 types at 600/200/200/100/50 cells
```

## Evaluation signals

- Output file (e.g., scdesign2_pbmc_simulated_counts.csv) exists and contains the expected number of cells (600+200+200+100+50 = 1,150).
- A ground-truth cell-type label column is present in the output file.
- Per-type cell counts match the configured sizes 600/200/200/100/50 exactly.
- The rarest simulated cell type constitutes <5% of all cells (50/1150).
- Gene dimension matches the target (16,653 genes) and the matrix contains integer count values.

## Limitations

- Simulation fidelity depends on the reference dataset: parameters are fitted from the 10x Genomics PBMC data, so simulated distributions reflect PBMC biology rather than arbitrary tissues.
- Cluster sizes must be configured explicitly; a rare type is intentionally set to <5% of cells, which makes the benchmark hard but also means results are sensitive to how the rare cluster is treated during clustering.
- Basic gene filtering (min_cells=3) is applied before fitting, so very sparsely expressed genes in the reference are excluded from the simulation.

## Evidence

- [other] The simulated dataset contained 16 653 genes and 1150 cells belonging to five cell types, with 600, 200, 200, 100, and 50 cells per cell type respectively.: [evidence span withheld — anchor status: not_found]
- [abstract] scAce outperforms state-of-the-art scRNA-seq clustering methods with better accuracy and robustness on simulated and real datasets: "we demonstrate that scAce outperforms state-of-the-art clustering methods for scRNA-seq data, and achieves better clustering accuracy and robustness"
- [readme] Basic filtering with scanpy before simulation setup: "# Basic filtering
sc.pp.filter_genes(adata, min_cells=3)
sc.pp.filter_cells(adata, min_genes=200)"
