---
name: gene-symbol-standardization
description: Use when combining marker gene sets from multiple published marker databases
  or cell-sorting collections and you need consistent gene nomenclature and cell type
  labels before computing cross-source overlaps or integrating markers.
when_to_use_negative:
- The data is a single-database marker list that will not be compared or merged with
  other sources — standardization adds overhead without benefit
- Cell type annotations are already ontology-mapped (e.g., Cell Ontology terms from
  the Cell Marker Accordion Shiny app) and symbols are already canonical HGNC/MGI
- The task is single-cell clustering or annotation itself rather than harmonizing
  reference marker sets
tools:
- name: Cell Marker Accordion
  role: Platform whose database construction performs filtering, standardization,
    and integration of multiple marker databases into a consistency-weighted marker
    resource
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: R
  role: Implementation language for the standardization and Jaccard overlap workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:cb69f05f0757991fbfc9a896ee35d102ede4a4e8f7cf2090b006a889d9b79b1e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - Cell Marker Accordion
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# gene-symbol-standardization

## Summary

Harmonizes marker gene lists from heterogeneous cell-marker databases (e.g., CellMarker2.0, PanglaoDB) by standardizing gene symbols to HGNC (2024) for human and Mouse Genome Informatics (v. 6.24) for mouse, and mapping cell type labels to the Cell Ontology (release 2024-08-16), enabling fair cross-database comparisons such as Jaccard overlap.

## When to use

Use when combining marker gene sets from multiple published marker databases or cell-sorting collections and you need consistent gene nomenclature and cell type labels before computing cross-source overlaps or integrating markers.

## When NOT to use

- The data is a single-database marker list that will not be compared or merged with other sources — standardization adds overhead without benefit
- Cell type annotations are already ontology-mapped (e.g., Cell Ontology terms from the Cell Marker Accordion Shiny app) and symbols are already canonical HGNC/MGI
- The task is single-cell clustering or annotation itself rather than harmonizing reference marker sets

## Inputs

- Marker gene lists exported from published marker databases (e.g., CellMarker2.0, PanglaoDB) and cell-sorting marker collections
- Original cell type label strings as used by each source database
- HGNC gene nomenclature files (2024) for human symbols
- Mouse Genome Informatics (v. 6.24) nomenclature resources for mouse symbols
- Cell Ontology (release 2024-08-16) for cell type label mapping

## Outputs

- Standardized marker gene tables with canonical HGNC/MGI symbols per cell type
- Cell type labels mapped to Cell Ontology terms for cross-database matching
- Pairwise Jaccard similarity values between database pairs (e.g., jaccard_similarity_matrix.csv)

## How to apply

Download marker gene sets from each source (in the Cell Marker Accordion work, CellMarker2.0, PanglaoDB and seven published marker/cell-sorting collections). Standardize human gene symbols against the HUGO Gene Nomenclature Committee (2024) resource and mouse symbols against Mouse Genome Informatics (v. 6.24), so that synonyms and aliases collapse to one canonical symbol per gene. Map each database's original cell type labels to the Cell Ontology (release 2024-08-16) so that the same biological cell type is identified identically across sources. Only after this dual standardization (genes + cell types) restrict to cell types common between database pairs and compute Jaccard similarity — intersection over union of marker genes — since unstandardized symbols or labels would inflate apparent database disagreement. This procedure underlies the Cell Marker Accordion database, which integrates 15,479 marker genes across 728 cell types in 212 human tissues.

## Related tools

- **Cell Marker Accordion** (Platform whose database construction performs filtering, standardization, and integration of multiple marker databases into a consistency-weighted marker resource) — https://github.com/TebaldiLab/cellmarkeraccordion
- **R** (Implementation language for the standardization and Jaccard overlap workflow)

## Evaluation signals

- Every output gene symbol resolves to a valid current HGNC (human) or MGI (mouse) identifier; no deprecated aliases remain
- Cell type labels are matched across databases only via Cell Ontology (release 2024-08-16) terms, not raw strings
- Reproducing the pairwise overlap after standardization yields max Jaccard 0.23 between CellMarker2.0 and PanglaoDB, and for the seven-database comparison an average Jaccard of 0.08 with a maximum of 0.13
- The standardized Cell Marker Accordion database contains 15,479 marker genes associated with 728 cell types in 212 human tissues

## Limitations

- Marker databases remain extremely heterogeneous with different marker sets for the same cell type and non-standard nomenclature; standardization reduces but does not eliminate disagreement (average pairwise Jaccard only 0.08 across seven databases)
- Mapping cell type labels depends on the chosen ontology release (2024-08-16); updates may change matches
- Only cell types common between a pair of databases can be compared, so overlap statistics are restricted to shared types

## Evidence

- [methods] HGNC and MGI standardization: "HUGO Gene Nomenclature Committee (2024) and Mouse Genome Informatics (v. 6.24) resources were employed to standardize human and mouse gene names"
- [methods] Cell Ontology mapping: "mapping them to the Cell Ontology (release 2024-08-16)"
- [methods] Jaccard computation: "we calculated the Jaccard pairwise similarity values, i.e. the ratio between the cardinality of the intersection and the cardinality of the union"
- [readme] Harmonization rationale: [evidence span withheld — anchor status: external_doc]
- [intro] Heterogeneity motivation: "currently available gene marker databases are extremely heterogeneous, contain different marker sets for the same cell type, and employ a non-standard nomenclature and classification"
