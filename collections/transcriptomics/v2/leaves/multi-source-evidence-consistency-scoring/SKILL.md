---
name: multi-source-evidence-consistency-scoring
description: Use when integrating marker gene sets from multiple published databases
  (e.g. CellMarker2.0, PanglaoDB) and you need to quantify and reconcile source agreement/conflict
  to weight markers for cell type annotation.
when_to_use_negative:
- You have a single authoritative marker source with no cross-source agreement to
  measure — SPs/ECs weighting adds no consistency information
- Input is expression data without a marker gene database — use annotation directly
  (accordion()) rather than rebuilding weights
- Non-marker-based annotation is required (e.g. reference-mapping tools); the article
  notes no direct comparison was conducted with annotation tools that do not rely
  on marker genes
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: Cell Marker Accordion
  role: Provides the SPs/ECs-weighted marker database and the accordion() R function
    that consumes these weights for automatic cell type annotation
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: ontologyIndex
  role: Performs Cell Ontology and Uberon tree lookups to standardize cell-type and
    tissue labels before computing cross-source consistency
- name: Seurat
  role: Pre-processes single-cell counts (NormalizeData, ScaleData) so scaled expression
    on input marker genes can be multiplied by ECs_reg x SPs_reg to obtain gene impact
    scores
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:74f1eb54b9076aa8d3abf2df6c0166d91427afe000e546801c9fd10abd020c67
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - Cell Marker Accordion
  - R
  - Seurat
  - ontologyIndex
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# multi-source-evidence-consistency-scoring

## Summary

Weights marker genes from multiple heterogeneous annotation sources by combining a specificity score (SPs = 1/number of cell types a gene marks) with an evidence consistency score (ECs = number of sources defining the gene as a marker), regularized via log10 transforms, to ensure consistency-weighted cell type annotation in the Cell Marker Accordion database.

## When to use

Use when integrating marker gene sets from multiple published databases (e.g. CellMarker2.0, PanglaoDB) and you need to quantify and reconcile source agreement/conflict to weight markers for cell type annotation.

## When NOT to use

- You have a single authoritative marker source with no cross-source agreement to measure — SPs/ECs weighting adds no consistency information
- Input is expression data without a marker gene database — use annotation directly (accordion()) rather than rebuilding weights
- Non-marker-based annotation is required (e.g. reference-mapping tools); the article notes no direct comparison was conducted with annotation tools that do not rely on marker genes

## Inputs

- Aggregated marker gene tables from multiple published databases (CellMarker2.0, PanglaoDB, cell-sorting marker collections), with positive/negative marker flags
- Built-in accordion_marker database from the cellmarkeraccordion R package (15,479 marker genes, 728 cell types, 212 human tissues)
- Cell Ontology (release 2024-08-16) and Uberon (release 2024-09-03) ontology files via ontologyIndex for label standardization

## Outputs

- Per-marker specificity score (SPs) in [0,1] and regularized ECs_reg and SPs_reg values
- Final consistency-weighted marker weights (range 1–5.6 in the current database), exportable as marker_weights.csv
- Weighted marker database usable by accordion() to produce cell type annotations (e.g. cell_type_annotations.csv)

## How to apply

For each marker gene G and cell type CT, compute SPs = 1/(number of cell types G marks) and ECs = number of sources defining G as marker of CT, separately for positive and negative markers. When sources conflict, subtract negative ECs from positive ECs, take the absolute value, and use the sign to set marker polarity (negative if <0, positive if >0, drop if =0). Regularize scores by log10-transforming ECs (range 1–23 in the current database) to ECs_reg, then reverse-scale 1/SPs, rescale it to the ECs range and log10-transform to obtain SPs_reg; multiply the two to get the final marker weight (maximum 5.6 in the current database). Standardize cell-type labels to Cell Ontology (release 2024-08-16) and tissues to Uberon (release 2024-09-03) before scoring so markers from different sources are comparable. The resulting weights feed gene impact scores (G_IMs = scaled expression x ECs_reg x SPs_reg) used by the accordion() function for automatic annotation.

## Related tools

- **Cell Marker Accordion** (Provides the SPs/ECs-weighted marker database and the accordion() R function that consumes these weights for automatic cell type annotation) — https://github.com/TebaldiLab/cellmarkeraccordion
- **ontologyIndex** (Performs Cell Ontology and Uberon tree lookups to standardize cell-type and tissue labels before computing cross-source consistency)
- **Seurat** (Pre-processes single-cell counts (NormalizeData, ScaleData) so scaled expression on input marker genes can be multiplied by ECs_reg x SPs_reg to obtain gene impact scores)

## Examples

```
data(accordion_marker); data <- accordion(data, assay="RNA", species="Human", tissue="blood", annotation_resolution="cluster", max_n_marker=30)
```

## Evaluation signals

- All computed SPs values lie in [0,1]
- ECs_reg values derive from the reported 1–23 ECs range (log10-transformed)
- Final marker weights fall within the reported 1–5.6 range
- Genes with ECs sign = 0 after positive/negative reconciliation are dropped (no zero-polarity markers retained)
- Weighted database matches the published scale: 15,479 marker genes, 728 cell types, 212 human tissues

## Limitations

- ECs range (1–23) and maximum weight (5.6) are database-version-specific and will shift as sources are added
- Database is limited to human and mouse markers; other species are out of scope
- Markers for the same cell type across sources are extremely heterogeneous and use non-standard nomenclature, so ontology mapping quality directly affects scoring

## Evidence

- [methods] SPs/ECs definition: "the Cell Marker Accordion database describes each marker with a specificity score (SPs) and an evidence consistency score (ECs)"
- [intro] Weighting rationale: "The Accordion database allows marker genes to be weighted not only by their specificity but also by their evidence consistency score, measuring the agreement of different annotation sources"
- [methods] Database scale after integration: "After standardization and integration, Cell Marker Accordion includes a comprehensive collection of 15479 marker genes associated with 728 cell types in 212 human tissues"
- [readme] Consistency-weighted annotation purpose: "This tool provides automated annotation and biological interpretation of single-cell populations using consistency-weighted markers."
