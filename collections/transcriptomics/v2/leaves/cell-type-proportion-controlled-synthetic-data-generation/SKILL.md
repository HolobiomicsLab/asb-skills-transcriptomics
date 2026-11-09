---
name: cell-type-proportion-controlled-synthetic-data-generation
description: Use when you need a simulated scRNA-seq count matrix with known ground-truth
  cell-type labels and controlled per-type cell numbers — e.g. to benchmark clustering
  methods on a rare cell type (<5% of cells).
when_to_use_negative:
- You need real, not simulated, single-cell data for your benchmark — this skill produces
  synthetic counts with ground-truth labels only
- Your reference data has no cell-type annotations, since scDesign2 must fit per-cell-type
  marginal parameters
- You require cluster number to be unknown a priori; here the five ground-truth types
  are imposed by the simulation design
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDesign2
  role: R package used to fit per-cell-type gene expression marginals and gene-gene
    correlations from PBMC reference data and simulate the synthetic count matrix
- name: scanpy
  role: AnnData container and sc.pp.filter_genes(min_cells=3) preprocessing before
    scDesign2 parameter fitting
  repo: https://github.com/sldyns/scAce
- name: Python 3.8
  role: Implementation environment for the loading, filtering, and simulation-output
    steps
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad546
    title: 'scAce: an adaptive embedding and clustering method for single-cell gene
      expression data'
  source_content_hash: sha256:a7ff79b0948e547d401063b39b45d116cc10e6904e8c922f13ac405d305f0980
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sldyns/scAce
  tools:
  - Python 3.8
  - scDesign2
  - scanpy
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# cell-type-proportion-controlled-synthetic-data-generation

## Summary

Generates a synthetic scRNA-seq count matrix with scDesign2 by fitting per-cell-type gene marginal and gene-gene correlation parameters to a real reference dataset (10x PBMC, Zheng et al. 2017) and simulating specified cell-type sizes, so rare populations are represented at controlled frequencies for clustering benchmarks.

## When to use

Use when you need a simulated scRNA-seq count matrix with known ground-truth cell-type labels and controlled per-type cell numbers — e.g. to benchmark clustering methods on a rare cell type (<5% of cells).

## When NOT to use

- You need real, not simulated, single-cell data for your benchmark — this skill produces synthetic counts with ground-truth labels only
- Your reference data has no cell-type annotations, since scDesign2 must fit per-cell-type marginal parameters
- You require cluster number to be unknown a priori; here the five ground-truth types are imposed by the simulation design

## Inputs

- Public 10x Genomics PBMC gene expression count matrix (Zheng et al. 2017)
- Per-cell-type labels for the reference PBMC data
- Target cell counts per simulated cell type (e.g. 600, 200, 200, 100, 50)

## Outputs

- Simulated scRNA-seq count matrix (16653 genes x 1150 cells) with a ground-truth cell-type label column, saved as scdesign2_pbmc_simulated_counts.csv

## How to apply

Load the public 10x Genomics PBMC gene expression counts (Zheng et al. 2017) into a scanpy AnnData object and apply basic gene filtering (sc.pp.filter_genes with min_cells=3) so only informative genes enter parameter fitting. Fit the scDesign2 model per cell type to learn gene expression marginal distributions and gene-gene correlations from the reference PBMC data. Simulate a synthetic count matrix with five cell types at specified sizes — 600/200/200/100/50 cells (1150 total) — deliberately setting the rarest type below 5% of cells so rare-population recovery can be tested. Write the simulated count matrix together with a ground-truth cell-type label column to a CSV (e.g. scdesign2_pbmc_simulated_counts.csv) for downstream benchmarking of methods such as scAce.

## Related tools

- **scDesign2** (R package used to fit per-cell-type gene expression marginals and gene-gene correlations from PBMC reference data and simulate the synthetic count matrix)
- **scanpy** (AnnData container and sc.pp.filter_genes(min_cells=3) preprocessing before scDesign2 parameter fitting) — https://github.com/sldyns/scAce
- **Python 3.8** (Implementation environment for the loading, filtering, and simulation-output steps)

## Examples

```
import scanpy as sc; adata = sc.AnnData(counts); sc.pp.filter_genes(adata, min_cells=3)  # then fit scDesign2 per cell type and simulate 600/200/200/100/50 cells
```

## Evaluation signals

- Output CSV file exists and contains exactly 1150 cells (600+200+200+100+50)
- The file includes a cell-type label column whose five category sizes match 600/200/200/100/50
- The rarest cell type accounts for less than 5% of simulated cells (50/1150 ≈ 4.3%)
- Gene dimension matches the expected 16653 informative genes after min_cells=3 filtering of the PBMC reference
- Simulated counts are integer-valued and sparse, consistent with scDesign2 count simulation from the PBMC fit

## Limitations

- Simulation fidelity is bounded by the scDesign2 fit to the single PBMC reference dataset; cell types outside the reference distribution are not represented
- The five-type design with a fixed rare type is tailored to benchmarking rare-population recovery, not general data augmentation
- The simulated dataset benchmarks methods like scAce but does not by itself validate them; downstream clustering accuracy must be assessed separately

## Evidence

- [results] Simulated dataset specs: [evidence span withheld — anchor status: not_found]
- [readme] Preprocessing before fitting: "sc.pp.filter_genes(adata, min_cells=3)
sc.pp.filter_cells(adata, min_genes=200)"
- [abstract] Benchmark context: "we demonstrate that scAce outperforms state-of-the-art clustering methods for scRNA-seq data, and achieves better clustering accuracy and robustness"
