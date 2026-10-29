---
name: cluster-separation-quality-assessment
description: Use when you have 2D UMAP coordinates plus cell-type labels (e.g., from
  Seurat KNN graph clustering on 10 PCs) and need a numeric, per-sample measure of
  cluster separation to compare across feature spaces such as gene, aTAR, and uTAR
  expression.
when_to_use_negative:
- Input is a raw UMI count matrix without cell-type labels or a low-dimensional embedding
  — silhouette requires labels and coordinates first
- You need to identify the TARs themselves (use groHMM-based TAR identification from
  the TAR-scRNA-seq workflow instead); this skill only scores separation of pre-defined
  clusters
- Cell-type labels are derived from the same feature space being scored, making the
  comparison circular rather than an independent validation
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: Defines the reference cell-type labels via KNN graph clustering on the first
    10 principal components of gene-expression data
- name: UMAP
  role: Provides the 2D embedding coordinates on which silhouette coefficients are
    computed with Euclidean distance
- name: groHMM
  role: 'Upstream: identifies transcriptionally active regions (TARs, including uTARs)
    whose expression matrices supply the alternative feature spaces being scored'
  repo: https://bioconductor.org/packages/release/bioc/html/groHMM.html
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:503aa00eb7a665d1bba684dd9d1df1c5bbed6d33f1b3fdc96c47374920017d45
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# cluster-separation-quality-assessment

## Summary

Quantifies how well cell types separate in an embedding or feature space by computing silhouette coefficients on 2D UMAP coordinates, so that alternative feature spaces (e.g., uTAR/aTAR reads outside gene annotations) can be compared against gene-expression-based clustering as in Fig. 2b of the TAR-scRNA-seq paper.

## When to use

Use when you have 2D UMAP coordinates plus cell-type labels (e.g., from Seurat KNN graph clustering on 10 PCs) and need a numeric, per-sample measure of cluster separation to compare across feature spaces such as gene, aTAR, and uTAR expression.

## When NOT to use

- Input is a raw UMI count matrix without cell-type labels or a low-dimensional embedding — silhouette requires labels and coordinates first
- You need to identify the TARs themselves (use groHMM-based TAR identification from the TAR-scRNA-seq workflow instead); this skill only scores separation of pre-defined clusters
- Cell-type labels are derived from the same feature space being scored, making the comparison circular rather than an independent validation

## Inputs

- 2D UMAP coordinate matrices per sample and per feature space (gene, aTAR, uTAR)
- Cell-type labels from Seurat KNN graph clustering on the first 10 principal components (gene-annotation-based)
- Sample metadata identifying the 11 datasets (human, mouse, chicken, mole rat, lemur, sea urchin)

## Outputs

- Table with one row per sample and silhouette coefficient columns for gene expression, aTARs, and uTARs (Fig. 2b layout)
- Per-sample comparison of uTAR vs. gene-based silhouette values used to verify close agreement

## How to apply

Load the 2D UMAP coordinates for each feature space (gene, aTAR, uTAR) for all samples (11 datasets in the paper), together with cell-type labels defined by gene-annotation-based Seurat KNN graph clustering on the first 10 PCs. For each sample-feature combination, call scikit-learn silhouette_score with Euclidean distance on the 2D embedding; the silhouette ranges from -1 to 1, with higher values indicating better-defined clusters. Assemble a per-sample table with columns for gene (blue), aTAR (red), and uTAR (maroon) silhouette values, mirroring Fig. 2b. Rationale: because UMAP geometry reflects the underlying cluster structure, silhouette values on 2D UMAP coordinates act as a comparable separation statistic across heterogeneous feature spaces. Judge correctness by checking that uTAR silhouette values closely track gene-based values for samples where the paper reports agreement (mouse spleen, lemur lung, naked mole rat spleen, sea urchin embryo).

## Related tools

- **Seurat** (Defines the reference cell-type labels via KNN graph clustering on the first 10 principal components of gene-expression data)
- **UMAP** (Provides the 2D embedding coordinates on which silhouette coefficients are computed with Euclidean distance)
- **groHMM** (Upstream: identifies transcriptionally active regions (TARs, including uTARs) whose expression matrices supply the alternative feature spaces being scored) — https://bioconductor.org/packages/release/bioc/html/groHMM.html

## Examples

```
from sklearn.metrics import silhouette_score; s = silhouette_score(umap_2d_coords, cell_type_labels, metric='euclidean')
```

## Evaluation signals

- The output table contains silhouette values for all 11 samples across all three feature types (gene, aTAR, uTAR)
- For samples where the paper reports agreement, uTAR silhouette values qualitatively track the gene-based silhouette values, reproducing Fig. 2b
- All silhouette scores fall within the valid [-1, 1] range
- aTAR-based silhouette values show close agreement with gene-expression-based values across samples
- Euclidean distance on the 2D UMAP coordinates is used as the silhouette metric (scikit-learn silhouette_score)

## Limitations

- uTAR silhouette agreement with gene-based clustering is dataset-dependent: the degree of relevant information in uTARs is strongly dependent on organism, tissue, cell type, and stage of development
- Silhouette is computed on 2D UMAP coordinates, so it reflects embedding geometry rather than high-dimensional separability
- The upstream groHMM step can fail (e.g., missing chr*_HMM.bed files from incomplete installation of groHMM/rtracklayer), preventing uTAR/aTAR feature matrices from being generated at all

## Evidence

- [results] Silhouette values computed from 2D UMAP coordinates show close agreement between aTAR- and gene-expression-based clustering: [evidence span withheld — anchor status: not_found]
- [intro] uTAR information content is context-dependent: "we find that the degree of relevant information in uTARs is strongly dependent on organism, tissue, cell type, and stage of development"
- [intro] groHMM defines TARs that produce the feature spaces being compared: "groHMM uses a hidden Markov model to divide the genome into two states (transcribed and untranscribed) based on read coverage in each bin"
- [readme] Upstream TAR generation can fail from missing packages: "This error indicates that the groHMM R script did not finish running to generate groHMM bed files."
