---
name: binary-pattern-gene-clustering
description: Use when a sparse, dropout-prone scRNA-seq matrix (e.g. SMARTer-protocol
  data such as Xin GSE81608) must be converted into a binary low-dimensional cell
  representation for cell-type clustering, and dropouts should be leveraged as weak
  signals rather than imputed.
when_to_use_negative:
- The analysis goal is a quantitative (expression-magnitude) representation — use
  scQA's LC2 module, which clusters genes with quasi-trend-preserved quantitative
  patterns, instead of LC1
- A predetermined number of clusters is required upstream, or the input is already
  a low-dimensional feature/embedding table rather than a raw sparse cell × gene matrix
- Only ~2000 retained HVGs would exclude critical informative genes for the biological
  question (a limitation the authors note for their own clustering)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scQA
  role: Provides the LC1 module that binarizes the matrix and clusters genes with
    qualitative quasi-trend-preserved expression patterns to form landmarks
  repo: https://github.com/LD-Lyndee/scQA
- name: Seurat
  role: Seurat-style highly variable gene selection (method of references [11],[25])
    used during pre-processing to retain 2000 HVGs before binarization
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.12.021
    title: 'scQA: A dual-perspective cell type identification model for single cell
      transcriptome data'
  source_content_hash: sha256:41c2482aa039fcda65180334b8c14e7f1eed2d2c367053bc7df94c75b29e7bda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LD-Lyndee/scQA
  tools:
  - Seurat
  - scQA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# binary-pattern-gene-clustering

## Summary

Implements scQA's qualitative landmark constructor (LC1): binarize dropout-prone scRNA-seq expression, cluster genes sharing analogous binary expression patterns via normalized Hamming similarity, and emit a binary low-dimensional cell-landmark matrix (Q1) that treats dropouts as weak signals to be strengthened by gene amalgamation.

## When to use

Use when a sparse, dropout-prone scRNA-seq matrix (e.g. SMARTer-protocol data such as Xin GSE81608) must be converted into a binary low-dimensional cell representation for cell-type clustering, and dropouts should be leveraged as weak signals rather than imputed.

## When NOT to use

- The analysis goal is a quantitative (expression-magnitude) representation — use scQA's LC2 module, which clusters genes with quasi-trend-preserved quantitative patterns, instead of LC1
- A predetermined number of clusters is required upstream, or the input is already a low-dimensional feature/embedding table rather than a raw sparse cell × gene matrix
- Only ~2000 retained HVGs would exclude critical informative genes for the biological question (a limitation the authors note for their own clustering)

## Inputs

- Raw scRNA-seq expression matrix of cells × genes (e.g. Xin GSE81608, human pancreas, 1492 cells, 239,851 genes, SMARTer protocol), passed through R pre-processing (pre.R) that filters genes by 2.5%/97.5% expression quantiles and retains 2000 highly variable genes
- Normalized, log-transformed gene expression matrix with all entries in [0,1]

## Outputs

- Qualitative cell-landmark matrix Q1 (cells as rows, one consensus binary landmark column per retained gene cluster), saved as q1_landmark_matrix.csv
- Gene clusters (with template and subsidiary vectors) representing qualitative quasi-trend-preserved expression patterns

## How to apply

Preprocess the raw count matrix in R (pre.R): filter genes expressed in fewer than 2.5% or more than 97.5% of cells, retain 2000 highly variable genes (Seurat-style method of references [11],[25]), then log-transform and normalize all entries to [0,1]. In LC1, binarize each gene vector (0 if expression is zero, 1 otherwise), sort genes by decreasing number of zeros, and compute binary gene-gene similarity S = 1 − h/n (normalized Hamming distance) against the 0.05×m neighbors on each side, retaining the top 1999 gene pairs (one thousandth of all pairs) as graph edges. Take connected components with more than two vertices as initial clusters, define each cluster template vector with component 1 where at least half (fraction ≥ 0.5) of the cluster's genes are expressed, then iteratively assign remaining genes using component-wise similarity increments with subsidiary vector a_jk = max(0.5 − mean expression of the cluster at component k, 0); add gene Gi to cluster C_J if S(Gi,C_J) > ρ = 0.7, otherwise keep it as a singleton, updating template and subsidiary vectors. Run one iteration at ρ = 0.7, prune clusters with fewer than 3 genes, and assemble the qualitative cell-landmark matrix Q1 with cells as rows and one consensus binary landmark column per retained cluster.

## Related tools

- **scQA** (Provides the LC1 module that binarizes the matrix and clusters genes with qualitative quasi-trend-preserved expression patterns to form landmarks) — https://github.com/LD-Lyndee/scQA
- **Seurat** (Seurat-style highly variable gene selection (method of references [11],[25]) used during pre-processing to retain 2000 HVGs before binarization)

## Examples

```
git clone https://github.com/LD-Lyndee/scQA.git; Rscript pre.R xin_matrix.txt; ./main xin_matrix.txt o
```

## Evaluation signals

- q1_landmark_matrix.csv exists and is binary: every entry is 0 or 1, with cells as rows and one landmark column per retained gene cluster
- Gene clusters entering Q1 all contain at least 3 genes (clusters with fewer than 3 genes are pruned after the ρ = 0.7 assignment iteration)
- Cluster template vectors have component 1 only where at least half (fraction ≥ 0.5) of the cluster's genes are expressed
- The gene similarity graph retains approximately the top 1999 gene pairs (one thousandth of all gene pairs) as edges, with edge weights S = 1 − h/n
- The number of landmarks should be minimized toward approximately a dozen via the iterative process while maximizing inclusion of quasi-trend-preserved genes

## Limitations

- Genes may be misclassified in LC1 (and LC2) due to the influence of high dimensionality
- Retaining only 2000 highly variable genes for subsequent clustering led to the exclusion of some informative genes
- The R version used for pre-processing is not stated, and the source code of the cited HVG method (references [11],[25]) is not named, complicating exact reproduction

## Evidence

- [methods] Qualitative landmark constructor (LC1) initially binarizes the matrix and persists to cluster genes with qualitative quasi-trend-preserved expression patterns to form landmarks.: "Qualitative landmark constructor (LC1) initially binarizes the matrix and persists to cluster genes with qualitative quasi-trend-preserved expression patterns to form landmarks."
- [intro] we regard dropouts as weak signals, strengthened by amalgamation of genes sharing analogous binary expression patterns and performing consensus, yielding a binary low-dimensional representation of: "we regard dropouts as weak signals, strengthened by amalgamation of genes sharing analogous binary expression patterns and performing consensus, yielding a binary low-dimensional representation of"
- [methods] we initially filter out genes expressed in fewer than x% cells or more than (100−x)% cells, with x set to 2.5: "we initially filter out genes expressed in fewer than x% cells or more than (100−x)% cells, with x set to 2.5"
- [methods] To identify genes containing valuable information, we retain 2000 highly variable genes: "To identify genes containing valuable information, we retain 2000 highly variable genes"
- [readme] Pre-processing with R provided as pre.R: "Pre-processing with R provided as pre.R"
