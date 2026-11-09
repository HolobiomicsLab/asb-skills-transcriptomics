---
name: cross-feature-benchmark-analysis
description: Use when you have cell coordinates in a common low-dimensional embedding
  (e.g., 2D UMAP) from multiple feature spaces for the same cells, plus reference
  cell-type labels, and need to score how well each feature space reproduces the reference
  structure.
when_to_use_negative:
- Cells from different feature spaces do not correspond 1:1 (no shared cell set),
  so silhouette scores are not computed against a common reference structure
- Only one feature space is available — there is nothing to benchmark against, use
  a standard single-space clustering evaluation instead
- You have raw expression/count matrices rather than a common low-dimensional embedding;
  the paper's protocol scores silhouette on the 2D UMAP coordinates, not on high-dimensional
  feature matrices
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Provides the reference cell-type labels via gene-annotation-based KNN graph
    clustering on the first 10 PCs, against which all feature spaces are scored
- name: UMAP
  role: Supplies the 2D coordinates on which silhouette coefficients are computed
    for the gene, aTAR, and uTAR feature spaces
- name: scikit-learn
  role: supplies silhouette_score, the metric used to quantify cluster separation
    per feature space
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-22496-3
    title: Uncovering transcriptional dark matter via gene annotation independent
      single-cell RNA sequencing analysis
  source_content_hash: sha256:925fb98abb32e0736127f1beb69be79ea6bf70cde7c395e63896439df10376b8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Seurat
  - UMAP
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# cross-feature-benchmark-analysis

## Summary

Quantitative benchmark that compares clustering/structure quality across different feature spaces (annotated genes, aTARs, uTARs) by computing silhouette coefficients on a shared 2D UMAP embedding, as done in the TAR-scRNA-seq paper's Fig. 2b to show uTAR reads separate cell types comparably to gene expression.

## When to use

Use when you have cell coordinates in a common low-dimensional embedding (e.g., 2D UMAP) from multiple feature spaces for the same cells, plus reference cell-type labels, and need to score how well each feature space reproduces the reference structure.

## When NOT to use

- Cells from different feature spaces do not correspond 1:1 (no shared cell set), so silhouette scores are not computed against a common reference structure
- Only one feature space is available — there is nothing to benchmark against, use a standard single-space clustering evaluation instead
- You have raw expression/count matrices rather than a common low-dimensional embedding; the paper's protocol scores silhouette on the 2D UMAP coordinates, not on high-dimensional feature matrices

## Inputs

- 2D UMAP coordinate matrices for gene, aTAR, and uTAR feature spaces, per sample (11 samples in the paper)
- Cell-type labels from gene-annotation-based Seurat KNN graph clustering computed on the first 10 principal components
- Sample-to-feature-space mapping / metadata table linking each cell to its sample and label

## Outputs

- A silhouette-coefficient table with one row per sample and columns for gene-expression (blue), aTAR (red), and uTAR (maroon) values, reproducing the layout of Fig. 2b
- A per-sample agreement assessment of uTAR vs. gene-based silhouette values

## How to apply

Load the 2D UMAP coordinates for each feature type (gene, aTAR, uTAR) for every sample, together with cell-type labels defined by gene-annotation-based KNN graph clustering on the first 10 PCs in Seurat. For each sample-feature combination, compute the silhouette coefficient with scikit-learn silhouette_score using Euclidean distance directly on the 2D embedding coordinates. Assemble one row per sample with silhouette columns for gene expression (blue), aTARs (red), and uTARs (maroon), mirroring Fig. 2b. Judge agreement qualitatively per sample: uTAR silhouette values should closely track gene-based values (e.g., mouse spleen, lemur lung, naked mole rat spleen, sea urchin embryo), indicating uTAR reads accurately separate cell types. The rationale is that silhouette on a shared embedding isolates feature-space information content from clustering/embedding parameter differences, since all feature types are scored against the same labels on comparable 2D coordinates.

## Related tools

- **Seurat** (Provides the reference cell-type labels via gene-annotation-based KNN graph clustering on the first 10 PCs, against which all feature spaces are scored)
- **UMAP** (Supplies the 2D coordinates on which silhouette coefficients are computed for the gene, aTAR, and uTAR feature spaces)
- **scikit-learn** (supplies silhouette_score, the metric used to quantify cluster separation per feature space)

## Examples

```
from sklearn.metrics import silhouette_score; s_ut = silhouette_score(X_umap2d_ut, labels_seurat, metric='euclidean')
```

## Evaluation signals

- The output table contains silhouette values for all 11 samples across the three feature types (gene, aTAR, uTAR)
- For samples where the paper reports agreement (mouse spleen, lemur lung, naked mole rat spleen, sea urchin embryo), uTAR silhouette values closely track gene-based silhouette values, qualitatively reproducing Fig. 2b
- aTAR-based clustering shows close agreement with gene-expression-based clustering across samples
- Silhouette scores are computed with Euclidean distance on the 2D UMAP coordinates, consistent with the reported metric

## Limitations

- The degree of relevant information in uTARs is strongly dependent on organism, tissue, cell type, and stage of development — agreement between uTAR and gene-based silhouette values holds only for several datasets, not universally
- Silhouette is computed on 2D UMAP coordinates, so the benchmark reflects structure in the 2D embedding rather than in the full-dimensional feature space
- uTAR content depends on annotation completeness of the organism; results span human, mouse, chicken, mole rat, lemur, and sea urchin and may not transfer to organisms outside these

## Evidence

- [other] compute the silhouette coefficient on the 2D UMAP coordinates with the scikit-learn silhouette_score metric (Euclidean distance on the 2D embedding): [evidence span withheld — anchor status: not_found]
- [other] Silhouette values from 2D UMAP show close aTAR–gene agreement and good uTAR–gene agreement: [evidence span withheld — anchor status: not_found]
- [other] the uTAR silhouette values should closely track the gene-based silhouette values, qualitatively reproducing Fig. 2b: [evidence span withheld — anchor status: not_found]
- [intro] Single-cell TAR expression analyses are then performed, using both annotated TARs (aTARs) and unannotated TARs (uTARs): "Single-cell TAR expression analyses are then performed, using both annotated TARs (aTARs) and unannotated TARs (uTARs), revealing the degree of biologically relevant information that exists both"
- [intro] we find that the degree of relevant information in uTARs is strongly dependent on organism, tissue, cell type, and stage of development: "we find that the degree of relevant information in uTARs is strongly dependent on organism, tissue, cell type, and stage of development"
- [readme] README: tool performs scRNA-seq analysis on any region in the genome for which transcriptional products are detected: "recovers biologically relevant transcriptional activity beyond the scope of the best available genome annotation by performing scRNA-seq analysis on any region in the genome for which transcriptional"
