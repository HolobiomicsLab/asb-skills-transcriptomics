---
name: jaccard-similarity-computation
description: Use when quantifying overlap between marker gene sets for the same cell
  types across multiple marker databases or cell-sorting marker collections to assess
  database consistency.
when_to_use_negative:
- When the goal is cell type annotation of single-cell data rather than quantifying
  marker database overlap — use the Cell Marker Accordion accordion() function instead
- When cell type labels between sources cannot be reconciled (e.g., no mapping to
  a shared ontology), since Jaccard on non-common cell types is meaningless
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0605
tools:
- name: Cell Marker Accordion
  role: Platform providing the harmonized marker database and R environment used for
    the database-overlap analysis
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: R
  role: Language in which the Jaccard similarity computations are implemented
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:74f8284c100b177157e927b27adca769f47fca94beb367ea87d8887d52d73421
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - Cell Marker Accordion
  - R
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# jaccard-similarity-computation

## Summary

Computes pairwise Jaccard similarity (intersection/union of marker gene sets) between cell-type marker databases such as CellMarker2.0 and PanglaoDB to quantify marker-set consistency, as done in the Cell Marker Accordion paper.

## When to use

Use when quantifying overlap between marker gene sets for the same cell types across multiple marker databases or cell-sorting marker collections to assess database consistency.

## When NOT to use

- When the goal is cell type annotation of single-cell data rather than quantifying marker database overlap — use the Cell Marker Accordion accordion() function instead
- When cell type labels between sources cannot be reconciled (e.g., no mapping to a shared ontology), since Jaccard on non-common cell types is meaningless

## Inputs

- Marker gene sets downloaded from CellMarker2.0 and PanglaoDB
- Seven published marker gene / cell-sorting marker collections
- HGNC (2024) human gene nomenclature for symbol standardization
- Mouse Genome Informatics (v. 6.24) for mouse gene symbol standardization
- Cell Ontology (release 2024-08-16) for mapping cell type labels

## Outputs

- Pairwise Jaccard similarity values for each database pair and shared cell type
- jaccard_similarity_matrix.csv summarizing pairwise overlaps
- Summary statistics: average and maximum Jaccard across database comparisons

## How to apply

Download marker gene sets from the sources under comparison (e.g., CellMarker2.0 and PanglaoDB, plus seven published marker/cell-sorting collections). Standardize gene symbols with HGNC (2024) for human and Mouse Genome Informatics (v. 6.24) for mouse markers, and map cell type labels across sources to the Cell Ontology (release 2024-08-16) to identify common cell types. For each database pair, restrict to shared cell types and compute the Jaccard pairwise similarity — the ratio between the cardinality of the intersection and the cardinality of the union of marker genes — then average over shared cell types. Assemble results into a pairwise matrix such as jaccard_similarity_matrix.csv. Validation: reproduce reported extremes of a maximum pairwise Jaccard of 0.23 between CellMarker2.0 and PanglaoDB, and across seven databases an average Jaccard of 0.08 with a maximum of 0.13.

## Related tools

- **Cell Marker Accordion** (Platform providing the harmonized marker database and R environment used for the database-overlap analysis) — https://github.com/TebaldiLab/cellmarkeraccordion
- **R** (Language in which the Jaccard similarity computations are implemented)

## Examples

```
jaccard <- length(intersect(markers_db1, markers_db2)) / length(union(markers_db1, markers_db2))
```

## Evaluation signals

- Maximum pairwise Jaccard between CellMarker2.0 and PanglaoDB equals 0.23
- Seven-database comparison yields an average Jaccard of 0.08 and a maximum of 0.13
- Each Jaccard value equals |intersection| / |union| for the restricted marker gene sets of a common cell type
- Gene symbols are fully standardized to HGNC (2024) for human and MGI (v. 6.24) for mouse before comparison
- jaccard_similarity_matrix.csv is complete with one entry per database pair over common cell types

## Limitations

- Marker gene databases are extremely heterogeneous, contain different marker sets for the same cell type, and employ non-standard nomenclature and classification, so results depend heavily on symbol standardization and label mapping
- Only cell types shared between a database pair can be compared; database-specific cell types are excluded from the average

## Evidence

- [methods] Overlap Among Published Marker Gene Databases: "we calculated the Jaccard pairwise similarity values, i.e. the ratio between the cardinality of the intersection and the cardinality of the union"
- [intro] Heterogeneity of marker databases: "currently available gene marker databases are extremely heterogeneous, contain different marker sets for the same cell type, and employ a non-standard nomenclature and classification"
- [methods] Cell ontology mapping: "mapping them to the Cell Ontology (release 2024-08-16)"
- [methods] Gene name standardization: "HUGO Gene Nomenclature Committee (2024) and Mouse Genome Informatics (v. 6.24) resources were employed to standardize human and mouse gene names"
- [readme] Accordion database scope: [evidence span withheld — anchor status: external_doc]
