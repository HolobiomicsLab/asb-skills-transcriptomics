---
name: seurat-clustering-and-marker-analysis
description: Use when a combined gene+TE scRNA-Seq expression matrix needs clustering
  and marker discovery to define cell populations via known markers (e.g., Zscan4c/Tcstv3
  for 2C-like cells).
when_to_use_negative:
- Input is a TE-family-level metagene matrix (e.g., from scTE) when locus-specific
  genomic location of TE expression is required
- Raw FASTQ/BAM files that have not been processed into a gene+TE count matrix — run
  STAR and SoloTE_pipeline.py first
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0654
tools:
- name: Seurat
  role: Normalization, clustering, UMAP dimensional reduction, and marker analysis
    via FindAllMarkers on the combined gene+TE matrix
- name: SoloTE
  role: Produces the combined gene and TE locus-specific expression matrix that is
    loaded into Seurat
  repo: https://github.com/bvaldebenitom/SoloTE
- name: scTE
  role: Comparison tool summarizing TE expression at family level; benchmarked against
    SoloTE for marker TE detection
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:051bd7e52e6b6ba4f5188f730bc62a2f9c817e9559dec1bd6a66357298025580
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bvaldebenitom/SoloTE
  tools:
  - BEDTools
  - Seurat
  - SoloTE
  - scTE
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# seurat-clustering-and-marker-analysis

## Summary

Normalize and cluster single-cell expression data (genes plus locus-specific TEs) in Seurat, then identify cell-population markers with FindAllMarkers under adjusted P-value ≤0.05 and average log2(Fold Change) ≥1, as done to define 2C-like cells via Zscan4c/Tcstv3 in the murine two-cell stage dataset.

## When to use

Use when a combined gene+TE scRNA-Seq expression matrix needs clustering and marker discovery to define cell populations via known markers (e.g., Zscan4c/Tcstv3 for 2C-like cells).

## When NOT to use

- Input is a TE-family-level metagene matrix (e.g., from scTE) when locus-specific genomic location of TE expression is required
- Raw FASTQ/BAM files that have not been processed into a gene+TE count matrix — run STAR and SoloTE_pipeline.py first

## Inputs

- Combined gene and TE expression matrix per cell (SoloTE output, compatible with Seurat)
- Known marker gene names used to define the target population (e.g., Zscan4c, Tcstv3)

## Outputs

- Clustered Seurat object with UMAP dimensional reduction
- Marker table (CSV, e.g., solote_2c_marker_TEs.csv) of significantly enriched genes/TEs per cluster (adjusted P ≤0.05, avg log2FC ≥1)

## How to apply

Load the combined gene and TE expression matrix (output of SoloTE_pipeline.py, which appends locus-level TE columns to the gene matrix) into Seurat, normalize, and cluster the cells. Define the population of interest by expression of known markers — in the murine two-cell stage dataset, cells expressing Zscan4c and Tcstv3 define the 2C-like population. Run Seurat's FindAllMarkers function, keeping only results with adjusted P-value ≤0.05 and average log2(Fold Change) ≥1. Rationale: with these filters SoloTE recovered 106 marker TEs in 2C-like cells (vs 28 for scTE), including MERVL-int and MT2-Mm showing the expected 2C-marker expression pattern; visualize clusters with UMAP dimensional reduction to confirm population separation.

## Related tools

- **Seurat** (Normalization, clustering, UMAP dimensional reduction, and marker analysis via FindAllMarkers on the combined gene+TE matrix)
- **SoloTE** (Produces the combined gene and TE locus-specific expression matrix that is loaded into Seurat) — https://github.com/bvaldebenitom/SoloTE
- **scTE** (Comparison tool summarizing TE expression at family level; benchmarked against SoloTE for marker TE detection)

## Examples

```
R: markers <- FindAllMarkers(seurat_obj, only.pos = TRUE); subset(markers, p_val_adj <= 0.05 & avg_log2FC >= 1)
```

## Evaluation signals

- All reported markers have adjusted P-value ≤0.05 and average log2(Fold Change) ≥1
- Known biology recapitulated: MERVL-int and MT2-Mm show the expected 2C-marker expression pattern in the Zscan4c/Tcstv3-positive cluster
- Benchmark expectation: SoloTE locus-level matrix yields ~106 marker TEs in 2C-like cells vs ~28 with scTE family-level summarization
- UMAP dimensional reduction of the combined gene+TE matrix resolves the 2C-like population as a distinct cluster

## Limitations

- scTE indexes do not filter non-TE repeats such as Satellites, which can contaminate family-level marker results
- 3' end 10X data: not all TEs are uniquely mappable with 100 nt reads, and TEs acting as alternative transcription start sites cannot be detected due to 3' bias
- Gain/loss-of-function experimental validation is still needed to confirm TE influence on gene expression

## Evidence

- [discussion] FindAllMarkers filtering thresholds: [evidence span withheld — anchor status: not_found]
- [discussion] SoloTE output compatible with Seurat: "a new matrix having both genes and TEs expression per cell is obtained as output, which is compatible with downstream analysis tools, such as Seurat"
- [readme] SoloTE README: "SoloTE requires the following tools to be installed, and available in your PATH environment variable"
- [intro] scTE family-level limitation: "which summarizes TE expression on metagenes corresponding to TE families, thus, losing the genomic location from which each TE is expressed"
- [intro] UMAP objective: "First, we compare it to scTE, in terms of (i) detecting TE expression, (ii) UMAP dimensional reduction, and (iii) detection of cell marker TEs."
