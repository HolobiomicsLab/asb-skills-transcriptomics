---
name: cell-type-homology-annotation
description: Use when integrating single-cell datasets from multiple species whose
  cell type annotations use different or dataset-specific ontology terms, and a shared
  cell type homology mapping is required before integration and scoring.
when_to_use_negative:
- The task requires non-one-to-one cell type mappings — the paper's quantitative scoring
  uses one-to-one mappings only and does not assess valid non-one-to-one homologies
- Cell type annotations lack ontology IDs or are not curated against a standard ontology
  system, so ontology-based matching cannot be trusted
- Only a single dataset is being analysed and no cross-dataset label harmonisation
  is needed
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3308
tools:
- name: scOntoMatch
  role: Provides ontoMultiMinimal, ontoMultiMatch, ontoTranslate and check_ontology_translate
    to trim, match and verify cell ontology annotations across datasets
  repo: https://github.com/Papatheodorou-Group/scOntoMatch
- name: BENGAL
  role: Benchmarking pipeline within which the homology mapping feeds integration
    strategy evaluation and iso_F1 scoring
- name: Seurat
  role: Data format for per-dataset objects loaded via getSeuratRds and annotated
    with ontology columns
  repo: https://github.com/satijalab/seurat-wrappers
- name: ontologyIndex
  role: Reads the Cell Ontology OBO file with get_OBO and propagates is_a/part_of
    relationships
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-41855-w
    title: Benchmarking strategies for cross-species integration of single-cell RNA
      sequencing data
  source_content_hash: sha256:8c415cb29646dc1cac0e2b13a2c570b97d818384954b44320ce7d08acd69295c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Papatheodorou-Group/scOntoMatch
  tools:
  - BENGAL
  - Harmony
  - LIGER
  - LIGER UINMF
  - SAMap
  - Scanorama
  - Seurat
  - SeuratV4
  - fastMNN
  - ontologyIndex
  - scANVI
  - scOntoMatch
  - scVI
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# cell-type-homology-annotation

## Summary

Unify cell type ontology annotations across species-specific scRNA-seq datasets so that cell types have comparable homology labels before cross-species integration and biology-conservation scoring (e.g. iso_F1) can be computed in the BENGAL pipeline.

## When to use

Use when integrating single-cell datasets from multiple species whose cell type annotations use different or dataset-specific ontology terms, and a shared cell type homology mapping is required before integration and scoring.

## When NOT to use

- The task requires non-one-to-one cell type mappings — the paper's quantitative scoring uses one-to-one mappings only and does not assess valid non-one-to-one homologies
- Cell type annotations lack ontology IDs or are not curated against a standard ontology system, so ontology-based matching cannot be trusted
- Only a single dataset is being analysed and no cross-dataset label harmonisation is needed

## Inputs

- Per-dataset Seurat objects stored as RDS files with cell ontology annotation columns (anno_col, onto_id_col)
- Cell Ontology OBO file (e.g. cl-basic.obo) from the OBO Foundry
- Metadata table mapping datasets to annotation columns (tab-separated)

## Outputs

- Matched cross-dataset cell ontology annotation (cell_ontology_mapped) per cell
- Matched ontology tree plots per dataset and across datasets
- Shared cell type homology mapping used as reference for iso_F1 and other biology-conservation metrics

## How to apply

Load the per-dataset Seurat objects with getSeuratRds and read the Cell Ontology (.obo, cl-basic.obo) via ontologyIndex::get_OBO with propagate_relationships c('is_a','part_of'). Run scOntoMatch::ontoMultiMinimal to merge descendant terms to existing ancestor terms, yielding a minimal ontology representation per dataset, then manual re-annotation where needed (e.g. 'nk cell' -> 'natural killer cell') and ontoMultiMatch to match ontology annotations cross datasets by direct mapping and descendant-to-ancestor mapping. Apply ontoTranslate to relabel annotations, and check_ontology_translate plus plotMatchedOntoTree to verify the matched tree. The resulting one-to-one homology mapping defines the ground truth against which biology-conservation metrics such as iso_F1 (isolated label F1) are computed in the heart task, where only 8 of 22 cell types are shared by all species.

## Related tools

- **scOntoMatch** (Provides ontoMultiMinimal, ontoMultiMatch, ontoTranslate and check_ontology_translate to trim, match and verify cell ontology annotations across datasets) — https://github.com/Papatheodorou-Group/scOntoMatch
- **BENGAL** (Benchmarking pipeline within which the homology mapping feeds integration strategy evaluation and iso_F1 scoring)
- **Seurat** (Data format for per-dataset objects loaded via getSeuratRds and annotated with ontology columns) — https://github.com/satijalab/seurat-wrappers
- **ontologyIndex** (Reads the Cell Ontology OBO file with get_OBO and propagates is_a/part_of relationships)

## Examples

```
obj_list_minimal = scOntoMatch::ontoMultiMinimal(obj_list = obj_list, ont = ont, anno_col = anno_col, onto_id_col = onto_id_col); obj_list_matched = scOntoMatch::ontoMultiMatch(obj_list_minimal, anno_col = 'cell_ontology_base', onto_id_col = onto_id_col, ont = ont)
```

## Evaluation signals

- After ontoMultiMinimal, each dataset's annotation uses a minimal ontology tree with descendant terms merged to existing ancestor terms
- plotMatchedOntoTree output shows consistent matched labels (cell_ontology_mapped) across all input datasets
- check_ontology_translate raises no unresolved terms after ontoTranslate relabelling
- The matched mapping yields the expected shared-vs-isolated structure (e.g. heart: 8 of 22 cell types shared by all four species), and iso_F1 can be computed against it per strategy and species subset

## Limitations

- Quantitative scoring assumes cell type annotations are accurate and known cell type homology from literature is confident
- The benchmark is limited to one-to-one cell type mappings; valid non-one-to-one mappings across species are not assessed
- Cell ontology annotation must be curated and standardised per dataset before matching; unannotated or ad hoc labels cannot be matched

## Evidence

- [methods] trim per-dataset ontology: "We provide `ontoMultiMinimal` for Merging descendant terms to existing ancestor terms in one dataset, to get a minimum ontology representation of the cell type tree."
- [methods] cross-dataset matching: [evidence span withheld — anchor status: not_found]
- [readme] scOntoMatch purpose: "scOntoMatch is an R package which unifies ontology annotation of scRNA-seq datasets to make them comparable across studies."
- [results] iso_F1 ground truth: [evidence span withheld — anchor status: not_found]
- [abstract] metric rationale: "using 9 established metrics. We also develop a new biology conservation metric to address the maintenance of cell type distinguishability."
