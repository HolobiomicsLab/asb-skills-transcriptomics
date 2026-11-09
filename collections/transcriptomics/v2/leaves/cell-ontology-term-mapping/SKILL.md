---
name: cell-ontology-term-mapping
description: Use when marker cell-type and tissue labels from heterogeneous marker
  databases must be harmonized onto Cell Ontology and Uberon terms before computing
  consistency-weighted marker scores or running accordion() annotation.
when_to_use_negative:
- Input labels are already Cell Ontology / Uberon standardized and need no harmonization
  before scoring
- The analysis goal is non-marker-based annotation (e.g., reference-mapping or CNV-based
  tools like SCEVAN or CopyKAT) where ontology harmonization of marker labels is not
  the operative step
- Only a single, internally consistent marker source is used, so cross-source evidence
  consistency (ECs) cannot be computed
edam_topics:
- http://edamontology.org/topic_0089
tools:
- name: cellmarkeraccordion
  role: Provides accordion_marker, cell_onto (Cell Ontology) and uberon_onto (Uberon)
    resources and performs the standardized, integrated marker database construction
    and downstream accordion() annotation
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: ontologyIndex
  role: Used for tree lookups over the Cell Ontology and Uberon hierarchies when mapping
    cell-type and tissue labels to ontology terms
- name: R Shiny app
  role: Interactive browsing, filtering and downloading of the ontology-standardized
    Cell Marker Accordion database
  repo: https://github.com/TebaldiLab/shiny_cellmarkeraccordion
- name: Seurat
  role: Consumes the harmonized database for automatic annotation; ontology mapping
    ensures its cluster/cell annotations use standard cell-type nomenclature
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:b9385a7525307bc73a27f72f8e3a3ec84a00de170c3424aa59e5e2f4276a3f3e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - Cell Marker Accordion
  - R
  - R Shiny app
  - Seurat
  - cellmarkeraccordion
  - ontologyIndex
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cell-ontology-term-mapping

## Summary

Standardizes heterogeneous marker gene cell-type labels and tissue names from multiple marker databases (e.g., CellMarker2.0, PanglaoDB) onto Cell Ontology (release 2024-08-16) and Uberon (release 2024-09-03) terms so that the Cell Marker Accordion can compute consistent SPs/ECs marker weights.

## When to use

Use when marker cell-type and tissue labels from heterogeneous marker databases must be harmonized onto Cell Ontology and Uberon terms before computing consistency-weighted marker scores or running accordion() annotation.

## When NOT to use

- Input labels are already Cell Ontology / Uberon standardized and need no harmonization before scoring
- The analysis goal is non-marker-based annotation (e.g., reference-mapping or CNV-based tools like SCEVAN or CopyKAT) where ontology harmonization of marker labels is not the operative step
- Only a single, internally consistent marker source is used, so cross-source evidence consistency (ECs) cannot be computed

## Inputs

- accordion_marker / disease_accordion_marker data objects from the cellmarkeraccordion R package (marker, cell-type, tissue, source per row)
- cell_onto Cell Ontology (release 2024-08-16) resource object
- uberon_onto Uberon ontology (release 2024-09-03) resource object
- Raw marker tables from source databases such as CellMarker2.0 and PanglaoDB with non-standard cell-type and tissue labels
- Human gene symbols standardized against HUGO Gene Nomenclature Committee (2024); mouse symbols against Mouse Genome Informatics (v. 6.24)

## Outputs

- Harmonized marker database in which every marker entry carries a Cell Ontology cell-type term and an Uberon tissue term
- Ontology-consistent cell-type and tissue keys suitable for downstream SPs (specificity) and ECs (evidence consistency) scoring
- TheCellMarkerAccordion_database_v1.0.0.xlsx-style downloadable database view via the Shiny app

## How to apply

Load the built-in accordion_marker database, the cell_onto Cell Ontology and uberon_onto Uberon resources shipped with the cellmarkeraccordion R package, and use ontologyIndex for tree lookups over the ontology hierarchies. Map each marker gene's source cell-type label to a Cell Ontology term (release 2024-08-16) and each tissue name to an Uberon term (release 2024-09-03), collapsing synonymous source-specific labels into a single standard term so that evidence from different sources can be counted per cell type. Simultaneously standardize human and mouse gene symbols against HGNC (2024) and Mouse Genome Informatics (v. 6.24). Rationale: source databases use non-standard nomenclature, and only after label harmonization can ECs (number of sources defining gene G as marker of cell type CT) and SPs (1/number of cell types gene G marks) be computed consistently across the 15,479 markers covering 728 cell types in 212 human tissues. Use ontology tree relationships (ancestors/descendants in cell_onto and uberon_onto) to resolve near-miss or hierarchical label matches.

## Related tools

- **cellmarkeraccordion** (Provides accordion_marker, cell_onto (Cell Ontology) and uberon_onto (Uberon) resources and performs the standardized, integrated marker database construction and downstream accordion() annotation) — https://github.com/TebaldiLab/cellmarkeraccordion
- **ontologyIndex** (Used for tree lookups over the Cell Ontology and Uberon hierarchies when mapping cell-type and tissue labels to ontology terms)
- **R Shiny app** (Interactive browsing, filtering and downloading of the ontology-standardized Cell Marker Accordion database) — https://github.com/TebaldiLab/shiny_cellmarkeraccordion
- **Seurat** (Consumes the harmonized database for automatic annotation; ontology mapping ensures its cluster/cell annotations use standard cell-type nomenclature)

## Examples

```
library(cellmarkeraccordion); library(ontologyIndex); data(accordion_marker); data <- accordion(data, assay="RNA", species="Human", tissue="blood", annotation_resolution="cluster", max_n_marker=30)
```

## Evaluation signals

- Every marker entry resolves to exactly one Cell Ontology term and one Uberon term after mapping, with no unmapped source labels left in the integrated database
- Mapped terms derive from the stated ontology releases: Cell Ontology 2024-08-16 and Uberon 2024-09-03
- Gene symbols validate against HGNC (2024) for human and MGI (v. 6.24) for mouse
- After harmonization, SPs values fall in [0,1] and ECs values fall within the reported 1-23 range in the current database
- The resulting database reproduces the reported scope of 15,479 marker genes across 728 cell types and 212 human tissues

## Limitations

- Mapping depends on the fixed ontology releases (Cell Ontology 2024-08-16, Uberon 2024-09-03); newer or non-standard source labels may fail to resolve
- Only human and mouse marker databases (CellMarker2.0, PanglaoDB) and cell-sorting marker collections were harmonized in the published database
- Ontology-mapped labels alone do not resolve conflicts between sources; conflicting positive/negative evidence must still be reconciled at the ECs polarity step

## Evidence

- [other] Standardize marker cell-type labels via ontology mapping: [evidence span withheld — anchor status: not_found]
- [methods] SPs and ECs definition: "the Cell Marker Accordion database describes each marker with a specificity score (SPs) and an evidence consistency score (ECs)"
- [methods] Ontology releases used: "HUGO Gene Nomenclature Committee (2024) and Mouse Genome Informatics (v. 6.24) resources were employed to standardize human and mouse gene names"
- [intro] Motivation for harmonization: "currently available gene marker databases are extremely heterogeneous, contain different marker sets for the same cell type, and employ a non-standard nomenclature and classification"
- [readme] Package resources and dependencies: "Once installed, load `cellmarkeraccordion` along with `Seurat` and `data.table` packages required for this tutorial"
