---
name: multi-tool-clustering-benchmarking
description: Use when comparing a new single-cell clustering method's accuracy against
  established tools on datasets with known cell-type labels, using ARI as the external
  validation metric.
when_to_use_negative:
- Datasets lack ground-truth cell-type annotations, so ARI cannot be computed and
  external validation is impossible
- The goal is clustering a single dataset rather than comparing methods across many
  datasets
- Input is already a preprocessed/normalized feature table — rerunning the 2.5%–97.5%
  gene filter and 2000-HVG selection would discard different genes than intended
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scQA
  role: Method under evaluation; clusters cells via qualitative (LC1) and quantitative
    (LC2) landmark construction plus label propagation, run with default binning of
    6
  repo: https://github.com/LD-Lyndee/scQA
- name: SC3
  role: 'Compared baseline: consensus k-means clustering on transformed distance matrices
    using PCA or eigenvectors of Laplacian'
- name: Seurat
  role: 'Compared baseline: shared nearest neighbor graphs with Louvain community
    detection following PCA'
- name: CIDR
  role: 'Compared baseline: PCA while considering dropout events'
- name: scGNN
  role: 'Compared baseline: graph convolutional network in multi-modal autoencoders
    for gene imputation and cell clustering'
- name: pcaReduce
  role: Compared baseline clustering method
- name: RaceID2
  role: Compared baseline clustering method
- name: SIMLR
  role: Compared baseline clustering method; failed on Shekhar dataset due to excessive
    memory usage
- name: TSCAN
  role: Compared baseline clustering method
- name: scHFC
  role: Compared baseline clustering method
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.12.021
    title: 'scQA: A dual-perspective cell type identification model for single cell
      transcriptome data'
  source_content_hash: sha256:3174720f541f9c04e3a5d9e78eb95bb5d0edcb683fd0a36517496b9622a0829f
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# multi-tool-clustering-benchmarking

## Summary

Benchmark scQA against nine clustering algorithms (SC3, Seurat, CIDR, pcaReduce, SIMLR, TSCAN, RaceID2, scGNN, scHFC) on 20 public scRNA-seq datasets with annotated cell types, scoring each with Adjusted Rand Index (ARI).

## When to use

Use when comparing a new single-cell clustering method's accuracy against established tools on datasets with known cell-type labels, using ARI as the external validation metric.

## When NOT to use

- Datasets lack ground-truth cell-type annotations, so ARI cannot be computed and external validation is impossible
- The goal is clustering a single dataset rather than comparing methods across many datasets
- Input is already a preprocessed/normalized feature table — rerunning the 2.5%–97.5% gene filter and 2000-HVG selection would discard different genes than intended

## Inputs

- Twenty raw scRNA-seq count matrices from GEO/ArrayExpress and hemberg-lab with annotated cell type labels (cells x genes)
- pre.R preprocessing script from the scQA repository
- scQA compiled binary (./main) from https://github.com/LD-Lyndee/scQA
- Compared tools: SC3, Seurat, CIDR, pcaReduce, SIMLR, TSCAN, RaceID2, scGNN, scHFC

## Outputs

- labels.txt cluster assignments from scQA per dataset
- ari_comparison_table.csv: per-dataset per-method ARI values (20 rows x 10 methods)
- Per-method failure log (e.g. SIMLR memory failure on Shekhar)

## How to apply

Download the 20 annotated scRNA-seq datasets (e.g. GSE57249 Biase, GSE36552 Yan, GSE81608 Xin, GSE84133 Baronm, E-MTAB-5061 Segerstolpe, GSE81904 Shekhar) with cell-type labels. Preprocess each raw count matrix with the scQA pre.R script: filter genes expressed in fewer than 2.5% or more than 97.5% of cells, retain 2000 highly variable genes, log-transform, and normalize entries to [0,1] as a cells-by-genes matrix. Run scQA with './main <matrix> o' using the default binning parameter of 6, reading clusters from labels.txt; run the nine compared tools with default/recommended settings, recording failures such as SIMLR exhausting memory on the 26,830-cell Shekhar dataset. Compute ARI (chance-corrected Rand Index, Eq. 14) between predicted clusters and annotations for each method-dataset pair and assemble a 20-row x 10-method comparison table.

## Related tools

- **scQA** (Method under evaluation; clusters cells via qualitative (LC1) and quantitative (LC2) landmark construction plus label propagation, run with default binning of 6) — https://github.com/LD-Lyndee/scQA
- **SC3** (Compared baseline: consensus k-means clustering on transformed distance matrices using PCA or eigenvectors of Laplacian)
- **Seurat** (Compared baseline: shared nearest neighbor graphs with Louvain community detection following PCA)
- **CIDR** (Compared baseline: PCA while considering dropout events)
- **scGNN** (Compared baseline: graph convolutional network in multi-modal autoencoders for gene imputation and cell clustering)
- **pcaReduce** (Compared baseline clustering method)
- **RaceID2** (Compared baseline clustering method)
- **SIMLR** (Compared baseline clustering method; failed on Shekhar dataset due to excessive memory usage)
- **TSCAN** (Compared baseline clustering method)
- **scHFC** (Compared baseline clustering method)

## Examples

```
git clone https://github.com/LD-Lyndee/scQA.git && cd scQA && ./make && ./main xin_matrix.txt o
```

## Evaluation signals

- Every computed ARI value falls within [-1, 1]
- scQA achieves the highest ARI on 16 of the 20 datasets and is competitive with the best tool on the remaining four
- Preprocessed matrices are cells-by-genes, contain ~2000 highly variable genes, and entries lie in [0,1]
- scQA labels.txt contains one cluster assignment per cell matching the input matrix row count
- Method failures (e.g. SIMLR memory exhaustion on Shekhar's 26,830 cells) are recorded rather than scored as ARI 0

## Limitations

- SIMLR failed to estimate the number of clusters and cluster cells on the Shekhar dataset due to excessive memory usage
- Retaining only 2000 HVGs excluded some informative genes
- R version used for pre-processing is not stated; HVG selection method cites references [11], [25] without naming code
- Compared tools are run with default/recommended settings, which may not be optimal per dataset

## Evidence

- [abstract] Validated on 20 publicly available scRNA-seq datasets, scQA consistently outperforms other salient tools.: "Validated on 20 publicly available scRNA-seq datasets, scQA consistently outperforms other salient tools."
- [methods] Gene filtering thresholds: "we initially filter out genes expressed in fewer than x% cells or more than (100−x)% cells, with x set to 2.5"
- [methods] HVG retention and normalization: "To identify genes containing valuable information, we retain 2000 highly variable genes"
- [results] SIMLR failure: "SIMLR failed to estimate the number of clusters and cluster cells on Shekhar dataset due to excessive memory usage"
- [readme] README usage: "There are two input parameters：

- **filename** Input matrix file. 
- **binning** Define number of bins, default: 6."
