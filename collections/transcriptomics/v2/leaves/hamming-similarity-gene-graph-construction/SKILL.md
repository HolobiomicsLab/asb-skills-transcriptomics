---
name: hamming-similarity-gene-graph-construction
description: Use when converting a sparse, dropout-prone scRNA-seq expression matrix
  (e.g. Xin GSE81608, 2000 highly variable genes, entries in [0,1]) into a binary
  gene-gene similarity graph to derive qualitative cell landmarks.
when_to_use_negative:
- The input is already a qualitative cell-landmark matrix Q1 or a feature table with
  no raw gene expression values to binarize
- The analysis goal is the quantitative low-dimensional representation (LC2), which
  clusters genes with quasi-trend-preserved expression patterns instead of binary
  patterns
- Very few genes remain after preprocessing, so the top 1999 gene pairs (one thousandth
  of all gene pairs) cannot be meaningfully retained as graph edges
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scQA
  role: Implements the LC1 module that binarizes the matrix, computes normalized Hamming
    similarity, and builds the gene similarity graph
  repo: https://github.com/LD-Lyndee/scQA
- name: Seurat
  role: Reference method (references [11], [25]) whose style is used to select the
    2000 highly variable genes fed into the binarization step
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.12.021
    title: 'scQA: A dual-perspective cell type identification model for single cell
      transcriptome data'
  source_content_hash: sha256:0b08b92f9d5420bd9e80830c6242fa994c1c9ec694e948f4041958b96bcc8bac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LD-Lyndee/scQA
  tools:
  - Seurat
  - scQA
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# hamming-similarity-gene-graph-construction

## Summary

Builds a binary gene similarity graph inside scQA's qualitative landmark constructor (LC1) by computing normalized Hamming similarity S = 1 − h/n between genes binarized from dropout-prone scRNA-seq data, then retaining the top 1999 gene pairs as edges so genes sharing analogous zero/non-zero expression patterns can be amalgamated into landmarks.

## When to use

Use when converting a sparse, dropout-prone scRNA-seq expression matrix (e.g. Xin GSE81608, 2000 highly variable genes, entries in [0,1]) into a binary gene-gene similarity graph to derive qualitative cell landmarks.

## When NOT to use

- The input is already a qualitative cell-landmark matrix Q1 or a feature table with no raw gene expression values to binarize
- The analysis goal is the quantitative low-dimensional representation (LC2), which clusters genes with quasi-trend-preserved expression patterns instead of binary patterns
- Very few genes remain after preprocessing, so the top 1999 gene pairs (one thousandth of all gene pairs) cannot be meaningfully retained as graph edges

## Inputs

- Preprocessed expression matrix (cells × genes) for Xin GSE81608 after filtering genes expressed in fewer than 2.5% or more than 97.5% of cells, retaining 2000 highly variable genes, log-transformed and normalized to [0,1]
- Binarized gene vectors sorted by decreasing number of zeros

## Outputs

- Binary gene-gene similarity matrix S = 1 − h/n (normalized Hamming distance)
- Gene similarity graph with the top 1999 gene pairs retained as edges
- Qualitative cell-landmark matrix Q1 (cells as rows, one consensus binary landmark column per retained cluster), saved as q1_landmark_matrix.csv

## How to apply

First binarize each gene vector in LC1 (entry 0 if expression is zero, 1 otherwise) and sort genes by decreasing number of zeros. Compute binary gene-gene similarity S = 1 − h/n (normalized Hamming distance) between each gene and its 0.05×m neighbors on each side in the sorted sequence, then build a gene similarity graph retaining the top 1999 gene pairs (one thousandth of all gene pairs) as edges. Take connected components with more than two vertices as initial gene clusters and define each cluster template vector with component 1 where at least half (fraction ≥ 0.5) of the cluster's genes are expressed. Iteratively assign remaining genes by component-wise similarity increments with the subsidiary vector a_jk = max(0.5 − mean expression of the cluster at component k, 0): add gene Gi to cluster C_J if S(Gi,C_J) exceeds ρ = 0.7, otherwise leave it as a singleton; run one iteration with ρ = 0.7, then prune clusters with fewer than 3 genes. The rationale is that dropouts are treated as weak signals and strengthened by amalgamating genes with similar binary expression patterns via consensus, yielding a binary low-dimensional qualitative representation of cells assembled into matrix Q1 (cells as rows, one consensus binary landmark column per retained cluster, saved as q1_landmark_matrix.csv).

## Related tools

- **scQA** (Implements the LC1 module that binarizes the matrix, computes normalized Hamming similarity, and builds the gene similarity graph) — https://github.com/LD-Lyndee/scQA
- **Seurat** (Reference method (references [11], [25]) whose style is used to select the 2000 highly variable genes fed into the binarization step)

## Examples

```
git clone https://github.com/LD-Lyndee/scQA.git; cd scQA; ./make; Rscript pre.R; ./main xin_matrix.txt o
```

## Evaluation signals

- The binarized gene vectors are sorted by decreasing number of zeros before similarity computation
- The similarity matrix uses S = 1 − h/n with comparisons restricted to 0.05×m neighbors on each side in the sorted sequence
- Exactly the top 1999 gene pairs (one thousandth of all gene pairs) are retained as graph edges
- Initial gene clusters are connected components with more than two vertices, and clusters with fewer than 3 genes are pruned after one assignment iteration with ρ = 0.7
- The output q1_landmark_matrix.csv has cells as rows and one consensus binary landmark column per retained cluster, with 1 where at least half (≥ 0.5) of a cluster's genes are expressed

## Limitations

- Genes may be misclassified in LC1 (and LC2) due to the influence of high dimensionality
- Retaining only 2000 HVGs for subsequent clustering analysis led to the exclusion of some informative genes
- The README notes the input must be a cells × genes matrix without row or column names, and only the final clustering labels.txt (not Q1) is emitted by the ./main executable

## Evidence

- [methods] Qualitative landmark constructor (LC1) initially binarizes the matrix and persists to cluster genes with qualitative quasi-trend-preserved expression patterns to form landmarks.: "Qualitative landmark constructor (LC1) initially binarizes the matrix and persists to cluster genes with qualitative quasi-trend-preserved expression patterns to form landmarks."
- [intro] we regard dropouts as weak signals, strengthened by amalgamation of genes sharing analogous binary expression patterns and performing consensus, yielding a binary low-dimensional representation of: "we regard dropouts as weak signals, strengthened by amalgamation of genes sharing analogous binary expression patterns and performing consensus, yielding a binary low-dimensional representation of"
- [readme] The input file must be in the form of a matrix of cells * genes, without names of rows and columns.: "The input file must be in the form of a matrix of cells * genes, without names of rows and columns."
- [results] genes may be misclassified in LC1 and LC2 as the influence of the high dimensionality: "genes may be misclassified in LC1 and LC2 as the influence of the high dimensionality"
