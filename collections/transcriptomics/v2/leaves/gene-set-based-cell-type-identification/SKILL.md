---
name: gene-set-based-cell-type-identification
description: Use when the goal is to quantify cross-dataset replicability of known
  cell-type labels (e.g., neuron vs non-neuron across eight scRNA-seq studies) using
  gene sets and AUROC, rather than cluster-level unsupervised matching.
when_to_use_negative:
- Cell-type labels are absent, ambiguous, or unmatched across datasets — the supervised
  MetaNeighbor analysis requires corresponding cell types across datasets; use the
  unsupervised MetaNeighborUS/topHits reciprocal-top-hits workflow instead
- Gene sets contain a single gene or gene names do not match the expression matrix
  row names — the README states these must hold or MetaNeighbor cannot run
- Only one dataset is available with no cross-dataset validation possible — leave-one-dataset-out
  folds are undefined
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MetaNeighbor
  role: Runs neighbor-voting on the cell-cell rank correlation network over gene sets
    and returns AUROC scores across leave-one-dataset-out folds
  repo: https://github.com/gillislab/MetaNeighbor
- name: R
  role: Runtime environment in which the MetaNeighbor package is loaded and the analysis
    is executed
  repo: https://github.com/gillislab/MetaNeighbor
- name: SummarizedExperiment
  role: Data container used to hold the combined multi-study expression matrix with
    study_id and cell_type metadata
- name: gplots
  role: Provides heatmap.2 used to visualize the celltype_NV matrix of AUROC scores
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-03282-0
    title: Characterizing the replicability of cell types defined by single cell RNA-sequencing
      data using MetaNeighbor
  source_content_hash: sha256:bfe433076653bae51532178f864b74b89a53c93fcf197cec9e16ad4828ff7e2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gillislab/MetaNeighbor
  tools:
  - MetaNeighbor
  - R
  - SummarizedExperiment
  - beanplot
  - gplots
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# gene-set-based-cell-type-identification

## Summary

A supervised MetaNeighbor workflow that scores how well cell-type labels (e.g., neurons vs non-neuronal cells) replicate across scRNA-seq datasets by neighbor voting on a cell-cell rank correlation network built from functional (e.g., GOmouse) and random gene sets, reporting mean AUROC across leave-one-dataset-out folds.

## When to use

Use when the goal is to quantify cross-dataset replicability of known cell-type labels (e.g., neuron vs non-neuron across eight scRNA-seq studies) using gene sets and AUROC, rather than cluster-level unsupervised matching.

## When NOT to use

- Cell-type labels are absent, ambiguous, or unmatched across datasets — the supervised MetaNeighbor analysis requires corresponding cell types across datasets; use the unsupervised MetaNeighborUS/topHits reciprocal-top-hits workflow instead
- Gene sets contain a single gene or gene names do not match the expression matrix row names — the README states these must hold or MetaNeighbor cannot run
- Only one dataset is available with no cross-dataset validation possible — leave-one-dataset-out folds are undefined

## Inputs

- Gene-by-sample expression matrix (single SummarizedExperiment or SingleCellExperiment object) combining the eight Crow et al (2018) scRNA-seq datasets
- Experiment labels indicating the study_id of origin for each sample
- Binary cell-type label matrix (celltype_labels) with row names matching expression matrix column names
- Gene sets of interest as a list of vectors, e.g. GOmouse GO gene sets loaded via data(GOmouse), plus matched-size random gene set controls

## Outputs

- Vector of mean AUROC scores, one per gene set tested
- Gene set-by-cell type matrix of mean AUROC scores across leave-one-dataset-out cross-validation folds
- Per-cell-type AUROC distribution plotted with the MetaNeighbor beanplot (bplot=TRUE), comparable to Fig. 2a

## How to apply

Assemble the scRNA-seq studies into a single SummarizedExperiment object with study_id (experiment_labels) and binary celltype_labels metadata, ensuring gene-set names match the expression matrix row names and cell-type label row names match sample columns. Load the GOmouse GO gene sets from the MetaNeighbor package and generate matched-size random gene sets as controls. For each gene set, build a cell-cell rank correlation network, then run the neighbor-voting predictor, which matrix-multiplies the network by the binary cell-type membership vector and normalizes by node degree, iterating through all leave-one-dataset-out cross-validation folds. Report the gene set-by-cell type matrix of mean AUROC scores across folds; scores are interpretable as the probability of correctly ranking a held-out cell, with 0.5 equal to chance. For task one (neuron vs non-neuron across eight Crow et al. 2018 datasets), expect mean AUROC of about 0.80 +/- 0.1 with all gene sets, including random ones, significantly above chance; visualize per-cell-type AUROC distributions with the built-in beanplot (bplot=TRUE) as in Fig. 2a.

## Related tools

- **MetaNeighbor** (Runs neighbor-voting on the cell-cell rank correlation network over gene sets and returns AUROC scores across leave-one-dataset-out folds) — https://github.com/gillislab/MetaNeighbor
- **R** (Runtime environment in which the MetaNeighbor package is loaded and the analysis is executed) — https://github.com/gillislab/MetaNeighbor
- **SummarizedExperiment** (Data container used to hold the combined multi-study expression matrix with study_id and cell_type metadata)
- **gplots** (Provides heatmap.2 used to visualize the celltype_NV matrix of AUROC scores)

## Examples

```
library(MetaNeighbor); library(SummarizedExperiment); data(mn_data); data(GOmouse)
```

## Evaluation signals

- Mean AUROC across gene sets for task one (neuron vs non-neuron) falls at 0.80 +/- 0.1, reproducing Fig. 2a
- All gene sets, including randomly chosen matched-size sets, yield AUROC scores significantly higher than chance (0.5)
- The GO:0055085 transmembrane transport gene set achieves an AUROC score greater than 0.85
- Per-cell-type AUROC beanplot distributions visually match the reported Fig. 2a smoothed distributions
- AUROC values are reported as means across all leave-one-dataset-out cross-validation folds for each gene set

## Limitations

- Because there is a ranking step, the method scales poorly with total sample count — 10,000 samples with 10 gene sets took about 3653 s on a laptop; speed-ups require parallelization
- This is a fully supervised analysis requiring knowledge of corresponding cell types across datasets; more moderate AUROC scores below 0.95 indicate lower-confidence replicability
- AUROC of 0.5 means chance-level performance, so small effect sizes must be interpreted against random-gene-set controls
- Current cell-type replicability estimates are optimistic since most clusters are defined from gene expression alone and orthogonal data would be needed to establish biological identity

## Evidence

- [methods] Network construction from gene sets: "first, we build a network of rank correlations between all cells for a gene set"
- [methods] Neighbor-voting predictor mechanics: "the neighbor voting predictor produces a weighted matrix of predicted labels by performing matrix multiplication between the network"
- [intro] Cross-dataset validation by label hiding: [evidence span withheld — anchor status: not_found]
- [readme] Chance-level interpretation of AUROC: "An AUROC score of 0.5 means that we have performed as well as if we had randomly guessed the cell’s identity."
- [abstract] Gene sets identify replicable cell types: "Across tasks we find that large sets of variably expressed genes can identify replicable cell types with high accuracy"
- [readme] Input requirements from README: [evidence span withheld — anchor status: external_doc]
