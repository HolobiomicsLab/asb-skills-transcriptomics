---
name: gene-signature-per-cell-scoring
description: Use when you have an annotated Seurat object with condition metadata
  and want per-cell scores and top genes for a pathway signature (e.g., innate immune
  response) across conditions and cell types.
when_to_use_negative:
- The input is already a summarized per-cluster or bulk feature table rather than
  per-cell single-cell data
- No condition/cell-type grouping metadata is available, since accordion_custom requires
  condition_group_info and celltype_group_info for condition- and cell-type-specific
  scoring
- The goal is cell type annotation itself rather than signature scoring — use accordion()
  or accordion_cellcycle() instead
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Cell Marker Accordion
  role: R package providing accordion_custom() for per-cell gene signature scoring
    and accordion() for the prerequisite cell type annotations
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: Seurat
  role: Holds the processed single-cell data (assay, meta.data with condition and
    accordion_per_cluster columns) and the misc slot where scores and top genes are
    returned
- name: Shiny
  role: Interactive web app to explore, filter and download the Cell Marker Accordion
    marker database used for annotation and signatures
  repo: https://rdds.it/CellMarkerAccordion/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:20810b750a95c4b5c8bc685c89c9a74e5682b13605ffa11c87f5462b4c85ad9f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - Cell Marker Accordion
  - R
  - Seurat
  - Shiny
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# gene-signature-per-cell-scoring

## Summary

Scores per-cell activity of a gene signature (e.g., innate immune response) in single-cell data using the Cell Marker Accordion's accordion_custom() on Seurat objects, enabling condition- and cell-type-specific pathway activation comparisons.

## When to use

Use when you have an annotated Seurat object with condition metadata and want per-cell scores and top genes for a pathway signature (e.g., innate immune response) across conditions and cell types.

## When NOT to use

- The input is already a summarized per-cluster or bulk feature table rather than per-cell single-cell data
- No condition/cell-type grouping metadata is available, since accordion_custom requires condition_group_info and celltype_group_info for condition- and cell-type-specific scoring
- The goal is cell type annotation itself rather than signature scoring — use accordion() or accordion_cellcycle() instead

## Inputs

- Processed Seurat object with count/normalized data and a condition column in meta.data
- Cell type annotations (e.g., accordion_per_cluster labels from accordion())
- Gene signature table with a category/terms column and a Symbol/gene column (e.g., built-in in_im_resp_sig)

## Outputs

- Per-cell signature scores stored in the Seurat misc slot
- Top 5 marker genes per condition and cell type (Seurat misc slot)
- Condition-split feature plots and dot plots of gene impact scores comparing WT vs Mettl3 KO and Vehicle vs STM2457

## How to apply

Load the processed Seurat objects (e.g., murine bone marrow from Mettl3 KO/WT and STM2457/vehicle) and the signature table — the cellmarkeraccordion package ships an innate immune response signature (in_im_resp_sig) with 'terms' and 'Symbol' columns. Annotate cell populations first with accordion() (species='Mouse', tissue='bone marrow', annotation_resolution='cluster', max_n_marker=30, allow_unknown=FALSE) so that accordion_per_cluster labels can drive grouping. Run accordion_custom() with category_column='terms', marker_column='Symbol', annotation_resolution='cell', max_n_marker=NULL, passing condition_group_info='condition' and celltype_group_info='accordion_per_cluster' so scores and top genes are computed per condition and cell type. Extract per-cell scores and top 5 marker genes per condition/cell type from the Seurat misc slot, then compare score distributions between conditions (WT vs KO, Vehicle vs STM2457). The rationale is that consistency-weighted markers identify disease-critical cells and pathological processes, here linking METTL3 modulation to aberrant endogenous dsRNA-driven innate immune activation.

## Related tools

- **Cell Marker Accordion** (R package providing accordion_custom() for per-cell gene signature scoring and accordion() for the prerequisite cell type annotations) — https://github.com/TebaldiLab/cellmarkeraccordion
- **Seurat** (Holds the processed single-cell data (assay, meta.data with condition and accordion_per_cluster columns) and the misc slot where scores and top genes are returned)
- **Shiny** (Interactive web app to explore, filter and download the Cell Marker Accordion marker database used for annotation and signatures) — https://rdds.it/CellMarkerAccordion/

## Examples

```
data <- accordion_custom(data, category_column="terms", marker_column="Symbol", annotation_resolution="cell", max_n_marker=NULL, condition_group_info="condition", celltype_group_info="accordion_per_cluster")
```

## Evaluation signals

- Per-cell scores for the innate immune response signature are present in the Seurat misc slot for all cells
- Top 5 marker genes per condition and per cell type are extracted as expected from the misc slot
- Score distributions show a statistically significant increase in the innate immune response score in both Mettl3 KO and STM2457 conditions versus controls, matching Fig. 8C,D,G,H,I
- Cell cycle assignments from accordion_cellcycle() show an increased G0 fraction upon Mettl3 modulation, consistent with the reported lineage shifts (increased stem cell and megakaryocyte fractions, decreased erythroid fractions)
- Annotation labels from accordion() are consistent (allow_unknown=FALSE forces every cluster to a valid bone marrow cell type)

## Limitations

- Annotation performance depends on the built-in Cell Marker Accordion database; marker databases are heterogeneous and databases employ non-standard nomenclature, so signature definitions may differ across sources
- Validation of the tool was limited to specific benchmark contexts (e.g., two liquid and two solid tumors in the cancer benchmark), so generalization to other disease contexts may require re-validation
- MDS single-cell evidence came from a small patient cohort, suggesting caution when generalizing disease conclusions from small cohorts

## Evidence

- [discussion] The Cell Marker Accordion identified cell cycle changes and activation of immune response pathways in specific hematopoietic cell types: "The Cell Marker Accordion identified cell cycle changes and activation of immune response pathways in specific hematopoietic cell types, consistent with the formation of aberrant endogenous dsRNAs"
- [methods] Users can annotate clusters or cells exploiting the built-in Cell Marker Accordion database with the accordion() function: "Users can annotate clusters or cells exploiting the built-in Cell Marker Accordion database with the accordion() function"
- [readme] cellmarkeraccordion allows to automatically identifies cell populations in multiple tissues in single-cell dataset by running function accordion: "<strong>cellmarkeraccordion</strong> allows to automatically identifies cell populations in multiple tissues in single-cell dataset by running function  accordion "
