---
name: scatac-seq-gene-activity-matrix-analysis
description: Use when input is a scATAC-seq dataset converted to a cell-by-gene activity
  score matrix and the goal is cell clustering and marker-gene identification via
  differential expression of the resulting clusters.
when_to_use_negative:
- Input is a raw scATAC-seq cell-by-bin/peak matrix not yet converted to gene activity
  scores — use a converter such as BinToGene or cisTopic topic modeling first.
- Data are already annotated single-cell gene expression with reference labels available
  for supervised transfer — scanpy Ingest or SingleR label transfer is more appropriate
  than unsupervised cluster-then-DE annotation.
- Protein/imaging data (CODEX, CITE-seq) — Cellar's protein workflows such as STvEA
  mapping, not gene-activity DE analysis, apply.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: Cellar
  role: Interactive graphical pipeline orchestrating preprocessing, PCA, clustering,
    and DE for the cell-by-gene scATAC-seq analysis
  repo: https://github.com/euxhenh/cellar
- name: scanpy
  role: Preprocessing of the data (CPM 1e5 normalization, log1p, scaling)
- name: scikit-learn
  role: PCA with randomized SVD solver to reduce the matrix to 40 dimensions
- name: faiss
  role: Approximate nearest-neighbor graph construction (15 neighbors) to speed up
    Leiden clustering
  repo: https://github.com/facebookresearch/faiss
- name: diffxpy
  role: One-vs-rest differential gene testing per Leiden cluster with Welch's t-test
  repo: https://github.com/theislab/diffxpy
- name: AnnData
  role: In-memory/mapped storage of the cell-by-gene matrix for large datasets
- name: GSEApy
  role: Optional enrichment analysis of the top 500 DE genes by fold-change
  repo: https://github.com/zqfang/GSEApy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:f435b071cadf5076fd6892b6577d82aad61c7636a38520abecdff3945000c74e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/euxhenh/cellar
  tools:
  - AnnData
  - Cellar
  - GSEApy
  - diffxpy
  - faiss
  - scanpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# scatac-seq-gene-activity-matrix-analysis

## Summary

Analyze scATAC-seq cell-by-gene activity score matrices (e.g., PBMC 10k converted to gene activity scores) by CPM/log1p normalization, randomized-SVD PCA, faiss approximate-neighbor Leiden clustering, and diffxpy Welch's t-test DE to recover known cell-type markers such as the NK-cell marker KLRD1.

## When to use

Use when input is a scATAC-seq dataset converted to a cell-by-gene activity score matrix and the goal is cell clustering and marker-gene identification via differential expression of the resulting clusters.

## When NOT to use

- Input is a raw scATAC-seq cell-by-bin/peak matrix not yet converted to gene activity scores — use a converter such as BinToGene or cisTopic topic modeling first.
- Data are already annotated single-cell gene expression with reference labels available for supervised transfer — scanpy Ingest or SingleR label transfer is more appropriate than unsupervised cluster-then-DE annotation.
- Protein/imaging data (CODEX, CITE-seq) — Cellar's protein workflows such as STvEA mapping, not gene-activity DE analysis, apply.

## Inputs

- scATAC-seq cell-by-gene activity score matrix (e.g., PBMC 10k converted from a cell-by-bin matrix via BinToGene with GENCODE v35 gene annotations)
- AnnData object (memory-mappable .h5ad)
- Optional cell-type annotation reference (e.g., Supplementary Fig. 3 NK-cell labels)

## Outputs

- Leiden cluster labels per cell (resolution 1, 15 faiss neighbors)
- Per-cluster one-vs-rest DE results (Welch's t-test statistics, log fold-changes) compiled into de_genes_per_cluster.csv
- Top marker genes per cluster (e.g., KLRD1 for NK-cell clusters 0 and 4)

## How to apply

Load the cell-by-gene activity score matrix as an AnnData object (memory-mapped mode for large datasets), then CPM total-count normalize to 1e5, log1p-transform, and scale to unit variance and zero mean. Reduce to 40 dimensions with scikit-learn's PCA using a randomized SVD solver, matching the Cellar workflow for scATAC-seq data. Build an approximate neighbors graph with faiss using 15 neighbors and cluster cells with Leiden community detection at resolution 1. Finally, perform one-vs-rest differential gene testing per Leiden cluster with diffxpy using Welch's t-test and rank genes by log fold-change; optionally select the top 500 DE genes by fold-change for downstream enrichment analysis (e.g., gseapy). Correct application is judged by confirming known markers — e.g., KLRD1 for natural killer cells — appear among the top-ranked DE genes of the appropriate clusters (clusters 0 and 4 in the PBMC 10k analysis, consistent with Supplementary Fig. 3).

## Related tools

- **Cellar** (Interactive graphical pipeline orchestrating preprocessing, PCA, clustering, and DE for the cell-by-gene scATAC-seq analysis) — https://github.com/euxhenh/cellar
- **scanpy** (Preprocessing of the data (CPM 1e5 normalization, log1p, scaling))
- **scikit-learn** (PCA with randomized SVD solver to reduce the matrix to 40 dimensions)
- **faiss** (Approximate nearest-neighbor graph construction (15 neighbors) to speed up Leiden clustering) — https://github.com/facebookresearch/faiss
- **diffxpy** (One-vs-rest differential gene testing per Leiden cluster with Welch's t-test) — https://github.com/theislab/diffxpy
- **AnnData** (In-memory/mapped storage of the cell-by-gene matrix for large datasets)
- **GSEApy** (Optional enrichment analysis of the top 500 DE genes by fold-change) — https://github.com/zqfang/GSEApy

## Examples

```
import scanpy as sc; adata = sc.read_h5ad('pbmc_gene_activity.h5ad'); sc.pp.normalize_total(adata, target_sum=1e5); sc.pp.log1p(adata); sc.pp.scale(adata)
```

## Evaluation signals

- Output table de_genes_per_cluster.csv exists and contains per-cluster Welch's t-test results ranked by log fold-change.
- KLRD1 appears among the top DE genes for clusters 0 and 4, consistent with the NK-cell annotation in Supplementary Fig. 3.
- Matrix passes through CPM normalization to total count 1e5 followed by log1p-transform and scaling to zero mean and unit variance.
- PCA reduces the matrix to exactly 40 dimensions using the randomized SVD solver.
- Neighbors graph built with faiss using 15 neighbors; Leiden clustering at resolution 1 produces a partition covering all filtered cells.

## Limitations

- The diffxpy repository is unmaintained; the README recommends PyDESeq2 for pure-python differential expression analysis.
- Software versions for Cellar, Python, Dash, and faiss are not stated, and reproducibility depends on an external web server (https://cellar.cmu.hubmapconsortium.org/app/cellar) rather than a locally installable archived artifact.
- DE-based annotation is unsupervised and relies on known markers; cells with no annotations cannot be validated against a reference.
- Conversion from cell-by-bin to cell-by-gene depends on GENCODE gene annotations (v34/v35) and interval extension parameters in BinToGene.

## Evidence

- [methods] The PBMC scATAC-seq dataset was converted to a gene activity score matrix: "The PBMC scATAC-seq dataset was converted to a gene activity score matrix"
- [methods] CPM normalization and scaling parameters: "The data matrix was then CPM total count normalized (total count = 1e5) and log1p-transformed."
- [methods] PCA dimensionality reduction parameters: "We used the PCA implementation of the scikit-learn package with a randomized SVD solver."
- [methods] faiss neighbor graph for Leiden: [evidence span withheld — anchor status: not_found]
- [methods] diffxpy for DE testing: "Differential gene expression analysis was performed with diffxpy (https://github.com/theislab/diffxpy)"
- [readme] BinToGene purpose (readme): [evidence span withheld — anchor status: not_found]
- [readme] diffxpy unmaintained warning (readme): [evidence span withheld — anchor status: external_doc]
