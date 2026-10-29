---
name: auroc-cross-validation-scoring
description: Use when you need a quantitative, comparable score (AUROC) for how well
  a cell-type label replicates across independent scRNA-seq datasets, e.g. neuron
  vs non-neuron classification across eight diverse studies.
when_to_use_negative:
- Cell-type labels across datasets are unknown or ambiguous — use unsupervised MetaNeighborUS
  with variableGenes-selected high-variance genes and reciprocal top hits instead
- Datasets have too few samples for reliable ranking — runtime scales poorly (e.g.
  ~3653 s for 10000 samples, 10 gene sets on a laptop)
- You need absolute cell-type annotation of new cells rather than a replicability
  score between labeled datasets
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MetaNeighbor
  role: Runs neighbor-voting with leave-one-dataset-out cross-validation and returns
    AUROC scores for each gene set and cell type
  repo: github.com/gillislab/MetaNeighbor
- name: R
  role: Execution environment; the MetaNeighbor Bioconductor package is loaded into
    an R session
- name: SummarizedExperiment
  role: Container format for the combined multi-study expression matrix with study_id
    and cell_type metadata
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-03282-0
    title: Characterizing the replicability of cell types defined by single cell RNA-sequencing
      data using MetaNeighbor
  source_content_hash: sha256:17acc46e85badb18229b8f8ff033db002fc1606bcd41890c282e38faec38e996
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaNeighbor
  - R
  - SummarizedExperiment
  - beanplot
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# auroc-cross-validation-scoring

## Summary

Score cell-type replicability across scRNA-seq datasets as mean AUROC from leave-one-dataset-out cross-validation of MetaNeighbor's neighbor-voting predictor on a cell-cell rank correlation network, as applied in Crow et al (2018) to tasks such as neurons vs non-neuronal cells.

## When to use

Use when you need a quantitative, comparable score (AUROC) for how well a cell-type label replicates across independent scRNA-seq datasets, e.g. neuron vs non-neuron classification across eight diverse studies.

## When NOT to use

- Cell-type labels across datasets are unknown or ambiguous — use unsupervised MetaNeighborUS with variableGenes-selected high-variance genes and reciprocal top hits instead
- Datasets have too few samples for reliable ranking — runtime scales poorly (e.g. ~3653 s for 10000 samples, 10 gene sets on a laptop)
- You need absolute cell-type annotation of new cells rather than a replicability score between labeled datasets

## Inputs

- Gene-by-sample scRNA-seq expression matrix (e.g. SummarizedExperiment / SingleCellExperiment object)
- Experiment labels indicating the source dataset of each sample (study_id)
- Binary cell-type label matrix with row names matching expression matrix column names
- Gene sets of interest as a list of vectors (e.g. GOmouse GO gene sets, matched-size random gene sets)

## Outputs

- Vector of mean AUROC scores per gene set tested
- List of AUROC scores for each dataset and cell type across cross-validation folds
- Gene set-by-cell type matrix of mean AUROC scores across folds
- Optional beanplot of smoothed AUROC distribution per cell type (bplot=TRUE)

## How to apply

Format the data as a SummarizedExperiment object with study_id (experiment_labels) and cell_type metadata, ensuring gene names in each gene set match the row names of the expression matrix. Build a network of rank correlations between all cells for each gene set (functional sets like GOmouse, or matched-size random gene sets as controls), then run the neighbor-voting predictor, which multiplies the network by the binary cell-type membership vector and divides by node degree. Permute through all leave-one-dataset-out cross-validation folds, hiding all cell-type labels for one dataset at a time to serve as the test set, and compute the AUROC per fold. Report the mean AUROC across folds for each gene set-by-cell type pair; 0.5 indicates chance-level performance. For the neuron vs non-neuron task, validate against the reported mean AUROC of 0.80 +/- 0.1 across gene sets, and optionally render results with the built-in beanplot (bplot=TRUE).

## Related tools

- **MetaNeighbor** (Runs neighbor-voting with leave-one-dataset-out cross-validation and returns AUROC scores for each gene set and cell type) — github.com/gillislab/MetaNeighbor
- **R** (Execution environment; the MetaNeighbor Bioconductor package is loaded into an R session)
- **SummarizedExperiment** (Container format for the combined multi-study expression matrix with study_id and cell_type metadata)

## Examples

```
library(MetaNeighbor); data(mn_data); data(GOmouse); AUROC_scores = MetaNeighbor(dat = mn_data, experiment_labels = as.numeric(factor(mn_data$study_id)), celltype_labels = mn_data@colData@metadata$cell_types, gene_sets = GOmouse, bplot = TRUE)
```

## Evaluation signals

- Mean AUROC across gene sets for the neuron vs non-neuron task falls at 0.80 +/- 0.1, matching Fig. 2a
- GO:0055085 transmembrane transport gene set achieves an AUROC score greater than 0.85
- AUROC scores are significantly above chance (0.5) for all gene sets tested, including randomly chosen sets
- Row names of the cell-type label matrix match column names of the expression matrix and all gene-set genes match the expression matrix gene names
- All leave-one-dataset-out folds complete and per-fold AUROC values are present in the returned output files

## Limitations

- The ranking step scales poorly in R: estimated runtime reaches ~3653 s for 10000 samples with 10 gene sets on a laptop; parallelization or MRAN can speed it up
- Gene sets must contain more than one gene and gene names must match the expression matrix exactly or the run fails
- Fully supervised mode requires known corresponding cell types across datasets; AUROC >0.95 and reciprocal top hits reflect extreme confidence, but moderate scores are still meaningful

## Evidence

- [results] AUROC scores were significantly higher than chance for all gene sets tested, including randomly chosen sets, with mean AUROC = 0.80 +/- 0.1: [evidence span withheld — anchor status: not_found]
- [methods] Neighbor voting via rank correlation network with leave-one-dataset-out CV: [evidence span withheld — anchor status: external_doc]
- [intro] Cross-dataset validation hides labels for one dataset at a time: [evidence span withheld — anchor status: not_found]
- [readme] AUROC interpretation as probability of correct binary classification: "An AUROC score of 0.5 means that we have performed as well as if we had randomly guessed the cell’s identity."
- [readme] Input-matching requirements for MetaNeighbor to run: [evidence span withheld — anchor status: external_doc]
