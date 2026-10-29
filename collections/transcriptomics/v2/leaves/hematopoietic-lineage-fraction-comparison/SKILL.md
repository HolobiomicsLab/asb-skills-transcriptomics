---
name: hematopoietic-lineage-fraction-comparison
description: Use when single-cell RNA-seq of murine bone marrow under hematopoietic
  perturbation (gene knockout or drug inhibition such as Mettl3 KO/STM2457) must be
  compared to a control for changes in lineage composition, cell cycle phase fractions
  (G0), or innate immune response signatures.
when_to_use_negative:
- Input is bulk RNA-seq or already-summarized cell type fractions rather than a single-cell
  Seurat object or count matrix
- Tissue/species outside the Accordion database coverage — verify available tissues
  first with list_tissues()
- The question requires variant calling rather than cell type/signature annotation
  (FreeBayes/VarTrix tools in this article serve CNV/variant workflows, not annotation)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cell Marker Accordion
  role: R package providing accordion(), accordion_cellcycle() and accordion_custom()
    for cell type annotation, cell cycle classification, and custom signature scoring
    of bone marrow single-cell data
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: Seurat
  role: Holds the processed bone marrow single-cell objects; annotation results, cell
    cycle assignments and per-cell scores are stored in and retrieved from the Seurat
    object (meta.data and misc slots)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:c06150b519710c2c9ba4c1725f15a9bbc937f85355817b2f5707d45747da31f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - Cell Marker Accordion
  - R
  - Seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# hematopoietic-lineage-fraction-comparison

## Summary

Annotates murine bone marrow single-cell data with the Cell Marker Accordion (accordion(), accordion_cellcycle(), accordion_custom()) and compares lineage/cell-cycle composition and innate immune response scores between conditions, e.g. Mettl3 KO vs WT and STM2457 vs vehicle, to detect hematopoietic lineage shifts and innate immune activation.

## When to use

Use when single-cell RNA-seq of murine bone marrow under hematopoietic perturbation (gene knockout or drug inhibition such as Mettl3 KO/STM2457) must be compared to a control for changes in lineage composition, cell cycle phase fractions (G0), or innate immune response signatures.

## When NOT to use

- Input is bulk RNA-seq or already-summarized cell type fractions rather than a single-cell Seurat object or count matrix
- Tissue/species outside the Accordion database coverage — verify available tissues first with list_tissues()
- The question requires variant calling rather than cell type/signature annotation (FreeBayes/VarTrix tools in this article serve CNV/variant workflows, not annotation)

## Inputs

- Processed Seurat objects of mouse bone marrow cells from two conditions (e.g. WT vs Mettl3 KO, vehicle vs STM2457) with a 'condition' metadata column
- Built-in innate immune response signature table (in_im_resp_sig) from the cellmarkeraccordion R package
- Cell Marker Accordion marker database (accordion_marker) covering mouse bone marrow cell types

## Outputs

- Per-cluster cell type annotations (accordion_per_cluster) with top impacting marker genes
- Per-cell cell cycle phase assignments (G0, G1, S, G2M)
- Per-cell innate immune response scores and top 5 marker genes per condition and cell type (Seurat misc slot)
- Comparison statistics and plots: condition-split feature plots, dot plots of gene impact scores, lineage and G0 fraction comparisons

## How to apply

Load the processed mouse bone marrow Seurat objects (Mettl3 KO/WT and STM2457/vehicle) together with the built-in innate immune response signature table (in_im_resp_sig) from the cellmarkeraccordion package. Annotate populations with accordion() using species="Mouse", tissue="bone marrow", annotation_resolution="cluster", max_n_marker=30, allow_unknown=FALSE, then classify G0/G1/S/G2M phases with accordion_cellcycle(). Compute per-cell innate immune response scores with accordion_custom() setting category_column="terms", marker_column="Symbol", annotation_resolution="cell", max_n_marker=NULL, and condition_group_info="condition" plus celltype_group_info="accordion_per_cluster" to obtain condition- and cell-type-specific top genes. Extract per-cell scores and top-5 marker genes from the Seurat misc slot, then compare score distributions, cell cycle fractions, and lineage fractions between conditions; consistency-weighted markers (specificity and evidence consistency scores) underpin the annotations, so the same weighting drives both annotation and signature scoring.

## Related tools

- **Cell Marker Accordion** (R package providing accordion(), accordion_cellcycle() and accordion_custom() for cell type annotation, cell cycle classification, and custom signature scoring of bone marrow single-cell data) — https://github.com/TebaldiLab/cellmarkeraccordion
- **Seurat** (Holds the processed bone marrow single-cell objects; annotation results, cell cycle assignments and per-cell scores are stored in and retrieved from the Seurat object (meta.data and misc slots))

## Examples

```
data <- accordion(data, assay="RNA", species="Mouse", tissue="bone marrow", annotation_resolution="cluster", max_n_marker=30, allow_unknown=FALSE)
```

## Evaluation signals

- Innate immune response score shows a statistically significant increase in both Mettl3 KO and STM2457 conditions relative to controls, matching Fig. 8C,D,G,H,I
- Lineage shifts reproduce the reported pattern: increased stem cell and megakaryocyte fractions with decreased erythroid fractions upon Mettl3 modulation
- G0 cell cycle fraction is higher in the Mettl3-modulated condition than in control
- Annotated populations are consistency-weighted labels from accordion() with allow_unknown=FALSE and no unexpected unknown clusters
- Per-cell scores and top-5 marker genes per condition/cell type are recoverable from the Seurat misc slot and match the condition grouping in meta.data

## Limitations

- The Accordion database is limited to tissues and species covered by CellMarker2.0/PanglaoDB-derived markers (15479 marker genes, 728 cell types in 212 human tissues); check list_tissues() for mouse bone marrow availability
- Direct comparison with annotation tools not relying on marker genes was not conducted in the study
- Findings in the article derive from single-cell data of a small patient cohort / specific perturbation models; lineage-shift conclusions may not generalize to other hematopoietic perturbations

## Evidence

- [discussion] Cell Marker Accordion identified cell cycle changes and immune response pathway activation upon METTL3 modulation: "The Cell Marker Accordion identified cell cycle changes and activation of immune response pathways in specific hematopoietic cell types, consistent with the formation of aberrant endogenous dsRNAs"
- [methods] accordion() annotates clusters using the built-in database: "Users can annotate clusters or cells exploiting the built-in Cell Marker Accordion database with the accordion() function"
- [readme] Example accordion() invocation with species, tissue, resolution and max_n_marker parameters: "data <- accordion(data, assay ="RNA", species ="Human", tissue="blood", annotation_resolution = "cluster", max_n_marker = 30, include_detailed_annotation_info = TRUE, plot = TRUE)"
- [methods] Markers weighted by specificity and evidence consistency scores: "the Cell Marker Accordion database describes each marker with a specificity score (SPs) and an evidence consistency score (ECs)"
