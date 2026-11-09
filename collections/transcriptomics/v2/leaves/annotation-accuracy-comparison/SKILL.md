---
name: annotation-accuracy-comparison
description: Use when you need to quantitatively evaluate how well automated cell-type
  annotation (reference-based or marker-based) reproduces manual cell-type assignments
  in plant single-cell datasets, especially across species.
when_to_use_negative:
- You lack any reference cell map or curated marker gene database for the tissue/species
  — scPlant's automatic annotation relies on one of these two inputs
- Your query data are bulk transcriptomes rather than single-cell Seurat objects —
  deconvolution (decomposeBulk), not per-cell label transfer, is the appropriate module
- You need a final annotation with no manual labels available for validation — this
  skill specifically measures agreement against ground-truth annotations
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0780
tools:
- name: scPlant
  role: Framework providing AutoAnnotate_SingleR, AutoAnnotate_Garnett, AutoAnnotate_scCATCH,
    AutoAnnotate_CellAssign, get_marker_file, and get_marker_scCATCH for automatic
    cell-type annotation
  repo: github.com/compbioNJU/scPlant
- name: SingleR
  role: Reference-based label transfer backend invoked by AutoAnnotate_SingleR with
    ref_type = "single-cell"
- name: Garnett
  role: Marker-file-driven annotation backend invoked by AutoAnnotate_Garnett
- name: scCATCH
  role: Marker-based annotation backend invoked by AutoAnnotate_scCATCH with a custom
    marker object
- name: CellAssign
  role: Marker-data-frame-driven annotation backend invoked by AutoAnnotate_CellAssign
- name: CellFunTopic
  role: Functional-signature-based annotation via CellFunTopic::predictFun after RunGSEA_plant
    on the reference
  repo: github.com/compbioNJU/CellFunTopic
- name: Seurat
  role: Seurat object handling and as.SingleCellExperiment conversion of references
    for label transfer
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:63695dd4308e8cdcc32b3a4b4dcd936abc977eeb43afb6bfc59753f92343ada4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/compbioNJU/scPlant
  tools:
  - AutoAnnotate_CellAssign
  - AutoAnnotate_Garnett
  - AutoAnnotate_SingleR
  - AutoAnnotate_scCATCH
  - CellAssign
  - CellFunTopic
  - Garnett
  - Seurat
  - SingleR
  - scCATCH
  - scPlant
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 20
---

# annotation-accuracy-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks scPlant's automatic cell-type annotation strategies (SingleR, Garnett, scCATCH, CellAssign, CellFunTopic) against manual ground-truth annotations for plant scRNA-seq/snRNA-seq data, including cross-species reference-based annotation (e.g., wheat root cells annotated against a rice reference).

## When to use

Use when you need to quantitatively evaluate how well automated cell-type annotation (reference-based or marker-based) reproduces manual cell-type assignments in plant single-cell datasets, especially across species.

## When NOT to use

- You lack any reference cell map or curated marker gene database for the tissue/species — scPlant's automatic annotation relies on one of these two inputs
- Your query data are bulk transcriptomes rather than single-cell Seurat objects — deconvolution (decomposeBulk), not per-cell label transfer, is the appropriate module
- You need a final annotation with no manual labels available for validation — this skill specifically measures agreement against ground-truth annotations

## Inputs

- Pre-processed query Seurat objects (e.g., PRJNA507252 Arabidopsis root, CRA008788 wheat root)
- Reference scRNA-seq cell maps as Seurat objects (e.g., PRJNA517021, PRJNA706435), convertible via Seurat::as.SingleCellExperiment
- Gene orthology mapping between query and reference species (e.g., wheat genes to rice orthologs)
- Marker gene data frame (celltype, gene) per species/tissue, plus scCATCH-format marker_custom objects

## Outputs

- Per-cell predicted labels in SeuratObj$predicted_label from each annotation strategy
- annotation_predictions.csv with per-cell predictions from all tools and agreement metrics versus manual ground truth

## How to apply

Load pre-processed query Seurat objects (e.g., PRJNA507252 Arabidopsis root, CRA008788 wheat root) and reference cell maps (PRJNA517021, PRJNA706435), converting references with Seurat::as.SingleCellExperiment where needed; for wheat-vs-rice, first map query genes to reference orthologs. Run reference-based label transfer with scPlant::AutoAnnotate_SingleR(SeuratObj, SCE_ref, ref_type = "single-cell"), storing predictions in SeuratObj$predicted_label. Run marker-based alternatives: AutoAnnotate_Garnett with a marker file from get_marker_file(markerDF, SeuratObj, file = "markers.txt") (check the marker file format first if Garnett errors), AutoAnnotate_scCATCH after building marker_custom via get_marker_scCATCH(markerDF, species = 'Arabidopsis thaliana', tissue = 'root'), and AutoAnnotate_CellAssign(SeuratObj, markerDF) with a two-column (celltype, gene) marker data frame; optionally add CellFunTopic::predictFun (group_by = "cell_type", cluster_by = "seurat_clusters", species = "Arabidopsis thaliana") after RunGSEA_plant on the reference. Compare each tool's predicted labels against manual ground-truth annotations (Figure 2D / Supplemental Figure 1) and write per-cell predictions and agreement metrics to annotation_predictions.csv.

## Related tools

- **scPlant** (Framework providing AutoAnnotate_SingleR, AutoAnnotate_Garnett, AutoAnnotate_scCATCH, AutoAnnotate_CellAssign, get_marker_file, and get_marker_scCATCH for automatic cell-type annotation) — github.com/compbioNJU/scPlant
- **SingleR** (Reference-based label transfer backend invoked by AutoAnnotate_SingleR with ref_type = "single-cell")
- **Garnett** (Marker-file-driven annotation backend invoked by AutoAnnotate_Garnett)
- **scCATCH** (Marker-based annotation backend invoked by AutoAnnotate_scCATCH with a custom marker object)
- **CellAssign** (Marker-data-frame-driven annotation backend invoked by AutoAnnotate_CellAssign)
- **CellFunTopic** (Functional-signature-based annotation via CellFunTopic::predictFun after RunGSEA_plant on the reference) — github.com/compbioNJU/CellFunTopic
- **Seurat** (Seurat object handling and as.SingleCellExperiment conversion of references for label transfer)

## Examples

```
SeuratObj <- scPlant::AutoAnnotate_SingleR(SeuratObj, SCE_ref, ref_type = "single-cell")
```

## Evaluation signals

- predicted_label values for both query datasets (PRJNA507252 Arabidopsis, CRA008788 wheat) reproduce the manual cell-type assignments reported in Figure 2D / Supplemental Figure 1 of the scPlant paper
- Each strategy stores predictions in SeuratObj$predicted_label for direct cross-tool comparison
- Per-cell predictions and agreement metrics for all tools are present in annotation_predictions.csv
- Cross-species wheat-vs-rice annotation requires prior ortholog mapping of query genes to reference genes; accuracy indicates successful cross-species transfer
- Garnett runs only after get_marker_file output passes format checking (check marker file if Garnett errors)

## Limitations

- Marker-based tools (Garnett, scCATCH, CellAssign) depend on the quality and completeness of the marker gene database; Garnett is sensitive to marker file formatting errors
- Reference-based annotation across species (wheat vs rice) requires ortholog mapping, and accuracy may vary with evolutionary distance and reference quality
- scPlant's automatic annotation is described as yielding 'promising predictions' — agreement with manual ground truth is not guaranteed to be perfect for all strategies and datasets

## Evidence

- [methods] scPlant have implemented several different solutions for automatic cell-type annotation: "scPlant have implemented several different solutions for automatic cell-type annotation that are based on popular tools"
- [methods] annotation tool list: [evidence span withheld — anchor status: not_found]
- [methods] workflow step RunGSEA_plant: "SeuratObj <- RunGSEA_plant(SeuratObj, by = 'GO', GeneIDtype = 'TAIR', minpct = 0.25)"
- [readme] CellFunTopic README: [evidence span withheld — anchor status: external_doc]
- [abstract] scPlant framework scope: "ranging from basic data processing to advanced demands such as cell type annotation and deconvolution, trajectory inference, cross-species data integration"
- [other] task finding on wheat annotation: [evidence span withheld — anchor status: not_found]
