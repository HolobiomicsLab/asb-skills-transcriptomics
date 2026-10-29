---
name: cross-dataset-cell-type-replicability-assessment
description: Use when you need to measure whether cell types (e.g., neurons vs non-neurons,
  excitatory vs inhibitory, interneuron subtypes) identified in one scRNA-seq study
  replicate in independent studies, or to find reciprocal top-hit cell-type pairs
  across datasets.
when_to_use_negative:
- Input has no cross-dataset replication question — e.g., a single-dataset analysis
  where no leave-one-dataset-out folds can be formed
- Gene sets contain fewer than two genes or gene names do not match the expression
  matrix row names — the README states this is critical for MetaNeighbor to run
- The goal is definitive biological validation of cell identity rather than transcriptomic
  replicability — the article notes orthogonal data will be required and current estimates
  are optimistic
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MetaNeighbor
  role: Core package implementing neighbor voting on cell-cell rank correlation networks
    (MetaNeighbor/MetaNeighborUS) plus variableGenes and topHits helpers for HVG selection
    and reciprocal-top-hit extraction
  repo: github.com/gillislab/MetaNeighbor
- name: SummarizedExperiment
  role: Container for the gene-by-sample expression matrix with study_id and cell_type
    metadata columns
- name: gplots
  role: heatmap.2 rendering of the cell type-by-cell type AUROC matrix
- name: RColorBrewer
  role: Supplies the RdYlBu palette (11 colors, 100 shades, breaks seq(0,1,length=101))
    for the AUROC heatmap
- name: R
  role: Analysis environment; MetaNeighbor is installed via BiocManager or devtools
    from GitHub
  repo: github.com/gillislab/MetaNeighbor
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_005
  - task_007
  - task_008
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-03282-0
    title: Characterizing the replicability of cell types defined by single cell RNA-sequencing
      data using MetaNeighbor
  source_content_hash: sha256:d60889f0ae0cd32b09f92fa5187bca71650cf131f9331160ac9a09ed21200542
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Matrix
  - MetaNeighbor
  - R
  - RColorBrewer
  - SingleCellExperiment
  - SummarizedExperiment
  - beanplot
  - gplots
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# cross-dataset-cell-type-replicability-assessment

## Summary

Quantifies how well cell types defined by scRNA-seq replicate across datasets using MetaNeighbor's neighbor-voting on a cell-cell rank correlation network, reporting AUROC scores for cell-type recovery under leave-one-dataset-out cross-validation, with both supervised (GO gene sets) and unsupervised (HVG) variants.

## When to use

Use when you need to measure whether cell types (e.g., neurons vs non-neurons, excitatory vs inhibitory, interneuron subtypes) identified in one scRNA-seq study replicate in independent studies, or to find reciprocal top-hit cell-type pairs across datasets.

## When NOT to use

- Input has no cross-dataset replication question — e.g., a single-dataset analysis where no leave-one-dataset-out folds can be formed
- Gene sets contain fewer than two genes or gene names do not match the expression matrix row names — the README states this is critical for MetaNeighbor to run
- The goal is definitive biological validation of cell identity rather than transcriptomic replicability — the article notes orthogonal data will be required and current estimates are optimistic

## Inputs

- Gene-by-sample scRNA-seq expression matrix (e.g., RPKM) as SummarizedExperiment/SingleCellExperiment with study_id and cell_type metadata
- Experiment labels indicating each sample's dataset of origin
- Binary cell-type label matrix (supervised mode) or sample metadata pheno table (unsupervised mode)
- Gene sets as a list of vectors (GOmouse GO terms, random matched-size controls) or an HVG vector from variableGenes (>= ~200 genes for >0.9 AUROC applicability)

## Outputs

- Gene set-by-cell type matrix of mean AUROC scores across leave-one-dataset-out folds (supervised mode)
- Cell type-by-cell type matrix of average AUROC scores (MetaNeighborUS, unsupervised mode)
- Reciprocal top hits / high-scoring cell-type pairs table (AUROC > 0.9 or >= 0.95 threshold)
- Beanplot or heatmap.2 visualization of AUROC distributions and matrices

## How to apply

Assemble the datasets as a SummarizedExperiment/SingleCellExperiment object with study_id experiment labels and binary cell-type labels, plus gene sets (GOmouse GO terms, matched-size random sets, or highly variable genes selected via variableGenes, which takes the top quartile of variable genes across all but the top decile of expression bins). For each gene set, build a network of Spearman rank correlations between all pairs of cells, re-rank network values to lie between 0 and 1, then run the neighbor-voting predictor by multiplying the network with the binary cell-type membership vector and dividing each element by the null predictor (node degree), so each cell scores the fraction of its neighbors (including itself) in a given type. Cycle through all leave-one-dataset-out folds, compute AUROC per fold, and report the mean AUROC across folds per gene set-by-cell type pair. For unsupervised replication, run MetaNeighborUS on the HVG set to get a cell type-by-cell type AUROC matrix, then extract replicate pairs with topHits at AUROC >= 0.95 (reciprocal top hits); visualize with gplots::heatmap.2 using the RdYlBu palette. Note the reported baselines: mean AUROC ~0.80 +/- 0.1 for neuron vs non-neuron across eight datasets and ~0.69 +/- 0.1 for excitatory vs inhibitory; HVGs reach ~0.99 for excitatory vs inhibitory.

## Related tools

- **MetaNeighbor** (Core package implementing neighbor voting on cell-cell rank correlation networks (MetaNeighbor/MetaNeighborUS) plus variableGenes and topHits helpers for HVG selection and reciprocal-top-hit extraction) — github.com/gillislab/MetaNeighbor
- **SummarizedExperiment** (Container for the gene-by-sample expression matrix with study_id and cell_type metadata columns)
- **gplots** (heatmap.2 rendering of the cell type-by-cell type AUROC matrix)
- **RColorBrewer** (Supplies the RdYlBu palette (11 colors, 100 shades, breaks seq(0,1,length=101)) for the AUROC heatmap)
- **R** (Analysis environment; MetaNeighbor is installed via BiocManager or devtools from GitHub) — github.com/gillislab/MetaNeighbor

## Examples

```
library(MetaNeighbor); data(mn_data); var_genes <- variableGenes(dat = mn_data, exp_labels = mn_data$study_id); celltype_NV <- MetaNeighborUS(var_genes = var_genes, dat = mn_data, study_id = mn_data$study_id, cell_type = mn_data$cell_type)
```

## Evaluation signals

- Mean AUROC across gene sets falls at 0.80 +/- 0.1 for neuron vs non-neuron across eight datasets, matching Fig. 2a; GO:0055085 exceeds 0.85
- Mean AUROC ~0.69 +/- 0.1 for cortical excitatory vs inhibitory neurons across four studies; GO:0019748 scores markedly higher than GO:0040011 (Fig. 2b–2c)
- HVG-based unsupervised run yields excitatory vs inhibitory AUROC ~0.99 and a matrix diagonal < 1, with cross-dataset pairs > 0.9 as reciprocal top hits
- topHits with threshold = 0.95 on Tasic/Zeisel/Paul data recovers 11 replicated interneuron subtypes covering 24/45 (53%) of types
- Reconstructed neighbor-voting AUROC matrix matches the deposited MetaNeighbor::MetaNeighbor() output on the same mn_data/GOmouse inputs

## Limitations

- Ranking step scales poorly in R — runtime grows steeply with sample count (10,000 samples x 10 gene sets took 3653 s in the README benchmarks); use fast_version = TRUE above ~10,000 cells
- Requires matching gene names between gene sets and the expression matrix, gene sets with more than one gene, and cell-type label row names matching expression column names
- Results are limited where studies cluster to different levels of specificity; distinguishing subtly different cell types is difficult and requires appropriate controls
- AUROC 0.5 indicates chance performance; AUROC scores reflect transcriptomic similarity only and are optimistic estimates of cell identity absent orthogonal data

## Evidence

- [methods] MetaNeighbor's operation: cell-type labels are hidden for one dataset at a time (the test set), and a neighbor-voting algorithm predicts the identity of those held-out cells: [evidence span withheld — anchor status: not_found]
- [readme] Neighbor voting divides the network prediction by node degree: "the neighbor voting predictor produces a weighted matrix of predicted labels by performing matrix multiplication between the network and the binary vector indicating cell type membership"
- [methods] HVG selection via variableGenes: "the function variableGenes, which picks the top quartile of variable genes across all but the top decile of expression bins for each dataset"
- [discussion] Replication threshold of reciprocal top hits with AUROC >= 0.95: "While reciprocal top hits and AUROCs >0.95 reflect extreme confidence in a highly concordant cell type, more moderate scores are still meaningful"
- [readme] AUROC interpretation as binary classification probability: "An AUROC score of 0.5 means that we have performed as well as if we had randomly guessed the cell’s identity."
