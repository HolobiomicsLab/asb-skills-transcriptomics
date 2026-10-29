---
name: marker-gene-specificity-scoring
description: Use when harmonizing cell-type marker genes from multiple annotation
  sources (e.g., CellMarker2.0, PanglaoDB) and you need consistent, evidence-weighted
  marker scores for downstream cell type annotation.
when_to_use_negative:
- You need to annotate cells directly from raw counts without a pre-built or user-provided
  weighted marker database — use the accordion() annotation workflow instead
- Your markers come from a single source with no conflicting annotations, so evidence
  consistency (ECs) provides no discriminative signal
- Input is already a weighted signature/marker score table and does not require integration
  across heterogeneous marker databases
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: Cell Marker Accordion
  role: Provides the marker database, the SPs/ECs integration scores and the accordion()
    annotation function
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: ontologyIndex
  role: R package used for Cell Ontology and Uberon tree lookups when standardizing
    cell-type and tissue labels
- name: Shiny
  role: Interactive web application to explore, filter and download the Cell Marker
    Accordion database
  repo: https://github.com/TebaldiLab/shiny_cellmarkeraccordion
- name: data.table
  role: R package used for fast handling of the marker weight tables
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:b59f0336d6a8c6dec0f0e82ba4930a41be9623d6c5d03b9adb0462d226c9ecbc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - Cell Marker Accordion
  - R
  - Shiny
  - data.table
  - ontologyIndex
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# marker-gene-specificity-scoring

## Summary

Weights marker genes from the Cell Marker Accordion database by combining a specificity score (SPs) and an evidence consistency score (ECs) so that annotation sources with heterogeneous marker sets are harmonized into a single interpretable weight per gene-cell type pair.

## When to use

Use when harmonizing cell-type marker genes from multiple annotation sources (e.g., CellMarker2.0, PanglaoDB) and you need consistent, evidence-weighted marker scores for downstream cell type annotation.

## When NOT to use

- You need to annotate cells directly from raw counts without a pre-built or user-provided weighted marker database — use the accordion() annotation workflow instead
- Your markers come from a single source with no conflicting annotations, so evidence consistency (ECs) provides no discriminative signal
- Input is already a weighted signature/marker score table and does not require integration across heterogeneous marker databases

## Inputs

- Built-in accordion_marker database from the cellmarkeraccordion R package (healthy) or disease_accordion_marker (disease)
- Cell Ontology OBO file (release 2024-08-16) and Uberon OBO file (release 2024-09-03)
- Marker gene sets from multiple published databases (e.g., CellMarker2.0, PanglaoDB) and cell-sorting marker collections

## Outputs

- marker_weights.csv with SPs, ECs, ECs_reg, SPs_reg and final marker weight per gene-cell type pair
- Weighted marker table mapping standardized cell types to gene weights usable by the accordion() scoring loop

## How to apply

Standardize cell-type labels to Cell Ontology (release 2024-08-16) and tissues to Uberon (release 2024-09-03) terms using ontologyIndex tree lookups. For each gene G, compute SPs = 1/(number of cell types gene G marks) and ECs = number of sources defining G as a marker of cell type CT; for conflicting sources, subtract negative ECs from positive ECs, take the absolute value, and use the sign to set marker polarity (negative if <0, positive if >0, dropped if =0). Regularize ECs by log10 transformation (observed range 1-23), and reverse-scale the reciprocal of SPs, rescale it to the ECs range, and log10-transform it to obtain SPs_reg; multiply the two to get the final marker weight (maximum 5.6 in the current database). The rationale is that weighting by both how specific a gene is and how many sources agree makes annotations robust to database heterogeneity.

## Related tools

- **Cell Marker Accordion** (Provides the marker database, the SPs/ECs integration scores and the accordion() annotation function) — https://github.com/TebaldiLab/cellmarkeraccordion
- **ontologyIndex** (R package used for Cell Ontology and Uberon tree lookups when standardizing cell-type and tissue labels)
- **Shiny** (Interactive web application to explore, filter and download the Cell Marker Accordion database) — https://github.com/TebaldiLab/shiny_cellmarkeraccordion
- **data.table** (R package used for fast handling of the marker weight tables)

## Examples

```
data(accordion_marker); data <- accordion(data, assay="RNA", species="Human", tissue="blood", annotation_resolution="cluster", max_n_marker=30)
```

## Evaluation signals

- SPs values all lie in the interval [0, 1]
- ECs_reg values derive from the reported ECs range of 1-23 in the current database
- Final marker weights fall within the reported range of 1-5.6
- Every conflicting gene-cell type pair is resolved to positive, negative, or dropped according to the sign of (positive ECs - negative ECs), with zero pairs dropped
- The scoring script runs end-to-end and each input cell receives exactly one winning cell type annotation

## Limitations

- The current database covers 15479 marker genes, 728 cell types and 212 human tissues; markers outside this standardized space cannot be scored directly
- The benchmark and weights are focused on specific liquid and solid tumor contexts, with expansion planned
- Ontology-based harmonization depends on the specific Cell Ontology (2024-08-16) and Uberon (2024-09-03) releases; newer or older releases may change label mapping

## Evidence

- [intro] The Accordion database allows marker genes to be weighted not only by their specificity but also by their evidence consistency score, measuring the agreement of different annotation sources: "The Accordion database allows marker genes to be weighted not only by their specificity but also by their evidence consistency score, measuring the agreement of different annotation sources"
- [methods] SPs and ECs definition: "the Cell Marker Accordion database describes each marker with a specificity score (SPs) and an evidence consistency score (ECs)"
- [methods] Database scale after integration: "After standardization and integration, Cell Marker Accordion includes a comprehensive collection of 15479 marker genes associated with 728 cell types in 212 human tissues"
- [methods] Ontology mapping releases: "mapping them to the Cell Ontology (release 2024-08-16)"
- [discussion] Accuracy improvement claim: "the Cell Marker Accordion improved the identification of cell types compared to other available marker-based annotation tools12,17,19,40,41,64,65, with an average increase of 23%"
- [readme] accordion function usage from README: "data <- accordion(data, assay ="RNA", species ="Human", tissue="blood", annotation_resolution = "cluster", max_n_marker = 30, include_detailed_annotation_info = TRUE, plot = TRUE)"
