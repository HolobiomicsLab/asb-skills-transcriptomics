---
name: rarity-and-subtlety-parameter-sweep-analysis
description: Use when you must determine how far MetaNeighbor AUROC performance degrades
  as a held-out cell type becomes rare (1-20% incidence) or only subtly different
  from the out-group, before trusting a replicability call.
when_to_use_negative:
- Cell-type labels are unknown or ambiguous across datasets — use the unsupervised
  MetaNeighbor variant (variableGenes + MetaNeighborUS) instead of the supervised
  sweep
- Gene sets contain fewer than two genes or gene names do not match the expression
  matrix row names — MetaNeighbor cannot run on such inputs
- The goal is discovering new subtypes rather than quantifying how robust identification
  of a known target type is under rarity/subtlety degradation
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MetaNeighbor
  role: 'Supervised neighbor-voting pipeline: builds cell-cell rank correlation network
    per gene set, performs leave-one-dataset-out cross-validation, returns AUROC scores'
  repo: https://github.com/gillislab/MetaNeighbor
- name: R
  role: Host language for loading datasets, subsampling cells to instantiate rarity/subtlety
    conditions, and running 100-repeat sweeps
- name: SummarizedExperiment
  role: Container for scRNA-seq expression data loaded into the R session
- name: Matrix
  role: Sparse matrix handling for large gene-by-sample expression matrices
- name: matrixStats
  role: Fast column/row statistics used during subsampling and standardization of
    network values
- name: gplots
  role: Heatmap rendering of cell type-by-cell type AUROC matrices (heatmap.2)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-03282-0
    title: Characterizing the replicability of cell types defined by single cell RNA-sequencing
      data using MetaNeighbor
  source_content_hash: sha256:68b7bd75bd1bc097438bc1b39ebf0cd5348bd1da521369f425a2733d8f0bcb23
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
  - gplots
  - matrixStats
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# rarity-and-subtlety-parameter-sweep-analysis

## Summary

Empirically stress-tests MetaNeighbor's supervised neighbor-voting replicability framework by sweeping target cell-type incidence (rarity, 1-20%) and transcriptional subtlety (up to ~88-90%) using the GO:0022857 transporter-activity gene set versus matched random 20-gene controls, quantifying robustness via mean AUROC (e.g. ~0.71 at ~88% subtlety for excitatory neurons).

## When to use

Use when you must determine how far MetaNeighbor AUROC performance degrades as a held-out cell type becomes rare (1-20% incidence) or only subtly different from the out-group, before trusting a replicability call.

## When NOT to use

- Cell-type labels are unknown or ambiguous across datasets — use the unsupervised MetaNeighbor variant (variableGenes + MetaNeighborUS) instead of the supervised sweep
- Gene sets contain fewer than two genes or gene names do not match the expression matrix row names — MetaNeighbor cannot run on such inputs
- The goal is discovering new subtypes rather than quantifying how robust identification of a known target type is under rarity/subtlety degradation

## Inputs

- Tasic, Zeisel, and Habib scRNA-seq gene-by-sample expression matrices
- GO:0022857 transporter-activity gene set (list of gene vectors matching expression matrix row names)
- Random 20-gene control sets size-matched to GO:0022857
- Experiment labels (dataset of origin per sample) and binary cell-type label matrix

## Outputs

- Mean AUROC per condition (rarity level x subtlety level) and per gene-set class
- rarity_subtlety_aurocs.csv condition-by-condition AUROC summary table
- Robustness profile: expected near-flat AUROC across 1-20% incidence, ~0.71 mean AUROC at ~88% subtlety

## How to apply

Load Tasic, Zeisel, and Habib scRNA-seq expression datasets in R and extract the GO:0022857 transporter-activity gene set (size-matched random 20-gene sets serve as the null). For each condition, subsample cells so the target type is instantiated at incidence levels spanning 1-20% and at subtlety levels up to ~88%, defining the held-out type against the remaining cell types as the out-group. For each of 100 repeats per condition, run MetaNeighbor's supervised pipeline: build a cell-cell rank correlation network for the gene set, standardize network values to 0-1, then perform leave-one-dataset-out cross-validation where all cell-type labels are hidden for one dataset at a time and a neighbor-voting predictor recovers the held-out type; report AUROC. Aggregate mean AUROCs per condition and gene-set class. The rationale is that AUROC reflects the probability of correct binary classification (0.5 = random), so near-flat AUROC across 1-20% incidence and ~0.71 at ~88% subtlety (dropping to chance only beyond ~99% subtlety) bounds the method's operating regime, with GO:0022857 outperforming random controls confirming the result is gene-set driven.

## Related tools

- **MetaNeighbor** (Supervised neighbor-voting pipeline: builds cell-cell rank correlation network per gene set, performs leave-one-dataset-out cross-validation, returns AUROC scores) — https://github.com/gillislab/MetaNeighbor
- **R** (Host language for loading datasets, subsampling cells to instantiate rarity/subtlety conditions, and running 100-repeat sweeps)
- **SummarizedExperiment** (Container for scRNA-seq expression data loaded into the R session)
- **Matrix** (Sparse matrix handling for large gene-by-sample expression matrices)
- **matrixStats** (Fast column/row statistics used during subsampling and standardization of network values)
- **gplots** (Heatmap rendering of cell type-by-cell type AUROC matrices (heatmap.2))

## Examples

```
library(MetaNeighbor); data(mn_data); data(GOmouse); AUROC_scores = MetaNeighbor(dat = mn_data, experiment_labels = as.numeric(factor(mn_data$study_id)), celltype_labels = mn_data@colData@metadata$cell_labels, gene_sets = GOmouse, i = 1)
```

## Evaluation signals

- Mean AUROC for GO:0022857 exceeds the matched random 20-gene control sets at every condition
- AUROC is nearly flat (performance nearly unaffected) across incidence levels from 1% to 20%
- Mean AUROC reaches ~0.71 at ~88% subtlety for excitatory neurons and falls toward chance (0.5) only beyond ~99% subtlety
- 100 repeats per condition are completed and aggregated; each fold uses leave-one-dataset-out cross-validation with all labels hidden for the test dataset
- Output table rarity_subtlety_aurocs.csv contains one row per condition with a numeric mean AUROC in [0, 1]

## Limitations

- Distinguishing only subtly different cell types is difficult for any analysis; beyond ~99% subtlety performance drops to chance, and discovery of such types requires appropriate controls
- Because of the ranking step, supervised MetaNeighbor scales poorly with total sample count (e.g. ~3653 s for 10 gene sets on 10,000 samples on a laptop), so 100-repeat sweeps are compute-heavy
- AUROC scores below the >0.95 'extreme confidence' regime must be interpreted cautiously; more moderate scores are still meaningful but reflect weaker replicability evidence

## Evidence

- [methods] Build rank-correlation network per gene set before neighbor voting: "first, we build a network of rank correlations between all cells for a gene set"
- [intro] Leave-one-dataset-out cross-validation defines the test set: [evidence span withheld — anchor status: not_found]
- [readme] Neighbor-voting predictor mechanism underlying AUROC output: "the neighbor voting predictor produces a weighted matrix of predicted labels by performing matrix multiplication between the network and the binary vector"
- [discussion] Subtle cell-type distinctions are the known difficulty this sweep quantifies: "Moving past this point and distinguishing between only subtly different types will be difficult for any analysis, and their discovery will require consideration of appropriate controls"
