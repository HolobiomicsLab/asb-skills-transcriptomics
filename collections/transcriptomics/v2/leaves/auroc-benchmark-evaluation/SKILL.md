---
name: auroc-benchmark-evaluation
description: Use when quantifying how gene set size or expression variability (CV)
  affects MetaNeighbor AUROC for cross-dataset cell-type identification, or when validating
  that a gene set performs above chance (AUROC ~0.5).
when_to_use_negative:
- Cell-type labels are missing or ambiguous across datasets — use unsupervised MetaNeighborUS
  with variableGenes instead of the supervised AUROC benchmark.
- The goal is identifying replicate cell-type pairs across studies (reciprocal top
  hits, AUROC > 0.95), not benchmarking gene set size/variability effects on a single
  task.
- Gene sets contain a single gene or gene names do not match the expression matrix
  row names — MetaNeighbor requires multi-gene sets with matching nomenclature.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: MetaNeighbor
  role: Supervised neighbor-voting framework that builds rank-correlation cell networks
    per gene set and returns mean AUROC scores via leave-one-dataset-out cross-validation
  repo: github.com/gillislab/MetaNeighbor
- name: R
  role: Analysis environment in which the MetaNeighbor package, random gene set sampling,
    Wilcoxon rank-sum and Spearman correlation tests are run
- name: SummarizedExperiment
  role: Container format for the mn_data expression sample data loaded via data(mn_data)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-018-03282-0
    title: Characterizing the replicability of cell types defined by single cell RNA-sequencing
      data using MetaNeighbor
  source_content_hash: sha256:28e1864ea9a16991f3f25d7f6365062b956f06f8bd960497a7e03dba735a68d6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MetaNeighbor
  - R
  - SummarizedExperiment
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# auroc-benchmark-evaluation

## Summary

Benchmarks MetaNeighbor's supervised neighbor-voting AUROC performance as a function of gene set size and within-set expression variability (coefficient of variation), using randomly sampled 100-gene and 800-gene sets from the mn_data/GOmouse sample data to quantify how set composition affects cell-type discrimination (neurons vs non-neuronal).

## When to use

Use when quantifying how gene set size or expression variability (CV) affects MetaNeighbor AUROC for cross-dataset cell-type identification, or when validating that a gene set performs above chance (AUROC ~0.5).

## When NOT to use

- Cell-type labels are missing or ambiguous across datasets — use unsupervised MetaNeighborUS with variableGenes instead of the supervised AUROC benchmark.
- The goal is identifying replicate cell-type pairs across studies (reciprocal top hits, AUROC > 0.95), not benchmarking gene set size/variability effects on a single task.
- Gene sets contain a single gene or gene names do not match the expression matrix row names — MetaNeighbor requires multi-gene sets with matching nomenclature.

## Inputs

- Gene-by-sample expression matrix (SummarizedExperiment), e.g. mn_data sample data
- Experiment labels indicating dataset of origin per sample
- Cell type label matrix with row names matching expression matrix column names
- Gene sets as a list of vectors (randomly sampled 100-gene and 800-gene sets, CV-binned sets, top-expression-centile control set)

## Outputs

- Gene set-by-cell type matrix of mean AUROC scores from supervised MetaNeighbor
- AUROC summary statistics per set size (mean ± sd, e.g. 0.75 ± 0.06 vs 0.87 ± 0.02)
- Wilcoxon rank-sum P-value comparing 100- vs 800-gene AUROC distributions
- Spearman correlation between CV bin and AUROC (~0.67)
- gene_set_size_auroc_results.csv with AUROC score matrix and summary statistics

## How to apply

Load MetaNeighbor, mn_data (SummarizedExperiment sample data) and GOmouse in R, then randomly sample 100 gene sets of 100 genes and 100 gene sets of 800 genes from expressed genes. Also construct CV-performance-binned gene sets and an uninformative control set of top-expression-centile genes. For each set, run the supervised MetaNeighbor function with leave-one-dataset-out cross-validation and rank-correlation neighbor voting (bplot disabled), producing a gene set-by-cell type matrix of mean AUROC scores. Compare 100-gene vs 800-gene AUROC distributions with a Wilcoxon rank-sum test and correlate CV binning with AUROC using Spearman correlation. Rationale: larger, more variably expressed sets carry more replicable signal — 800-gene sets reach ~0.87 ± 0.02 AUROC vs ~0.75 ± 0.06 for 100-gene sets, while an uninformative expression-centile control scores ~0.47, below the 0.5 random-guessing baseline.

## Related tools

- **MetaNeighbor** (Supervised neighbor-voting framework that builds rank-correlation cell networks per gene set and returns mean AUROC scores via leave-one-dataset-out cross-validation) — github.com/gillislab/MetaNeighbor
- **R** (Analysis environment in which the MetaNeighbor package, random gene set sampling, Wilcoxon rank-sum and Spearman correlation tests are run)
- **SummarizedExperiment** (Container format for the mn_data expression sample data loaded via data(mn_data))

## Examples

```
library(MetaNeighbor); data(mn_data); data(GOmouse); AUROC_scores = MetaNeighbor(dat = mn_data, experiment_labels = ..., celltype_labels = ..., gene_sets = gene_sets, bplot = FALSE)
```

## Evaluation signals

- 800-gene AUROC distribution is significantly higher than the 100-gene distribution (Wilcoxon rank-sum P < 2.2E-16)
- Spearman correlation between CV-binned set performance and AUROC is approximately 0.67
- Uninformative top-expression-centile control set scores near chance (AUROC ~0.47, below the 0.5 random-guessing baseline)
- 100-gene random sets yield AUROC ≈ 0.75 ± 0.06 and 800-gene sets ≈ 0.87 ± 0.02
- Output matrix has one mean AUROC per gene set per cell type, consistent with leave-one-dataset-out folds

## Limitations

- AUROC of 0.5 corresponds to random guessing; scores at or below this level indicate the gene set carries no cell-type-specific signal for the task
- Ranking step limits scaling in R: run time grows steeply with sample count (e.g. ~3653 s for 2 experiments, 10 cell types, 10000 samples, 10 gene sets)
- Supervised analysis requires known corresponding cell types across datasets; it cannot be applied when labels are ambiguous or uninformative
- More moderate AUROC scores are still meaningful but only AUROC > 0.95 with reciprocal top hits reflects extreme confidence in a concordant cell type

## Evidence

- [abstract] Large sets of variably expressed genes can identify replicable cell types with high accuracy: "Across tasks we find that large sets of variably expressed genes can identify replicable cell types with high accuracy"
- [readme] An AUROC of 0.5 means random guessing: "An AUROC score of 0.5 means that we have performed as well as if we had randomly guessed the cell’s identity."
- [readme] Neighbor voting via rank-correlation network and leave-one-dataset-out CV: [evidence span withheld — anchor status: external_doc]
- [intro] Cross-dataset validation hides labels of one dataset at a time: [evidence span withheld — anchor status: not_found]
