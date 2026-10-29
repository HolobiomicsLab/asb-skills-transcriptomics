---
name: spearman-rank-correlation-network-construction
description: Use when you need a cross-dataset cell-cell similarity network from scRNA-seq
  expression data for a gene set (e.g. GOmouse, HVGs) as the first step of MetaNeighbor
  neighbor voting or AUROC estimation.
when_to_use_negative:
- The gene set contains fewer than two genes, or gene set names do not match the gene
  names of the expression matrix (README lists these as critical requirements)
- You need an unsupervised marker gene ranking rather than a similarity network over
  cells
- Sample count is very large without parallelization, since the ranking step does
  not scale well in the R environment
tools:
- name: MetaNeighbor
  role: Implements the rank-correlation network construction and downstream neighbor-voting
    predictor (MetaNeighbor supervised and MetaNeighborUS functions)
  repo: https://github.com/gillislab/MetaNeighbor
- name: R
  role: Runtime environment in which the package and correlation/ranking computations
    are executed
- name: SummarizedExperiment
  role: Container for the gene-by-sample expression matrix (e.g. mn_data) passed to
    the network construction
- name: gplots
  role: Heatmap.2 visualization of the resulting AUROC matrix derived from the network
provenance:
  source_task_ids:
  - task_008
  - task_005
  - task_004
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-03282-0
    title: Characterizing the replicability of cell types defined by single cell RNA-sequencing
      data using MetaNeighbor
  source_content_hash: sha256:397f1567ffb5177dc5329d72429b93c145171bebfefe0b647b93db97f0b93313
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gillislab/MetaNeighbor
  tools:
  - Matrix
  - MetaNeighbor
  - R
  - RColorBrewer
  - SingleCellExperiment
  - SummarizedExperiment
  - gplots
  - matrixStats
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# spearman-rank-correlation-network-construction

## Summary

Constructs a cell-cell similarity network by computing Spearman rank correlations between all pairs of cells across scRNA-seq datasets based on a chosen gene set, then re-ranking and standardizing network values to lie between 0 and 1 for use in MetaNeighbor's neighbor-voting predictor.

## When to use

Use when you need a cross-dataset cell-cell similarity network from scRNA-seq expression data for a gene set (e.g. GOmouse, HVGs) as the first step of MetaNeighbor neighbor voting or AUROC estimation.

## When NOT to use

- The gene set contains fewer than two genes, or gene set names do not match the gene names of the expression matrix (README lists these as critical requirements)
- You need an unsupervised marker gene ranking rather than a similarity network over cells
- Sample count is very large without parallelization, since the ranking step does not scale well in the R environment

## Inputs

- Gene-by-sample scRNA-seq expression matrix (SummarizedExperiment, e.g. mn_data, RPKM)
- A gene set of interest as a list of vectors (e.g. GOmouse GO terms, HVGs from variableGenes)
- Experiment/study labels indicating the source dataset of each sample
- Binary cell-type membership matrix with row names matching expression column names

## Outputs

- Cell-cell rank-correlation network, re-ranked and standardized to values between 0 and 1
- Downstream mean AUROC scores per gene set-by-cell type pair (cell type-by-cell type matrix)

## How to apply

Starting from a gene-by-sample expression matrix (e.g. the mn_data SummarizedExperiment of cortical interneurons from GSE60361 and GSE71585 in RPKM) and a gene set (GO:0022857, GOmouse, or a variableGenes HVG set), build a network of rank correlations between all cells for that gene set. Then re-rank all network values and standardize them to lie between 0 and 1. The standardized network feeds the neighbor-voting predictor: matrix multiplication with the binary cell-type membership vector, each element divided by the null predictor (node degree), so each cell scores the fraction of its neighbors (including itself) in a given cell type. Cycle through leave-one-dataset-out folds and report the mean AUROC across folds. Note the ranking step is the scaling bottleneck in R.

## Related tools

- **MetaNeighbor** (Implements the rank-correlation network construction and downstream neighbor-voting predictor (MetaNeighbor supervised and MetaNeighborUS functions)) — https://github.com/gillislab/MetaNeighbor
- **R** (Runtime environment in which the package and correlation/ranking computations are executed)
- **SummarizedExperiment** (Container for the gene-by-sample expression matrix (e.g. mn_data) passed to the network construction)
- **gplots** (Heatmap.2 visualization of the resulting AUROC matrix derived from the network)

## Examples

```
celltype_NV = MetaNeighborUS(var_genes = var_genes, dat = mn_data, study_id = mn_data$study_id, cell_type = mn_data$cell_type)
```

## Evaluation signals

- All network values are re-ranked and standardized to lie between 0 and 1 before neighbor voting
- The resulting AUROC score matrix matches the scores from the deposited MetaNeighbor::MetaNeighbor() call on the same mn_data/GOmouse inputs
- Each cell's voting score equals the fraction of its neighbors (including itself) belonging to the given cell type after division by node degree
- AUROC scores above 0.5 indicate above-chance recovery of held-out cells (e.g. Sst-Chodl/Int1, Smad3/Int14); an uninformative control gene set scores near chance (~0.47)
- The unsupervised AUROC matrix diagonal is <1 and cross-dataset high-scoring pairs (AUROC >0.9, reciprocal top hits) reproduce expected cell-type separation

## Limitations

- The ranking step does not scale ideally in R; runtimes grow steeply with sample count (e.g. 10,000 samples x 10 gene sets took ~3653 s on a laptop), so speed-ups require parallelization
- Genes within gene sets must match the gene names of the expression matrix, gene sets must contain more than one gene, and cell-type label row names must match expression column names
- AUROC of 0.5 means performance equal to random guessing, so scores near chance indicate the gene set carries no replicable cell-type signal

## Evidence

- [methods] Rank correlation network is the first MetaNeighbor step: "first, we build a network of rank correlations between all cells for a gene set"
- [methods] Neighbor voting consumes the standardized network: "the neighbor voting predictor produces a weighted matrix of predicted labels by performing matrix multiplication between the network and the binary vector indicating cell type membership"
- [intro] Correlations computed across datasets for a gene set: "We calculate correlations between all pairs of cells that we aim to compare across datasets based on the expression of a set of genes."
- [readme] Ranking step limits scalability in R: "Because there is a ranking step, the code is not ideal for scaling in the R environment as is."
