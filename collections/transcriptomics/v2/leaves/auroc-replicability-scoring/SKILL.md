---
name: auroc-replicability-scoring
description: Use when cell-type labels exist for multiple scRNA-seq datasets and you
  need to quantify cross-dataset replicability of a cell type (or validate a reconstructed
  scoring module) via AUROC from neighbor voting over a cell-cell correlation network.
when_to_use_negative:
- Cell-type labels are absent or ambiguous across datasets — use unsupervised MetaNeighbor
  (variableGenes + MetaNeighborUS + topHits) instead of the supervised voting pipeline
- The goal is within-dataset cell typing/clustering rather than cross-dataset replicability
  assessment; AUROC here only measures recovery of same-type held-out cells
- Gene set names do not match expression matrix gene names or contain fewer than two
  genes, so the correlation network cannot be built as specified
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: MetaNeighbor
  role: Reference implementation whose neighbor-voting and leave-one-dataset-out AUROC
    output serves as the deposited baseline the reconstructed module must match
  repo: https://github.com/gillislab/MetaNeighbor
- name: R
  role: Runtime environment in which the reconstructed scoring module and the MetaNeighbor::MetaNeighbor()
    baseline are executed
- name: SummarizedExperiment
  role: Container holding the mn_data expression matrix, study_id and cell-type label
    metadata consumed by the pipeline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-03282-0
    title: Characterizing the replicability of cell types defined by single cell RNA-sequencing
      data using MetaNeighbor
  source_content_hash: sha256:7194d340958a7d27881eda9dbed4b4fe0824a14ca098823718a69281f1bffe82
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gillislab/MetaNeighbor
  tools:
  - MetaNeighbor
  - R
  - SummarizedExperiment
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# auroc-replicability-scoring

## Summary

Reconstructs MetaNeighbor's neighbor-voting scoring pipeline from its Methods description: Spearman rank-correlation cell-cell networks built per gene set are combined with leave-one-dataset-out cross-validation to produce AUROC scores quantifying how well cell-type identities replicate across scRNA-seq datasets.

## When to use

Use when cell-type labels exist for multiple scRNA-seq datasets and you need to quantify cross-dataset replicability of a cell type (or validate a reconstructed scoring module) via AUROC from neighbor voting over a cell-cell correlation network.

## When NOT to use

- Cell-type labels are absent or ambiguous across datasets — use unsupervised MetaNeighbor (variableGenes + MetaNeighborUS + topHits) instead of the supervised voting pipeline
- The goal is within-dataset cell typing/clustering rather than cross-dataset replicability assessment; AUROC here only measures recovery of same-type held-out cells
- Gene set names do not match expression matrix gene names or contain fewer than two genes, so the correlation network cannot be built as specified

## Inputs

- Gene-by-sample expression matrix (e.g. mn_data SummarizedExperiment of cortical interneurons from GSE60361 and GSE71585, RPKM)
- Numerical vector of experiment labels indicating each sample's dataset of origin (study_id)
- Binary cell-type label matrix with row names matching expression matrix column names
- Gene sets as a list of vectors (e.g. GOmouse), with gene names matching the expression matrix

## Outputs

- AUROC score vector (mean per gene set)
- AUROC score matrix of per-fold/per-dataset scores for each gene set-by-cell type pair (written to e.g. auroc_scores.csv)
- Cell type-by-cell type matrix of mean AUROC scores across datasets

## How to apply

Load the mn_data sample SummarizedExperiment (cortical interneurons from GSE60361 and GSE71585, RPKM) and the GOmouse gene sets from the MetaNeighbor package, encoding experiment labels via study_id and cell-type membership as a binary label matrix. For each gene set, compute Spearman rank correlations between all pairs of cells to build the cell-cell network, then re-rank all network values and standardize them to lie between 0 and 1. Run the neighbor-voting predictor by matrix-multiplying the standardized network with the binary cell-type membership vector and dividing each element by the null predictor (node degree), so each cell scores the fraction of its neighbors (including itself) belonging to a given cell type. Perform leave-one-dataset-out cross-validation over all folds, hiding labels for one dataset (the test set) at a time, and average AUROC across folds for each gene set-by-cell type pair. Judge correctness by comparing the module's AUROC matrix against scores from the deposited MetaNeighbor::MetaNeighbor() call on the same inputs; scores >0.5 indicate above-chance recovery, and highly concordant pairs score ≥0.95.

## Related tools

- **MetaNeighbor** (Reference implementation whose neighbor-voting and leave-one-dataset-out AUROC output serves as the deposited baseline the reconstructed module must match) — https://github.com/gillislab/MetaNeighbor
- **R** (Runtime environment in which the reconstructed scoring module and the MetaNeighbor::MetaNeighbor() baseline are executed)
- **SummarizedExperiment** (Container holding the mn_data expression matrix, study_id and cell-type label metadata consumed by the pipeline)

## Examples

```
data(mn_data); data(GOmouse); AUROC_scores = MetaNeighbor(dat = mn_data, experiment_labels = as.numeric(factor(pData(mn_data)$study_id)), celltype_labels = colData(mn_data)$cell_type, gene_sets = GOmouse, file_ext = 'default')
```

## Evaluation signals

- Reconstructed module's AUROC score matrix matches the AUROC values produced by the deposited MetaNeighbor::MetaNeighbor() call on the same mn_data/GOmouse inputs
- Mean AUROC per gene set is >0.5 (random-guess baseline) for the Sst-Chodl/Int1 and Smad3/Int14 subtypes, indicating above-chance recovery
- Per-fold AUROC values are computed for every leave-one-dataset-out fold and averaged, so each gene set-by-cell type pair has one mean AUROC
- Standardized network values lie in [0, 1] after re-ranking, and each cell's score equals the fraction of neighbors (including itself) of the given cell type
- Results are written to auroc_scores.csv with rows/columns keyed to gene sets and cell types

## Limitations

- The ranking step scales poorly with sample count in R — runtime grows steeply (e.g. 10000 samples x 10 gene sets took ~3653 s on a laptop); parallelization is advised
- It is a fully supervised analysis requiring knowledge of corresponding cell types across datasets; when labels are ambiguous, use the unsupervised variant
- AUROC of 0.5 means performance equivalent to random guessing; moderate scores are still meaningful but reciprocal top hits with AUROC >0.95 reflect extreme confidence
- Studies clustering to different levels of specificity limit interpretation, reflecting ambiguity about the term 'cell type'

## Evidence

- [methods] neighbor voting predictor produces weighted matrix of predicted labels: "the neighbor voting predictor produces a weighted matrix of predicted labels by performing matrix multiplication between the network and the binary vector indicating cell type membership"
- [readme] cross-validation and AUROC reporting: "For cross-validation, we permute through all possible combinations of leave-one-dataset-out cross-validation, and we report how well we can recover cells of the same type as area under the receiver"
- [readme] network construction and standardization: "MetaNeighbor runs as follows: first, we build a network of rank correlations between all cells for a gene set."
- [intro] cross-dataset validation design: [evidence span withheld — anchor status: not_found]
- [readme] AUROC interpretation: "An AUROC score of 0.5 means that we have performed as well as if we had randomly guessed the cell’s identity."
