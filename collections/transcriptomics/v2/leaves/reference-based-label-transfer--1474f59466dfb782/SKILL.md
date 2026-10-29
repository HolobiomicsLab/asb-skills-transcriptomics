---
name: reference-based-label-transfer
description: Use when a pre-processed query Seurat object (e.g., Arabidopsis root
  PRJNA507252 or wheat root CRA008788) has clustered cells and a curated reference
  cell map (e.g., PRJNA517021 or PRJNA706435) is available for label transfer.
when_to_use_negative:
- No curated single-cell reference cell map exists for the species or a sufficiently
  close relative, so only marker-database-based annotation is feasible
- Query data is bulk transcriptomics — use scPlant's deconvolution module (decomposeBulk)
  instead of label transfer
- Query clusters have not yet been defined (unclustered raw matrix) — run scPlant
  preprocessing (QCfun, RunSeurat) first
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0780
tools:
- name: scPlant
  role: Framework wrapping the annotation strategies; provides AutoAnnotate_SingleR
    for reference-based label transfer
  repo: github.com/compbioNJU/scPlant
- name: SingleR
  role: Underlying reference-based label transfer engine invoked by AutoAnnotate_SingleR
    with ref_type = 'single-cell'
- name: Seurat
  role: Holds query and reference objects; as.SingleCellExperiment converts references
    for SingleR input
- name: scCATCH
  role: Marker-based annotation alternative via AutoAnnotate_scCATCH using get_marker_scCATCH
    custom marker lists
- name: CellFunTopic
  role: Functional-signature-based annotation transfer via predictFun after RunGSEA_plant
    on the reference
  repo: github.com/compbioNJU/CellFunTopic
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:3a7c092feee3f7e357d6d40107cc4649688635f510711b835445eeb34439da03
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
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# reference-based-label-transfer

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automatic plant cell-type annotation by transferring labels from a curated reference scRNA-seq cell map (e.g., scPlant-DB Seurat/SCE objects) to a query dataset, including cross-species transfer such as hexaploid wheat root cells annotated against a rice reference.

## When to use

Use when a pre-processed query Seurat object (e.g., Arabidopsis root PRJNA507252 or wheat root CRA008788) has clustered cells and a curated reference cell map (e.g., PRJNA517021 or PRJNA706435) is available for label transfer.

## When NOT to use

- No curated single-cell reference cell map exists for the species or a sufficiently close relative, so only marker-database-based annotation is feasible
- Query data is bulk transcriptomics — use scPlant's deconvolution module (decomposeBulk) instead of label transfer
- Query clusters have not yet been defined (unclustered raw matrix) — run scPlant preprocessing (QCfun, RunSeurat) first

## Inputs

- Pre-processed query Seurat object (e.g., Arabidopsis root PRJNA507252 or wheat root CRA008788)
- Reference scRNA-seq cell map as Seurat object or SingleCellExperiment (e.g., PRJNA517021 or PRJNA706435 from scPlant-DB)
- Ortholog mapping table for cross-species gene mapping (e.g., wheat genes to rice orthologs)
- Optional marker data frame (two-column celltype, gene) for complementary marker-based annotation

## Outputs

- Per-cell predicted labels stored in SeuratObj$predicted_label
- Per-cell predictions and agreement metrics vs. manual annotations written to annotation_predictions.csv

## How to apply

Load the pre-processed query Seurat object and the reference scRNA-seq cell map, converting the reference with Seurat::as.SingleCellExperiment where needed. For cross-species comparisons (wheat query vs. rice reference), first map query genes to reference orthologs. Then run scPlant::AutoAnnotate_SingleR(SeuratObj, SCE_ref, ref_type = "single-cell"), which performs reference-based label transfer with SingleR and stores predictions in SeuratObj$predicted_label. Optionally complement with marker-based strategies (AutoAnnotate_Garnett, AutoAnnotate_scCATCH, AutoAnnotate_CellAssign, CellFunTopic::predictFun) and compare each tool's predicted labels against manual ground-truth annotations to assess agreement. Validation: predictions should reproduce the manual cell-type assignments reported in Figure 2D / Supplemental Figure 1 of the scPlant paper.

## Related tools

- **scPlant** (Framework wrapping the annotation strategies; provides AutoAnnotate_SingleR for reference-based label transfer) — github.com/compbioNJU/scPlant
- **SingleR** (Underlying reference-based label transfer engine invoked by AutoAnnotate_SingleR with ref_type = 'single-cell')
- **Seurat** (Holds query and reference objects; as.SingleCellExperiment converts references for SingleR input)
- **scCATCH** (Marker-based annotation alternative via AutoAnnotate_scCATCH using get_marker_scCATCH custom marker lists)
- **CellFunTopic** (Functional-signature-based annotation transfer via predictFun after RunGSEA_plant on the reference) — github.com/compbioNJU/CellFunTopic

## Examples

```
SeuratObj <- scPlant::AutoAnnotate_SingleR(SeuratObj, SCE_ref, ref_type = 'single-cell'); table(SeuratObj$predicted_label)
```

## Evaluation signals

- predicted_label column is populated for all cells in SeuratObj$predicted_label with valid cell-type names
- Per-cell agreement metrics vs. manual ground-truth annotations (Figure 2D / Supplemental Figure 1) are computed and written to annotation_predictions.csv
- For wheat-vs-rice transfer, query gene symbols map to reference orthologs with a non-trivial mapping rate before SingleR scoring
- Annotation accuracy is comparable to that reported for scPlant's diverse automatic annotation strategies (SingleR, scCATCH, CellAssign, Garnett, CellFunTopic)

## Limitations

- Reference-based transfer quality depends on reference coverage and orthology; cross-species transfer (wheat vs. rice) requires correct ortholog mapping of query genes
- Marker-based tools can fail on input format issues — check the marker file generated by scPlant::get_marker_file if Garnett errors
- scPlant's annotation depends on the accompanying plant knowledgebase (scPlant-DB); lack of a plant knowledgebase was a key gap motivating the framework

## Evidence

- [full_text] Workflow step for SingleR label transfer: [evidence span withheld — anchor status: not_found]
