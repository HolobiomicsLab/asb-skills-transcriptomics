---
name: single-cell-clustering-and-dam-annotation
description: Use when a combined gene + TE single-cell expression matrix needs clustering,
  UMAP visualization, and cell-cluster annotation, e.g. identifying DAM clusters in
  the APP/PS1 AD mouse model via Cst7, Lpl, and Clec7a.
when_to_use_negative:
- Input is a bulk RNA-Seq pseudobulk count table rather than a per-cell expression
  matrix — use edgeR differential expression instead of clustering/annotation.
- TE expression is only available at family-level metagene resolution (e.g. scTE output)
  with no genomic locus information, so locus-specific marker TEs cannot be resolved.
- The organism or cell type lacks validated marker genes, making the annotation step
  (Cst7, Lpl, Clec7a analogs) unsupported.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: Seurat
  role: R toolkit used to import the combined gene + TE matrix, normalize, cluster
    cells, run UMAP, annotate DAM clusters with Cst7/Lpl/Clec7a, and run FindAllMarkers
- name: SoloTE
  role: Produces the combined gene + TE expression matrix that is the clustering input;
    locus-level TE counts enable cluster marker TE discovery
  repo: https://github.com/bvaldebenitom/SoloTE
- name: scTE
  role: 'Contrast method: family-level metagene TE summarization that detected 0 marker
    TEs where locus-level analysis detected 71'
- name: edgeR
  role: Used downstream for pseudobulk differential expression between disease and
    control conditions after clustering/annotation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-022-04020-5
    title: SoloTE for improved analysis of transposable elements in single-cell RNA-Seq
      data using locus-specific expression
  source_content_hash: sha256:9c66ecc8baf4568a62554945a8a1f868f87ec6470e8e288f50352f9f6d0fc819
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bvaldebenitom/SoloTE
  tools:
  - BEDTools
  - Samtools
  - Seurat
  - SoloTE
  - edgeR
  - scTE
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# single-cell-clustering-and-dam-annotation

## Summary

Clusters single-cell expression data (including locus-level TE counts from SoloTE) in Seurat, performs UMAP dimensional reduction, and annotates disease-associated microglia (DAM) clusters using the marker genes Cst7, Lpl, and Clec7a before running FindAllMarkers to detect cell-type marker TEs.

## When to use

Use when a combined gene + TE single-cell expression matrix needs clustering, UMAP visualization, and cell-cluster annotation, e.g. identifying DAM clusters in the APP/PS1 AD mouse model via Cst7, Lpl, and Clec7a.

## When NOT to use

- Input is a bulk RNA-Seq pseudobulk count table rather than a per-cell expression matrix — use edgeR differential expression instead of clustering/annotation.
- TE expression is only available at family-level metagene resolution (e.g. scTE output) with no genomic locus information, so locus-specific marker TEs cannot be resolved.
- The organism or cell type lacks validated marker genes, making the annotation step (Cst7, Lpl, Clec7a analogs) unsupported.

## Inputs

- Combined gene + TE expression matrix produced by SoloTE (compatible with Seurat)
- Cell-level metadata (e.g. disease vs wild-type condition) for the APP/PS1 AD mouse model scRNA-Seq data
- DAM marker gene list (Cst7, Lpl, Clec7a)

## Outputs

- Seurat object with normalized data, clusters, and UMAP dimensional reduction
- Annotated disease-associated microglia (DAM) cluster labels
- FindAllMarkers table of cluster marker TEs filtered to adjusted P-value ≤ 0.05 and average log2(FC) ≥ 1 (71 TEs in the APP/PS1 study)

## How to apply

Import the combined gene + TE expression matrix output by SoloTE_pipeline.py into Seurat and normalize, then cluster cells and perform UMAP dimensional reduction. Annotate disease-associated microglia clusters using the marker genes Cst7, Lpl, and Clec7a rather than generic cluster labels. Run Seurat FindAllMarkers on the annotated clusters, keeping only results with adjusted P-value ≤ 0.05 and average log2(Fold Change) ≥ 1, which in the APP/PS1 analysis yielded 71 marker TEs at the locus level while scTE family-level metagenes detected 0. The rationale is that locus-specific TE expression preserves genomic coordinates, so cluster-marker testing can identify individual transcribed TE loci that family-level summaries collapse away.

## Related tools

- **Seurat** (R toolkit used to import the combined gene + TE matrix, normalize, cluster cells, run UMAP, annotate DAM clusters with Cst7/Lpl/Clec7a, and run FindAllMarkers)
- **SoloTE** (Produces the combined gene + TE expression matrix that is the clustering input; locus-level TE counts enable cluster marker TE discovery) — https://github.com/bvaldebenitom/SoloTE
- **scTE** (Contrast method: family-level metagene TE summarization that detected 0 marker TEs where locus-level analysis detected 71)
- **edgeR** (Used downstream for pseudobulk differential expression between disease and control conditions after clustering/annotation)

## Examples

```
R: seurat_obj <- CreateSeuratObject(Read10X(combined_gene_TE_matrix)); seurat_obj <- NormalizeData(seurat_obj); seurat_obj <- RunUMAP(seurat_obj, dims = 1:30); FindAllMarkers(seurat_obj, only.pos = TRUE, logfc.threshold = 1)
```

## Evaluation signals

- UMAP dimensional reduction separates clusters and the DAM clusters are identifiable by expression of Cst7, Lpl, and Clec7a.
- FindAllMarkers output restricted to adjusted P-value ≤ 0.05 and average log2(Fold Change) ≥ 1 yields 71 marker TEs in the APP/PS1 dataset.
- Locus-level marker TE detection (71 TEs) contrasts with scTE family-level metagenes detecting 0, confirming genomic-location resolution matters.
- Downstream edgeR pseudobulk disease-vs-control testing on annotated clusters identifies 9 disease-upregulated TEs including Arthur1B, RLTR14-int, and MMVL30-int.

## Limitations

- Marker TE detection depends on MAPQ ≥ 255 uniquely mapped reads for locus-level assignment; multi-mapping reads are only summarized at family level.
- Not all TEs can be uniquely mappable with short 100-nucleotide reads, limiting locus-level detection in 3' end 10X data.
- scTE-style indexes do not filter non-TE repeats such as Satellites, which can contaminate TE marker tables if the wrong annotation is used.
- TE-derived isoform analysis and EM-based multi-mapper assignment are not yet integrated into the pipeline.

## Evidence

- [intro] UMAP dimensional reduction and detection of cell marker TEs: "First, we compare it to scTE, in terms of (i) detecting TE expression, (ii) UMAP dimensional reduction, and (iii) detection of cell marker TEs."
- [discussion] a new matrix having both genes and TEs expression per cell is obtained as output, which is compatible with downstream analysis tools, such as Seurat: "a new matrix having both genes and TEs expression per cell is obtained as output, which is compatible with downstream analysis tools, such as Seurat67"
- [discussion] FindAllMarkers filtering thresholds: [evidence span withheld — anchor status: not_found]
- [other] 71 marker TEs and DAM annotation from the task card workflow: [evidence span withheld — anchor status: not_found]
- [other] locus-level hits Arthur1B, RLTR14-int, and MMVL30-int from the task card workflow: [evidence span withheld — anchor status: not_found]
- [discussion] scTE does not filter repetitive elements in such indexes, and thus, in the resulting matrix other types of non-TE repeats, such as Satellites, appear: "scTE does not filter repetitive elements in such indexes, and thus, in the resulting matrix other types of non-TE repeats, such as Satellites, appear"
