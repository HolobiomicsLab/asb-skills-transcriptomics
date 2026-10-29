---
name: scrnaseq-dataset-harmonization
description: Use when several scRNA-seq datasets with expression matrices, dataset-of-origin
  labels, and cell-type labels must be combined to test whether cell types (e.g. neurons
  vs non-neuronal cells) replicate across studies via AUROC.
when_to_use_negative:
- Cell-type labels are ambiguous or uninformative across datasets — the supervised
  MetaNeighbor requires known corresponding types; use the unsupervised MetaNeighborUS
  with variableGenes HVGs instead
- You only have a single dataset and no cross-dataset replication question — neighbor
  voting with leave-one-dataset-out folds is meaningless without at least two studies
- Gene sets contain fewer than two genes or gene names do not match the expression
  matrix — the method will fail; harmonize identifiers first
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MetaNeighbor
  role: Supervised neighbor-voting framework that builds cell-cell rank correlation
    networks and returns mean AUROC scores across leave-one-dataset-out folds
  repo: https://github.com/gillislab/MetaNeighbor
- name: SummarizedExperiment
  role: Container for the combined multi-study expression matrix with study_id and
    cell_type colData metadata
- name: SingleCellExperiment
  role: Alternative supported container for the harmonized single-cell expression
    data
- name: gplots
  role: heatmap.2 visualization of the celltype_NV AUROC matrix
- name: Matrix
  role: Sparse matrix support for the gene-by-sample expression data
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-03282-0
    title: Characterizing the replicability of cell types defined by single cell RNA-sequencing
      data using MetaNeighbor
  source_content_hash: sha256:1cd4fcf43a0daa76833d299598c529c33660e5ec51fd833d1dbaf4ff0f06fb85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gillislab/MetaNeighbor
  tools:
  - Matrix
  - MetaNeighbor
  - R
  - SingleCellExperiment
  - SummarizedExperiment
  - beanplot
  - gplots
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 22
---

# scrnaseq-dataset-harmonization

## Summary

Harmonizes multiple scRNA-seq datasets into a single SummarizedExperiment object with study_id and cell_type metadata so that MetaNeighbor's neighbor-voting framework can quantify cross-dataset cell-type replicability via mean AUROC scores over leave-one-dataset-out folds.

## When to use

Use when several scRNA-seq datasets with expression matrices, dataset-of-origin labels, and cell-type labels must be combined to test whether cell types (e.g. neurons vs non-neuronal cells) replicate across studies via AUROC.

## When NOT to use

- Cell-type labels are ambiguous or uninformative across datasets — the supervised MetaNeighbor requires known corresponding types; use the unsupervised MetaNeighborUS with variableGenes HVGs instead
- You only have a single dataset and no cross-dataset replication question — neighbor voting with leave-one-dataset-out folds is meaningless without at least two studies
- Gene sets contain fewer than two genes or gene names do not match the expression matrix — the method will fail; harmonize identifiers first

## Inputs

- Gene-by-sample expression matrix (single-cell RNA-seq count/RPKM data)
- Experiment labels: numerical/character vector indicating each sample's dataset of origin (study_id)
- Cell-type labels: binary matrix or metadata indicating each sample's cell type, with row names matching expression matrix column names
- Gene sets of interest as a list of vectors (e.g. GOmouse GO gene sets, randomly chosen matched-size control sets)

## Outputs

- Vector of mean AUROC scores per gene set tested
- List of AUROC scores per dataset and cell type
- Matrix of mean AUROC scores per cell type across datasets (gene set-by-cell type matrix)
- Smoothed AUROC distribution beanplot per cell type (bplot=TRUE) comparable to Fig. 2a

## How to apply

Download the public scRNA-seq datasets (e.g. the eight diverse studies used by Crow et al 2018, including GSE60361 and GSE71585) and format them as one gene-by-sample SummarizedExperiment object whose colData carries study_id (experiment labels) and cell_type metadata, ensuring row names of the cell-type label object match the column names of the expression matrix. Load gene sets of interest, such as GOmouse from the MetaNeighbor package, plus matched-size random gene sets as controls, verifying that gene-set gene names match the expression matrix row names and that each set contains more than one gene. Run MetaNeighbor's supervised neighbor-voting pipeline: it builds a cell-cell rank correlation network for each gene set, predicts held-out labels by matrix multiplication against the binary cell-type membership vector, and permutes through all leave-one-dataset-out cross-validation folds. Collect the gene-set-by-cell-type matrix of mean AUROC scores and optionally visualize with the built-in beanplot (bplot=TRUE). Correct application is judged by whether mean AUROC significantly exceeds the 0.5 chance level and matches published distributions (task one mean AUROC = 0.80 +/- 0.1; GO:0055085 transmembrane transport set AUROC > 0.85).

## Related tools

- **MetaNeighbor** (Supervised neighbor-voting framework that builds cell-cell rank correlation networks and returns mean AUROC scores across leave-one-dataset-out folds) — https://github.com/gillislab/MetaNeighbor
- **SummarizedExperiment** (Container for the combined multi-study expression matrix with study_id and cell_type colData metadata)
- **SingleCellExperiment** (Alternative supported container for the harmonized single-cell expression data)
- **gplots** (heatmap.2 visualization of the celltype_NV AUROC matrix)
- **Matrix** (Sparse matrix support for the gene-by-sample expression data)

## Examples

```
library(MetaNeighbor); library(SummarizedExperiment); data(mn_data); data(GOmouse); AUROC_scores = MetaNeighbor(dat = mn_data, experiment_labels = as.numeric(factor(mn_data$study_id)), celltype_labels = mn_data@colData@metadata$cell_labels, gene_sets = GOmouse, bplot = TRUE)
```

## Evaluation signals

- Mean AUROC across gene sets for the neuron vs non-neuron task falls at 0.80 +/- 0.1, significantly above the 0.5 chance level
- The GO:0055085 transmembrane transport gene set achieves an AUROC score greater than 0.85
- Randomly chosen matched-size gene sets also score significantly above chance, serving as a negative control baseline for interpreting GO set performance
- Row names of the cell-type label object match the column names of the expression matrix, and all genes in each gene set match expression matrix gene names with >1 gene per set
- AUROC scores are reported as means across all folds of leave-one-dataset-out cross-validation, with every dataset held out exactly once

## Limitations

- The ranking step does not scale well in R; runtime grows steeply with sample count (e.g. ~3653 s for 10000 samples), so speed-ups may require parallelization
- Supervised analysis requires knowledge of corresponding cell types across datasets; ambiguous or uninformative labels need the unsupervised heuristic variant
- An AUROC of 0.5 equals random guessing, and moderate scores are still meaningful but reflect weaker replicability than reciprocal top hits with AUROCs >0.95
- Common data-sharing issues such as absence of inferred cell-type labels or missing metadata (viability, capture method, library prep, batch, age, sex, strain) hinder harmonization

## Evidence

- [readme] MetaNeighbor takes four inputs: "MetaNeighbor takes four inputs: a gene-by-sample expression matrix (“data”), a set of labels indicating each sample’s dataset of origin (“experiment labels”), a set of labels"
- [readme] For MetaNeighbor to run, it is critical that genes within gene sets match: [evidence span withheld — anchor status: external_doc]
- [methods] we build a network of rank correlations between all cells: "first, we build a network of rank correlations between all cells for a gene set"
- [methods] we permute through all possible combinations of leave-one-dataset-out cross-validation: "For cross-validation, we permute through all possible combinations of leave-one-dataset-out cross-validation, and we report how well we can recover cells of the same type as area"
- [abstract] we first measure the replicability of neuronal identity: "We first measure the replicability of neuronal identity, comparing results across eight technically and biologically diverse datasets to define best practices"
- [readme] AUROC score of 0.5 means that we have performed as well as random: "An AUROC score of 0.5 means that we have performed as well as if we had randomly guessed the cell’s identity."
