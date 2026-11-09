---
name: spearman-rank-correlation-gene-clustering
description: Use when scRNA-seq count matrices need a quantitative low-dimensional
  cell representation built by clustering genes with akin expression-rank trends,
  as in scQA's LC2 module.
when_to_use_negative:
- If the goal is a binary (qualitative) low-dimensional representation based on dropout
  patterns — that is the separate LC1 module, not LC2.
- If input is not a cells-by-genes expression matrix without row/column names — the
  scQA executable requires this exact unlabeled matrix format.
- If the number of cell types is already fixed and a supervised or k-means-style assignment
  suffices; LC2 is an unsupervised gene-clustering step within scQA's pipeline.
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scQA
  role: Primary tool whose LC2 module performs the Spearman-correlation-based gene
    clustering to build the quantitative landmark matrix (LM2).
  repo: https://github.com/LD-Lyndee/scQA
- name: R (pre.R)
  role: 'Upstream preprocessing: gene filtering, 2000 HVG retention, log-transformation,
    and normalization to [0,1] before LC2.'
  repo: https://github.com/LD-Lyndee/scQA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.12.021
    title: 'scQA: A dual-perspective cell type identification model for single cell
      transcriptome data'
  source_content_hash: sha256:b460f67caaa61862621d39c910bca662e186a63deb596d33a7163b0c159a3569
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LD-Lyndee/scQA
  tools:
  - R (pre.R)
  - scQA
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# spearman-rank-correlation-gene-clustering

## Summary

Implementation of scQA's quantitative landmark constructor (LC2), which groups quasi-trend-preserved genes via Spearman rank correlation similarity graphs and iterative cluster-gene assignment to build a quantitative low-dimensional cell-landmark matrix (LM2/Q2).

## When to use

Use when scRNA-seq count matrices need a quantitative low-dimensional cell representation built by clustering genes with akin expression-rank trends, as in scQA's LC2 module.

## When NOT to use

- If the goal is a binary (qualitative) low-dimensional representation based on dropout patterns — that is the separate LC1 module, not LC2.
- If input is not a cells-by-genes expression matrix without row/column names — the scQA executable requires this exact unlabeled matrix format.
- If the number of cell types is already fixed and a supervised or k-means-style assignment suffices; LC2 is an unsupervised gene-clustering step within scQA's pipeline.

## Inputs

- Preprocessed scRNA-seq expression matrix (cells x genes), e.g. the Xin GSE81608 human pancreas dataset with 1492 cells, filtered to 2000 highly variable genes and normalized to [0,1]
- Binning parameter k (default 6 bins over [0,1]) passed to the scQA executable
- Similarity threshold rho=0.7 and top 0.1% Spearman correlation cutoff

## Outputs

- lm2_cell_landmark_matrix.txt: quantitative cell-landmark matrix Q2 (LM2) with 1492 rows (one per cell) and a reduced set of landmark columns, each the consensus of a quasi-trend-preserved gene cluster

## How to apply

Preprocess with R (pre.R): filter genes expressed in fewer than 2.5% or more than 97.5% of cells, retain 2000 highly variable genes, log-transform and normalize entries to [0,1]. Discretize values by dividing [0,1] into k=6 bins, replace each value by its bin index to form D', and group genes by the number of bins they utilize. Within each group, form initial clusters as connected components (>2 vertices) of a gene similarity graph built from the top 0.1% of Spearman's rank correlation scores, defining each cluster's template vector as the mean expression per component. For each unclustered gene, compute similarity S(Gi,Cj) as the fraction of n components where Gi's expression falls between the lower- and upper-quartile bound vectors of Cj; iteratively assign genes above threshold rho=0.7 while updating template and bound vectors, then prune clusters with fewer than 3 genes. Join retained clusters across groups into the quantitative cell-landmark matrix Q2 (LM2) with cells as rows and landmark consensus vectors as columns.

## Related tools

- **scQA** (Primary tool whose LC2 module performs the Spearman-correlation-based gene clustering to build the quantitative landmark matrix (LM2).) — https://github.com/LD-Lyndee/scQA
- **R (pre.R)** (Upstream preprocessing: gene filtering, 2000 HVG retention, log-transformation, and normalization to [0,1] before LC2.) — https://github.com/LD-Lyndee/scQA

## Examples

```
git clone https://github.com/LD-Lyndee/scQA.git && cd scQA && ./make && Rscript pre.R && ./main xin_matrix.txt o
```

## Evaluation signals

- lm2_cell_landmark_matrix.txt exists and has exactly 1492 rows (one per Xin cell).
- The number of landmark columns is substantially reduced (approximately a dozen) relative to the 2000 input genes, per scQA's landmark-minimization goal.
- Each landmark column equals the consensus (mean template) of its gene cluster, and all entries are consistent with the cluster consensus definitions.
- Every retained cluster contains at least 3 genes after pruning of smaller clusters.
- Assignment of borderline genes respects the rho=0.7 quartile-bound similarity threshold, and the binning used k=6 as specified.

## Limitations

- Genes may be misclassified in LC1 and LC2 due to the influence of high dimensionality.
- Retention of only 2000 HVGs excluded some informative genes from the quantitative representation.
- The R version used for preprocessing is not stated in the methods.
- The specific source code of the cited HVG-selection method (references [11], [25]) is not named, complicating exact reproduction of the filtering step.

## Evidence

- [methods] Quantitative landmark constructor (LC2) divides genes into several groups and creates landmarks by clustering genes within each group: "Quantitative landmark constructor (LC2) divides genes into several groups and creates landmarks by clustering genes within each group"
- [abstract] Through an iterative process, our approach aims to minimize the number of landmarks to approximately a dozen while maximizing the inclusion of quasi-trend-preserved genes with dropouts both: "Through an iterative process, our approach aims to minimize the number of landmarks to approximately a dozen while maximizing the inclusion of quasi-trend-preserved genes"
- [intro] cluster genes exhibiting quasi-trend-preserved patterns to derive the quantitative low-dimensional representation of cells: "cluster genes exhibiting quasi-trend-preserved patterns to derive the quantitative low-dimensional representation of cells"
- [readme] The input file must be in the form of a matrix of cells * genes, without names of rows and columns.: "The input file must be in the form of a matrix of cells * genes, without names of rows and columns."
- [methods] All data are log-transformed and normalized to ensure that each entry falls in the range [0,1].: "All data are log-transformed and normalized to ensure that each entry falls in the range [0,1]."
