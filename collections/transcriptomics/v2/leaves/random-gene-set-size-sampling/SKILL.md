---
name: random-gene-set-size-sampling
description: Use when benchmarking cell-type replicability (e.g., neuron vs non-neuron
  AUROC) with random gene sets and you need to test how set size (100 vs 800 genes)
  or cross-validation variability bins influence MetaNeighbor performance.
when_to_use_negative:
- When the analysis goal is cross-dataset matching without known cell-type labels
  — unsupervised MetaNeighborUS with variableGenes (top-quartile variable genes) is
  the appropriate mode instead.
- When gene sets contain fewer than two genes or gene names do not match the expression
  matrix row names — MetaNeighbor requires matching names and multi-gene sets to run.
- When the question is identifying candidate marker genes for replicable subtypes
  rather than characterizing gene-set-size effects on AUROC.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MetaNeighbor
  role: supervised neighbor-voting framework run per gene set to produce mean AUROC
    scores via leave-one-dataset-out cross-validation
  repo: github.com/gillislab/MetaNeighbor
- name: R
  role: environment in which MetaNeighbor, data loading, random gene set sampling,
    and Wilcoxon/Spearman statistics are executed
- name: SummarizedExperiment
  role: container class for the mn_data sample expression data loaded with data(mn_data)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-03282-0
    title: Characterizing the replicability of cell types defined by single cell RNA-sequencing
      data using MetaNeighbor
  source_content_hash: sha256:bfd800cb22c4495bc762164aea69d3caa5b855c24fe5cf2eee74621d6d2cfc9f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaNeighbor
  - R
  - SummarizedExperiment
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# random-gene-set-size-sampling

## Summary

Skill for quantifying how gene set size and expression variability affect MetaNeighbor AUROC by sampling random gene sets (e.g., 100 sets of 100 genes vs 800 genes) from the mn_data/GOmouse R environment and running supervised MetaNeighbor with leave-one-dataset-out cross-validation.

## When to use

Use when benchmarking cell-type replicability (e.g., neuron vs non-neuron AUROC) with random gene sets and you need to test how set size (100 vs 800 genes) or cross-validation variability bins influence MetaNeighbor performance.

## When NOT to use

- When the analysis goal is cross-dataset matching without known cell-type labels — unsupervised MetaNeighborUS with variableGenes (top-quartile variable genes) is the appropriate mode instead.
- When gene sets contain fewer than two genes or gene names do not match the expression matrix row names — MetaNeighbor requires matching names and multi-gene sets to run.
- When the question is identifying candidate marker genes for replicable subtypes rather than characterizing gene-set-size effects on AUROC.

## Inputs

- gene-by-sample expression matrix (SummarizedExperiment, mn_data sample data)
- experiment labels indicating dataset of origin per sample
- cell type labels (e.g., neuron vs non-neuronal) with row names matching expression column names
- GOmouse functional gene sets (list of gene vectors)
- randomly sampled gene sets of fixed sizes (100 and 800 genes) from expressed genes

## Outputs

- gene set-by-cell type matrix of mean AUROC scores from supervised MetaNeighbor
- AUROC distributions for 100-gene and 800-gene sets with mean +/- sd (0.75 +/- 0.06 vs 0.87 +/- 0.02)
- Wilcoxon rank-sum P-value (P < 2.2E-16) and Spearman correlation (~0.67) between CV bins and AUROC
- summary statistics written to gene_set_size_auroc_results.csv

## How to apply

In R, load MetaNeighbor, mn_data (SummarizedExperiment sample data), and GOmouse via data(mn_data) and data(GOmouse). Randomly sample 100 gene sets of 100 genes and 100 sets of 800 genes from expressed genes, plus CV-performance-binned sets and an uninformative top-expression-centile control set. Run supervised MetaNeighbor with leave-one-dataset-out cross-validation and rank-correlation neighbor voting, obtaining the gene set-by-cell type AUROC matrix (bplot disabled). Compare 100-gene vs 800-gene AUROC distributions with a Wilcoxon rank-sum test and correlate CV bins with AUROC by Spearman correlation. Larger sets are expected to yield higher, more stable AUROCs (0.87 +/- 0.02 vs 0.75 +/- 0.06, P < 2.2E-16), while the control set should score near chance (~0.47).

## Related tools

- **MetaNeighbor** (supervised neighbor-voting framework run per gene set to produce mean AUROC scores via leave-one-dataset-out cross-validation) — github.com/gillislab/MetaNeighbor
- **R** (environment in which MetaNeighbor, data loading, random gene set sampling, and Wilcoxon/Spearman statistics are executed)
- **SummarizedExperiment** (container class for the mn_data sample expression data loaded with data(mn_data))

## Examples

```
library(MetaNeighbor); data(mn_data); data(GOmouse); auroc_scores <- MetaNeighbor(dat = mn_data, experiment_labels = ..., celltype_labels = ..., gene_sets = sampled_sets, bplot = FALSE)
```

## Evaluation signals

- 800-gene AUROC distribution is significantly higher than the 100-gene distribution (Wilcoxon rank-sum P < 2.2E-16).
- Spearman correlation between cross-validation variability bins and AUROC is approximately 0.67.
- Uninformative top-expression-centile control gene set scores below the 0.5 random-guessing baseline (AUROC ~0.47).
- Mean +/- sd AUROC reproduces ~0.75 +/- 0.06 for 100-gene sets and ~0.87 +/- 0.02 for 800-gene sets.
- Output gene set-by-cell type AUROC matrix is returned with bplot disabled and summary statistics written to gene_set_size_auroc_results.csv.

## Limitations

- The ranking step in MetaNeighbor scales poorly with total sample size; run times can reach hours for 10,000 samples and speed-ups require parallelization.
- Genes within gene sets must match expression matrix gene names and each set must contain more than one gene, or the method cannot run.
- An AUROC of 0.5 indicates performance no better than random guessing; uninformative sets may even fall slightly below chance (~0.47).

## Evidence

- [methods] Random 100 vs 800 gene set AUROC and Wilcoxon: "first, we build a network of rank correlations between all cells for a gene set"
- [methods] Neighbor voting predictor mechanics: "the neighbor voting predictor produces a weighted matrix of predicted labels by performing matrix multiplication between the network and the binary vector indicating cell type membership"
- [readme] AUROC interpretation and chance baseline: "An AUROC score of 0.5 means that we have performed as well as if we had randomly guessed the cell’s identity."
- [readme] Required inputs for supervised MetaNeighbor: "MetaNeighbor takes four inputs: a gene-by-sample expression matrix (“data”), a set of labels indicating each sample’s dataset of origin (“experiment labels”), a set of labels indicating each sample’s"
- [abstract] Large variable gene sets identify replicable types accurately: "Across tasks we find that large sets of variably expressed genes can identify replicable cell types with high accuracy"
