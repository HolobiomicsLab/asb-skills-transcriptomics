---
name: consensus-landmark-vector-construction-scrna
description: Use when a scRNA-seq expression matrix must be reduced to a quantitative
  low-dimensional cell representation via clustering of genes with akin (quasi-trend-preserved)
  expression-rank trends, e.g. as the LC2 step feeding scQA's cluster constructor.
when_to_use_negative:
- The goal is a binary/qualitative representation of cells from dropout-strengthened
  genes — that is the LC1 module, not LC2
- You need final cell cluster labels rather than the quantitative landmark feature
  matrix — that requires the CC (cluster constructor) and label propagation modules
  downstream
- Fewer than ~2000 informative genes or datasets where filtering 2.5%/97.5% and HVG
  retention would remove the signal under study (article notes HVG retention excluded
  some informative genes)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scQA
  role: Implements the LC2 quantitative landmark constructor module that bins expression,
    clusters quasi-trend-preserved genes, and emits the Q2 cell-landmark matrix
  repo: https://github.com/LD-Lyndee/scQA
- name: pre.R
  role: R pre-processing script that filters genes (2.5%/97.5%), retains 2000 HVGs,
    log-transforms and normalizes entries to [0,1] prior to binning
  repo: https://github.com/LD-Lyndee/scQA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.12.021
    title: 'scQA: A dual-perspective cell type identification model for single cell
      transcriptome data'
  source_content_hash: sha256:e0f906c8356715385eb00f63dbde76862955d87d0c44aa72dde1533c213e556a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LD-Lyndee/scQA
  tools:
  - pre.R
  - scQA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# consensus-landmark-vector-construction-scrna

## Summary

Constructs the quantitative low-dimensional landmark representation (LM2) of single-cell RNA-seq data by discretizing expression into k bins, clustering quasi-trend-preserved genes within bin-count groups via Spearman-rank similarity graphs and iterative assignment, and joining retained gene clusters into a cell-by-landmark consensus matrix.

## When to use

Use when a scRNA-seq expression matrix must be reduced to a quantitative low-dimensional cell representation via clustering of genes with akin (quasi-trend-preserved) expression-rank trends, e.g. as the LC2 step feeding scQA's cluster constructor.

## When NOT to use

- The goal is a binary/qualitative representation of cells from dropout-strengthened genes — that is the LC1 module, not LC2
- You need final cell cluster labels rather than the quantitative landmark feature matrix — that requires the CC (cluster constructor) and label propagation modules downstream
- Fewer than ~2000 informative genes or datasets where filtering 2.5%/97.5% and HVG retention would remove the signal under study (article notes HVG retention excluded some informative genes)

## Inputs

- Preprocessed cell-by-gene scRNA-seq expression matrix (cells × genes, numeric, values in [0,1] after log-transform/normalization, e.g. from pre.R on Xin GSE81608)
- Binning parameter k (number of discretization bins, default 6 in scQA)

## Outputs

- Quantitative cell-landmark matrix Q2 (LM2), with one row per cell (1492 for Xin GSE81608) and one column per landmark consensus vector
- Retained gene clusters with template (mean) and lower-/upper-quartile bound vectors

## How to apply

Preprocess the cell-by-gene matrix (e.g. Xin GSE81608, 1492 cells): filter genes expressed in fewer than 2.5% or more than 97.5% of cells, retain 2000 highly variable genes, log-transform and normalize entries to [0,1]. Divide [0,1] into k=6 bins, replace each expression value by its bin index to form D', and partition genes into groups by the number of bins they utilize. Within each group, build a gene similarity graph from the top 0.1% of Spearman's rank correlations; take connected components of >2 vertices as initial clusters, with each cluster's template vector the mean expression per component. For unassigned genes, compute similarity S(Gi,Cj) as the fraction of n components where Gi's expression falls between the lower- and upper-quartile bound vectors of Cj; iteratively assign genes above threshold rho=0.7 while updating template and bound vectors, then prune clusters with fewer than 3 genes and join all retained clusters across groups into the single quantitative cell-landmark matrix Q2 (LM2).

## Related tools

- **scQA** (Implements the LC2 quantitative landmark constructor module that bins expression, clusters quasi-trend-preserved genes, and emits the Q2 cell-landmark matrix) — https://github.com/LD-Lyndee/scQA
- **pre.R** (R pre-processing script that filters genes (2.5%/97.5%), retains 2000 HVGs, log-transforms and normalizes entries to [0,1] prior to binning) — https://github.com/LD-Lyndee/scQA

## Examples

```
git clone https://github.com/LD-Lyndee/scQA.git; cd scQA; ./main xin_matrix.txt o
```

## Evaluation signals

- lm2_cell_landmark_matrix.txt exists and has exactly 1492 rows when run on Xin GSE81608 (one per cell)
- Number of landmark columns is greatly reduced relative to the 2000 input genes (article: iterative process minimizes landmarks to approximately a dozen)
- Each landmark column equals the consensus (template/mean) vector of its gene cluster, with entries consistent with cluster consensus definitions
- No retained cluster contains fewer than 3 genes after pruning
- All gene assignments above similarity threshold rho=0.7 (fraction of components where expression lies within the quartile bound vectors)

## Limitations

- Genes may be misclassified in LC1 and LC2 due to the influence of high dimensionality
- Retention of only 2000 highly variable genes for downstream analysis led to the exclusion of some informative genes
- The number of bins k is a parameter (default 6; 'o' uses the default) and input matrices must be cells × genes without row or column names

## Evidence

- [methods] Quantitative landmark constructor (LC2) divides genes into several groups and creates landmarks by clustering genes within each group: "Quantitative landmark constructor (LC2) divides genes into several groups and creates landmarks by clustering genes within each group"
- [methods] Discretization and grouping by bin count: "we initially filter out genes expressed in fewer than x% cells or more than (100−x)% cells, with x set to 2.5"
- [methods] All data are log-transformed and normalized to ensure that each entry falls in the range [0,1].: "All data are log-transformed and normalized to ensure that each entry falls in the range [0,1]."
- [intro] cluster genes exhibiting quasi-trend-preserved patterns to derive the quantitative low-dimensional representation of cells: "cluster genes exhibiting quasi-trend-preserved patterns to derive the quantitative low-dimensional representation of cells"
- [abstract] Through an iterative process, our approach aims to minimize the number of landmarks to approximately a dozen: "Through an iterative process, our approach aims to minimize the number of landmarks to approximately a dozen"
- [readme] Input matrix format requirement: "The input file must be in the form of a matrix of cells * genes, without names of rows and columns."
