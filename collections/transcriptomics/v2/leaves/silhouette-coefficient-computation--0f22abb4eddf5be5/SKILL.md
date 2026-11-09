---
name: silhouette-coefficient-computation
description: Use when you have 2D UMAP coordinates plus cell-type labels from gene-annotation-based
  KNN/Seurat clustering and want to test whether an alternative feature space (e.g.,
  uTAR reads outside gene annotations) clusters cell types comparably to annotated
  gene expression.
when_to_use_negative:
- Input is raw read counts or a gene-by-cell expression matrix rather than an existing
  2D embedding with cluster labels — silhouette is computed on the UMAP coordinates,
  not raw profiles
- You want to detect differentially expressed uTARs or annotate them by sequence homology
  — use the TAR-scRNA-seq differential-expression and homology steps instead
- No cell-type labels exist for the dataset — silhouette requires pre-defined clusters
  to score
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Defines the reference cell-type labels via KNN graph clustering on the first
    10 principal components of gene expression, which the silhouette scores are evaluated
    against
- name: UMAP
  role: Provides the 2D embedding coordinates on which silhouette coefficients (Euclidean
    distance) are computed for each feature space
- name: groHMM
  role: 'Upstream: identifies transcriptionally active regions (TARs) whose aTAR/uTAR
    expression profiles supply the alternative feature spaces being scored'
  repo: https://bioconductor.org/packages/release/bioc/html/groHMM.html
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:d13ae4ccfbfbfb4d6d58d3e8d3bd6fbaeae25ed93422cd129ec43758ebe4eb6a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Seurat
  - UMAP
  - groHMM
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# silhouette-coefficient-computation

## Summary

Compute silhouette coefficients on 2D UMAP embeddings for each scRNA-seq feature space (annotated gene expression, aTARs, uTARs) to quantify how well cell-type clusters are separated, following Fig. 2b of the TAR-scRNA-seq paper.

## When to use

Use when you have 2D UMAP coordinates plus cell-type labels from gene-annotation-based KNN/Seurat clustering and want to test whether an alternative feature space (e.g., uTAR reads outside gene annotations) clusters cell types comparably to annotated gene expression.

## When NOT to use

- Input is raw read counts or a gene-by-cell expression matrix rather than an existing 2D embedding with cluster labels — silhouette is computed on the UMAP coordinates, not raw profiles
- You want to detect differentially expressed uTARs or annotate them by sequence homology — use the TAR-scRNA-seq differential-expression and homology steps instead
- No cell-type labels exist for the dataset — silhouette requires pre-defined clusters to score

## Inputs

- 2D UMAP coordinates per cell for gene-expression, aTAR, and uTAR feature spaces
- Cell-type labels from gene-annotation-based KNN graph clustering on the first 10 PCs (Seurat)
- Sample metadata covering all 11 samples (human, mouse, chicken, mole rat, lemur, sea urchin datasets)

## Outputs

- Table with one row per sample and silhouette values for gene, aTAR, and uTAR feature spaces (Fig. 2b layout)
- Per-sample comparison of uTAR vs gene-based silhouette values

## How to apply

Load the 2D UMAP coordinates for the gene, aTAR, and uTAR feature spaces for each of the 11 samples, together with cell-type labels derived from KNN graph clustering on the first 10 principal components (Seurat). For each sample and each feature type, run scikit-learn's silhouette_score with Euclidean distance on the 2D embedding coordinates, treating each cell-type label as a cluster. Assemble a table with one row per sample and three columns for gene (blue), aTAR (red), and uTAR (maroon) silhouette values, matching the Fig. 2b layout. The rationale is that silhouette values capture cluster separation independently of the feature space used, so uTAR-based scores that closely track gene-based scores indicate that reads outside gene annotations carry cell-type-specific structure.

## Related tools

- **Seurat** (Defines the reference cell-type labels via KNN graph clustering on the first 10 principal components of gene expression, which the silhouette scores are evaluated against)
- **UMAP** (Provides the 2D embedding coordinates on which silhouette coefficients (Euclidean distance) are computed for each feature space)
- **groHMM** (Upstream: identifies transcriptionally active regions (TARs) whose aTAR/uTAR expression profiles supply the alternative feature spaces being scored) — https://bioconductor.org/packages/release/bioc/html/groHMM.html

## Examples

```
from sklearn.metrics import silhouette_score; s_ut = silhouette_score(umap_2d_coords, cell_type_labels, metric='euclidean')
```

## Evaluation signals

- The output table contains silhouette values for all 11 samples across all three feature types (gene, aTAR, uTAR)
- For samples where the paper reports agreement (mouse spleen, lemur lung, naked mole rat spleen, sea urchin embryo), uTAR silhouette values closely track the gene-based silhouette values, qualitatively reproducing Fig. 2b
- Silhouette scores are computed on 2D UMAP coordinates with Euclidean distance, matching the paper's metric choice
- aTAR-based silhouette values show close agreement with gene-expression-based values across samples
- Each silhouette value falls in the range [-1, 1] as a sanity check on the scikit-learn silhouette_score output

## Limitations

- The degree of cell-type information in uTARs is strongly dependent on organism, tissue, cell type, and stage of development, so uTAR silhouette agreement is not uniform across all 11 samples
- Silhouette scores are computed on the 2D UMAP embedding, so they assess cluster separation in the projected space rather than the high-dimensional expression space
- groHMM/Rscript dependencies can fail (e.g., 'no lines available in input' in rule calcHMMrefFlat when groHMM bed files are missing), blocking generation of TAR feature spaces upstream

## Evidence

- [results] Silhouette values computed from 2D UMAP coordinates show close agreement between aTAR- and gene-expression-based clustering: [evidence span withheld — anchor status: not_found]
- [methods] compute the silhouette coefficient on the 2D UMAP coordinates with the scikit-learn silhouette_score metric (Euclidean distance on the 2D embedding): [evidence span withheld — anchor status: not_found]
- [intro] uTAR information is context-dependent: "we find that the degree of relevant information in uTARs is strongly dependent on organism, tissue, cell type, and stage of development"
- [readme] groHMM failure mode README: "This error indicates that the groHMM R script did not finish running to generate groHMM bed files."
