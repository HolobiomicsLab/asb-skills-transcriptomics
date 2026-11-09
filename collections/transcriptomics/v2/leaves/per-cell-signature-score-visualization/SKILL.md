---
name: per-cell-signature-score-visualization
description: Use when per-cell annotation or disease scores (e.g., LHSC_per_cell_score,
  Neoplastic_monocyte_per_cell_score) are stored in a Seurat object's metadata and
  you need to compare their spatial/UMAP distributions across biological conditions
  such as healthy vs AML or diagnosis vs relapse.
when_to_use_negative:
- Input data are raw counts without per-cell signature scores in meta.data — run accordion_disease()
  scoring first rather than visualizing counts directly
- The goal is cluster-level annotation (accordion_per_cluster) rather than continuous
  per-cell score comparison across conditions; use DimPlot instead
- The dataset is spatial omics or a non-Seurat feature table where FeaturePlot/UMAP
  splitting by condition is not applicable
tools:
- name: Cell Marker Accordion
  role: Computes the per-cell disease signature scores (LHSC, Neoplastic Monocyte)
    via accordion_disease() that are then visualized
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: Seurat
  role: Hosts the preprocessed bone marrow Seurat object and provides FeaturePlot
    with split.by and min.cutoff/max.cutoff quantile scaling for the score visualizations
- name: ggplot2
  role: Supports rendering and styling of the grouped score distribution plots as
    PNG figures
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:b4dfb264ded71508778d16908b63fbbe783eb0312a1d416d4a411088c39c7e5e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - Cell Marker Accordion
  - R
  - Seurat
  - ggplot2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# per-cell-signature-score-visualization

## Summary

Visualize per-cell disease-signature scores (e.g., LHSC and Neoplastic Monocyte scores computed by the Cell Marker Accordion accordion_disease function) on a Seurat object using FeaturePlot split by condition (healthy vs AML), with quantile-based color cutoffs to expose aberrant-cell enrichment patterns.

## When to use

Use when per-cell annotation or disease scores (e.g., LHSC_per_cell_score, Neoplastic_monocyte_per_cell_score) are stored in a Seurat object's metadata and you need to compare their spatial/UMAP distributions across biological conditions such as healthy vs AML or diagnosis vs relapse.

## When NOT to use

- Input data are raw counts without per-cell signature scores in meta.data — run accordion_disease() scoring first rather than visualizing counts directly
- The goal is cluster-level annotation (accordion_per_cluster) rather than continuous per-cell score comparison across conditions; use DimPlot instead
- The dataset is spatial omics or a non-Seurat feature table where FeaturePlot/UMAP splitting by condition is not applicable

## Inputs

- Preprocessed Seurat object of CD34+ bone marrow cells (e.g., bone_marrow_data.rda from cellmarkeraccordion extdata) with per-cell score columns in meta.data
- Per-cell disease signature scores (LHSC_per_cell_score, Neoplastic_monocyte_per_cell_score) from accordion_disease() runs
- Condition metadata (healthy vs AML, diagnosis vs relapse/venetoclax-treated)

## Outputs

- FeaturePlot PNG files showing grouped score distributions split by condition with quantile cutoffs (q15/q95 for LHSC, q10/q90 for neoplastic monocyte)
- Exported score table (e.g., aml_disease_scores.csv) containing per-cell scores and condition labels

## How to apply

Load the preprocessed Seurat object (e.g., CD34+ bone marrow cells from system.file('extdata', 'bone_marrow_data.rda', package='cellmarkeraccordion')) and confirm the per-cell score columns (LHSC_per_cell_score, Neoplastic_monocyte_per_cell_score) exist in meta.data alongside condition metadata. Use Seurat's FeaturePlot on the UMAP reduction, group/split by the condition variable (healthy vs AML, or diagnosis vs relapse) so paired panels reveal where each signature is enriched. Apply quantile-based cutoffs to tame outlier scaling — min.cutoff='q15'/max.cutoff='q95' for LHSC scores and min.cutoff='q10'/max.cutoff='q90' for neoplastic monocyte scores — so the disease-enriched populations are visually distinguishable without outlier compression. Render each plot to PNG and export the scores plus condition labels (e.g., aml_disease_scores.csv) to make the visualization reproducible. Interpretation follows Fig. 5C-H patterns: LHSC scores should dominate AML patient cells and vanish at relapse after venetoclax, while neoplastic monocyte scores persist, indicating venetoclax resistance resides in the aberrant monocytic population.

## Related tools

- **Cell Marker Accordion** (Computes the per-cell disease signature scores (LHSC, Neoplastic Monocyte) via accordion_disease() that are then visualized) — https://github.com/TebaldiLab/cellmarkeraccordion
- **Seurat** (Hosts the preprocessed bone marrow Seurat object and provides FeaturePlot with split.by and min.cutoff/max.cutoff quantile scaling for the score visualizations)
- **ggplot2** (Supports rendering and styling of the grouped score distribution plots as PNG figures)

## Examples

```
aml <- accordion_disease(aml, assay="RNA", species="Human", tissue="bone marrow", disease="acute myeloid leukemia", NCIT_celltypes="Leukemic Hematopoietic Stem Cell", annotation_resolution="cell", max_n_marker=30, log2FC_threshold=1, annotation_name="LHSC"); png("lhsc_featureplot.png"); FeaturePlot(aml, features="LHSC_per_cell_score", split.by="condition", min.cutoff="q15", max.cutoff="q95"); dev.off()
```

## Evaluation signals

- LHSC per-cell scores are detectable/enriched in AML patient cells relative to healthy controls, reproducing the aberrant-cell pattern of Fig. 5C-H
- LHSC scores disappear at relapse after venetoclax treatment, while Neoplastic Monocyte scores persist — confirming resistance resides in the aberrant monocytic population
- FeaturePlot color scaling uses the specified quantile cutoffs (min.cutoff='q15'/max.cutoff='q95' for LHSC; q10/q90 for neoplastic monocyte), preventing outlier-driven saturation
- Exported aml_disease_scores.csv contains per-cell scores paired with correct condition labels (healthy vs AML / diagnosis vs relapse) for every cell in the Seurat object
- Plots are successfully rendered as PNG files split by condition without errors on the UMAP reduction

## Limitations

- The demonstrated bone marrow example is specific to AML CD34+ cells and the LHSC/Neoplastic Monocyte disease cell types in the Cell Marker Accordion disease database (NCIT); other diseases require their own NCIT cell types and validated score patterns
- Quantile cutoff values (q15/q95, q10/q90) are example-specific choices; applying them without checking score distributions can clip genuine signal in other datasets
- Visualization is only as good as the upstream accordion_disease() scoring — the paper notes the cancer benchmark was limited to two liquid and two solid tumors and is planned to expand

## Evidence

- [other] Visualize the grouped score distributions with Seurat FeaturePlot split by condition, using min.cutoff="q15"/max.cutoff="q95" for LHSC: [evidence span withheld — anchor status: not_found]
- [methods] Single-cell datasets were analyzed employing a standard pre-processing pipeline for single-cell RNA-seq data, using Seurat: "Single-cell datasets were analyzed employing a standard pre-processing pipeline for single-cell RNA-seq data, using Seurat"
- [other] Load the preprocessed Seurat object of CD34+ bone marrow cells from the package extdata: [evidence span withheld — anchor status: not_found]
- [readme] DimPlot(data, group.by = "accordion_per_cluster"): "DimPlot(data, group.by = "accordion_per_cluster")"
- [abstract] the Cell Marker Accordion can identify disease-critical cells and pathological processes: "we show that the Cell Marker Accordion can identify disease-critical cells and pathological processes, extracting potential biomarkers in a wide variety of disease contexts"
