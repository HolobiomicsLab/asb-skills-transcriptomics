---
name: uberon-tissue-standardization
description: Use when integrating marker databases or cell-sorting marker collections
  whose tissue annotations use non-standard nomenclature and need to be mapped to
  a common anatomy ontology before cross-source aggregation.
when_to_use_negative:
- Input tissue labels are already standardized Uberon terms and no cross-source integration
  is needed
- The task is cell-type annotation of a count matrix with the built-in accordion()
  database rather than building or re-mapping the database itself
- The organism/tissue of interest is not covered by the Uberon release used by the
  Cell Marker Accordion (212 human tissues)
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3308
tools:
- name: cellmarkeraccordion
  role: Provides the built-in cell_onto (Cell Ontology) and uberon_onto (Uberon) resources
    and the standardized accordion_marker database the mapping feeds into
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: ontologyIndex
  role: R package used for ontology tree lookups when mapping labels to Cell Ontology
    and Uberon terms
- name: Uberon
  role: Target anatomy ontology (release 2024-09-03) onto which tissue labels are
    standardized
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:f18f5dea713628c3a8ac903f760d2a68ce605584134b06fffb030955e9f7e956
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - R
  - Uberon
  - cellmarkeraccordion
  - ontologyIndex
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# uberon-tissue-standardization

## Summary

Standardizes heterogeneous tissue labels from marker gene sources onto Uberon anatomy ontology terms (release 2024-09-03) so that markers from different databases can be integrated consistently by tissue in the Cell Marker Accordion database.

## When to use

Use when integrating marker databases or cell-sorting marker collections whose tissue annotations use non-standard nomenclature and need to be mapped to a common anatomy ontology before cross-source aggregation.

## When NOT to use

- Input tissue labels are already standardized Uberon terms and no cross-source integration is needed
- The task is cell-type annotation of a count matrix with the built-in accordion() database rather than building or re-mapping the database itself
- The organism/tissue of interest is not covered by the Uberon release used by the Cell Marker Accordion (212 human tissues)

## Inputs

- Marker gene tables from published databases (CellMarker2.0, PanglaoDB) and cell-sorting marker collections with free-text tissue and cell-type labels
- Uberon ontology (release 2024-09-03) as uberon_onto from the cellmarkeraccordion package
- Cell Ontology (release 2024-08-16) as cell_onto from the cellmarkeraccordion package
- ontologyIndex ontology objects for tree lookups

## Outputs

- Marker tables with tissue labels mapped to Uberon terms and cell-type labels mapped to Cell Ontology terms
- Tissue-standardized marker subsets compatible with accordion() (e.g. tissue='blood', species='Human')

## How to apply

Load the Uberon ontology (release 2024-09-03) and Cell Ontology (release 2024-08-16) resources (cell_onto, uberon_onto) shipped with the cellmarkeraccordion R package, and use the ontologyIndex package for tree lookups. Map each source database's free-text tissue labels to Uberon terms and cell-type labels to Cell Ontology terms, exploiting the ontology hierarchy (ancestors/descendants) to resolve naming variants and classification discrepancies. Apply the same standardization to gene names via HGNC (2024, human) and MGI (v. 6.24, mouse). Rationale: published marker databases are heterogeneous, contain different marker sets for the same cell type, and use non-standard nomenclature, so ontology-based harmonization is the prerequisite step before computing SPs/ECs consistency weights. The standardized tissue terms determine which tissue-specific marker subset (e.g. 'blood') is passed to the accordion() annotation function.

## Related tools

- **cellmarkeraccordion** (Provides the built-in cell_onto (Cell Ontology) and uberon_onto (Uberon) resources and the standardized accordion_marker database the mapping feeds into) — https://github.com/TebaldiLab/cellmarkeraccordion
- **ontologyIndex** (R package used for ontology tree lookups when mapping labels to Cell Ontology and Uberon terms)
- **Uberon** (Target anatomy ontology (release 2024-09-03) onto which tissue labels are standardized)

## Examples

```
library(cellmarkeraccordion); data(cell_onto); data(uberon_onto); library(ontologyIndex); tissue_uberon <- map_labels(tissue_labels, uberon_onto)
```

## Evaluation signals

- Every source tissue label resolves to exactly one Uberon term from release 2024-09-03, with no unmapped labels remaining
- Standardized cell-type labels resolve to Cell Ontology terms from release 2024-08-16
- The standardized database reproduces the reported scope: 15479 marker genes associated with 728 cell types in 212 human tissues
- list_tissues(species='Human') on the standardized database returns the expected tissue set and the requested tissue (e.g. 'blood') is present
- Downstream SPs values lie in [0,1] and ECs values fall in the reported 1-23 range, indicating the mapping did not fragment or duplicate marker-source entries

## Limitations

- Only the specific ontology releases bundled with the package (Cell Ontology 2024-08-16, Uberon 2024-09-03) are referenced; newer releases may shift term IDs
- Tissue coverage is bounded by the built-in database (212 human tissues); tissues absent from Uberon mappings used by the Accordion cannot be annotated via the built-in path
- Source databases are heterogeneous and may contain conflicting annotations for the same cell type, which the mapping alone does not resolve — conflicts are later handled by the ECs polarity rule

## Evidence

- [other] Standardize marker cell-type labels by mapping them to Cell Ontology terms (release 2024-08-16) and tissues to Uberon terms (release 2024-09-03), using ontologyIndex for tree lookups.: "mapping them to the Cell Ontology (release 2024-08-16)"
- [methods] The Uberon anatomy ontology provides the target terms for tissue label standardization.: "The Uber-anatomy ontology (Uberon, release 2024-09-03)"
- [intro] Heterogeneous source databases with non-standard nomenclature motivate ontology-based harmonization.: "currently available gene marker databases are extremely heterogeneous, contain different marker sets for the same cell type, and employ a non-standard nomenclature and classification"
- [methods] Gene names are standardized using HGNC and MGI resources alongside the ontology mapping.: "HUGO Gene Nomenclature Committee (2024) and Mouse Genome Informatics (v. 6.24) resources were employed to standardize human and mouse gene names"
- [readme] The Shiny app is the recommended interface for filtering and exploring the standardized database by tissue.: "To explore, filter and download the Cell Marker database we recommend using our Shiny app"
