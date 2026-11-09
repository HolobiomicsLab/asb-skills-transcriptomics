---
name: quasi-trend-preserved-gene-grouping
description: Use when building a quantitative low-dimensional representation of single-cell
  RNA-seq cells by clustering genes whose expression-rank vectors show akin (not identical)
  trends into landmark consensus vectors.
when_to_use_negative:
- You need a binary/qualitative cell representation — that is the LC1 module, which
  binarizes the matrix and clusters genes with qualitative quasi-trend-preserved patterns
  instead
- You want final cell cluster labels directly — clustering cells from Q1/Q2 requires
  the downstream Cluster Constructor (CC) and label propagation modules
- The dataset has not been filtered/HVG-selected and normalized to [0,1]; raw sparse
  counts violate the binning assumption of LC2
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scQA
  role: Provides the LC2 module that discretizes expression into bins, clusters quasi-trend-preserved
    genes into landmarks, and emits the Q2 cell-landmark matrix
  repo: https://github.com/LD-Lyndee/scQA
- name: pre.R
  role: R pre-processing script that filters genes (2.5%/97.5% thresholds), selects
    2000 highly variable genes, and log-transforms/normalizes entries to [0,1] before
    LC2
  repo: https://github.com/LD-Lyndee/scQA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.12.021
    title: 'scQA: A dual-perspective cell type identification model for single cell
      transcriptome data'
  source_content_hash: sha256:a8d340376f264d2647485515bf0dc03ece22a52d0dc48d990a8a3759d1c1441d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LD-Lyndee/scQA
  tools:
  - pre.R
  - scQA
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# quasi-trend-preserved-gene-grouping

## Summary

Implements scQA's quantitative landmark constructor (LC2): discretize normalized gene expression into k bins, group genes by bin usage, and cluster quasi-trend-preserved genes within each group into landmark consensus vectors that form the quantitative low-dimensional cell representation (LM2/Q2 matrix).

## When to use

Use when building a quantitative low-dimensional representation of single-cell RNA-seq cells by clustering genes whose expression-rank vectors show akin (not identical) trends into landmark consensus vectors.

## When NOT to use

- You need a binary/qualitative cell representation — that is the LC1 module, which binarizes the matrix and clusters genes with qualitative quasi-trend-preserved patterns instead
- You want final cell cluster labels directly — clustering cells from Q1/Q2 requires the downstream Cluster Constructor (CC) and label propagation modules
- The dataset has not been filtered/HVG-selected and normalized to [0,1]; raw sparse counts violate the binning assumption of LC2

## Inputs

- Preprocessed gene expression matrix (cells x genes), log-transformed and normalized to [0,1], after filtering genes expressed in <2.5% or >97.5% of cells and retaining 2000 highly variable genes (e.g., xin_matrix.txt)
- Number of bins k (default 6) for discretizing expression values into D'

## Outputs

- lm2_cell_landmark_matrix.txt — quantitative cell-landmark matrix Q2 (LM2) with one row per cell (e.g., 1492 Xin cells) and one column per landmark consensus vector
- Retained gene clusters (each with >= 3 genes) with template vectors and lower-/upper-quartile bound vectors per landmark

## How to apply

Preprocess the count matrix (e.g., Xin GSE81608) by filtering genes expressed in fewer than 2.5% or more than 97.5% of cells, retaining 2000 highly variable genes, then log-transform and normalize all entries to [0,1]. Discretize by dividing [0,1] into k=6 bins and replace each expression value with its bin index to form D', grouping genes by the number of bins they utilize. Within each gene group, build a gene similarity graph from the top 0.1% of Spearman's rank correlation scores; initial clusters are connected components with more than 2 vertices, each represented by a template vector of mean expression per component. Assign each remaining gene iteratively if its similarity S(Gi,Cj) — the fraction of n components where Gi's expression falls between the lower- and upper-quartile bound vectors of Cj — exceeds rho=0.7, updating template and quartile bound vectors after each assignment, then prune clusters with fewer than 3 genes. Join all retained clusters across gene groups into the single quantitative cell-landmark matrix Q2 (LM2), with cells as rows and landmark consensus vectors as columns; the iterative process aims to minimize landmarks to approximately a dozen while maximizing inclusion of quasi-trend-preserved genes.

## Related tools

- **scQA** (Provides the LC2 module that discretizes expression into bins, clusters quasi-trend-preserved genes into landmarks, and emits the Q2 cell-landmark matrix) — https://github.com/LD-Lyndee/scQA
- **pre.R** (R pre-processing script that filters genes (2.5%/97.5% thresholds), selects 2000 highly variable genes, and log-transforms/normalizes entries to [0,1] before LC2) — https://github.com/LD-Lyndee/scQA

## Examples

```
git clone https://github.com/LD-Lyndee/scQA.git && cd scQA && ./make && ./main xin_matrix.txt o
```

## Evaluation signals

- lm2_cell_landmark_matrix.txt exists and has 1492 rows (one per Xin GSE81608 cell) with a reduced set of landmark columns, each the consensus of its gene cluster
- All entries in Q2 are consistent with the cluster consensus definitions (mean/template vectors of the assigned gene clusters)
- Each retained landmark cluster contains at least 3 genes after pruning; clusters with fewer than 3 genes were removed
- Only genes with S(Gi,Cj) above rho=0.7 were assigned to clusters, with template and quartile bound vectors updated iteratively during assignment
- The number of landmarks is small (approximately a dozen), reflecting the iterative minimization while maximizing quasi-trend-preserved gene inclusion

## Limitations

- Genes may be misclassified in LC1 and LC2 due to the influence of high dimensionality
- Retaining only 2000 HVGs for downstream analysis led to the exclusion of some informative genes
- The iterative landmark minimization targets approximately a dozen landmarks, which may not suit datasets requiring finer quantitative resolution

## Evidence

- [methods] Quantitative landmark constructor (LC2) divides genes into several groups and creates landmarks by clustering genes within each group: "Quantitative landmark constructor (LC2) divides genes into several groups and creates landmarks by clustering genes within each group"
- [methods] All data are log-transformed and normalized to ensure that each entry falls in the range [0,1].: "All data are log-transformed and normalized to ensure that each entry falls in the range [0,1]."
- [methods] we retain 2000 highly variable genes, employing the method outlined in references [11], [25]: "we retain 2000 highly variable genes, employing the method outlined in references [11], [25]"
- [intro] cluster genes exhibiting quasi-trend-preserved patterns to derive the quantitative low-dimensional representation of cells: "cluster genes exhibiting quasi-trend-preserved patterns to derive the quantitative low-dimensional representation of cells"
- [readme] The input file must be in the form of a matrix of cells * genes, without names of rows and columns.: "The input file must be in the form of a matrix of cells * genes, without names of rows and columns."
