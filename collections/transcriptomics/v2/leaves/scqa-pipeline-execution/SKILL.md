---
name: scqa-pipeline-execution
description: Use when raw scRNA-seq count matrices (e.g., SMARTer, Smart-Seq2, CEL-Seq2,
  inDrop data from GEO/ArrayExpress accessions like GSE81608 or E-MTAB-5061) need
  cell type clustering and cluster-specific key gene identification without knowing
  the number of cell types in advance.
when_to_use_negative:
- When the input is already a feature-reduced or pre-clustered representation (e.g.,
  a dimensionality-reduced embedding), since scQA expects the filtered cells-by-genes
  matrix from pre.R
- When the number of cell types must be specified in advance or clusters must be constrained
  — scQA deliberately obviates a predetermined number of cell types
- When genes expressed outside the 2.5%–97.5% cell range carry the signal of interest
  — the pre.R filter removes them, and retaining only 2000 HVGs excluded some informative
  genes
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scQA
  role: 'Main clustering executable: builds qualitative (LC1) and quantitative (LC2)
    landmarks and clusters cells by label propagation; outputs labels.txt'
  repo: https://github.com/LD-Lyndee/scQA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.12.021
    title: 'scQA: A dual-perspective cell type identification model for single cell
      transcriptome data'
  source_content_hash: sha256:c42f745e24636558dace812032bd786313195c960229e28d5efa39a89398f8cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LD-Lyndee/scQA
  tools:
  - CIDR
  - RaceID2
  - SC3
  - SIMLR
  - Seurat
  - TSCAN
  - pcaReduce
  - scGNN
  - scHFC
  - scQA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# scqa-pipeline-execution

## Summary

Run the scQA dual-perspective (qualitative + quantitative) cell type identification pipeline on a preprocessed single-cell RNA-seq count matrix: filter genes, select 2000 highly variable genes, build landmark-based binary and quantitative representations, and obtain cluster labels via label propagation. scQA identifies cell types without requiring a predetermined number of clusters.

## When to use

Use when raw scRNA-seq count matrices (e.g., SMARTer, Smart-Seq2, CEL-Seq2, inDrop data from GEO/ArrayExpress accessions like GSE81608 or E-MTAB-5061) need cell type clustering and cluster-specific key gene identification without knowing the number of cell types in advance.

## When NOT to use

- When the input is already a feature-reduced or pre-clustered representation (e.g., a dimensionality-reduced embedding), since scQA expects the filtered cells-by-genes matrix from pre.R
- When the number of cell types must be specified in advance or clusters must be constrained — scQA deliberately obviates a predetermined number of cell types
- When genes expressed outside the 2.5%–97.5% cell range carry the signal of interest — the pre.R filter removes them, and retaining only 2000 HVGs excluded some informative genes

## Inputs

- Raw scRNA-seq gene count matrix (cells x genes) from public repositories (GEO/ArrayExpress), preprocessed via pre.R into a filtered, log-transformed, [0,1]-normalized cells-by-genes matrix
- Input matrix file for the scQA binary with no row or column names (e.g., xin_matrix.txt)

## Outputs

- labels.txt: per-cell cluster assignments
- Optionally, an Adjusted Rand Index (ARI) against annotated cell type labels, with DEG/key-gene selections using |logFC| > 2 and adjusted p-value < 0.01

## How to apply

Preprocess each raw count matrix with the provided R script pre.R: filter out genes expressed in fewer than 2.5% or more than 97.5% of cells, retain 2000 highly variable genes, log-transform, and normalize entries to the range [0,1] as a cells-by-genes matrix. The scQA binary expects the input matrix in cells-by-genes form with no row or column names. Compile from source via git clone and ./make, then run './main <matrix> o' where 'o' selects the default binning parameter of 6 bins; internally the LC1 module binarizes the matrix to build qualitative landmarks and LC2 clusters quasi-trend-preserved genes to build quantitative landmarks, and the CC module clusters cells by label propagation from density-based seeds along a directed k-nearest neighbor graph. Read the clustering result from labels.txt; optionally evaluate clusters against annotated cell type labels using ARI.

## Related tools

- **scQA** (Main clustering executable: builds qualitative (LC1) and quantitative (LC2) landmarks and clusters cells by label propagation; outputs labels.txt) — https://github.com/LD-Lyndee/scQA

## Examples

```
./main xin_matrix.txt o
```

## Evaluation signals

- labels.txt exists and contains one cluster label per input cell (same number of rows as the cells-by-genes input matrix)
- Run with the 'o' argument confirms the default binning parameter of 6 was used
- When annotated cell type labels are available, compute ARI per dataset; ARI values must fall in [-1, 1] and scQA's benchmark shows the highest ARI on 16 of 20 datasets
- Input file format check: matrix is cells x genes with no row or column names, entries normalized to [0,1]
- Preprocessing check: genes expressed in <2.5% or >97.5% of cells removed and exactly 2000 highly variable genes retained before running the binary

## Limitations

- Retaining only 2000 HVGs for subsequent clustering analysis led to the exclusion of some informative genes
- Genes may be misclassified in LC1 and LC2 as a result of high dimensionality
- Clustering can merge distinct populations — e.g., endothelial and mesenchymal cells were incorrectly grouped into a single cluster (Cluster3), and Cluster8 contained only three outlier cells
- Tested on a Linux server; the input matrix must be headerless (no row/column names)

## Evidence

- [methods] we initially filter out genes expressed in fewer than x% cells or more than (100−x)% cells, with x set to 2.5: "we initially filter out genes expressed in fewer than x% cells or more than (100−x)% cells, with x set to 2.5"
- [methods] All data are log-transformed and normalized to ensure that each entry falls in the range [0,1].: "All data are log-transformed and normalized to ensure that each entry falls in the range [0,1]."
- [readme] The input file must be in the form of a matrix of cells * genes, without names of rows and columns.: "The input file must be in the form of a matrix of cells * genes, without names of rows and columns."
- [abstract] It then clusters cells by employing an ingenious label propagation strategy, obviating the requirement for a predetermined number of cell types.: "It then clusters cells by employing an ingenious label propagation strategy, obviating the requirement for a predetermined number of cell types."
- [results] the retention of only 2000 HVGs for subsequent clustering analysis led to the exclusion of some informative genes: "the retention of only 2000 HVGs for subsequent clustering analysis led to the exclusion of some informative genes"
