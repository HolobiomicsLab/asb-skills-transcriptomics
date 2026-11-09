---
name: adjusted-rand-index-evaluation
description: Use when a clustering method's predicted cell labels must be scored against
  ground-truth cell type annotations on single-cell RNA-seq data, or when assembling
  a per-dataset, per-method clustering accuracy benchmark table.
when_to_use_negative:
- No ground-truth cell type labels exist for the dataset — ARI requires reference
  annotations and cannot be computed from predicted labels alone
- The task is evaluating identified key genes rather than clustering accuracy — the
  paper validates key genes via DEG criteria (|logFC| > 2, adjusted p-value < 0.01),
  not ARI
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: scQA
  role: Method under evaluation; run with default binning of 6 and cluster labels
    collected from labels.txt for ARI computation
  repo: https://github.com/LD-Lyndee/scQA
- name: SC3
  role: Compared baseline clustering method run on each preprocessed dataset at default
    settings before ARI scoring
- name: Seurat
  role: Compared baseline clustering method (shared nearest neighbor graphs + Louvain
    after PCA) scored by ARI
- name: CIDR
  role: Compared baseline clustering method (PCA with dropout imputation) scored by
    ARI
- name: pcaReduce
  role: Compared baseline clustering method scored by ARI
- name: SIMLR
  role: Compared baseline clustering method; failed with excessive memory usage on
    the 26,830-cell Shekhar dataset
- name: TSCAN
  role: Compared baseline clustering method scored by ARI
- name: RaceID2
  role: Compared baseline clustering method scored by ARI
- name: scGNN
  role: Compared baseline clustering method (graph convolutional network autoencoders)
    scored by ARI
- name: scHFC
  role: Compared baseline clustering method scored by ARI
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.12.021
    title: 'scQA: A dual-perspective cell type identification model for single cell
      transcriptome data'
  source_content_hash: sha256:e19d5bcfb77650a955cba5ec23b07064f06cee48720759e1d7d3635a3470fda7
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# adjusted-rand-index-evaluation

## Summary

Compute the Adjusted Rand Index (ARI), a chance-corrected Rand Index, to score predicted single-cell cluster assignments against annotated cell type labels — used here to benchmark scQA against nine clustering tools (SC3, Seurat, CIDR, pcaReduce, SIMLR, TSCAN, RaceID2, scGNN, scHFC) across 20 public scRNA-seq datasets.

## When to use

Use when a clustering method's predicted cell labels must be scored against ground-truth cell type annotations on single-cell RNA-seq data, or when assembling a per-dataset, per-method clustering accuracy benchmark table.

## When NOT to use

- No ground-truth cell type labels exist for the dataset — ARI requires reference annotations and cannot be computed from predicted labels alone
- The task is evaluating identified key genes rather than clustering accuracy — the paper validates key genes via DEG criteria (|logFC| > 2, adjusted p-value < 0.01), not ARI

## Inputs

- Predicted cluster assignments per dataset (scQA labels.txt; equivalently the label vectors of the nine compared tools)
- Ground-truth cell type annotations for each of the 20 scRNA-seq datasets (e.g. GSE57249 Biase, GSE36552 Yan, GSE45719 Deng, GSE81904 Shekhar, etc.)
- Preprocessed cells-by-genes count matrices (2.5%/97.5% gene filter, 2000 HVGs, log-transformed, normalized to [0,1])

## Outputs

- Per-dataset, per-method ARI value in [-1, 1]
- Benchmark comparison table (ari_comparison_table.csv, 20 datasets x 10 methods) with per-dataset method rankings

## How to apply

Run each clustering tool (scQA via './main <matrix> o' with default binning of 6; comparison tools at default/recommended settings) on a preprocessed cells-by-genes matrix — genes filtered at the 2.5%/97.5% expression boundaries, 2000 highly variable genes retained, log-transformed and normalized to [0,1] — and collect predicted labels from labels.txt. Then compute the ARI between predicted clusters and the annotated cell type labels for every dataset, per the paper's metric definition (Rand Index with chance correction, Eq. 14). Record tool failures as missing entries rather than zero scores (e.g. SIMLR exhausted memory on the 26,830-cell Shekhar dataset). Assemble the per-dataset, per-method ARI values into a single table (e.g. 20 rows x 10 methods, ari_comparison_table.csv) and compare per-dataset rankings.

## Related tools

- **scQA** (Method under evaluation; run with default binning of 6 and cluster labels collected from labels.txt for ARI computation) — https://github.com/LD-Lyndee/scQA
- **SC3** (Compared baseline clustering method run on each preprocessed dataset at default settings before ARI scoring)
- **Seurat** (Compared baseline clustering method (shared nearest neighbor graphs + Louvain after PCA) scored by ARI)
- **CIDR** (Compared baseline clustering method (PCA with dropout imputation) scored by ARI)
- **pcaReduce** (Compared baseline clustering method scored by ARI)
- **SIMLR** (Compared baseline clustering method; failed with excessive memory usage on the 26,830-cell Shekhar dataset)
- **TSCAN** (Compared baseline clustering method scored by ARI)
- **RaceID2** (Compared baseline clustering method scored by ARI)
- **scGNN** (Compared baseline clustering method (graph convolutional network autoencoders) scored by ARI)
- **scHFC** (Compared baseline clustering method scored by ARI)

## Examples

```
./main xin_matrix.txt o && python -c "from sklearn.metrics import adjusted_rand_score; print(adjusted_rand_score(labels_true, [int(l) for l in open('labels.txt')]))"
```

## Evaluation signals

- Every computed ARI value falls within the valid range [-1, 1]
- Per-dataset rankings reproduce the paper's result that scQA attains the highest ARI on 16 of 20 datasets and is competitive on the remaining four
- The assembled table has exactly 20 dataset rows x 10 method columns; SIMLR's entry for the Shekhar dataset is recorded as a failure, not an ARI score
- Cluster labels input to the ARI computation cover the same cell set as the reference annotations (e.g. 56 cells for Biase GSE57249, 26,830 cells for Shekhar GSE81904)
- ARI is computed as the chance-corrected Rand Index per Eq. 14, not the unadjusted Rand Index

## Limitations

- SIMLR failed to estimate the number of clusters and cluster cells on the Shekhar dataset due to excessive memory usage, leaving a gap in the benchmark table
- The upstream preprocessing (retaining only 2000 highly variable genes) excluded some informative genes, affecting clustering quality and hence ARI values
- ARI depends on the quality of the reference cell type annotations; scQA itself does not require a predetermined number of cell types, but the reference labels do

## Evidence

- [abstract] Validated on 20 publicly available scRNA-seq datasets, scQA consistently outperforms other salient tools.: "Validated on 20 publicly available scRNA-seq datasets, scQA consistently outperforms other salient tools."
- [results] SIMLR failure on the largest dataset: "SIMLR failed to estimate the number of clusters and cluster cells on Shekhar dataset due to excessive memory usage"
- [methods] Preprocessing thresholds feeding the clustering that ARI scores: "we initially filter out genes expressed in fewer than x% cells or more than (100−x)% cells, with x set to 2.5"
- [readme] scQA cluster output used as ARI input: "There are two input parameters：

- **filename** Input matrix file. 
- **binning** Define number of bins, default: 6. "o" for using the default parameter."
