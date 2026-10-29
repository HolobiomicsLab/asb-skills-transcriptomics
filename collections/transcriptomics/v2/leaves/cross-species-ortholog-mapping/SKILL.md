---
name: cross-species-ortholog-mapping
description: Use when the query scRNA-seq/snRNA-seq dataset is from a different species
  than the available reference cell map (e.g., wheat query vs. rice reference) and
  reference-based annotation is required.
when_to_use_negative:
- A same-species reference cell map is available, making within-species annotation
  more appropriate
- Only marker-gene databases are available and no single-cell reference exists for
  label transfer
- The query and reference gene identifiers cannot be reconciled through orthology,
  leaving incompatible feature spaces
edam_operation: http://edamontology.org/operation_0326
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0797
tools:
- name: scPlant
  role: Provides AutoAnnotate_SingleR for cross-species reference-based label transfer
    and crossSpecies_integrate for multi-species data integration
  repo: https://github.com/compbioNJU/scPlant
- name: SingleR
  role: Underlying reference-based label transfer method invoked through scPlant::AutoAnnotate_SingleR
    with ref_type = "single-cell"
- name: Seurat
  role: Seurat::as.SingleCellExperiment converts the reference Seurat object into
    the format required by SingleR
- name: CellFunTopic
  role: Alternative cross-species annotation-transfer tool enabling automated cell
    type annotation transfer across datasets and species
  repo: https://github.com/compbioNJU/CellFunTopic
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:a4279d7561eda9c7dcda4deb72cadd3da89d0ac8e8a004dbac9fcbd089c5e2bd
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
  edam_operation: http://edamontology.org/operation_0326
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# cross-species-ortholog-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps query genes to orthologs in a reference species so that reference-based cell-type annotation (e.g., SingleR label transfer via scPlant) can run across species, as demonstrated by annotating hexaploid wheat root cells using a rice reference cell map.

## When to use

Use when the query scRNA-seq/snRNA-seq dataset is from a different species than the available reference cell map (e.g., wheat query vs. rice reference) and reference-based annotation is required.

## When NOT to use

- A same-species reference cell map is available, making within-species annotation more appropriate
- Only marker-gene databases are available and no single-cell reference exists for label transfer
- The query and reference gene identifiers cannot be reconciled through orthology, leaving incompatible feature spaces

## Inputs

- Pre-processed query Seurat object (e.g., CRA008788 wheat root scRNA-seq)
- Reference scRNA-seq cell map Seurat/SingleCellExperiment object from a different species (e.g., PRJNA706435 rice root)
- Query-to-reference ortholog mapping (gene-to-gene ortholog table)

## Outputs

- Per-cell predicted labels stored in SeuratObj$predicted_label
- Agreement metrics versus manual ground-truth annotations (e.g., written to annotation_predictions.csv)

## How to apply

When no same-species reference exists, convert the reference Seurat object to a SingleCellExperiment with Seurat::as.SingleCellExperiment, then map the query genes (e.g., wheat from CRA008788) to reference orthologs (e.g., rice from PRJNA706435) so the gene spaces are compatible for label transfer. Run scPlant::AutoAnnotate_SingleR(SeuratObj, SCE_ref, ref_type = "single-cell"), which stores predictions in SeuratObj$predicted_label. This cross-species strategy was the one that produced highly accurate cell-type annotation for hexaploid wheat despite the reference being from a different species; scPlant additionally supports crossSpecies_integrate for combining datasets from multiple plant species (Arabidopsis, rice, maize).

## Related tools

- **scPlant** (Provides AutoAnnotate_SingleR for cross-species reference-based label transfer and crossSpecies_integrate for multi-species data integration) — https://github.com/compbioNJU/scPlant
- **SingleR** (Underlying reference-based label transfer method invoked through scPlant::AutoAnnotate_SingleR with ref_type = "single-cell")
- **Seurat** (Seurat::as.SingleCellExperiment converts the reference Seurat object into the format required by SingleR)
- **CellFunTopic** (Alternative cross-species annotation-transfer tool enabling automated cell type annotation transfer across datasets and species) — https://github.com/compbioNJU/CellFunTopic

## Examples

```
SeuratObj <- scPlant::AutoAnnotate_SingleR(SeuratObj, SCE_ref, ref_type = "single-cell")  # wheat query vs. rice ortholog-mapped reference
```

## Evaluation signals

- predicted_label values in SeuratObj$predicted_label reproduce the manual cell-type assignments reported in Figure 2D / Supplemental Figure 1 of the paper
- Per-cell predictions and agreement metrics versus ground truth are successfully written to annotation_predictions.csv
- Wheat query genes are fully mapped onto the rice reference ortholog space before SingleR label transfer, with no unanchored features blocking the comparison
- Cross-species wheat-vs-rice annotation yields agreement with manual annotations comparable to the paper's reported highly accurate result for hexaploid wheat

## Limitations

- Accuracy depends on the quality/completeness of the ortholog mapping between query and reference species
- Cross-species annotation via SingleR requires a single-cell reference from another species, so results degrade when orthologs or cell types are poorly conserved
- The paper's validation is specific to the wheat-vs-rice and Arabidopsis-root comparisons against manual Figure 2D / Supplemental Figure 1 annotations; other species pairs are untested here

## Evidence

- [other] promising predictions, including highly accurate cell-type annotation for hexaploid wheat using references from a different species: [evidence span withheld — anchor status: not_found]
- [other] map query genes to reference orthologs for the wheat-vs-rice comparison: [evidence span withheld — anchor status: not_found]
- [other] Run reference-based label transfer with scPlant::AutoAnnotate_SingleR(SeuratObj, SCE_ref, ref_type = "single-cell"): [evidence span withheld — anchor status: not_found]
- [readme] CellFunTopic enables rapid, automated, effective cell type annotation transfer across datasets and species: [evidence span withheld — anchor status: external_doc]
