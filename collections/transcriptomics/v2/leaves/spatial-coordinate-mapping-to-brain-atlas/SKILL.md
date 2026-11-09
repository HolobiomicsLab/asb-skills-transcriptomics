---
name: spatial-coordinate-mapping-to-brain-atlas
description: Use when an imaging-based spatial dataset (e.g., MERFISH mouse brain
  sections with per-cell x/y coordinates) needs anatomical registration to a reference
  atlas (Allen CCF 2020) to validate cell type annotations spatially.
when_to_use_negative:
- Input is non-spatial scRNA-seq data with no per-cell coordinates — no atlas registration
  is possible
- Tissue is not brain or the coordinates are not in the coordinate space of Allen
  CCF 2020 NIfTI reference images
- The dataset already carries authoritative anatomical region labels, making re-registration
  redundant
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Standard pre-processing pipeline for the imaging-based MERFISH spatial dataset
    before atlas mapping and annotation
- name: ontologySimilarity
  role: Scores predicted vs. atlas-ground-truth annotations as average percentage
    similarity over the Cell Ontology tree via get_sim_grid (v2.7)
- name: Cell Marker Accordion
  role: Produces the predicted cell type annotations that are spatially mapped onto
    the atlas-registered section
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:4b0531a7aa82a77763c0731af4872cfdf6200d38b047099c13b39cd57755f37b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - Cell Marker Accordion
  - SCINA
  - ScType
  - Seurat
  - clustifyR
  - ontologySimilarity
  - scCATCH
  - scSorter
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# spatial-coordinate-mapping-to-brain-atlas

## Summary

Registers imaging-based spatial transcriptomics cell coordinates (e.g., MERFISH coronal brain sections) onto the Allen CCF 2020 brain atlas via NIfTI images, providing anatomical ground truth against which predicted cell type annotations are spatially validated.

## When to use

Use when an imaging-based spatial dataset (e.g., MERFISH mouse brain sections with per-cell x/y coordinates) needs anatomical registration to a reference atlas (Allen CCF 2020) to validate cell type annotations spatially.

## When NOT to use

- Input is non-spatial scRNA-seq data with no per-cell coordinates — no atlas registration is possible
- Tissue is not brain or the coordinates are not in the coordinate space of Allen CCF 2020 NIfTI reference images
- The dataset already carries authoritative anatomical region labels, making re-registration redundant

## Inputs

- MERFISH mouse brain spatial dataset with per-cell spatial coordinates (coronal section Zhuang-ABCA-1.080, 37,068 barcodes)
- Allen CCF 2020 atlas reference images in NIfTI format with 0.01 mm voxel spacing
- Ground-truth per-cell cell type annotation for the section

## Outputs

- Per-barcode anatomical region labels from the Allen CCF 2020 atlas
- Per-tool performance comparison table (merfish_annotation_performance.csv)
- Spatial plot of winning cell type annotations on tissue coordinates (merfish_spatial_annotation.png)

## How to apply

Preprocess the imaging-based spatial dataset with a standard Seurat pipeline, then select the target coronal brain section — in the benchmark, section ID 'Zhuang-ABCA-1.080' with 37,068 barcodes. Map each cell's spatial coordinates to the Allen CCF 2020 using NIfTI reference images with 0.01 mm voxel spacing, assigning each barcode an anatomical region label. Use the atlas-mapped ground truth annotation as the reference for scoring tools: compute average percentage similarity between predicted and original cell types via the Cell Ontology tree with the get_sim_grid function of the ontologySimilarity R package (v2.7). Finally, export the per-tool performance table (e.g., merfish_annotation_performance.csv) and render the winning annotations on the tissue spatial coordinates (e.g., merfish_spatial_annotation.png). Rationale: high similarity between the tool's spatial mapping and the original annotation demonstrates that marker-based annotation preserves spatial anatomy.

## Related tools

- **Seurat** (Standard pre-processing pipeline for the imaging-based MERFISH spatial dataset before atlas mapping and annotation)
- **ontologySimilarity** (Scores predicted vs. atlas-ground-truth annotations as average percentage similarity over the Cell Ontology tree via get_sim_grid (v2.7))
- **Cell Marker Accordion** (Produces the predicted cell type annotations that are spatially mapped onto the atlas-registered section) — https://github.com/TebaldiLab/cellmarkeraccordion

## Evaluation signals

- All 37,068 barcodes of section Zhuang-ABCA-1.080 receive a non-null Allen CCF 2020 region label after NIfTI mapping
- NIfTI reference images are used at the stated 0.01 mm voxel spacing; misregistration would shift cells across region boundaries
- Predicted annotations show high spatial similarity with the original annotation — benchmark reports >13% improvement over other marker-based tools (ScType, SCINA, clustifyR, scCATCH, scSorter)
- Output files merfish_annotation_performance.csv and merfish_spatial_annotation.png exist and the CSV contains one score per evaluated tool

## Limitations

- High-quality ground-truth datasets with atlas annotations are scarce, constraining benchmark expansion
- No direct comparison is made with annotation tools that do not rely on marker genes
- Spatial similarity scoring depends on the Cell Ontology tree and get_sim_grid; semantically distant labels may be penalized or forgiven depending on ontology structure

## Evidence

- [methods] MERFISH Seurat preprocessing: "The MERFISH mouse brain dataset was analyzed employing a standard pre-processing pipeline for imaging-based spatial dataset, using Seurat"
- [other] Section selection and Allen CCF 2020 registration: [evidence span withheld — anchor status: not_found]
- [other] Similarity scoring via Cell Ontology: [evidence span withheld — anchor status: not_found]
- [results] Accordion mapping fidelity: [evidence span withheld — anchor status: not_found]
