---
name: dropout-tolerant-consensus-landmark-computation
description: Use when scRNA-seq count data are sparse with high dropout rates and
  a robust binary low-dimensional cell representation is needed before clustering,
  e.g. SMARTer or CEL-Seq2 data such as Xin GSE81608.
when_to_use_negative:
- The input is already a binary or feature-reduced landmark table — LC1 expects raw/log-normalized
  continuous expression values to binarize
- The analysis goal is quantitative trend-preserved gene clustering — that is the
  separate scQA LC2 module, not LC1
- Preserving per-gene expression magnitude is required, since binarization discards
  expression levels entirely
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: scQA
  role: Implements the LC1 module that binarizes the matrix and clusters genes into
    qualitative landmarks; compiled and run via ./main
  repo: https://github.com/LD-Lyndee/scQA
- name: Seurat
  role: Highly variable gene selection method (references [11],[25]) applied during
    R preprocessing before LC1
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.12.021
    title: 'scQA: A dual-perspective cell type identification model for single cell
      transcriptome data'
  source_content_hash: sha256:04955c25769877d00ddc7ad2ab6a91ad57962209d1937a7bd8ff85ae5d3b2959
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LD-Lyndee/scQA
  tools:
  - Seurat
  - scQA
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# dropout-tolerant-consensus-landmark-computation

## Summary

Implements scQA's qualitative landmark constructor (LC1), which binarizes dropout-prone scRNA-seq expression and amalgamates genes sharing analogous binary expression patterns into consensus binary landmarks, yielding a low-dimensional qualitative representation of cells.

## When to use

Use when scRNA-seq count data are sparse with high dropout rates and a robust binary low-dimensional cell representation is needed before clustering, e.g. SMARTer or CEL-Seq2 data such as Xin GSE81608.

## When NOT to use

- The input is already a binary or feature-reduced landmark table — LC1 expects raw/log-normalized continuous expression values to binarize
- The analysis goal is quantitative trend-preserved gene clustering — that is the separate scQA LC2 module, not LC1
- Preserving per-gene expression magnitude is required, since binarization discards expression levels entirely

## Inputs

- Raw scRNA-seq gene expression matrix (cells × genes), e.g. Xin GSE81608 counts
- Preprocessed matrix from pre.R (2.5%/97.5% gene filter, 2000 HVGs, log-transformed, normalized to [0,1])

## Outputs

- Qualitative cell-landmark binary matrix Q1 (cells as rows, one consensus binary landmark column per retained gene cluster), saved as q1_landmark_matrix.csv

## How to apply

Preprocess the raw expression matrix in R (pre.R): filter genes expressed in fewer than 2.5% or more than 97.5% of cells, retain 2000 highly variable genes (Seurat-style method of references [11],[25]), then log-transform and normalize all entries to [0,1]. In scQA's LC1, binarize each gene vector (0 if expression is zero, 1 otherwise), sort genes by decreasing number of zeros, and compute binary gene-gene similarity S = 1 − h/n (normalized Hamming distance) against the 0.05×m neighbors on each side, keeping the top 1999 gene pairs (one thousandth of all pairs) as edges of a similarity graph. Take connected components with more than two vertices as initial clusters, defining each cluster's template vector with a 1 where at least half (fraction ≥ 0.5) of the cluster's genes are expressed. Iteratively assign remaining genes using component-wise similarity increments with the subsidiary vector a_jk = max(0.5 − mean expression of the cluster at component k, 0): add gene Gi to cluster C_J if S(Gi,C_J) exceeds ρ = 0.7, otherwise form a singleton; run one iteration with ρ = 0.7, then prune clusters with fewer than 3 genes. The rationale is to treat dropouts as weak signals and strengthen them via amalgamation and consensus, producing the qualitative cell-landmark matrix Q1 (one binary consensus column per retained cluster).

## Related tools

- **scQA** (Implements the LC1 module that binarizes the matrix and clusters genes into qualitative landmarks; compiled and run via ./main) — https://github.com/LD-Lyndee/scQA
- **Seurat** (Highly variable gene selection method (references [11],[25]) applied during R preprocessing before LC1)

## Examples

```
git clone https://github.com/LD-Lyndee/scQA.git && cd scQA && ./make && Rscript pre.R && ./main xin_matrix.txt o
```

## Evaluation signals

- Q1 matrix contains only binary entries (0/1) with cells as rows and one column per retained cluster
- All retained gene clusters have at least 3 genes after pruning clusters with fewer than 3 genes
- The gene similarity graph retains the top 1999 gene pairs (one thousandth of all gene pairs) as edges
- Every retained cluster's template vector has a 1 at components where at least half (fraction ≥ 0.5) of its genes are expressed
- Number of landmarks is small (the iterative process aims to minimize landmarks to approximately a dozen)

## Limitations

- Genes may be misclassified in LC1 (and LC2) due to the influence of high dimensionality
- Retaining only 2000 HVGs for subsequent clustering analysis led to the exclusion of some informative genes
- The R preprocessing script (pre.R) does not state the R version, so reproducibility of the preprocessing step may vary

## Evidence

- [intro] we regard dropouts as weak signals, strengthened by amalgamation of genes sharing analogous binary expression patterns and performing consensus, yielding a binary low-dimensional representation of: "we regard dropouts as weak signals, strengthened by amalgamation of genes sharing analogous binary expression patterns and performing consensus, yielding a binary low-dimensional representation of"
- [methods] Qualitative landmark constructor (LC1) initially binarizes the matrix: "Qualitative landmark constructor (LC1) initially binarizes the matrix and persists to cluster genes with qualitative quasi-trend-preserved expression patterns to form landmarks."
- [methods] we retain 2000 highly variable genes, employing the method outlined in references [11], [25]: "we retain 2000 highly variable genes, employing the method outlined in references [11], [25]"
- [methods] we initially filter out genes expressed in fewer than x% cells or more than (100−x)% cells, with x set to 2.5: "we initially filter out genes expressed in fewer than x% cells or more than (100−x)% cells, with x set to 2.5"
- [readme] Pre-processing with R provided as pre.R: "Pre-processing with R provided as pre.R"
