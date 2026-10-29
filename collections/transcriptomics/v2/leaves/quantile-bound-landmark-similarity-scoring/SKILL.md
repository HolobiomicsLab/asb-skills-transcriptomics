---
name: quantile-bound-landmark-similarity-scoring
description: Use when grouping genes with akin (quasi-trend-preserved) expression-rank
  vectors into consensus landmark clusters, and a rank/similarity score is needed
  to decide which unassigned genes join an existing gene cluster.
when_to_use_negative:
- Input is a raw, unnormalized count matrix — the quartile-bound comparison assumes
  entries normalized to [0,1] after log transformation
- The goal is the qualitative (binary) landmark representation — that is handled by
  the LC1 module using binary expression patterns, not quantitative quartile bounds
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scQA
  role: Implements the LC2 quantitative landmark constructor, including the quartile-bound
    similarity scoring S(Gi,Cj) used to assign genes to landmark clusters
  repo: https://github.com/LD-Lyndee/scQA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.12.021
    title: 'scQA: A dual-perspective cell type identification model for single cell
      transcriptome data'
  source_content_hash: sha256:b848cf3538a66da02637f113d0b0b01d731798f9a9f98524d6ca287ecf674790
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LD-Lyndee/scQA
  tools:
  - scQA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Quantile-bound landmark similarity scoring

## Summary

A similarity measure from the scQA quantitative landmark constructor (LC2) that scores each gene's membership in a gene cluster by the fraction of cluster components for which the gene's expression falls within the cluster's lower- and upper-quartile bound vectors, driving iterative assignment of quasi-trend-preserved genes into quantitative landmarks.

## When to use

Use when grouping genes with akin (quasi-trend-preserved) expression-rank vectors into consensus landmark clusters, and a rank/similarity score is needed to decide which unassigned genes join an existing gene cluster.

## When NOT to use

- Input is a raw, unnormalized count matrix — the quartile-bound comparison assumes entries normalized to [0,1] after log transformation
- The goal is the qualitative (binary) landmark representation — that is handled by the LC1 module using binary expression patterns, not quantitative quartile bounds

## Inputs

- Preprocessed gene-by-cell (or cell-by-gene) expression matrix, log-transformed and normalized so every entry falls in [0,1]
- Discretized matrix D' where each value is replaced by its bin index from dividing [0,1] into k=6 bins
- Gene groups defined by the number of bins each gene utilizes
- Initial gene clusters (connected components of the top 0.1% Spearman rank-correlation similarity graph) with template, lower-quartile, and upper-quartile bound vectors

## Outputs

- Per-gene similarity scores S(Gi,Cj) against each cluster
- Updated clusters (template and quartile bound vectors) after threshold-based assignment and pruning of clusters with <3 genes
- Quantitative cell-landmark matrix Q2 (LM2), e.g. lm2_cell_landmark_matrix.txt, with one row per cell (e.g. 1492 Xin cells) and landmark consensus-vector columns

## How to apply

First group genes by the number of discretization bins their normalized expression occupies (continuous values are binned by dividing [0,1] into k=6 bins), then build initial clusters as connected components (>2 vertices) of a gene similarity graph built from the top 0.1% of Spearman's rank correlation scores, with each cluster's template vector set to the mean expression per component. For each gene outside clusters, compute S(Gi,Cj) as the fraction of the n components where Gi's expression lies between the lower-quartile and upper-quartile bound vectors of cluster Cj, and iteratively assign genes whose score exceeds the threshold rho=0.7. After each assignment, update the cluster's template vector and its quartile bound vectors, then prune clusters with fewer than 3 genes. Retained clusters across gene groups are joined into the single quantitative cell-landmark matrix Q2 (LM2), with cells as rows and landmark consensus vectors as columns. The rationale is that genes whose expression stays within a cluster's quartile envelope across components share a quasi-trend-preserved pattern and can be represented by one consensus landmark.

## Related tools

- **scQA** (Implements the LC2 quantitative landmark constructor, including the quartile-bound similarity scoring S(Gi,Cj) used to assign genes to landmark clusters) — https://github.com/LD-Lyndee/scQA

## Examples

```
git clone https://github.com/LD-Lyndee/scQA.git; cd scQA; ./make; ./main xin_matrix.txt o
```

## Evaluation signals

- lm2_cell_landmark_matrix.txt exists and has one row per cell (1492 rows for the Xin GSE81608 dataset) with a reduced set of landmark columns
- Each landmark column is the consensus vector of its gene cluster and all entries match the cluster consensus definitions
- Every retained cluster contains at least 3 genes after pruning
- Number of landmarks is minimized to approximately a dozen while maximizing the inclusion of quasi-trend-preserved genes with dropouts
- Output matrix format matches the scQA convention: cells * genes-style plain matrix without row or column names as required by the pipeline

## Limitations

- Genes may be misclassified in LC1 and LC2 due to high dimensionality
- Retaining only 2000 highly variable genes excludes some informative genes before landmark construction
- The similarity threshold (rho=0.7), bin count (k=6), and Spearman top-0.1% cutoff are prespecified; inputs whose expression distribution differs markedly may require retuning

## Evidence

- [methods] Quantitative landmark constructor (LC2) divides genes into several groups and creates landmarks by clustering genes within each group: "Quantitative landmark constructor (LC2) divides genes into several groups and creates landmarks by clustering genes within each group"
- [intro] cluster genes exhibiting quasi-trend-preserved patterns to derive the quantitative low-dimensional representation of cells: "cluster genes exhibiting quasi-trend-preserved patterns to derive the quantitative low-dimensional representation of cells"
- [readme] The input file must be in the form of a matrix of cells * genes, without names of rows and columns.: "The input file must be in the form of a matrix of cells * genes, without names of rows and columns."
- [methods] All data are log-transformed and normalized to ensure that each entry falls in the range [0,1].: "All data are log-transformed and normalized to ensure that each entry falls in the range [0,1]."
