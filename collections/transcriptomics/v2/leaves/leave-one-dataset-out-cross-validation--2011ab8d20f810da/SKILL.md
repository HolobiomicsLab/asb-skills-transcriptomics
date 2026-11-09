---
name: leave-one-dataset-out-cross-validation
description: 'Use when assessing whether a cell type replicates across two or more
  scRNA-seq datasets: you have labeled cells per dataset, a gene set (e.g. GOmouse),
  and need an AUROC that reflects cross-dataset identity recovery rather than within-study
  clustering.'
when_to_use_negative:
- You lack corresponding cell-type labels across datasets — the supervised variant
  requires known correspondence; use unsupervised MetaNeighborUS with high-variance
  genes (variableGenes) instead
- All cells come from a single dataset — leave-one-dataset-out folds are undefined;
  there is no cross-dataset test set to hide
- The goal is de novo cluster discovery rather than scoring replication of already-defined
  cell types
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: MetaNeighbor
  role: Implements the neighbor-voting scoring and leave-one-dataset-out cross-validation
    loop; MetaNeighbor() on mn_data/GOmouse produces the reference AUROC matrix (MetaNeighborUS
    for the unsupervised variant)
  repo: github.com/gillislab/MetaNeighbor
- name: R
  role: Runtime environment in which the MetaNeighbor package, SummarizedExperiment
    sample data, and plotting libraries are loaded
- name: SummarizedExperiment
  role: Container format for the mn_data sample expression data (GSE60361/GSE71585
    cortical interneurons, RPKM)
- name: gplots
  role: heatmap.2 visualization of the celltype_NV AUROC matrix with an RdYlBu palette
    from RColorBrewer
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-03282-0
    title: Characterizing the replicability of cell types defined by single cell RNA-sequencing
      data using MetaNeighbor
  source_content_hash: sha256:02169432482a598df492c164302d5d629068207a4e98eed02eaa620c4cd6ce99
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaNeighbor
  - R
  - SummarizedExperiment
  - gplots
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# leave-one-dataset-out-cross-validation

## Summary

Cross-dataset validation scheme in which all cell-type labels are hidden for one dataset at a time (the test set) and a neighbor-voting predictor on a Spearman rank-correlation cell network predicts the held-out identities, with recovery measured as AUROC and averaged across folds per gene set-by-cell-type pair.

## When to use

Use when assessing whether a cell type replicates across two or more scRNA-seq datasets: you have labeled cells per dataset, a gene set (e.g. GOmouse), and need an AUROC that reflects cross-dataset identity recovery rather than within-study clustering.

## When NOT to use

- You lack corresponding cell-type labels across datasets — the supervised variant requires known correspondence; use unsupervised MetaNeighborUS with high-variance genes (variableGenes) instead
- All cells come from a single dataset — leave-one-dataset-out folds are undefined; there is no cross-dataset test set to hide
- The goal is de novo cluster discovery rather than scoring replication of already-defined cell types

## Inputs

- Gene-by-sample expression matrix ('data') for cells from multiple datasets (e.g. mn_data: cortical interneurons from GSE60361 and GSE71585, RPKM)
- Numerical vector of experiment labels indicating each sample's dataset of origin (study_id)
- Binary cell-type label matrix whose row names match the expression matrix column names
- Gene sets of interest as a list of vectors (e.g. GOmouse), with gene names matching the expression matrix

## Outputs

- AUROC score vector/matrix: mean AUROC across leave-one-dataset-out folds for each gene set-by-cell-type pair
- Per-dataset and per-cell-type AUROC breakdown (first output file) and a matrix of cell-type means across datasets (second output file)

## How to apply

For each gene set, build a network of Spearman rank correlations between all pairs of cells, then re-rank all network values and standardize them to lie between 0 and 1. Run the neighbor-voting predictor by matrix-multiplying the standardized network with the binary cell-type membership vector and dividing each element by the null predictor (node degree), so each cell is scored as the fraction of its neighbors (including itself) belonging to a given cell type. Cycle through every leave-one-dataset-out fold with one dataset held out as the test set and all labels hidden, compute AUROC for recovering same-type cells within each fold, and report the mean AUROC across folds for each gene set-by-cell-type pair. Scores >0.5 beat random guessing (e.g. neuron vs. non-neuronal classification), and AUROC >0.9 (or reciprocal top hits with >0.95) indicates high-confidence replication of a cell type across datasets.

## Related tools

- **MetaNeighbor** (Implements the neighbor-voting scoring and leave-one-dataset-out cross-validation loop; MetaNeighbor() on mn_data/GOmouse produces the reference AUROC matrix (MetaNeighborUS for the unsupervised variant)) — github.com/gillislab/MetaNeighbor
- **R** (Runtime environment in which the MetaNeighbor package, SummarizedExperiment sample data, and plotting libraries are loaded)
- **SummarizedExperiment** (Container format for the mn_data sample expression data (GSE60361/GSE71585 cortical interneurons, RPKM))
- **gplots** (heatmap.2 visualization of the celltype_NV AUROC matrix with an RdYlBu palette from RColorBrewer)

## Examples

```
celltype_NV = MetaNeighborUS(var_genes = var_genes, dat = mn_data, study_id = study_id, cell_type_labels = cell_types)
```

## Evaluation signals

- The reconstructed pipeline's AUROC matrix matches the AUROC values from the deposited MetaNeighbor::MetaNeighbor() call on the same mn_data/GOmouse inputs
- Standardized network values all lie in [0, 1] after the re-ranking step
- Mean AUROC across folds exceeds 0.5 for replicated types (e.g. Sst-Chodl/Int1, Smad3/Int14 subtypes score >0.5; high-confidence pairs score >0.9–0.95)
- Every possible leave-one-dataset-out fold is evaluated exactly once and per-fold AUROCs are averaged, not pooled
- Row names of the cell-type label matrix match the column names of the expression matrix, and gene-set gene names match the expression matrix row names

## Limitations

- The ranking step scales poorly with total sample count (e.g. 2 experiments × 10 cell types × 10,000 samples ≈ 3653 s on a laptop); speed-ups require parallelization
- Scores assume cells of the same type have more similar expression profiles than cells of different types; subtly different subtypes are difficult to distinguish and require appropriate controls
- Interpretation of intermediate AUROC values is context-dependent: reciprocal top hits and AUROC >0.95 indicate extreme confidence, but more moderate scores can still be meaningful

## Evidence

- [intro] Cross-dataset validation via hidden labels: [evidence span withheld — anchor status: not_found]
- [methods] Neighbor voting predictor definition: "the neighbor voting predictor produces a weighted matrix of predicted labels by performing matrix multiplication between the network and the binary vector indicating cell type membership"
- [readme] Network construction and null-predictor normalization: [evidence span withheld — anchor status: external_doc]
- [readme] Cross-validation and AUROC reporting: "For cross-validation, we permute through all possible combinations of leave-one-dataset-out cross-validation, and we report how well we can recover cells of the same type as area under the receiver"
- [readme] AUROC interpretation threshold: "An AUROC score of 0.5 means that we have performed as well as if we had randomly guessed the cell’s identity."
- [discussion] High-confidence replication threshold: "While reciprocal top hits and AUROCs >0.95 reflect extreme confidence in a highly concordant cell type, more moderate scores are still meaningful"
