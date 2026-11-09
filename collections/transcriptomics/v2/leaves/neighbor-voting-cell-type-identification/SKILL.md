---
name: neighbor-voting-cell-type-identification
description: Use when comparing cell types across two or more scRNA-seq datasets without
  shared labels, to identify reciprocal matches (AUROC >= 0.95) and assess how well
  novel subtypes replicate across studies.
when_to_use_negative:
- Corresponding cell type labels are already known across datasets — use the fully
  supervised MetaNeighbor mode instead of the unsupervised AUROC-matrix workflow
- The question is de novo clustering of cells within a single dataset rather than
  cross-dataset replication of pre-defined cell type labels
- Gene names in the gene sets do not match the expression matrix row names, or gene
  sets contain only one gene — MetaNeighbor will not run correctly
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: MetaNeighbor
  role: R/Bioconductor package implementing neighbor voting and AUROC scoring (MetaNeighborUS)
    plus HVG selection (variableGenes) and top-hit extraction (topHits)
  repo: https://github.com/gillislab/MetaNeighbor
- name: SummarizedExperiment
  role: Container for the fused gene-by-sample expression matrix with study and cell
    type metadata
- name: SingleCellExperiment
  role: Alternative container for combining datasets with cell-level labels before
    MetaNeighborUS
- name: gplots
  role: heatmap.2 used to plot the cell type AUROC matrix with a reversed RdYlBu color
    scale
- name: RColorBrewer
  role: Supplies the RdYlBu palette (brewer.pal(11, 'RdYlBu')) reversed for the AUROC
    heatmap
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-03282-0
    title: Characterizing the replicability of cell types defined by single cell RNA-sequencing
      data using MetaNeighbor
  source_content_hash: sha256:cb663f149870c14a893094cb50a86d7434348739794bf42b5b7cb65607e3f6a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gillislab/MetaNeighbor
  tools:
  - MetaNeighbor
  - R
  - RColorBrewer
  - SingleCellExperiment
  - SummarizedExperiment
  - gplots
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 22
---

# neighbor-voting-cell-type-identification

## Summary

MetaNeighbor-based quantification of cell type replicability across scRNA-seq datasets via neighbor voting, producing cell type-by-cell type mean AUROC scores from highly variable gene sets to identify replicated subtypes.

## When to use

Use when comparing cell types across two or more scRNA-seq datasets without shared labels, to identify reciprocal best matches (AUROC >= 0.95) and assess how well novel subtypes replicate across studies.

## When NOT to use

- Corresponding cell type labels are already known across datasets — use the fully supervised MetaNeighbor mode instead of the unsupervised AUROC-matrix workflow
- The question is de novo clustering of cells within a single dataset rather than cross-dataset replication of pre-defined cell type labels
- Gene names in the gene sets do not match the expression matrix row names, or gene sets contain only one gene — MetaNeighbor will not run correctly

## Inputs

- Gene-by-sample scRNA-seq expression matrix (SummarizedExperiment or SingleCellExperiment object)
- Per-cell study_id (dataset of origin) labels
- Per-cell cell_type labels
- Vector of highly variable gene names (from variableGenes) or functional gene sets (e.g., GOmouse)

## Outputs

- Cell type-by-cell type mean AUROC matrix (celltype_NV)
- Table of reciprocal top hits and high-scoring cell type pairs (topHits, AUROC >= 0.95)
- Heatmap of the AUROC matrix with reversed RdYlBu palette
- Summary of replicated subtypes and candidate marker genes (e.g., 11 replicated interneuron subtypes covering 24/45, 53%, of types)

## How to apply

Load each dataset (e.g., Tasic, Zeisel, Paul interneuron studies) into a combined SummarizedExperiment/SingleCellExperiment object with study_id and cell_type labels per cell. Select highly variable genes with variableGenes, which picks the top quartile of variable genes across all but the top decile of expression bins per dataset and returns the intersect (use the majority rule option if fewer than 200 genes result). Run MetaNeighborUS on the variable genes to obtain a cell type-by-cell type mean AUROC matrix (set fast_version = TRUE for fused datasets over 10,000 cells); internally, rank correlations between all cells form a network and neighbor voting predicts held-out labels under leave-one-dataset-out cross-validation. Extract replicate type pairs with topHits at threshold = 0.95, which selects reciprocal top hits with high AUROC, and visualize the matrix with gplots::heatmap.2 using a reversed RdYlBu palette (100 colors, breaks = seq(0, 1, length=101)).

## Related tools

- **MetaNeighbor** (R/Bioconductor package implementing neighbor voting and AUROC scoring (MetaNeighborUS) plus HVG selection (variableGenes) and top-hit extraction (topHits)) — https://github.com/gillislab/MetaNeighbor
- **SummarizedExperiment** (Container for the fused gene-by-sample expression matrix with study and cell type metadata)
- **SingleCellExperiment** (Alternative container for combining datasets with cell-level labels before MetaNeighborUS)
- **gplots** (heatmap.2 used to plot the cell type AUROC matrix with a reversed RdYlBu color scale)
- **RColorBrewer** (Supplies the RdYlBu palette (brewer.pal(11, 'RdYlBu')) reversed for the AUROC heatmap)

## Examples

```
celltype_NV = MetaNeighborUS(var_genes = var_genes, data = data, study_id = study_id, cell_type = cell_type, fast_version = TRUE)
```

## Evaluation signals

- AUROC matrix is square, symmetric, with cell types from all datasets as rows/columns and diagonal/cross-study entries in [0, 1]
- Identified replicate pairs are reciprocal top hits with mean AUROC >= 0.95 (the paper's extreme-confidence cutoff)
- Applied to the Tasic/Zeisel/Paul interneuron datasets, the workflow recovers 11 replicated subtypes corresponding to 24/45 (53%) of types
- A mean AUROC near 0.5 indicates random-guess-level performance and should not be counted as replication
- variableGenes returns a gene set matching expression matrix rownames, with >= 200 genes achievable via the majority rule option

## Limitations

- AUROC > 0.95 reciprocal top hits reflect extreme confidence, but moderate scores (e.g., >0.9) can still be meaningful; threshold choice affects subtype counts
- Hierarchical ambiguity: studies cluster to different levels of specificity, making one-to-one subtype matching difficult for subtly different types
- Scores are derived only from gene expression; orthogonal data are required to establish the biological basis of cell identity, so current estimates are optimistic
- Ranking step scales poorly with sample count; use fast_version = TRUE above 10,000 cells

## Evidence

- [abstract] MetaNeighbor quantifies cell type replicability: "quantifies the degree to which cell types replicate across datasets, and enables rapid identification of clusters with high similarity"
- [methods] HVG selection procedure: "the function variableGenes, which picks the top quartile of variable genes across all but the top decile of expression bins for each dataset"
- [methods] AUROC computation core: "first, we build a network of rank correlations between all cells for a gene set"
- [discussion] Reciprocal top hit threshold: "While reciprocal top hits and AUROCs >0.95 reflect extreme confidence in a highly concordant cell type, more moderate scores are still meaningful"
- [abstract] Replicated subtype counts: "We then apply this to novel interneuron subtypes, finding that 24/45 subtypes have evidence of replication"
- [readme] README workflow outline: "Identify reciprocal top hits and high scoring cell type pairs"
