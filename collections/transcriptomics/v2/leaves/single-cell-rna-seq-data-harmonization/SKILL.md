---
name: single-cell-rna-seq-data-harmonization
description: Use when cell-type labels must be compared across two or more scRNA-seq
  studies with matching labels (e.g., excitatory vs inhibitory neurons), and you need
  quantitative AUROC evidence of cross-study replicability.
when_to_use_negative:
- Cell-type labels are ambiguous or absent across datasets — use the unsupervised
  MetaNeighborUS variant instead of the supervised framework
- Only one dataset is present — leave-one-dataset-out cross-validation requires at
  least two studies to compare
- Genes in the gene sets do not match the expression matrix row names, or gene sets
  contain fewer than two genes — MetaNeighbor will not run correctly
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MetaNeighbor
  role: supervised neighbor-voting framework computing rank-correlation cell networks
    and cross-dataset AUROC scores
  repo: github.com/gillislab/MetaNeighbor
- name: SummarizedExperiment
  role: container for multi-study expression data with experiment and cell-type labels
- name: gplots
  role: heatmap.2 rendering of the gene set-by-cell type AUROC matrix
- name: RColorBrewer
  role: supplies the RdYlBu palette for the AUROC heatmap
- name: SingleCellExperiment
  role: alternative Bioconductor container accepted by MetaNeighbor
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-018-03282-0
    title: Characterizing the replicability of cell types defined by single cell RNA-sequencing
      data using MetaNeighbor
  source_content_hash: sha256:a65a75b6b638dfe615f02f1de554f2481476b26b03babb2afa23bad8831c9a1e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaNeighbor
  - R
  - RColorBrewer
  - SingleCellExperiment
  - SummarizedExperiment
  - gplots
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# single-cell-rna-seq-data-harmonization

## Summary

Harmonize multiple scRNA-seq studies into a shared cell-type label space and quantify cross-dataset replicability with MetaNeighbor's supervised neighbor-voting AUROC framework (e.g., 2809 excitatory vs 1162 inhibitory cortical neurons across four studies, mean AUROC ~0.69 ± 0.1).

## When to use

Use when cell-type labels must be compared across two or more scRNA-seq studies with matching labels (e.g., excitatory vs inhibitory neurons), and you need quantitative AUROC evidence of cross-study replicability.

## When NOT to use

- Cell-type labels are ambiguous or absent across datasets — use the unsupervised MetaNeighborUS variant instead of the supervised framework
- Only one dataset is present — leave-one-dataset-out cross-validation requires at least two studies to compare
- Genes in the gene sets do not match the expression matrix row names, or gene sets contain fewer than two genes — MetaNeighbor will not run correctly

## Inputs

- gene-by-sample expression matrix ('data') shared across studies
- numerical experiment-label vector indicating each sample's dataset of origin
- binary cell-type label matrix with row names matching expression matrix column names (e.g., excitatory vs inhibitory)
- gene sets of interest as a list of vectors (e.g., GO terms such as GO:0019748, GO:0040011), names matching the expression matrix
- RPKM-normalized public datasets (e.g., GSE60361, GSE71585) or SummarizedExperiment/SingleCellExperiment objects

## Outputs

- vector of mean AUROC scores per gene set (mean across leave-one-dataset-out CV folds)
- gene set-by-cell type AUROC score matrix (celltype_NV)
- beanplot of AUROC score distributions per cell type and heatmap.2 of the AUROC matrix
- list of reciprocal top hits and high-scoring cell-type pairs (AUROC > 0.9 via topHits)

## How to apply

Load each study as a SummarizedExperiment/SingleCellExperiment object in R with dataset-of-origin (experiment) labels and binary cell-type labels whose row names match the expression matrix column names. Optionally restrict to informative features using variableGenes (top quartile of variable genes across all but the top decile of expression bins per dataset). For each gene set, MetaNeighbor builds a network of rank correlations between all cells, rescales network values to [0,1], then the neighbor-voting predictor multiplies the network by the binary cell-type vector and divides each element by the null predictor (node degree). Run leave-one-dataset-out cross-validation over all folds, reporting mean AUROC per gene set-by-cell type, then visualize with gplots::heatmap.2 and the RColorBrewer RdYlBu palette or beanplots. Validate by checking AUROC ~0.5 means chance-level (random guessing) and AUROC >0.9 (reciprocal top hits) indicates strong cross-dataset concordance.

## Related tools

- **MetaNeighbor** (supervised neighbor-voting framework computing rank-correlation cell networks and cross-dataset AUROC scores) — github.com/gillislab/MetaNeighbor
- **SummarizedExperiment** (container for multi-study expression data with experiment and cell-type labels)
- **gplots** (heatmap.2 rendering of the gene set-by-cell type AUROC matrix)
- **RColorBrewer** (supplies the RdYlBu palette for the AUROC heatmap)
- **SingleCellExperiment** (alternative Bioconductor container accepted by MetaNeighbor)

## Examples

```
library(MetaNeighbor); data(mn_data); celltype_NV = MetaNeighborUS(var_genes = var_genes, data = mn_data, pheno = pheno, celltypes = celltypes)
```

## Evaluation signals

- Mean AUROC across gene sets is approximately 0.69 ± 0.1 for the four-study excitatory vs inhibitory task (Fig. 2b)
- AUROC of 0.5 indicates chance-level performance; scores should be significantly above 0.5 for replicable cell types
- GO:0019748 (neuron vs non-neuron) yields markedly higher AUROC than GO:0040011 (excitatory vs inhibitory classes), as in Fig. 2c
- AUROC scores >0.9 combined with reciprocal top hits indicate high-confidence cell-type matches (topHits)
- Row names of the cell-type label matrix match column names of the expression matrix, and all gene-set genes are present in the matrix

## Limitations

- The ranking step limits scalability: run time grows steeply with sample count (e.g., ~3653 s for 10,000 samples, 10 gene sets on a laptop)
- Supervised analysis requires known corresponding cell types across datasets; label ambiguity breaks the assumption
- AUROC reflects ranking of same-type cells above other cells within a dataset, not absolute biological identity — orthogonal data is still required to establish cell identity
- Studies clustering to different levels of specificity complicate label harmonization, reflecting ambiguity in the term 'cell type'

## Evidence

- [readme] AUROC of 0.5 means chance performance: "An AUROC score of 0.5 means that we have performed as well as if we had randomly guessed the cell's identity."
- [readme] neighbor voting method: "the neighbor voting predictor produces a weighted matrix of predicted labels by performing matrix multiplication between the network and the binary vector indicating cell type membership"
- [readme] cross-validation scheme: "For cross-validation, we permute through all possible combinations of leave-one-dataset-out cross-validation, and we report how well we can recover cells of the same type as AUROC."
- [methods] variableGenes filter: "the function variableGenes, which picks the top quartile of variable genes across all but the top decile of expression bins for each dataset"
- [readme] input requirements: [evidence span withheld — anchor status: external_doc]
- [discussion] high-confidence matches: "While reciprocal top hits and AUROCs >0.95 reflect extreme confidence in a highly concordant cell type, more moderate scores are still meaningful"
