---
name: seurat-clustering-umap-dimensionality-reduction
description: Use when a gene + TE cell-by-locus/family count matrix (e.g., from SoloTE
  or scTE) must be clustered and embedded in 2D to check whether TE expression recovers
  known cell populations, or to find cluster marker TEs.
when_to_use_negative:
- Input is already a feature table, metadata table, or pre-clustered object with no
  raw cell-by-feature counts to normalize and scale
- The goal is quantifying TE expression itself from BAM files rather than downstream
  clustering/visualization — run SoloTE_pipeline.py first to obtain the Seurat-compatible
  matrix
- Data are bulk RNA-Seq (e.g., prior bulk TE workflows such as SQuIRE/Telescope),
  not single-cell expression matrices
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: R package used to build Seurat objects from the gene+TE matrices, run standard
    preprocessing (normalize, variable features, scaling), PCA, UMAP dimensional reduction,
    graph-based clustering, and FindAllMarkers marker analysis
- name: SoloTE
  role: Produces the combined gene + TE expression matrix per cell that is fed into
    Seurat for clustering and UMAP; locus-level TE summarization (MAPQ ≥ 255) enables
    cluster-resolved TE markers
  repo: https://github.com/bvaldebenitom/SoloTE
- name: scTE
  role: Family-level comparator whose TE matrices are clustered/UMAP-reduced with
    the same Seurat workflow to benchmark cluster recovery (young-TE clusters are
    closer and even mixed)
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:b3f873ce47362b836bfb62cab2e387b33607b28e7d7782e660b342198e3e6185
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bvaldebenitom/SoloTE
  tools:
  - BEDTools
  - Pandas
  - Pysam
  - R
  - Samtools
  - Seurat
  - SoloTE
  - scTE
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# seurat-clustering-umap-dimensionality-reduction

## Summary

Standard Seurat preprocessing (normalize, variable-feature selection, scaling), PCA, UMAP dimensional reduction, and graph-based clustering applied to combined gene + TE expression matrices to test whether transposable-element signals recover expected cell clusters, benchmarking SoloTE against scTE.

## When to use

Use when a gene + TE cell-by-locus/family count matrix (e.g., from SoloTE or scTE) must be clustered and embedded in 2D to check whether TE expression recovers known cell populations, or to find cluster marker TEs.

## When NOT to use

- Input is already a feature table, metadata table, or pre-clustered object with no raw cell-by-feature counts to normalize and scale
- The goal is quantifying TE expression itself from BAM files rather than downstream clustering/visualization — run SoloTE_pipeline.py first to obtain the Seurat-compatible matrix
- Data are bulk RNA-Seq (e.g., prior bulk TE workflows such as SQuIRE/Telescope), not single-cell expression matrices

## Inputs

- Combined gene + TE cell expression matrix produced by SoloTE_pipeline.py (Seurat-compatible)
- Ground-truth and scTE gene/TE expression matrices from simulated hg38 and mm10 datasets
- Per-cell cluster-free count matrix derived from STAR/CellRanger BAM alignments (MAPQ ≥ 255 for locus-level TE assignment)

## Outputs

- Seurat object with normalized, scaled gene/TE features, PCA, and UMAP embedding
- Per-cell cluster assignments from Seurat graph-based clustering, stratified by genome and TE age (young/old)
- Cluster marker TE lists from FindAllMarkers (adjusted P-value ≤0.05, average log2FC ≥1), e.g., 24 marker TEs in the early gastric cancer dataset
- UMAP plots showing recovered clusters (e.g., 4 expected clusters in simulated data; 6 cell clusters in early gastric cancer)

## How to apply

Load the gene/TE cell expression matrix (e.g., the combined gene + TE matrix output by SoloTE_pipeline.py, or the scTE/ground-truth matrices) into R v4 or higher and construct a Seurat object. Apply Seurat's standard preprocessing: normalize counts, identify variable features, and scale the gene/TE features, then run PCA followed by UMAP dimensional reduction and graph-based clustering to record per-cell cluster assignments. Stratify results by genome (hg38, mm10) and TE age class (young: divergence from consensus ≤10%; old: divergence >10%) and count recovered clusters per condition against the four expected clusters. For marker discovery, run Seurat's FindAllMarkers keeping only results with adjusted P-value ≤0.05 and average log2(Fold Change) ≥1. The rationale is that Seurat consumes SoloTE's locus-level output directly, letting you judge whether locus-resolved TE expression (vs. scTE's family-level metagenes) separates known cell types such as the Cancer cluster in the early gastric cancer dataset.

## Related tools

- **Seurat** (R package used to build Seurat objects from the gene+TE matrices, run standard preprocessing (normalize, variable features, scaling), PCA, UMAP dimensional reduction, graph-based clustering, and FindAllMarkers marker analysis)
- **SoloTE** (Produces the combined gene + TE expression matrix per cell that is fed into Seurat for clustering and UMAP; locus-level TE summarization (MAPQ ≥ 255) enables cluster-resolved TE markers) — https://github.com/bvaldebenitom/SoloTE
- **scTE** (Family-level comparator whose TE matrices are clustered/UMAP-reduced with the same Seurat workflow to benchmark cluster recovery (young-TE clusters are closer and even mixed))

## Examples

```
library(Seurat); so <- CreateSeuratObject(counts); so <- NormalizeData(so); so <- FindVariableFeatures(so); so <- ScaleData(so); so <- RunPCA(so); so <- RunUMAP(so, dims = 1:10); so <- FindClusters(so)
```

## Evaluation signals

- Exactly the four expected clusters are recovered for old and young TE datasets in both hg38 and mm10 from the ground-truth, SoloTE, and scTE matrices
- SoloTE young-TE clustering yields the 4 expected clusters, whereas scTE young-TE clusters are closer and even mixed — the reported benchmark behavior
- The early gastric cancer dataset reproduces 6 cell clusters after Seurat clustering and UMAP
- FindAllMarkers with adjusted P-value ≤0.05 and average log2FC ≥1 yields 24 marker TEs, with cancer-cluster markers mapping to L1PA7 (chrY:18989629-18990627) and THE1D (chr8:61579930-61580458)
- UMAP embedding is computed from PCA on scaled gene/TE features, per Seurat's standard preprocessing workflow

## Limitations

- scTE young-TE clusters are closer and even mixed in the simulated datasets, so cluster-count validation against the 4 expected clusters can be ambiguous for family-level TE matrices
- scTE summarizes TE expression on metagenes corresponding to TE families, losing the genomic location from which each TE is expressed, which limits locus-specific cluster marker discovery
- With short 100 nucleotide reads (3' end 10X), not all TEs can be uniquely mappable, affecting the feature space available for clustering

## Evidence

- [discussion] a new matrix having both genes and TEs expression per cell is obtained as output, which is compatible with downstream analysis tools, such as Seurat67: "a new matrix having both genes and TEs expression per cell is obtained as output, which is compatible with downstream analysis tools, such as Seurat67"
- [intro] First, we compare it to scTE, in terms of (i) detecting TE expression, (ii) UMAP dimensional reduction, and (iii) detection of cell marker TEs.: "First, we compare it to scTE, in terms of (i) detecting TE expression, (ii) UMAP dimensional reduction, and (iii) detection of cell marker TEs."
- [discussion] marker analysis was carried out using the "FindAllMarkers" function of Seurat, only keeping the results having adjusted P-value ≤0.05 and average log2(Fold Change) ≥1: [evidence span withheld — anchor status: not_found]
- [readme] R v4 or higher (https://www.r-project.org/): "R v4 or higher (https://www.r-project.org/)"
