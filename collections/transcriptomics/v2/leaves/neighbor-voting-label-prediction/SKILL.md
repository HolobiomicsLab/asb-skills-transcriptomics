---
name: neighbor-voting-label-prediction
description: Use when you need to quantify how well cell-type labels replicate across
  two or more scRNA-seq datasets by predicting labels for cells from one held-out
  dataset based on their similarity to labeled cells in training datasets.
when_to_use_negative:
- Cell-type labels are ambiguous or missing across datasets — use the unsupervised
  variant (MetaNeighborUS with variableGenes and topHits) instead of the fully supervised
  neighbor-voting pipeline.
- There is only one dataset of origin, since cross-dataset validation requires hiding
  labels for one dataset at a time and scoring recovery on held-out data.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MetaNeighbor
  role: Implements the rank-correlation network construction, neighbor-voting predictor,
    leave-one-dataset-out cross-validation, and AUROC scoring (MetaNeighbor() and
    MetaNeighborUS() functions)
  repo: github.com/gillislab/MetaNeighbor
- name: R
  role: Runtime environment in which the MetaNeighbor package is loaded and executed
- name: SummarizedExperiment
  role: Container format for the gene-by-sample expression matrix with study_id and
    cell_type metadata
- name: gplots
  role: Heatmap visualization of the returned celltype_NV AUROC matrix via gplots::heatmap.2
provenance:
  source_task_ids:
  - task_001
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-03282-0
    title: Characterizing the replicability of cell types defined by single cell RNA-sequencing
      data using MetaNeighbor
  source_content_hash: sha256:249c3a82936e2913c80a4162f960e5208fc9f2d558d3be8cc729ed3546bc5f43
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaNeighbor
  - R
  - SummarizedExperiment
  - beanplot
  - gplots
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# neighbor-voting-label-prediction

## Summary

MetaNeighbor's neighbor-voting algorithm predicts held-out cell-type identities across scRNA-seq datasets by building a cell-cell Spearman rank correlation network for a gene set and voting with labeled training cells, scoring recovery with mean AUROC under leave-one-dataset-out cross-validation.

## When to use

Use when you need to quantify how well cell-type labels replicate across two or more scRNA-seq datasets by predicting labels for cells from one held-out dataset based on their similarity to labeled cells in training datasets.

## When NOT to use

- Cell-type labels are ambiguous or missing across datasets — use the unsupervised variant (MetaNeighborUS with variableGenes and topHits) instead of the fully supervised neighbor-voting pipeline.
- There is only one dataset of origin, since cross-dataset validation requires hiding labels for one dataset at a time and scoring recovery on held-out data.

## Inputs

- gene-by-sample expression matrix ('data', e.g., SummarizedExperiment mn_data with cortical interneurons from GSE60361 and GSE71585, RPKM)
- numerical vector of experiment labels indicating each sample's dataset of origin (study_id)
- binary matrix of cell type labels whose row names match expression matrix column names
- gene sets of interest provided as a list of vectors (e.g., GOmouse), with gene names matching the expression matrix and >1 gene per set

## Outputs

- vector of mean AUROC scores, one per gene set tested
- list containing the AUROC score for each dataset and cell type (first output file)
- matrix of mean AUROC scores per cell type across datasets (second output file, e.g., auroc_scores.csv)

## How to apply

Encode the data as a gene-by-sample expression matrix (e.g., SummarizedExperiment such as mn_data) with experiment labels (study_id) and a binary cell-type membership matrix whose row names match the expression matrix column names. For each gene set (e.g., GOmouse), compute Spearman rank correlations between all pairs of cells, then re-rank and standardize network values to lie between 0 and 1. Run the neighbor-voting predictor by matrix-multiplying the standardized network with the binary cell-type membership vector and dividing each element by the null predictor (node degree), giving each cell the fraction of its neighbors (including itself) in a given cell type. Cycle through all leave-one-dataset-out folds with one dataset's labels hidden as the test set, compute AUROC per fold, and report the mean AUROC across folds for each gene set-by-cell type pair. Interpretation: 0.5 equals chance; scores >0.5 indicate above-chance recovery, and AUROCs >0.95 reflect extreme confidence in highly concordant cell types.

## Related tools

- **MetaNeighbor** (Implements the rank-correlation network construction, neighbor-voting predictor, leave-one-dataset-out cross-validation, and AUROC scoring (MetaNeighbor() and MetaNeighborUS() functions)) — github.com/gillislab/MetaNeighbor
- **R** (Runtime environment in which the MetaNeighbor package is loaded and executed)
- **SummarizedExperiment** (Container format for the gene-by-sample expression matrix with study_id and cell_type metadata)
- **gplots** (Heatmap visualization of the returned celltype_NV AUROC matrix via gplots::heatmap.2)

## Examples

```
library(MetaNeighbor); data(mn_data); data(GOmouse); AUROC_scores = MetaNeighbor(dat = mn_data, experiment_labels = mn_data$study_id, celltype_labels = mn_data$cell_type, gene_sets = GOmouse, bplot = TRUE)
```

## Evaluation signals

- The reconstructed module's AUROC score matrix matches the reference MetaNeighbor::MetaNeighbor() output on the same mn_data/GOmouse inputs.
- Mean AUROC across gene sets is ~0.5-equivalent chance behavior only for random labels; for task one across eight datasets, mean AUROC = 0.80 +/- 0.1 and all gene sets including random sets score significantly above chance.
- GO:0055085 (transmembrane transport) gene set achieves an AUROC > 0.85; scores >0.5 indicate above-chance recovery of subtypes such as Sst-Chodl/Int1 and Smad3/Int14.
- Schema check: output is a gene set-by-cell type matrix of mean AUROCs across cross-validation folds, with row/column names matching gene sets and cell types.

## Limitations

- Because of the ranking step, the code does not scale well in R; runtime grows steeply with sample count (e.g., 10 experiments/10 gene sets/10,000 samples ≈ 3653 s on a laptop).
- Genes within gene sets must match the expression matrix gene names and contain more than one gene, and cell-type label row names must match expression matrix column names, or the run fails.
- Scores AUROC >0.95 indicate extreme confidence, but more moderate scores are still meaningful; distinguishing subtly different cell types is difficult and requires appropriate controls.
- AUROC reflects probability of correct binary classification only within the supervised setting; unlabeled or ambiguous cell types need the unsupervised heuristic variant.

## Evidence

- [methods] neighbor-voting predictor divides by null predictor (node degree): "the neighbor voting predictor produces a weighted matrix of predicted labels by performing matrix multiplication between the network and the binary vector indicating cell type membership"
- [methods] rank correlation network built per gene set: "first, we build a network of rank correlations between all cells for a gene set"
- [intro] cross-dataset validation with hidden labels: [evidence span withheld — anchor status: not_found]
- [readme] AUROC interpretation from README: "An AUROC score of 0.5 means that we have performed as well as if we had randomly guessed the cell’s identity."
- [discussion] high-confidence threshold: "While reciprocal top hits and AUROCs >0.95 reflect extreme confidence in a highly concordant cell type, more moderate scores are still meaningful"
