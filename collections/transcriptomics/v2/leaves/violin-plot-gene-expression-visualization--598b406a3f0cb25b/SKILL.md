---
name: violin-plot-gene-expression-visualization
description: Use when you have an annotated single-cell RNA-seq object (e.g. Seurat
  with cell-type labels) and need to determine which cell subsets show differential
  expression of specific marker genes.
when_to_use_negative:
- Input is bulk RNA-seq or a processed differential-expression table with no per-cell
  resolution — box plots of the Streamlit app counts/anova folders are the appropriate
  visualization instead
- Clusters are not yet annotated from canonical markers; violin plots across unlabelled
  clusters cannot localize dysregulation to CD14+ subsets
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: R toolkit used for QC, normalization, PCA, clustering, cell-type annotation,
    and VlnPlot generation of gene expression across cell-type clusters
- name: Harmony
  role: Batch integration on the top-20-PC embedding prior to clustering, so violin
    plots are computed on non-batch-confounded cell annotations
- name: UMAP
  role: 2D embedding used alongside violin plots to visualize and assign cell-type
    labels from canonical marker expression
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.ebiom.2023.104472
    title: Early peripheral blood MCEMP1 and HLA-DRA expression predicts COVID-19
      prognosis
  source_content_hash: sha256:3b6b8b5899a63cfdad8febb9599998ac9fd47350d87a49360d4310202bc05b90
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Harmony
  - Seurat
  - UMAP
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# violin-plot-gene-expression-visualization

## Summary

Visualize per-cell gene expression distributions across annotated immune cell types with Seurat's VlnPlot, as used to localize MCEMP1 upregulation and HLA-DRA downregulation to CD14+ monocytes in PBMCs of severe COVID-19 patients.

## When to use

Use when you have an annotated single-cell RNA-seq object (e.g. Seurat with cell-type labels) and need to determine which cell subsets show differential expression of specific marker genes.

## When NOT to use

- Input is bulk RNA-seq or a processed differential-expression table with no per-cell resolution — box plots of the Streamlit app counts/anova folders are the appropriate visualization instead
- Clusters are not yet annotated from canonical markers; violin plots across unlabelled clusters cannot localize dysregulation to CD14+ subsets

## Inputs

- Annotated Seurat object of PBMC single-cell RNA-seq counts (e.g. GSE149689 raw count matrix, 48,583 QC-passing cells)
- Cell-type annotations from canonical marker expression (CD14+ monocytes, proliferating cells, etc.)
- Gene list of interest (MCEMP1, HLA-DRA, ETS1)

## Outputs

- Violin plots of gene expression per annotated cell-type cluster (Seurat VlnPlot output)
- Qualitative localization of differential expression to specific immune cell subsets

## How to apply

After QC, normalization, PCA (top 20 PCs), Harmony batch integration, clustering at resolution 1, UMAP embedding, and annotation of clusters from canonical marker expression in Seurat, call VlnPlot for the genes of interest (here MCEMP1, HLA-DRA, ETS1) grouped by the annotated cell-type clusters. In the Chan et al. GSE149689 PBMC analysis, this retained 48,583 cells and produced violin plots whose shapes reveal whether differential expression is concentrated in a lineage (CD14+ monocytes) or spread broadly (reduced HLA-DRA across lineages, low ETS1 restricted to proliferating cells). Correctness is judged by whether the plots reproduce the published pattern: heightened MCEMP1 and reduced HLA-DRA primarily in CD14+ immune cell subsets, with reduced HLA-DRA also detected across other lineages.

## Related tools

- **Seurat** (R toolkit used for QC, normalization, PCA, clustering, cell-type annotation, and VlnPlot generation of gene expression across cell-type clusters)
- **Harmony** (Batch integration on the top-20-PC embedding prior to clustering, so violin plots are computed on non-batch-confounded cell annotations)
- **UMAP** (2D embedding used alongside violin plots to visualize and assign cell-type labels from canonical marker expression)

## Examples

```
VlnPlot(seurat_obj, features = c("MCEMP1", "HLA-DRA", "ETS1"), group.by = "celltype", split.by = "condition")
```

## Evaluation signals

- Violin plot shows significantly heightened MCEMP1 primarily in the CD14+ monocyte subset of severe COVID-19 PBMCs
- HLA-DRA expression is reduced in CD14+ cells and broadly reduced across other immune cell lineages
- Low ETS1 expression is restricted to proliferating cells, matching the reported pattern
- Cell count after QC thresholds matches the expected 48,583 cells from GSE149689 before plotting
- Cell-type axis labels correspond to clusters annotated from canonical marker expression, not unlabelled cluster IDs

## Limitations

- Violin plots localize differential expression visually but do not by themselves validate prognostic sensitivity or specificity of MCEMP1/HLA-DRA; prospective studies were flagged as still needed
- Heterogeneity across study datasets means the pattern may not generalize to all patient cohorts or SARS-CoV-2 variants
- Only one gene can be selected at a time for graph plotting in the accompanying app, so multi-gene comparisons require separate plots or custom Seurat code

## Evidence

- [other] Generate violin plots of MCEMP1, HLA-DRA, and ETS1 expression across the annotated cell-type clusters with Seurat's VlnPlot.: [evidence span withheld — anchor status: not_found]
- [abstract] Single cell RNA-sequencing of peripheral blood mononuclear cells from publicly available datasets was then used to determine the immune cell subsets involved.: "Single cell RNA-sequencing of peripheral blood mononuclear cells from publicly available datasets was then used to determine the immune cell subsets involved."
- [intro] we ascertained that the dysregulation of MCEMP1 and HLA-DRA transcripts occurs primarily in the CD14+ immune cell subsets: "we ascertained that the dysregulation of MCEMP1 and HLA-DRA transcripts occurs primarily in the CD14+ immune cell subsets"
- [readme] Tick the checkbox next to the gene of interest. Note that only one gene can be selected at a time for graph plotting.: "Tick the checkbox next to the gene of interest. Note that only one gene can be selected at a time for graph plotting."
