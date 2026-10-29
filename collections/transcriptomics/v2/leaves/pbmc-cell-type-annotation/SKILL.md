---
name: pbmc-cell-type-annotation
description: Use when a raw PBMC single-cell RNA-seq count matrix (e.g. a GEO accession
  like GSE149689) must be clustered and annotated into immune cell subsets to localise
  gene-expression dysregulation to a cell type.
when_to_use_negative:
- Input is bulk RNA-seq count or processed differential-expression data (the COVID19-severity
  Streamlit app handles those; this skill needs per-cell matrices)
- The single-cell data are already annotated and the question is only querying gene
  expression per cell type
- Analysis targets BALF single-cell data with different QC needs unless the PBMC thresholds
  are re-validated
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Seurat
  role: QC, normalization, PCA, resolution-1 clustering, UMAP embedding, and VlnPlot
    generation of MCEMP1/HLA-DRA/ETS1 across annotated clusters
- name: Harmony
  role: Batch integration on the top-20-PC Seurat embedding before clustering
- name: UMAP
  role: 2D embedding of integrated cells for visualisation and cell-type assignment
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:492e337b579f61c60c64e43d14183180aa409a7b3181c66119f3f20db96b9290
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Harmony
  - Seurat
  - UMAP
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# pbmc-cell-type-annotation

## Summary

Single-cell analysis of PBMC scRNA-seq count matrices that clusters cells into immune cell subsets so differential expression signals (e.g. MCEMP1 up, HLA-DRA down) can be localised to specific lineages such as CD14+ monocytes.

## When to use

Use when a raw PBMC single-cell RNA-seq count matrix (e.g. a GEO accession like GSE149689) must be clustered and annotated into immune cell subsets to localise gene-expression dysregulation to a cell type.

## When NOT to use

- Input is bulk RNA-seq count or processed differential-expression data (the COVID19-severity Streamlit app handles those; this skill needs per-cell matrices)
- The single-cell data are already annotated and the question is only querying gene expression per cell type
- Analysis targets BALF single-cell data with different QC needs unless the PBMC thresholds are re-validated

## Inputs

- Raw PBMC single-cell RNA-seq count matrix from a public GEO accession (e.g. GSE149689)
- Sample/condition metadata (severe vs mild COVID-19 status)

## Outputs

- QC-filtered, Harmony-integrated Seurat object with clusters at resolution 1
- Cell-type annotations (including CD14+ monocytes and proliferating cells) with 2D UMAP embedding
- VlnPlot violin plots of MCEMP1, HLA-DRA and ETS1 expression across annotated cell-type clusters

## How to apply

Download the public PBMC raw count matrix from GEO, then in Seurat apply the published quality-control thresholds (this study retained 48,583 cells), normalize counts, and run PCA keeping the top 20 principal components. Integrate batches with Harmony on the top-20-PC embedding and cluster cells at resolution 1, since the paper's workflow found this resolution sufficient to resolve CD14+ monocytes and proliferating cells. Embed integrated cells in 2D with UMAP for visualisation and assign cell-type labels from canonical marker expression rather than reference mapping. Generate VlnPlot violin plots of the genes of interest (MCEMP1, HLA-DRA, ETS1) across annotated clusters; accept the result if heightened MCEMP1 and reduced HLA-DRA appear primarily in CD14+ monocytes, reduced HLA-DRA extends broadly across other lineages, and low ETS1 is restricted to proliferating cells.

## Related tools

- **Seurat** (QC, normalization, PCA, resolution-1 clustering, UMAP embedding, and VlnPlot generation of MCEMP1/HLA-DRA/ETS1 across annotated clusters)
- **Harmony** (Batch integration on the top-20-PC Seurat embedding before clustering)
- **UMAP** (2D embedding of integrated cells for visualisation and cell-type assignment)

## Examples

```
VlnPlot(seurat_obj, features = c("MCEMP1", "HLA-DRA", "ETS1"), split.by = "condition")
```

## Evaluation signals

- QC thresholds retain approximately 48,583 cells as reported in the study workflow
- Clusters formed at resolution 1 on the Harmony-integrated, top-20-PC embedding separate CD14+ monocytes from other lineages and proliferating cells by canonical markers
- VlnPlot reproduces the reported pattern: significantly heightened MCEMP1 and reduced HLA-DRA primarily in CD14+ immune cell subsets
- Reduced HLA-DRA is broadly detected across other immune cell lineages, not confined to monocytes
- Low ETS1 expression is restricted to proliferating cells

## Limitations

- Annotation relies on canonical marker expression without an external ground-truth validation of labels
- Dataset heterogeneity was noted in the systematic review, so cell-subset proportions and expression patterns may vary across cohorts
- The included datasets did not directly evaluate the effect of different SARS-CoV-2 variants on transcriptomic responses
- Sensitivity and specificity of the MCEMP1/HLA-DRA signal for predicting severity have not been prospectively validated

## Evidence

- [other] Single-cell analysis of PBMCs revealed that heightened MCEMP1 and reduced HLA-DRA expression in severe COVID-19 patients were seen predominantly in CD14+ monocytes, with reduced HLA-DRA also broadly detected across other immune cell lineages.: "Single cell RNA-sequencing of peripheral blood mononuclear cells from publicly available datasets was then used to determine the immune cell subsets involved."
- [intro] Dysregulation of MCEMP1 and HLA-DRA transcripts occurs primarily in CD14+ immune cell subsets.: "we ascertained that the dysregulation of MCEMP1 and HLA-DRA transcripts occurs primarily in the CD14+ immune cell subsets"
- [discussion] Analysis of publicly available single cell sequencing data from PBMCs and BALF.: "we took advantage of the opportunity to analyse the publicly available single cell sequencing data from PBMCs and BALF"
