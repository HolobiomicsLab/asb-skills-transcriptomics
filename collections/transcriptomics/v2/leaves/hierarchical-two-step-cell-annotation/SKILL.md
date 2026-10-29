---
name: hierarchical-two-step-cell-annotation
description: Use when annotating unlabeled cancer scRNA-seq cells against the tumor
  microenvironment reference (113,976 cells, 13 cancer types) — e.g., MCC/CRC/LC datasets
  — and when major-type and subtype labels are both needed.
when_to_use_negative:
- Tissue-specific cell types (e.g., alveolar cells in lung cancers) are expected —
  scWizard may assign them to the Unknown cluster; a tissue-specific annotation method
  is preferable.
- Input data has not been quality-controlled — scWizard's default QC (<200 genes,
  >10,000 genes, >5% mitochondrial reads) should be applied or data checked with 'view
  quality' before annotation.
- The analysis goal is clustering or differential expression alone rather than cell-type
  classification — the Seurat-based Clustering module or 11 downstream analyses apply
  instead.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scWizard
  role: 'Packages the two-step hierarchical annotation: step 1 (PCA + DNN with ReLU/softmax
    for major cell types) and step 2 (Seurat subclustering + DNN majority-vote for
    subtypes); run via GUI after installing Python and the pretrained trainset.'
  repo: https://github.com/Dulab2020/scWizard
- name: Seurat
  role: Used inside the second annotation step to subcluster each major cell type
    at a specific resolution before DNN subtype prediction with the majority-vote
    rule.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:576d0650c31e5329d92d922b765e3b166fbd3a058ac9eb2883b4748da1bed3ad
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Dulab2020/scWizard
  repo_url: https://github.com/Dulab2020/scWizard
  tools:
  - Seurat
  - scWizard
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# hierarchical-two-step-cell-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Two-step hierarchical cell-type annotation from scWizard: a deep neural network (ReLU/softmax, PCA on combined training+prediction data) first assigns each cell to one of 7 major tumor-microenvironment cell types, then Seurat subclustering plus a second DNN with majority-vote assignment refines cells into 47 subtypes.

## When to use

Use when annotating unlabeled cancer scRNA-seq cells against the tumor microenvironment reference (113,976 cells, 13 cancer types) — e.g., MCC/CRC/LC datasets — and when major-type and subtype labels are both needed.

## When NOT to use

- Tissue-specific cell types (e.g., alveolar cells in lung cancers) are expected — scWizard may assign them to the Unknown cluster; a tissue-specific annotation method is preferable.
- Input data has not been quality-controlled — scWizard's default QC (<200 genes, >10,000 genes, >5% mitochondrial reads) should be applied or data checked with 'view quality' before annotation.
- The analysis goal is clustering or differential expression alone rather than cell-type classification — the Seurat-based Clustering module or 11 downstream analyses apply instead.

## Inputs

- scRNA-seq counts matrix (csv, rds, or h5 format; genes as rows, cells as columns)
- Seurat object ('Data-rds')
- 10X output folder ('Data-10X')
- Pretrained trainset in H5 format from the Baidu link (113,976 cells, 7 major types, 47 subtypes, 13 cancer types)
- Optional custom trainset + label pair (H5, rows = genes, columns = cells)

## Outputs

- Major cell-type label for every cell (7 TME major types)
- Cell subtype label for every cell (47 subtypes) via subcluster annotation
- Per-cluster annotation accuracies (consistent/inconsistent/unknown labels), e.g. annotation_accuracy.csv

## How to apply

Install scWizard from GitHub and click 'install Python'; download the pretrained trainset from the Baidu link (extraction code 0ccq) and copy it to the 'app/www/python' folder. Run the first step on the raw counts (csv/rds/h5, Seurat object, or 10X folder): PCA on the combined training+prediction data followed by a DNN using ReLU and softmax activation with max-likelihood assignment per cell, producing major cell-type labels. For subtype annotation, run the second step: Seurat subclustering of each major cell type at a chosen resolution, then DNN prediction with a majority-vote subcluster rule (adjustable hidden-layer nodes, or use defaults). Optionally supply a custom trainset as H5 format (rows = genes, columns = cells) via 'Choose custom trainset'. Validate by labeling each predicted cell as 'consistent', 'inconsistent', or 'unknown' versus reference annotations and computing per-cluster accuracy as consistent cells over total cells in each cluster.

## Related tools

- **scWizard** (Packages the two-step hierarchical annotation: step 1 (PCA + DNN with ReLU/softmax for major cell types) and step 2 (Seurat subclustering + DNN majority-vote for subtypes); run via GUI after installing Python and the pretrained trainset.) — https://github.com/Dulab2020/scWizard
- **Seurat** (Used inside the second annotation step to subcluster each major cell type at a specific resolution before DNN subtype prediction with the majority-vote rule.)

## Examples

```
devtools::install_github("Dulab2020/scWizard"); scWizard::run_app()
```

## Evaluation signals

- Independent MCC dataset (11,024 labeled cells): major cell-type accuracy ≈ 0.98.
- CRC myeloid dataset (11,717 cells): subtype accuracy ≈ 0.85.
- LC T cell dataset (45,555 cells): T/NK subtype accuracy ≈ 0.79; comparison datasets should reach >0.66 (LYM), 0.71 (T/NK_BRCA), 0.66 (T/NK_KIRC), 0.64 (Mye_PAAD), tolerance within a few percent.
- Every predicted cell can be labeled 'consistent', 'inconsistent', or 'unknown' against the original study annotations; per-cluster accuracy = consistent cells / total cells in each cluster.
- Major-type assignments should use max-likelihood output from the ReLU/softmax DNN; subtype assignments should follow the majority-vote subcluster rule after Seurat subclustering.

## Limitations

- Tissue-specific cell types such as alveolar cells in lung cancers may not be identified and are named in the Unknown cluster.
- Identification of tissue-specific subtypes and cellular states is deferred to future updates.
- Cells are classified directly by the hierarchical model only for the 7 major types and 47 subtypes covered by the built-in reference (113,976 cells, 13 cancer types); custom trainsets must be H5 with rows = genes and columns = cells.
- Before annotation, the Python environment must be installed via the 'install Python' button and the trainset manually copied to the app/www/python folder, or the function will not run.

## Evidence
