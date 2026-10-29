---
name: cell-type-nomenclature-standardization
description: Use when combining marker gene sets from multiple published databases
  or cell-sorting sources whose gene symbols and cell type labels follow non-standard,
  inconsistent nomenclature, and you need harmonized, comparable sets.
when_to_use_negative:
- The input is already a single, internally consistent marker database that uses standard
  HGNC/MGI gene symbols and Cell Ontology terms — re-mapping adds no value.
- The analysis goal is automatic cell type annotation of a single-cell/spatial dataset
  with the Cell Marker Accordion algorithm (use the accordion() annotation workflow
  instead of cross-database harmonization).
- Cell type labels cannot be confidently mapped to Cell Ontology terms (e.g., purely
  novel or disease-specific aberrant states with no ontology correspondence).
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3365
tools:
- name: Cell Marker Accordion
  role: Platform whose harmonization framework performs the filtering, standardization,
    and integration of multiple marker databases; source of the standardized database
    and the overlap-analysis workflow
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: R
  role: Language used to standardize gene symbols, map cell type labels to Cell Ontology,
    and compute pairwise Jaccard similarity across databases
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:ee20cae417e4c84f2d2b2ef905600ee974c4210e88ea93e8cf33eb2891e9c470
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - Cell Marker Accordion
  - R
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3365
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 3
---

# cell-type-nomenclature-standardization

## Summary

Standardizes gene symbols and cell type labels across heterogeneous marker gene databases (e.g., CellMarker2.0, PanglaoDB, cell-sorting collections) using HGNC/MGI for genes and the Cell Ontology for cell types, enabling consistent downstream comparisons such as Jaccard overlap analyses.

## When to use

Use when combining marker gene sets from multiple published databases or cell-sorting sources whose gene symbols and cell type labels follow non-standard, inconsistent nomenclature, and you need harmonized, comparable sets.

## When NOT to use

- The input is already a single, internally consistent marker database that uses standard HGNC/MGI gene symbols and Cell Ontology terms — re-mapping adds no value.
- The analysis goal is automatic cell type annotation of a single-cell/spatial dataset with the Cell Marker Accordion algorithm (use the accordion() annotation workflow instead of cross-database harmonization).
- Cell type labels cannot be confidently mapped to Cell Ontology terms (e.g., purely novel or disease-specific aberrant states with no ontology correspondence).

## Inputs

- Marker gene lists per cell type from CellMarker2.0
- Marker gene lists per cell type from PanglaoDB
- Published marker/cell-sorting marker collections for human and mouse
- HGNC gene nomenclature resource (2024)
- Mouse Genome Informatics nomenclature resource (v. 6.24)
- Cell Ontology (release 2024-08-16)

## Outputs

- Standardized (HGNC/MGI-compliant) marker gene sets per cell type per source
- Cell type labels mapped to Cell Ontology terms, defining the set of common cell types between database pairs
- Pairwise overlap table of common cell types across database pairs (e.g., jaccard_similarity_matrix.csv)

## How to apply

Download the marker gene sets from each source (e.g., CellMarker2.0, PanglaoDB, and the cell-sorting marker collections used by the Cell Marker Accordion). Standardize human gene symbols against the HUGO Gene Nomenclature Committee (2024) and mouse markers against Mouse Genome Informatics (v. 6.24). Map each source's original cell type labels to the Cell Ontology (release 2024-08-16) so that 'common cell types' between database pairs can be identified unambiguously. Then, for any downstream comparison (e.g., the Jaccard pairwise similarity used in the article), restrict each database pair to shared cell types and compute intersection-over-union of the standardized marker sets. The rationale is that marker databases are heterogeneous and use non-standard nomenclature, so skipping this harmonization inflates or deflates overlap measures.

## Related tools

- **Cell Marker Accordion** (Platform whose harmonization framework performs the filtering, standardization, and integration of multiple marker databases; source of the standardized database and the overlap-analysis workflow) — https://github.com/TebaldiLab/cellmarkeraccordion
- **R** (Language used to standardize gene symbols, map cell type labels to Cell Ontology, and compute pairwise Jaccard similarity across databases)

## Examples

```
Rscript -e 'j <- length(intersect(genes_a, genes_b))/length(union(genes_a, genes_b)); mean_jaccard <- tapply(j, ct_pair, mean)'
```

## Evaluation signals

- All output gene symbols resolve against HGNC (2024) for human and MGI (v. 6.24) for mouse — no unstandardized aliases remain.
- Every cell type label across sources maps to a Cell Ontology (release 2024-08-16) term; only pairs sharing an ontology term are compared.
- Reproducing the article's benchmark: maximum pairwise Jaccard of 0.23 between CellMarker2.0 and PanglaoDB; across the seven-database comparison, average Jaccard 0.08 and maximum 0.13.
- Jaccard values are computed only on common cell types (intersection-over-union of standardized marker sets) and averaged over shared cell types per database pair.
- The assembled jaccard_similarity_matrix.csv has one row/column per database with symmetric off-diagonal entries.

## Limitations

- Published marker gene databases are extremely heterogeneous and contain different marker sets for the same cell type, so even after standardization the resulting overlap is low (average Jaccard 0.08, max 0.13 across seven databases).
- Standardization quality depends on the ontology and nomenclature releases used (HGNC 2024, MGI v.6.24, Cell Ontology 2024-08-16); other releases may yield different mappings.
- Only cell types shared and mappable across databases can be compared; source-specific labels that cannot be mapped to Cell Ontology are excluded.

## Evidence

- [methods] Gene standardization resources: "HUGO Gene Nomenclature Committee (2024) and Mouse Genome Informatics (v. 6.24) resources were employed to standardize human and mouse gene names"
- [methods] Cell Ontology mapping: "mapping them to the Cell Ontology (release 2024-08-16)"
- [methods] Jaccard computation: "we calculated the Jaccard pairwise similarity values, i.e. the ratio between the cardinality of the intersection and the cardinality of the union"
- [intro] Motivating heterogeneity of databases: "currently available gene marker databases are extremely heterogeneous, contain different marker sets for the same cell type, and employ a non-standard nomenclature and classification"
- [readme] Harmonization framework (README): [evidence span withheld — anchor status: external_doc]
