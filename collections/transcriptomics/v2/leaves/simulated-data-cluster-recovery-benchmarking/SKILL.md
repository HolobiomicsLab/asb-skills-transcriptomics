---
name: simulated-data-cluster-recovery-benchmarking
description: Use when comparing TE-expression quantification tools against known ground-truth
  labels using simulated single-cell data, specifically to test whether expected cell
  clusters are recovered from gene/TE expression matrices.
when_to_use_negative:
- Real (non-simulated) datasets with no ground-truth cell labels — cluster-recovery
  counts cannot be validated against expected clusters
- Benchmarking computational runtime or memory usage between SoloTE and scTE — that
  requires a different resource-profiling protocol
- Detecting cell-marker TEs or gene-TE regulatory relationships — use the article's
  FindAllMarkers (adj. P ≤0.05, avg log2FC ≥1) and linear-model workflow instead
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Constructs Seurat objects from each expression matrix and performs normalization,
    PCA, UMAP dimensional reduction, and graph-based clustering for cluster recovery
- name: SoloTE
  role: Produces the locus-specific gene+TE expression matrices being benchmarked
    against ground truth; output matrix is Seurat-compatible
  repo: https://github.com/bvaldebenitom/SoloTE
- name: scTE
  role: Comparator tool whose TE-family metagene matrices are benchmarked for cluster
    recovery against SoloTE and ground truth
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:6f2dd1c03e69649dbcd05112783b716d3828a995f39af3e504caca44afda4db1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bvaldebenitom/SoloTE
  tools:
  - Seurat
  - SoloTE
  - scTE
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# simulated-data-cluster-recovery-benchmarking

## Summary

Benchmark of TE quantification tools (SoloTE vs scTE) by processing ground-truth, SoloTE, and scTE expression matrices from simulated hg38 and mm10 scRNA-Seq data through Seurat clustering and UMAP, then counting recovered cell clusters stratified by TE age class.

## When to use

Use when comparing TE-expression quantification tools against known ground-truth labels using simulated single-cell data, specifically to test whether expected cell clusters are recovered from gene/TE expression matrices.

## When NOT to use

- Real (non-simulated) datasets with no ground-truth cell labels — cluster-recovery counts cannot be validated against expected clusters
- Benchmarking computational runtime or memory usage between SoloTE and scTE — that requires a different resource-profiling protocol
- Detecting cell-marker TEs or gene-TE regulatory relationships — use the article's FindAllMarkers (adj. P ≤0.05, avg log2FC ≥1) and linear-model workflow instead

## Inputs

- Ground-truth gene/TE cell expression matrix from the simulated hg38 and mm10 scRNA-Seq experiments
- SoloTE gene+TE expression matrix (locus-level TE output, Seurat-compatible)
- scTE gene+TE expression matrix (TE-family metagene output)
- TE annotation in BED format with divergence-from-consensus values for young/old classification

## Outputs

- Seurat objects with PCA and UMAP embeddings for each input matrix
- Per-cell graph-based cluster assignments per matrix
- Cluster-recovery counts stratified by TE age class (young/old) and genome (hg38, mm10)
- Comparison of SoloTE vs scTE cluster assignments against ground-truth labels

## How to apply

Load the ground-truth, SoloTE, and scTE gene/TE cell expression matrices from the simulated hg38 and mm10 experiments into R (v4 or higher) and construct Seurat objects. Run Seurat's standard preprocessing (normalize counts, identify variable features, scale features), then PCA, UMAP dimensional reduction, and graph-based clustering, recording per-cell cluster assignments for each matrix. Stratify results by TE age class — young defined as divergence from the consensus sequence ≤10%, old as >10% — and by genome (hg38, mm10). Count recovered clusters per condition and compare assignments against the four expected clusters and the ground-truth labels, which is possible because the simulated data encode known cluster structure. Validation hinges on the reported behavior: exactly four expected clusters recovered for old TEs in both genomes with either tool, SoloTE recovering the 4 expected young-TE clusters, while scTE young-TE clusters are closer together and even mixed.

## Related tools

- **Seurat** (Constructs Seurat objects from each expression matrix and performs normalization, PCA, UMAP dimensional reduction, and graph-based clustering for cluster recovery)
- **SoloTE** (Produces the locus-specific gene+TE expression matrices being benchmarked against ground truth; output matrix is Seurat-compatible) — https://github.com/bvaldebenitom/SoloTE
- **scTE** (Comparator tool whose TE-family metagene matrices are benchmarked for cluster recovery against SoloTE and ground truth)

## Examples

```
obj <- RunPCA(obj); obj <- RunUMAP(obj, dims = 1:30); obj <- FindClusters(obj)  # per Seurat: normalize, find variable features, scale, PCA, UMAP, graph-based clustering
```

## Evaluation signals

- Exactly the four expected clusters are recovered for old TE datasets (divergence >10%) in both hg38 and mm10, using either SoloTE or scTE matrices
- SoloTE young-TE matrices (divergence ≤10%) recover the 4 expected clusters
- scTE young-TE clusters appear closer together or mixed relative to the expected cluster separation
- Per-cell cluster assignments match the ground-truth labels from the simulated data for the expected conditions
- Each condition (tool × genome × TE age class) yields a recorded cluster count for direct comparison

## Limitations

- Benchmark restricted to simulated data from the human genome hg38 and mouse genome mm10 versions
- TE age stratification depends on divergence-from-consensus thresholds (≤10% young, >10% old), which may not generalize to all TE classes
- scTE young-TE results show mixed/closer clusters, so cluster-number counting alone may understate assignment-level disagreement; compare assignments against ground truth, not just cluster count
- scTE indexes do not filter non-TE repeats such as Satellites, which can contaminate its resulting matrix

## Evidence

- [intro] First, we compare it to scTE, in terms of (i) detecting TE expression, (ii) UMAP dimensional reduction, and (iii) detection of cell marker TEs.: "First, we compare it to scTE, in terms of (i) detecting TE expression, (ii) UMAP dimensional reduction, and (iii) detection of cell marker TEs."
- [discussion] we simulated data from the human genome hg38 version, and from the mouse genome mm10 version: "we simulated data from the human genome hg38 version, and from the mouse genome mm10 version"
- [discussion] TEs were divided into young and old, if their divergence from the consensus sequence was ≤10% or >10%, respectively: "TEs were divided into young and old, if their divergence from the consensus sequence was ≤10% or >10%, respectively"
- [discussion] a new matrix having both genes and TEs expression per cell is obtained as output, which is compatible with downstream analysis tools, such as Seurat: "a new matrix having both genes and TEs expression per cell is obtained as output, which is compatible with downstream analysis tools, such as Seurat67"
