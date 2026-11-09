---
name: gene-gene-correlation-preservation-in-simulation
description: Use when a clustering benchmark needs a simulated scRNA-seq dataset that
  retains realistic gene-gene correlation structure and per-type marginals, with controlled
  group sizes including a rare cell type (<5% of cells).
when_to_use_negative:
- No per-cell-type annotations are available on the reference data, since scDesign2
  must be fitted per cell type to capture type-specific marginals and correlations
- The evaluation requires real experimental scRNA-seq data rather than a synthetic
  benchmark with known ground-truth labels
- The goal is running clustering itself (e.g. with scAce) rather than generating the
  benchmark data on which clustering is evaluated
tools:
- name: scDesign2
  role: Fitted per cell type to learn gene expression marginal parameters and gene-gene
    correlations from PBMC data, then used to simulate the synthetic count matrix
- name: scanpy
  role: Loads the PBMC counts into an AnnData object and applies basic gene filtering
    (min_cells=3) before parameter fitting
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad546
    title: 'scAce: an adaptive embedding and clustering method for single-cell gene
      expression data'
  source_content_hash: sha256:ddf9528d2420c59390a38bc774ff27832fe2ed40b114c82d797e8cf284a1e7b8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python 3.8
  - scDesign2
  - scanpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# gene-gene-correlation-preservation-in-simulation

## Summary

Generates a synthetic scRNA-seq benchmark count matrix by fitting scDesign2 per cell type so that gene expression marginals and gene-gene correlations learned from real data (10x Genomics PBMC, Zheng et al. 2017) are preserved, with controlled cell-type proportions including a rare (<5%) type used to benchmark scAce clustering.

## When to use

Use when a clustering benchmark needs a simulated scRNA-seq dataset that retains realistic gene-gene correlation structure and per-type marginals, with controlled group sizes including a rare cell type (<5% of cells).

## When NOT to use

- No per-cell-type annotations are available on the reference data, since scDesign2 must be fitted per cell type to capture type-specific marginals and correlations
- The evaluation requires real experimental scRNA-seq data rather than a synthetic benchmark with known ground-truth labels
- The goal is running clustering itself (e.g. with scAce) rather than generating the benchmark data on which clustering is evaluated

## Inputs

- Raw gene expression count matrix from the public 10x Genomics PBMC dataset (Zheng et al. 2017)
- Per-cell cell-type labels for the PBMC reference data (required for per-type scDesign2 fitting)
- scDesign2 R package and Python 3.8 with scanpy

## Outputs

- Simulated count matrix of 16653 genes x 1150 cells
- Ground-truth cell-type label column with sizes 600/200/200/100/50
- CSV file scdesign2_pbmc_simulated_counts.csv

## How to apply

Load the public 10x Genomics PBMC gene expression counts (Zheng et al. 2017) into a scanpy AnnData object and apply basic gene filtering with min_cells=3 so only informative genes enter parameter fitting. Fit the scDesign2 model separately per cell type to learn each gene's expression marginal parameters and the gene-gene correlations from the PBMC data. Simulate a synthetic count matrix with 16653 genes and 1150 cells spanning 5 cell types at sizes 600/200/200/100/50 so the rarest type is <5% of cells. Write the simulated count matrix together with a ground-truth cell-type label column to scdesign2_pbmc_simulated_counts.csv, then verify the per-type sizes and the <5% rare-type constraint before using the file to benchmark clustering methods such as scAce.

## Related tools

- **scDesign2** (Fitted per cell type to learn gene expression marginal parameters and gene-gene correlations from PBMC data, then used to simulate the synthetic count matrix)
- **scanpy** (Loads the PBMC counts into an AnnData object and applies basic gene filtering (min_cells=3) before parameter fitting)

## Examples

```
import scanpy as sc; adata = sc.AnnData(pbmc_counts); sc.pp.filter_genes(adata, min_cells=3); sc.pp.filter_cells(adata, min_genes=200)  # then fit scDesign2 per cell type and simulate 5 types sized 600/200/200/100/50
```

## Evaluation signals

- Output file scdesign2_pbmc_simulated_counts.csv exists and contains a cell-type label column
- Simulated matrix has 16653 genes and 1150 cells (600+200+200+100+50)
- Cell-type sizes match exactly 600/200/200/100/50
- Rarest cell type accounts for <5% of cells (50/1150)
- Gene-gene correlation structure in the simulated matrix reflects the correlations fitted by scDesign2 from the PBMC reference

## Limitations

- Simulation fidelity depends on the single 10x PBMC reference; marginals and correlations learned from PBMC may not transfer to other tissues or scRNA-seq protocols
- The rare-type challenge in the benchmark is fixed at 50 cells (<5%), so results may not generalize to other rarity levels
- Pre-simulation gene filtering (min_cells=3) discards very lowly expressed genes, which are therefore absent from the simulated matrix

## Evidence

- [methods] Fit the scDesign2 model per cell type to learn gene expression marginal parameters and gene-gene correlations from the PBMC data.: [evidence span withheld — anchor status: not_found]
- [methods] Simulate a synthetic count matrix with 5 cell types at sizes 600/200/200/100/50 cells so the rarest type is <5% of cells.: [evidence span withheld — anchor status: not_found]
- [results] The simulated dataset contained 16 653 genes and 1150 cells belonging to five cell types, with 600, 200, 200, 100, and 50 cells per cell type respectively.: [evidence span withheld — anchor status: not_found]
- [readme] sc.pp.filter_genes(adata, min_cells=3): "sc.pp.filter_genes(adata, min_cells=3)"
