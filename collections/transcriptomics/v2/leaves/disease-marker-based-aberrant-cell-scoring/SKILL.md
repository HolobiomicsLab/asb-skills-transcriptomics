---
name: disease-marker-based-aberrant-cell-scoring
description: Use when single-cell data from a diseased tissue (e.g., AML bone marrow)
  must be screened for aberrant/disease-critical cell populations defined in NCIT,
  and per-cell disease scores must be compared across conditions such as healthy vs
  AML or pre- vs post-treatment relapse.
when_to_use_negative:
- The goal is annotation of normal/physiological cell types across tissues — use the
  standard accordion() function with the healthy Accordion database instead of disease
  scoring
- The disease cell type of interest is not represented in the NCIT-backed disease
  Accordion database (728 cell types / 212 human tissues coverage), so no weighted
  marker set exists
- Input is bulk RNA-seq or a precomputed feature table with no per-cell resolution,
  since the method computes per-cell scores
edam_operation: http://edamontology.org/operation_0313
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0085
tools:
- name: Cell Marker Accordion
  role: R package providing accordion_disease() and the consistency-weighted disease
    marker database used to compute per-cell aberrant-cell scores
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: Seurat
  role: Holds the input single-cell object, supplies assay data and metadata, and
    renders FeaturePlot visualizations of the per-cell disease scores split by condition
- name: ggplot2
  role: Supports grouped visualization of score distributions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:77ba20ec4b4276f2e5cbeef3dd953502b9ce46561589b41e1a7697c765714069
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - Cell Marker Accordion
  - R
  - Seurat
  - ggplot2
  edam_operation: http://edamontology.org/operation_0313
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# disease-marker-based-aberrant-cell-scoring

## Summary

Score per-cell enrichment of disease-critical cell types (e.g., Leukemic Hematopoietic Stem Cell, Neoplastic Monocyte) in scRNA-seq data using the accordion_disease() function of the Cell Marker Accordion R package, which exploits the disease Accordion marker database built from consistency-weighted positive and negative markers.

## When to use

Use when single-cell data from a diseased tissue (e.g., AML bone marrow) must be screened for aberrant/disease-critical cell populations defined in NCIT, and per-cell disease scores must be compared across conditions such as healthy vs AML or pre- vs post-treatment relapse.

## When NOT to use

- The goal is annotation of normal/physiological cell types across tissues — use the standard accordion() function with the healthy Accordion database instead of disease scoring
- The disease cell type of interest is not represented in the NCIT-backed disease Accordion database (728 cell types / 212 human tissues coverage), so no weighted marker set exists
- Input is bulk RNA-seq or a precomputed feature table with no per-cell resolution, since the method computes per-cell scores

## Inputs

- Preprocessed Seurat object or raw/normalized count matrix (genes x cells) from diseased tissue, e.g., CD34+ AML bone marrow cells
- Species, tissue, and disease identifiers matching the Cell Marker Accordion disease database (e.g., Human / bone marrow / acute myeloid leukemia)
- NCIT-defined disease cell type(s) to score (e.g., 'Leukemic Hematopoietic Stem Cell', 'Neoplastic Monocyte')
- Condition metadata per cell (e.g., healthy vs AML, diagnosis vs relapse after venetoclax)

## Outputs

- Seurat object augmented with per-cell disease score metadata columns (e.g., LHSC_per_cell_score, Neoplastic_monocyte_per_cell_score)
- Exported score table (e.g., aml_disease_scores.csv) joining per-cell scores with condition labels
- PNG feature plots of grouped score distributions split by condition with quantile cutoffs (min.cutoff='q15'/max.cutoff='q95' for LHSC; 'q10'/'q90' for neoplastic monocyte)

## How to apply

Load a preprocessed Seurat object (e.g., CD34+ bone marrow cells via system.file('extdata','bone_marrow_data.rda',package='cellmarkeraccordion')) and run accordion_disease() once per disease cell type, supplying assay='RNA', species='Human', tissue='bone marrow', disease='acute myeloid leukemia', the NCIT cell type (e.g., 'Leukemic Hematopoietic Stem Cell' or 'Neoplastic Monocyte'), annotation_resolution='cell', max_n_marker=30, log2FC_threshold=1, and a distinct annotation_name (e.g., 'LHSC', 'Neoplastic_monocyte'). The function selects disease markers from the built-in disease Accordion database weighted by specificity score (SPs) and evidence consistency score (ECs), then computes a per-cell score column (e.g., LHSC_per_cell_score, Neoplastic_monocyte_per_cell_score) in the Seurat object. Export scores with condition metadata (healthy vs AML, diagnosis vs relapse) to CSV and visualize with Seurat FeaturePlot split by condition using quantile cutoffs (q15/q95 for LHSC, q10/q90 for neoplastic monocyte). Rationale: consistency-weighted markers reduce the heterogeneity and inconsistent nomenclature of raw marker databases, enabling identification of aberrant cell states responsible for disease persistence and therapy resistance.

## Related tools

- **Cell Marker Accordion** (R package providing accordion_disease() and the consistency-weighted disease marker database used to compute per-cell aberrant-cell scores) — https://github.com/TebaldiLab/cellmarkeraccordion
- **Seurat** (Holds the input single-cell object, supplies assay data and metadata, and renders FeaturePlot visualizations of the per-cell disease scores split by condition)
- **ggplot2** (Supports grouped visualization of score distributions)

## Examples

```
data <- accordion_disease(data, assay="RNA", species="Human", tissue="bone marrow", disease="acute myeloid leukemia", NCIT_celltypes="Leukemic Hematopoietic Stem Cell", annotation_resolution="cell", max_n_marker=30, log2FC_threshold=1, annotation_name="LHSC")
```

## Evaluation signals

- Per-cell disease scores are detectable/enriched in AML patient cells relative to healthy controls, reproducing the aberrant-cell scoring patterns of Fig. 5C-H
- Both requested score columns (e.g., LHSC_per_cell_score and Neoplastic_monocyte_per_cell_score) exist in the Seurat object metadata after the two accordion_disease() runs with distinct annotation_name values
- In the venetoclax diagnosis/relapse comparison, LHSC scores decrease/disappear at relapse while neoplastic monocyte scores persist, matching the reported resistance mechanism
- Feature plots use the specified quantile cutoffs (q15/q95 for LHSC, q10/q90 for neoplastic monocyte) and render without cells lacking condition labels
- accordion_disease() runs succeed with log2FC_threshold=1 and max_n_marker=30, returning top marker genes that impacted each disease annotation

## Limitations

- Disease cell-type scoring depends on coverage of the built-in disease Accordion database; cell types absent from NCIT-derived entries cannot be scored
- The benchmark of disease contexts was limited (e.g., two liquid and two solid tumors in the cancer benchmark), with expansion planned, so performance in other disease settings is less characterized
- Scores require prior standard preprocessing and dimensionality reduction of the Seurat object; unnormalized or heavily filtered data may yield unreliable scoring

## Evidence

- [methods] Disease scoring via accordion_disease() on NCIT cell types: "Users can annotate clusters or cells exploiting the built-in Cell Marker Accordion database with the accordion() function"
- [methods] Weighted marker integration rationale: "the Cell Marker Accordion database describes each marker with a specificity score (SPs) and an evidence consistency score (ECs)"
- [abstract] Disease-critical cell identification capability: "we show that the Cell Marker Accordion can identify disease-critical cells and pathological processes, extracting potential biomarkers in a wide variety of disease contexts."
- [readme] Package input types and loading of built-in data: "All the functions of the <strong>cellmarkeraccordion</strong> accept as input either a Seurat object or a raw or normalized count matrix."
- [readme] Disease database availability in the package: "To access the *disease* Accordion database run:
bash
data(disease_accordion_marker)
"
