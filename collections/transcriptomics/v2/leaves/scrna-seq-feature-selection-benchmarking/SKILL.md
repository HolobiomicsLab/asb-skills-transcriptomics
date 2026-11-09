---
name: scrna-seq-feature-selection-benchmarking
description: Use when evaluating whether a feature selection method improves single-cell
  clustering accuracy versus baselines, or when quantifying runtime/memory scaling
  on downsamplings up to million-cell scRNA-seq datasets.
when_to_use_negative:
- Input is already a curated feature table or the clustering features are fixed —
  feature selection benchmarking is unnecessary
- Dataset is so small that scalability profiling (memory/runtime scaling) is meaningless,
  e.g. a few hundred cells
- Reference-based cell type annotation is the goal rather than unsupervised feature
  selection for clustering
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: DUBStepR
  role: Correlation-based feature selection method under test; selects optimal feature
    gene set from gene-gene correlation matrix
  repo: github.com/prabhakarlab/DUBStepR
- name: Seurat
  role: Hosts the Seurat object, normalization, PCA, FindNeighbors and FindClusters
    for downstream clustering with each method's features
  repo: https://satijalab.org/seurat/
- name: M3Drop
  role: Baseline feature selection method (M3DropDANB) compared in the benchmark
- name: GiniClust
  role: Baseline Gini-index feature selection method; fails at 100k cells in the scalability
    test
- name: hdf5r
  role: Loads 10X h5 count matrices into the Seurat object
- name: R
  role: Runtime environment; requires version >= 3.5.0; fresh session per method for
    timing/memory profiling
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-26085-2
    title: DUBStepR is a scalable correlation-based feature selection method for accurately
      clustering single-cell data
  source_content_hash: sha256:b62138b5274d8c58c7fdfb77bb90589186c7ea19ace2b09bd3bba925b1043c0d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DUBStepR
  - GiniClust
  - M3Drop
  - R
  - Seurat
  - hdf5r
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# scrna-seq-feature-selection-benchmarking

## Summary

Benchmark correlation-based feature selection (DUBStepR) against existing scRNA-seq methods (HVGDisp, HVGVST, trendVar, devianceFS, M3DropDANB, GiniClust, HLG) for clustering accuracy and scalability in time/memory across dataset sizes up to 1M+ cells.

## When to use

Use when evaluating whether a feature selection method improves single-cell clustering accuracy versus baselines, or when quantifying runtime/memory scaling on downsamplings up to million-cell scRNA-seq datasets.

## When NOT to use

- Input is already a curated feature table or the clustering features are fixed — feature selection benchmarking is unnecessary
- Dataset is so small that scalability profiling (memory/runtime scaling) is meaningless, e.g. a few hundred cells
- Reference-based cell type annotation is the goal rather than unsupervised feature selection for clustering

## Inputs

- Raw scRNA-seq count matrix (feature-by-cell) as 10X h5 loaded via hdf5r/Read10X_h5
- Preprocessed log-normalized Seurat object
- Downsampled datasets (e.g. 10k, 100k cells) and full 1M+ cell Cao et al. mouse organogenesis dataset (GSE119945)

## Outputs

- Clustering assignments (Seurat FindClusters clusters) per feature selection method
- runtime_memory_benchmark.csv: comparison table of wall-clock time and peak memory versus cell count
- Set of selected variable/optimal feature genes per method

## How to apply

Load count matrices (e.g. 10X pbmc_1k_v2, GSE119945 mouse organogenesis) into a Seurat object via hdf5r and log-normalize. For each method run in a fresh R session (DUBStepR with min.cells = 0.05*ncol, optimise.features = TRUE, k = 10, num.pcs = 20), assign returned optimal feature genes as Seurat variable features, scale all genes, run PCA (npcs = 30), build the neighbor graph on PCs 1:10 and cluster with FindClusters. For the scalability arm, generate 10,000-cell and 100,000-cell downsamplings plus the full 1M+ cell dataset and record per-method wall-clock runtime and peak memory using R's system timing and memory profiling utilities. DUBStepR filters out mitochondrial, ribosomal, spike-in, and pseudogenes before constructing the GGC matrix, binning genes by expression to compute correlation range z-scores, running stepwise regression to find minimally redundant seed features, and expanding them by guilt-by-association to the elbow-point feature set.

## Related tools

- **DUBStepR** (Correlation-based feature selection method under test; selects optimal feature gene set from gene-gene correlation matrix) — github.com/prabhakarlab/DUBStepR
- **Seurat** (Hosts the Seurat object, normalization, PCA, FindNeighbors and FindClusters for downstream clustering with each method's features) — https://satijalab.org/seurat/
- **M3Drop** (Baseline feature selection method (M3DropDANB) compared in the benchmark)
- **GiniClust** (Baseline Gini-index feature selection method; fails at 100k cells in the scalability test)
- **hdf5r** (Loads 10X h5 count matrices into the Seurat object)
- **R** (Runtime environment; requires version >= 3.5.0; fresh session per method for timing/memory profiling)

## Examples

```
if (!require(DUBStepR)) install.packages("DUBStepR"); genes <- DUBStepR(input.data = norm.data, min.cells = 0.05*ncol(norm.data), optimise.features = TRUE, k = 10, num.pcs = 20, error = 0)
```

## Evaluation signals

- runtime_memory_benchmark.csv shows DUBStepR's memory increase from 100k to 1M cells is markedly smaller than the five correlation-free baselines (reported as 6.4x), reproducing Supp. Fig. S4
- GiniClust and HLG fail at 100k cells in the scalability run
- On the RA PBMC T/NK subset (5329 cells), DUBStepR with DI optimization resolves 10 discrete subtypes with rare clusters at RBC ~1.8%, proliferating ~2%, platelet-T ~3.4%, platelet-NK ~3%
- Clustering accuracy (e.g. silhouette index) with DUBStepR features exceeds baselines and no-feature-selection controls across benchmark datasets
- DUBStepR approaches its maximum silhouette index value at 200–500 feature genes

## Limitations

- Elbow point computation (findElbow) is not described in the main text — details are deferred to Supp. Note 1B
- No quantitative thresholds or effect sizes for scalability claims are reported in the main text (results in Supp. Fig. S4)
- SingleR misclassification can occur (e.g. platelet/RBC-like cells called T/NK) due to absent platelet and RBC transcriptomes in the reference panel, affecting subset annotation before clustering
- Existing feature selection methods perform inconsistently across datasets and can yield worse clustering accuracy than no feature selection

## Evidence

- [methods] DUBStepR(input.data, min.cells, species, optimise.features, k, num.pcs, error): "DUBStepR(input.data, min.cells, species, optimise.features, k, num.pcs, error)"
- [intro] After filtering out mitochondrial, ribosomal, spike-in, and pseudogenes, DUBStepR constructs a GGC matrix: "After filtering out mitochondrial, ribosomal, spike-in, and pseudogenes, DUBStepR constructs a GGC matrix"
- [abstract] DUBStepR is scalable to over a million cells: "DUBStepR is scalable to over a million cells, and can be straightforwardly applied to other data types such as single-cell ATAC-seq."
- [abstract] Despite selecting a relatively small number of genes, DUBStepR substantially outperformed existing single-cell feature selection methods: "Despite selecting a relatively small number of genes, DUBStepR substantially outperformed existing single-cell feature selection methods across diverse clustering benchmarks."
- [methods] DUBStepR requires your R version to be >= 3.5.0: "DUBStepR requires your R version to be >= 3.5.0"
- [readme] After filtering genes based on a correlation range score, DUBStepR exploits structure in the gene-gene correlation matrix: "After filtering genes based on a correlation range score, DUBStepR exploits structure in the gene-gene correlation matrix to prioritize genes as features for clustering."
- [discussion] Intriguingly, DUBStepR approaches its maximum silhouette index value at 200–500 feature genes (Supp. Fig. S3): "Intriguingly, DUBStepR approaches its maximum silhouette index value at 200–500 feature genes (Supp. Fig. S3)"
