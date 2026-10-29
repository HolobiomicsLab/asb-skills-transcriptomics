---
name: node-degree-null-standardization
description: 'Use when reconstructing or reimplementing MetaNeighbor''s neighbor-voting
  predictor from the Methods: you have a cell-cell similarity network plus a binary
  cell-type membership vector and need degree-normalized vote scores per cell for
  downstream AUROC computation.'
when_to_use_negative:
- Cell-type labels for the query dataset are already known and you only want a similarity
  map — use the unsupervised MetaNeighborUS/topHits route with variableGenes instead
  of supervised voting
- Gene sets contain fewer than two genes or gene names do not match the expression
  matrix row names — the predictor cannot run (README states both are critical requirements)
- The goal is de novo clustering rather than scoring replicability against existing
  labels
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MetaNeighbor
  role: Implements the rank-correlation network, standardization to 0-1, degree-normalized
    neighbor voting, and leave-one-dataset-out AUROC evaluation; used as the reference
    implementation the reconstructed module is validated against
  repo: github.com/gillislab/MetaNeighbor
- name: R
  role: Language in which the predictor is reconstructed via matrix multiplication
    of the standardized network with the binary membership vector and division by
    node degree
- name: SummarizedExperiment
  role: Container for the mn_data sample expression data consumed by the skill
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-03282-0
    title: Characterizing the replicability of cell types defined by single cell RNA-sequencing
      data using MetaNeighbor
  source_content_hash: sha256:c520732a8056d14b07c172e5d9bc4b0d6fb663a5584b0923749fc2ddb78bbd21
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaNeighbor
  - R
  - SummarizedExperiment
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Node-degree null standardization in neighbor voting

## Summary

Core MetaNeighbor scoring step: build a Spearman rank-correlation cell-cell network for a gene set, re-rank and standardize all network values to lie between 0 and 1, then compute each cell's vote as (standardized network x binary cell-type membership vector) divided by the null predictor (node degree), yielding the fraction of a cell's neighbors (including itself) in a given cell type.

## When to use

Use when reconstructing or reimplementing MetaNeighbor's neighbor-voting predictor from the Methods: you have a cell-cell similarity network plus a binary cell-type membership vector and need degree-normalized vote scores per cell for downstream AUROC computation.

## When NOT to use

- Cell-type labels for the query dataset are already known and you only want a similarity map — use the unsupervised MetaNeighborUS/topHits route with variableGenes instead of supervised voting
- Gene sets contain fewer than two genes or gene names do not match the expression matrix row names — the predictor cannot run (README states both are critical requirements)
- The goal is de novo clustering rather than scoring replicability against existing labels

## Inputs

- Gene-by-sample expression matrix (e.g., mn_data SummarizedExperiment, RPKM)
- Gene set of interest as a list of gene-name vectors (e.g., GOmouse)
- Vector of experiment/dataset labels per sample (study_id)
- Binary matrix of cell-type membership per sample, row names matching expression column names

## Outputs

- Per-cell vote score matrix (fraction of neighbors, including self, in each cell type)
- AUROC score per gene set-by-cell-type pair, averaged across leave-one-dataset-out folds (e.g., written to auroc_scores.csv)

## How to apply

First, compute Spearman rank correlations between all pairs of cells across datasets using only the genes in a given gene set (e.g., GOmouse terms applied to the mn_data cortical interneuron data from GSE60361 and GSE71585). Re-rank all network values and standardize them to lie between 0 and 1 so scores are comparable across gene sets. Multiply the standardized network by the binary vector/matrix indicating cell-type membership, then divide each element by the null predictor, i.e., the node degree, so each cell receives a score equal to the fraction of its neighbors (including itself) belonging to that cell type. Embed this predictor in leave-one-dataset-out cross-validation: hide all cell-type labels for one dataset at a time, score held-out cells, and compute AUROC for recovering same-type cells; average AUROC across folds per gene set-by-cell-type pair. The degree normalization is the null model that corrects for cells with many high-similarity neighbors, and scores >0.5 indicate above-chance recovery.

## Related tools

- **MetaNeighbor** (Implements the rank-correlation network, standardization to 0-1, degree-normalized neighbor voting, and leave-one-dataset-out AUROC evaluation; used as the reference implementation the reconstructed module is validated against) — github.com/gillislab/MetaNeighbor
- **R** (Language in which the predictor is reconstructed via matrix multiplication of the standardized network with the binary membership vector and division by node degree)
- **SummarizedExperiment** (Container for the mn_data sample expression data consumed by the skill)

## Examples

```
votes <- (net_std %*% celltype_labels) / rowSums(net_std)  # standardized Spearman network x binary membership, divided by node degree (null predictor)
```

## Evaluation signals

- Reconstructed module's AUROC matrix matches the scores from the deposited MetaNeighbor::MetaNeighbor() call on the same mn_data/GOmouse inputs
- Every vote score lies in [0, 1] after division by node degree (fraction-of-neighbors interpretation holds)
- Mean AUROC across leave-one-dataset-out folds is 0.5 for random gene sets (chance) and >0.5 for informative sets such as Sst-Chodl/Int1 and Smad3/Int14
- Network values are Spearman rank correlations, re-ranked and standardized to lie between 0 and 1 before voting
- Predictor output equals standardized network x binary membership matrix divided by row (node) degree, verifiable by element-wise comparison against the null predictor

## Limitations

- The ranking step scales poorly with total sample count in R (e.g., ~3653 s for 10000 samples, 2 experiments, 10 cell types on a laptop); speed-ups require parallelization
- AUROC = 0.5 means performance equal to random guessing; moderate scores still meaningful but only reciprocal top hits with AUROC > 0.95 (or > 0.9 for topHits filtering) reflect extreme confidence
- Fully supervised variant requires knowledge of corresponding cell types across datasets; ambiguous or uninformative labels need the unsupervised variant

## Evidence

- [methods] Degree normalization is the null predictor: "the neighbor voting predictor produces a weighted matrix of predicted labels by performing matrix multiplication between the network and the binary vector indicating cell type membership"
- [readme] Vote scores are fractions of neighbors: "each cell is given a score equal to the fraction of its neighbors (including itself), which are part of a given cell type"
- [intro] Network is built from pairwise rank correlations over a gene set: "We calculate correlations between all pairs of cells that we aim to compare across datasets based on the expression of a set of genes."
- [readme] AUROC chance level is 0.5: "An AUROC score of 0.5 means that we have performed as well as if we had randomly guessed the cell’s identity."
- [readme] Cross-validation protocol for the predictor: "we permute through all possible combinations of leave-one-dataset-out cross-validation, and we report how well we can recover cells of the same type"
