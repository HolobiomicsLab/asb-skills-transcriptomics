---
name: single-cell-cell-type-annotation
description: Use when a pre-processed, clustered Seurat object from plant single-cell
  data needs cell-type labels without manual curation, especially when a reference
  atlas (e.g., PRJNA517021, PRJNA706435) or a marker gene data frame is available.
when_to_use_negative:
- No reference atlas and no curated marker gene list exist for the organism/tissue,
  leaving the automatic strategies without supervision
- The input is raw count data that has not been processed through the scPlant/Seurat
  QC-and-clustering pipeline (QCfun, RunSeurat), since the annotation functions expect
  a processed Seurat object
- The goal is bulk RNA-seq deconvolution rather than single-cell labeling — scPlant
  handles that via decomposeBulk, not these annotation wrappers
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scPlant
  role: End-to-end framework providing AutoAnnotate_* wrappers, get_marker_file, and
    get_marker_scCATCH for plant cell-type annotation
  repo: https://github.com/compbioNJU/scPlant
- name: SingleR
  role: Reference-based label transfer via AutoAnnotate_SingleR using a single-cell
    reference map
- name: Garnett
  role: Marker-file-based classification via AutoAnnotate_Garnett
- name: scCATCH
  role: Marker-database-based annotation via AutoAnnotate_scCATCH with a custom marker
    object
- name: CellAssign
  role: Marker-gene probabilistic annotation via AutoAnnotate_CellAssign using a two-column
    marker data frame
- name: CellFunTopic
  role: Functional-signature-based annotation transfer via predictFun after RunGSEA_plant
  repo: https://github.com/compbioNJU/CellFunTopic
- name: Seurat
  role: Provides the query/reference object format; references converted with Seurat::as.SingleCellExperiment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:6b14f1979e0362692b27a226fa815cbe36f96947efefd58c341899f5e88bc1f2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/compbioNJU/scPlant
  repo_url: https://github.com/compbioNJU/scPlant
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
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# single-cell-cell-type-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automated cell-type annotation for plant scRNA-seq/snRNA-seq datasets using scPlant's wrapper functions built on SingleR, Garnett, scCATCH, CellAssign, and CellFunTopic, with reference cell maps or curated marker gene databases as supervision. Enables label transfer across datasets and even across species (e.g., wheat annotated with rice references).

## When to use

Use when a pre-processed, clustered Seurat object from plant single-cell data needs cell-type labels without manual curation, especially when a reference atlas (e.g., PRJNA517021, PRJNA706435) or a marker gene data frame is available.

## When NOT to use

- No reference atlas and no curated marker gene list exist for the organism/tissue, leaving the automatic strategies without supervision
- The input is raw count data that has not been processed through the scPlant/Seurat QC-and-clustering pipeline (QCfun, RunSeurat), since the annotation functions expect a processed Seurat object
- The goal is bulk RNA-seq deconvolution rather than single-cell labeling — scPlant handles that via decomposeBulk, not these annotation wrappers

## Inputs

- Pre-processed, clustered query Seurat object (e.g., PRJNA507252 Arabidopsis root, CRA008788 wheat root)
- Reference scRNA-seq cell map as Seurat or SingleCellExperiment object (e.g., PRJNA517021, PRJNA706435)
- Curated marker gene data frame with columns (celltype, gene)
- Gene-to-ortholog mapping table for cross-species annotation (e.g., wheat query genes to rice reference orthologs)

## Outputs

- Per-cell predicted labels stored in SeuratObj$predicted_label
- annotation_predictions.csv with per-cell predictions from each tool and agreement metrics versus manual ground truth

## How to apply

Load the pre-processed query Seurat objects (e.g., PRJNA507252 Arabidopsis root, CRA008788 wheat root) and reference scRNA-seq cell maps, converting references with Seurat::as.SingleCellExperiment where needed; map query genes to reference orthologs for cross-species cases like wheat-vs-rice. For reference-based label transfer, run scPlant::AutoAnnotate_SingleR(SeuratObj, SCE_ref, ref_type = 'single-cell'), which stores predictions in SeuratObj$predicted_label. For marker-based annotation, choose among AutoAnnotate_Garnett (after generating a marker file via scPlant::get_marker_file(markerDF, SeuratObj, file = 'markers.txt'), checking its format if Garnett errors), AutoAnnotate_scCATCH (after building marker_custom via get_marker_scCATCH with species/tissue, e.g., 'Arabidopsis thaliana'/'root'), and AutoAnnotate_CellAssign with a two-column (celltype, gene) marker data frame. Optionally run CellFunTopic::predictFun (group_by = 'cell_type', cluster_by = 'seurat_clusters') after RunGSEA_plant on the reference to transfer labels by functional signatures. Finally, compare each tool's predicted_label values against manual ground-truth annotations (Figure 2D / Supplemental Figure 1) and write per-cell predictions and agreement metrics to annotation_predictions.csv.

## Related tools

- **scPlant** (End-to-end framework providing AutoAnnotate_* wrappers, get_marker_file, and get_marker_scCATCH for plant cell-type annotation) — https://github.com/compbioNJU/scPlant
- **SingleR** (Reference-based label transfer via AutoAnnotate_SingleR using a single-cell reference map)
- **Garnett** (Marker-file-based classification via AutoAnnotate_Garnett)
- **scCATCH** (Marker-database-based annotation via AutoAnnotate_scCATCH with a custom marker object)
- **CellAssign** (Marker-gene probabilistic annotation via AutoAnnotate_CellAssign using a two-column marker data frame)
- **CellFunTopic** (Functional-signature-based annotation transfer via predictFun after RunGSEA_plant) — https://github.com/compbioNJU/CellFunTopic
- **Seurat** (Provides the query/reference object format; references converted with Seurat::as.SingleCellExperiment)

## Examples

```
SeuratObj <- scPlant::AutoAnnotate_SingleR(SeuratObj, SCE_ref, ref_type = 'single-cell')
```

## Evaluation signals

- predicted_label values for both query datasets reproduce the manual cell-type assignments reported in Figure 2D / Supplemental Figure 1
- Per-cell predictions for every tested tool (SingleR, Garnett, scCATCH, CellAssign, CellFunTopic) are non-empty and cover the clustered populations
- For cross-species annotation, query genes are correctly mapped to reference orthologs before label transfer (wheat genes to rice orthologs)
- annotation_predictions.csv exists with per-cell rows and agreement metrics against ground truth
- Garnett marker file generated by get_marker_file passes format checks before classification; errors prompt re-inspection of markers.txt format

## Limitations

- Annotation quality depends on reference relevance: wheat (hexaploid) was annotated with references from a different species (rice), so ortholog mapping is required and ortholog-poor lineages may be mislabeled
- Garnett can error on poorly formatted marker files; the marker file format must be checked before running AutoAnnotate_Garnett
- Marker-based strategies (scCATCH, CellAssign, Garnett) require a curated species- and tissue-specific marker list (e.g., via get_marker_scCATCH(species = 'Arabidopsis thaliana', tissue = 'root'))

## Evidence

- [methods] SingleR-based reference label transfer stores predictions in metadata: [evidence span withheld — anchor status: not_found]
- [results] Cross-species annotation of hexaploid wheat with rice references: [evidence span withheld — anchor status: not_found]
- [readme] CellFunTopic transfers annotation across datasets and species via functional signatures: [evidence span withheld — anchor status: external_doc]
