---
name: gene-ontology-geneset-scoring
description: Use when you have scRNA-seq data with known cell-type labels across datasets
  and want to test whether specific functional gene sets (e.g., GO terms) carry enough
  cell-type signal to recover labels via cross-dataset neighbor voting.
when_to_use_negative:
- Cell-type labels are absent or ambiguous — the supervised MetaNeighbor requires
  known corresponding cell types across datasets; use the unsupervised mode (MetaNeighborUS
  with variableGenes/topHits) instead
- Gene sets cannot be matched to the expression matrix gene names, or contain fewer
  than two genes — MetaNeighbor will not run
- You need per-cell subtype classification rather than scoring of predefined binary
  labels (e.g., neuron vs non-neuron, excitatory vs inhibitory)
tools:
- name: MetaNeighbor
  role: Builds rank-correlation cell networks per gene set and runs supervised leave-one-dataset-out
    neighbor voting to produce AUROC scores
  repo: github.com/gillislab/MetaNeighbor
- name: R
  role: Runtime environment in which the MetaNeighbor package and plotting are executed
- name: SummarizedExperiment
  role: Container for the multi-study expression data with study and cell-type label
    metadata
- name: gplots
  role: heatmap.2 used to render the gene set-by-cell type AUROC score matrix
- name: RColorBrewer
  role: Supplies the RdYlBu palette for the AUROC heatmap
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-03282-0
    title: Characterizing the replicability of cell types defined by single cell RNA-sequencing
      data using MetaNeighbor
  source_content_hash: sha256:3633ee9ce69fa0cc5603b30a9e52f24849c8e755de8d89ac0f1737ef9652d3c7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaNeighbor
  - R
  - RColorBrewer
  - SummarizedExperiment
  - gplots
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# gene-ontology-geneset-scoring

## Summary

Score how well Gene Ontology gene sets discriminate predefined cell types across scRNA-seq datasets using MetaNeighbor's supervised neighbor-voting framework, yielding a gene set-by-cell type AUROC matrix (e.g., mean AUROC ~0.69 ± 0.1 for cortical excitatory vs inhibitory neurons, with GO:0019748 distinguishing neurons from non-neurons while GO:0040011 distinguishes neuron classes).

## When to use

Use when you have scRNA-seq data with known cell-type labels across datasets and want to test whether specific functional gene sets (e.g., GO terms) carry enough cell-type signal to recover labels via cross-dataset neighbor voting.

## When NOT to use

- Cell-type labels are absent or ambiguous — the supervised MetaNeighbor requires known corresponding cell types across datasets; use the unsupervised mode (MetaNeighborUS with variableGenes/topHits) instead
- Gene sets cannot be matched to the expression matrix gene names, or contain fewer than two genes — MetaNeighbor will not run
- You need per-cell subtype classification rather than scoring of predefined binary labels (e.g., neuron vs non-neuron, excitatory vs inhibitory)

## Inputs

- Gene-by-sample expression matrix ('data') for multiple scRNA-seq studies
- Numerical vector of experiment labels indicating each sample's dataset of origin
- Matrix of cell-type labels (row names must match expression matrix column names)
- Gene sets of interest as a list of vectors (e.g., GOmouse GO terms) matching the expression matrix gene names, with >1 gene per set

## Outputs

- Vector of mean AUROC scores per gene set tested
- List of AUROC scores per dataset and cell type (first output file)
- Matrix of mean AUROC per cell type across datasets (second output file)
- Gene set-by-cell type AUROC heatmap (gplots::heatmap.2, RdYlBu palette) and beanplot of per-cell-type AUROC distributions

## How to apply

Load the studies as a SummarizedExperiment object with study identifiers and binary cell-type labels (e.g., 2809 excitatory and 1162 inhibitory cortical neurons). For each GO gene set, build a network of rank correlations between all cells and standardize network values to [0,1]. Run the neighbor-voting predictor, which produces predicted labels by matrix multiplication between the network and the binary cell-type membership vector, dividing each element by the null predictor (node degree); use leave-one-dataset-out cross-validation, repeating for all folds and reporting the mean AUROC. Aggregate results into a gene set-by-cell type AUROC matrix and visualize per-cell-type distributions (beanplot) and the heatmap via gplots::heatmap.2 with RColorBrewer's RdYlBu palette. Validate by checking the mean AUROC (~0.69 ± 0.1) and confirming gene-set specificity, e.g., GO:0019748 scores markedly higher than GO:0040011 for the neuron-vs-non-neuron contrast.

## Related tools

- **MetaNeighbor** (Builds rank-correlation cell networks per gene set and runs supervised leave-one-dataset-out neighbor voting to produce AUROC scores) — github.com/gillislab/MetaNeighbor
- **R** (Runtime environment in which the MetaNeighbor package and plotting are executed)
- **SummarizedExperiment** (Container for the multi-study expression data with study and cell-type label metadata)
- **gplots** (heatmap.2 used to render the gene set-by-cell type AUROC score matrix)
- **RColorBrewer** (Supplies the RdYlBu palette for the AUROC heatmap)

## Examples

```
library(MetaNeighbor); data(mn_data); data(GOmouse); aurocs <- MetaNeighbor(data = mn_data, experiment_labels = as.numeric(factor(mn_data$study_id)), celltype_labels = mn_data@colData@metadata$cell_labels, gene_sets = GOmouse, fileext = "GOmouse")
```

## Evaluation signals

- Mean AUROC across gene sets for the cortical excitatory vs inhibitory task is approximately 0.69 ± 0.1 (Fig. 2b)
- GO:0019748 yields markedly higher scores than GO:0040011, confirming gene-set specificity (Fig. 2c)
- An AUROC of 0.5 indicates chance-level performance — scores should exceed 0.5 for replicable signal
- Gene set names in the gene-set list must match expression matrix gene names, and cell-type label row names must match expression matrix column names, or the run fails
- AUROC scores from each cross-validation fold should be aggregated as the mean across folds per gene set-by-cell type combination

## Limitations

- The ranking step does not scale well in R; runtime grows steeply with sample count (e.g., 10,000 samples × 10 gene sets ≈ 3653 s)
- AUROC of 0.5 means performance no better than random guessing of cell identity
- Requires fully supervised knowledge of corresponding cell types across datasets; without informative labels, use the unsupervised variant

## Evidence

- [methods] build a cell-cell rank correlation network for a gene set: "first, we build a network of rank correlations between all cells for a gene set"
- [methods] neighbor voting predictor computes predicted labels via network multiplication: "the neighbor voting predictor produces a weighted matrix of predicted labels by performing matrix multiplication between the network and the binary vector indicating cell type membership"
- [readme] MetaNeighbor inputs and AUROC output definition: "MetaNeighbor takes four inputs: a gene-by-sample expression matrix (“data”), a set of labels indicating each sample’s dataset of origin (“experiment labels”)"
- [methods] cross-validation scheme: "we permute through all possible combinations of leave-one-dataset-out cross-validation, and we report how well we can recover cells of the same type as area under the receiver operator characteristic"
