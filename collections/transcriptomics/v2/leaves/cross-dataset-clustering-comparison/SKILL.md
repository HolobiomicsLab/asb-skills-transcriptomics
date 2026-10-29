---
name: cross-dataset-clustering-comparison
description: Use when you need to compare a scRNA-seq cell clustering method's accuracy
  (ARI) against published tools across many datasets with known cell type labels,
  e.g. benchmarking scQA on GEO/ArrayExpress data such as GSE57249 or E-MTAB-5061.
when_to_use_negative:
- The input data lacks annotated cell type labels, since the comparison metric (ARI)
  is computed against known labels and cannot be applied without them
- The goal is to identify cell type-specific key genes or marker genes for a single
  dataset rather than to benchmark clustering accuracy across datasets
- You need de novo clustering of a new dataset rather than a method comparison — use
  scQA directly instead of the benchmarking workflow
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scQA
  role: Method under test; run via './main <matrix> o' with default binning 6, output
    labels.txt
  repo: https://github.com/LD-Lyndee/scQA
- name: SC3
  role: Compared clustering method run with default/recommended settings to produce
    baseline clusters for ARI
- name: Seurat
  role: Compared clustering method (shared nearest neighbor graphs + Louvain after
    PCA) run with default settings
- name: CIDR
  role: Compared clustering method (PCA with dropout consideration) run with default
    settings
- name: pcaReduce
  role: Compared clustering method run with default settings
- name: SIMLR
  role: Compared clustering method; failed on the Shekhar dataset due to excessive
    memory usage
- name: TSCAN
  role: Compared clustering method run with default settings
- name: RaceID2
  role: Compared clustering method run with default settings
- name: scGNN
  role: Compared clustering method (graph convolutional network + multi-modal autoencoders)
    run with default settings
- name: scHFC
  role: Compared clustering method run with default settings
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.12.021
    title: 'scQA: A dual-perspective cell type identification model for single cell
      transcriptome data'
  source_content_hash: sha256:db71af611a1e29e21abe32074043368f1e42d10222a105be348c9db33fe11600
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LD-Lyndee/scQA
  tools:
  - CIDR
  - R
  - RaceID2
  - SC3
  - SIMLR
  - SSCC
  - Seurat
  - TSCAN
  - ZIFA
  - pcaReduce
  - scGNN
  - scHFC
  - scQA
  - sscClust
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- benchmark-dataset-clustering-comparison
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# cross-dataset-clustering-comparison

## Summary

Benchmarks a single-cell clustering method (scQA) against nine salient algorithms (SC3, Seurat, CIDR, pcaReduce, SIMLR, TSCAN, RaceID2, scGNN, scHFC) on 20 public scRNA-seq datasets by computing the Adjusted Rand Index (ARI) against annotated cell type labels, producing a per-dataset, per-method comparison table.

## When to use

Use when you need to compare a scRNA-seq cell clustering method's accuracy (ARI) against published tools across many datasets with known cell type labels, e.g. benchmarking scQA on GEO/ArrayExpress data such as GSE57249 or E-MTAB-5061.

## When NOT to use

- The input data lacks annotated cell type labels, since the comparison metric (ARI) is computed against known labels and cannot be applied without them
- The goal is to identify cell type-specific key genes or marker genes for a single dataset rather than to benchmark clustering accuracy across datasets
- You need de novo clustering of a new dataset rather than a method comparison — use scQA directly instead of the benchmarking workflow

## Inputs

- Raw scRNA-seq count matrices from 20 public GEO/ArrayExpress accessions (e.g. GSE57249, GSE36552, GSE45719, GSE75140, GSE86469, GSE81252, GSE81608, GSE84133, GSE85241, E-MTAB-5061, GSE109033, GSE65525, GSE74672, GSE60361, E-MTAB-7716, GSE87544, GSE127465, GSE93374, GSE81904) with annotated cell type labels
- Preprocessed cells-by-genes matrix (filtered to genes expressed in 2.5%-97.5% of cells, top 2000 HVGs, log-transformed, normalized to [0,1])

## Outputs

- Cluster assignments per method per dataset (scQA writes labels.txt)
- Per-dataset per-method ARI values assembled into a 20 x 10 comparison table (ari_comparison_table.csv)

## How to apply

Download the twenty annotated scRNA-seq datasets (e.g. Biase GSE57249, Yan GSE36552, Deng GSE45719, Xin GSE81608, Shekhar GSE81904) with their cell type labels. Preprocess each raw count matrix with the paper's R script (pre.R): filter out genes expressed in fewer than 2.5% or more than 97.5% of cells, retain 2000 highly variable genes, log-transform, and normalize entries to [0,1] as a cells-by-genes matrix. Run scQA on each preprocessed matrix with './main <matrix> o' (binning parameter default 6) and collect cluster assignments from labels.txt; run the nine compared tools with default/recommended settings, recording failures such as SIMLR exceeding memory on the 26,830-cell Shekhar dataset. Compute the ARI (chance-corrected Rand Index, Eq. 14) between each method's predicted clusters and the annotated labels per dataset, then assemble a 20-row x 10-method table (ari_comparison_table.csv).

## Related tools

- **scQA** (Method under test; run via './main <matrix> o' with default binning 6, output labels.txt) — https://github.com/LD-Lyndee/scQA
- **SC3** (Compared clustering method run with default/recommended settings to produce baseline clusters for ARI)
- **Seurat** (Compared clustering method (shared nearest neighbor graphs + Louvain after PCA) run with default settings)
- **CIDR** (Compared clustering method (PCA with dropout consideration) run with default settings)
- **pcaReduce** (Compared clustering method run with default settings)
- **SIMLR** (Compared clustering method; failed on the Shekhar dataset due to excessive memory usage)
- **TSCAN** (Compared clustering method run with default settings)
- **RaceID2** (Compared clustering method run with default settings)
- **scGNN** (Compared clustering method (graph convolutional network + multi-modal autoencoders) run with default settings)
- **scHFC** (Compared clustering method run with default settings)

## Examples

```
./main xin_matrix.txt o
```

## Evaluation signals

- Each computed ARI value falls within [-1, 1]
- scQA achieves the highest ARI on 16 of the 20 datasets and is competitive with the best-performing method on the remaining four, matching the paper's reported rankings
- The comparison table has 20 dataset rows and 10 method columns (scQA plus the nine compared tools)
- Per-dataset cluster counts from scQA's labels.txt match the annotated number of cell types on benchmark datasets (e.g. 4 types for Xin GSE81608, 9 for Muraro GSE85241)
- Failures are recorded explicitly rather than as ARI values — e.g. SIMLR failed on the 26,830-cell Shekhar dataset due to excessive memory usage

## Limitations

- SIMLR failed to estimate the number of clusters and cluster cells on the Shekhar dataset due to excessive memory usage, so its ARI is unavailable there
- Retention of only 2000 highly variable genes for clustering led to the exclusion of some informative genes
- Genes may be misclassified in LC1 and LC2 due to high dimensionality
- The comparison depends on the pre.R preprocessing choices (2.5%/97.5% gene filtering, 2000 HVGs, [0,1] normalization), which may not be the optimal defaults for every method

## Evidence

- [abstract] Validated on 20 publicly available scRNA-seq datasets, scQA consistently outperforms other salient tools.: "Validated on 20 publicly available scRNA-seq datasets, scQA consistently outperforms other salient tools."
- [methods] filter genes and retain 2000 HVGs: "we initially filter out genes expressed in fewer than x% cells or more than (100−x)% cells, with x set to 2.5"
- [methods] log-transform and normalize: "All data are log-transformed and normalized to ensure that each entry falls in the range [0,1]."
- [readme] README usage of scQA binary: "./main xin_matrix.txt o"
- [results] SIMLR memory failure on Shekhar: "SIMLR failed to estimate the number of clusters and cluster cells on Shekhar dataset due to excessive memory usage"
- [results] HVG limitation: "the retention of only 2000 HVGs for subsequent clustering analysis led to the exclusion of some informative genes"
