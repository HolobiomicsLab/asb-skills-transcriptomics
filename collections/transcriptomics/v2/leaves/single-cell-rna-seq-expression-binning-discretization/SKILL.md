---
name: single-cell-rna-seq-expression-binning-discretization
description: Use when a quantitative, rank-trend-based low-dimensional gene representation
  is needed from normalized scRNA-seq data, i.e., when binning continuous expression
  values into discrete levels to group quasi-trend-preserved genes.
when_to_use_negative:
- 'Input is not normalized to [0,1]: equal-width binning of [0,1] is meaningless on
  raw counts or differently scaled matrices'
- The goal is qualitative (binary) landmark construction — use LC1 binarization instead
  of quantitative k-bin discretization
- The input already contains discrete/categorical expression levels and only requires
  clustering rather than re-discretization
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: scQA
  role: 'Provides the binning-discretization workflow: pre.R preprocessing (filter
    2.5%/97.5%, 2000 HVGs, log-transform, normalize to [0,1]) and the ''binning''
    parameter (default 6) controlling bin count for quantitative landmark construction
    (LC2)'
  repo: https://github.com/LD-Lyndee/scQA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.12.021
    title: 'scQA: A dual-perspective cell type identification model for single cell
      transcriptome data'
  source_content_hash: sha256:3299292ef37aa7268c49fb8667cf5089f70ac75f6844a61f6266e16683dd2a0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LD-Lyndee/scQA
  tools:
  - scQA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# single-cell-rna-seq-expression-binning-discretization

## Summary

Discretize normalized single-cell RNA-seq expression values by dividing the [0,1] range into k equal bins (scQA uses k=6, tunable via the 'binning' parameter) and replacing each value with its bin index, forming matrix D' that enables grouping of quasi-trend-preserved genes by the number of bins they utilize for quantitative landmark construction (LC2).

## When to use

Use when a quantitative, rank-trend-based low-dimensional gene representation is needed from normalized scRNA-seq data, i.e., when binning continuous expression values into discrete levels to group quasi-trend-preserved genes.

## When NOT to use

- Input is not normalized to [0,1]: equal-width binning of [0,1] is meaningless on raw counts or differently scaled matrices
- The goal is qualitative (binary) landmark construction — use LC1 binarization instead of quantitative k-bin discretization
- The input already contains discrete/categorical expression levels and only requires clustering rather than re-discretization

## Inputs

- Raw scRNA-seq count matrix as cells × genes without row/column names (e.g., Xin GSE81608 human pancreas, 1492 cells)
- pre.R R script input: filtered, log-transformed expression matrix normalized to [0,1]

## Outputs

- Discretized expression matrix D' where each entry is a bin index from 1..k (k=6 by default)
- Gene groups partitioned by the number of expression bins each gene utilizes, feeding quantitative landmark (LM2) construction

## How to apply

First preprocess the raw count matrix (cells × genes) in R with pre.R: filter out genes expressed in fewer than 2.5% or more than 97.5% of cells, retain the 2000 most highly variable genes, log-transform, and normalize every entry to the range [0,1]. Then divide the [0,1] interval into k=6 equal-width bins and replace each gene's normalized expression value with its bin index, producing the discretized matrix D'. Group genes by the number of distinct bins they utilize, since genes within a group show akin (though not identical) expression-rank trends; this grouping is the substrate scQA's LC2 module uses to build similarity graphs (top 0.1% Spearman correlations) and cluster genes into quantitative landmarks. The bin count k is exposed as the 'binning' parameter of the scQA executable, defaulting to 6 ('o' selects the default).

## Related tools

- **scQA** (Provides the binning-discretization workflow: pre.R preprocessing (filter 2.5%/97.5%, 2000 HVGs, log-transform, normalize to [0,1]) and the 'binning' parameter (default 6) controlling bin count for quantitative landmark construction (LC2)) — https://github.com/LD-Lyndee/scQA

## Examples

```
Rscript pre.R  # then: ./main xin_matrix.txt 6   ('o' uses the default binning of 6)
```

## Evaluation signals

- All entries of the preprocessed matrix fall within [0,1] after log-transformation and normalization, a precondition for [0,1] binning
- Every entry of the discretized matrix D' is an integer bin index in {1,...,6} (or the chosen k)
- Gene groups are complete: each of the 2000 retained highly variable genes is assigned to exactly one group based on its number of utilized bins
- Downstream check: the LC2 output lm2_cell_landmark_matrix.txt exists with 1492 rows (one per Xin cell) and a reduced set of landmark columns, each the consensus of its gene cluster
- The 'binning' argument is honored: passing a non-default integer changes the bin count, while 'o' selects the default of 6

## Limitations

- Genes may be misclassified in LC1 and LC2 due to the influence of high dimensionality
- Retaining only 2000 highly variable genes excluded some informative genes before binning
- Only tested/preprocessed on Linux for the scQA binary; the R version used for pre.R is not stated

## Evidence

- [methods] we initially filter out genes expressed in fewer than x% cells or more than (100−x)% cells, with x set to 2.5: "we initially filter out genes expressed in fewer than x% cells or more than (100−x)% cells, with x set to 2.5"
- [methods] All data are log-transformed and normalized to ensure that each entry falls in the range [0,1].: "All data are log-transformed and normalized to ensure that each entry falls in the range [0,1]."
- [methods] Quantitative landmark constructor (LC2) divides genes into several groups and creates landmarks by clustering genes within each group: "Quantitative landmark constructor (LC2) divides genes into several groups and creates landmarks by clustering genes within each group"
- [readme] binning Define number of bins, default: 6. "o" for using the default parameter.: "binning Define number of bins, default: 6. "o" for using the default parameter."
- [readme] The input file must be in the form of a matrix of cells * genes, without names of rows and columns.: "The input file must be in the form of a matrix of cells * genes, without names of rows and columns."
