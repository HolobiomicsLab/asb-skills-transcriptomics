---
name: harmony-batch-integration
description: Use when a raw single-cell RNA-seq count matrix (e.g. GSE149689 PBMCs)
  pooled from multiple samples/donors shows batch effects that would confound clustering
  and cell-type annotation prior to differential-expression localisation.
when_to_use_negative:
- Input is bulk RNA-seq or bulk transcriptomic data (e.g. the 7 bulk severe-vs-mild
  COVID-19 datasets in the COVID19-severity database) — Harmony integration is defined
  on single-cell embeddings, not bulk samples.
- The single-cell data already come from a single sample/donor with no batch structure
  to correct — integration would remove real biological variation for nothing.
- Cells have already been integrated by another method and you only need cell-type
  annotation or VlnPlot output.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: QC of raw counts, normalization, PCA (top 20 PCs), clustering at resolution
    1, UMAP embedding, and VlnPlot of MCEMP1/HLA-DRA/ETS1 per annotated cluster
- name: Harmony
  role: Batch integration on the top-20-PC embedding so cells from different donors/samples
    can be clustered jointly
- name: UMAP
  role: 2D visualization of the Harmony-integrated cells for cell-type annotation
    and presentation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:d037db70b32c0c945391582faf43165809f6b7be6d6340624091b5ae9deb9a8d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Harmony
  - Seurat
  - UMAP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# harmony-batch-integration

## Summary

Harmony-based batch integration of a single-cell PBMC Seurat object on the top-20-PC embedding, so that cells pooled across samples/batches can be jointly clustered and cell-type-resolved. In this paper it enables localisation of MCEMP1 up-regulation and HLA-DRA down-regulation to CD14+ monocytes.

## When to use

Use when a raw single-cell RNA-seq count matrix (e.g. GSE149689 PBMCs) pooled from multiple samples/donors shows batch effects that would confound clustering and cell-type annotation prior to differential-expression localisation.

## When NOT to use

- Input is bulk RNA-seq or bulk transcriptomic data (e.g. the 7 bulk severe-vs-mild COVID-19 datasets in the COVID19-severity database) — Harmony integration is defined on single-cell embeddings, not bulk samples.
- The single-cell data already come from a single sample/donor with no batch structure to correct — integration would remove real biological variation for nothing.
- Cells have already been integrated by another method and you only need cell-type annotation or VlnPlot output.

## Inputs

- Raw single-cell RNA-seq count matrix from GEO (e.g. GSE149689 PBMC dataset)
- Seurat object with per-cell metadata (sample/batch labels) and normalized counts + top-20-PC PCA embedding

## Outputs

- Harmony-integrated Seurat embedding (corrected top-20-PC space)
- Cell clusters generated at resolution 1 on the integrated embedding
- 2D UMAP embedding of integrated cells
- Cell-type annotations (including CD14+ monocytes and proliferating cells) used for per-cluster gene-expression plots

## How to apply

In Seurat, first apply the methods' quality-control thresholds to the raw count matrix (this retained 48,583 cells), normalize counts, and run PCA keeping the top 20 principal components. Integrate batches by running Harmony on the top-20-PC embedding so cells align across donors/samples while preserving the biological signal of interest. Cluster the Harmony-corrected embedding at resolution 1 and embed the integrated cells in 2D UMAP for visualization. Assign cell-type labels (CD14+ monocytes, proliferating cells, and other lineages) from canonical marker expression, then examine MCEMP1, HLA-DRA, and ETS1 per cluster (e.g. Seurat VlnPlot) to attribute dysregulation to specific subsets.

## Related tools

- **Seurat** (QC of raw counts, normalization, PCA (top 20 PCs), clustering at resolution 1, UMAP embedding, and VlnPlot of MCEMP1/HLA-DRA/ETS1 per annotated cluster)
- **Harmony** (Batch integration on the top-20-PC embedding so cells from different donors/samples can be clustered jointly)
- **UMAP** (2D visualization of the Harmony-integrated cells for cell-type annotation and presentation)

## Examples

```
pbmc <- RunHarmony(pbmc, group.by.vars = "batch", reduction = "pca", dims = 1:20); pbmc <- FindNeighbors(pbmc, reduction = "harmony", dims = 1:20); pbmc <- FindClusters(pbmc, resolution = 1); pbmc <- RunUMAP(pbmc, reduction = "harmony", dims = 1:20)
```

## Evaluation signals

- Cell count after QC matches the methods' retained set (48,583 cells) before integration.
- Post-integration UMAP shows cells mixing across donor/sample batches rather than separating by batch.
- Clustering at resolution 1 yields a cluster whose canonical marker expression supports CD14+ monocyte identity.
- VlnPlot reproduces the reported pattern: heightened MCEMP1 and reduced HLA-DRA predominantly in CD14+ monocytes, reduced HLA-DRA also broadly across other lineages.
- ETS1 expression is low and restricted to proliferating cells, matching the validation criterion.

## Limitations

- Heterogeneity across study datasets was noted, so integration assumptions and resulting clusters may vary with patient/dataset composition.
- Downstream interpretation depends on correct cell-type labelling from canonical markers; mis-annotation would mis-attribute MCEMP1/HLA-DRA dysregulation.
- The reduced HLA-DRA signal extends broadly across other immune lineages, so CD14+ monocyte localisation holds mainly for the joint MCEMP1/HLA-DRA pattern.

## Evidence

- [abstract] Single cell RNA-sequencing of peripheral blood mononuclear cells from publicly available datasets was then used to determine the immune cell subsets involved.: "Single cell RNA-sequencing of peripheral blood mononuclear cells from publicly available datasets was then used to determine the immune cell subsets involved."
- [intro] Dysregulation of MCEMP1 and HLA-DRA transcripts occurs primarily in CD14+ immune cell subsets: "we ascertained that the dysregulation of MCEMP1 and HLA-DRA transcripts occurs primarily in the CD14+ immune cell subsets"
- [discussion] Analysis of publicly available single cell sequencing data from PBMCs and BALF: "we took advantage of the opportunity to analyse the publicly available single cell sequencing data from PBMCs and BALF"
