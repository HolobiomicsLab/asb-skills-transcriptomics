---
name: reference-based-cell-type-classification
description: Use when a clustered scRNA-seq count matrix (e.g., Seurat object such
  as demo_sc_data) lacks cell-type labels and cluster identities must be assigned
  via a pre-computed reference classifier before cell-cell communication analysis.
when_to_use_negative:
- The dataset already carries authoritative cell-type annotations (e.g., manually
  curated labels such as MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils) —
  re-classification is unnecessary.
- No suitable pre-computed reference training set exists for the tissue or species,
  since scDeepSort depends on pre-computed reference data.
- The goal is de novo cluster identification rather than labeling existing clusters
  — clustering, not reference-based classification, is the right step.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDeepSort
  role: Reference-based classifier run in pre-computed-reference mode to predict cell-type
    probabilities for each cell from reference-annotated training data
- name: Seurat
  role: Framework for the clustered single-cell count matrix / Seurat-compatible object
    that serves as classification input
  repo: https://github.com/multitalk/scCrossTalk
- name: scCrossTalk
  role: Downstream consumer of the cell_annotations.csv labels via create_scCrossTalk(sc_data,
    sc_celltype, species) for cell-cell communication inference
  repo: https://github.com/multitalk/scCrossTalk
- name: R
  role: Environment in which scDeepSort classification and export of cell_annotations.csv
    are executed
  repo: https://github.com/multitalk/scCrossTalk
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.eng.2023.12.004
    title: A Single-Cell Landscape of Human Liver Transplantation Reveals a Pathogenic
      Immune Niche Associated with Early Allograft Dysfunction
  source_content_hash: sha256:a4c301b5731dca81f636ad19eee0ed9583856bd354e5c3e61d165447dd4f9759
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/multitalk/scCrossTalk
  repo_url: https://github.com/multitalk/scCrossTalk
  tools:
  - R
  - Seurat
  - scCrossTalk
  - scDeepSort
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# reference-based-cell-type-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate cell types for each cluster of a single-cell transcriptomic dataset by applying scDeepSort, a classifier that uses pre-computed reference-annotated training data, then export a cell-to-celltype mapping consumable by scCrossTalk's create_scCrossTalk.

## When to use

Use when a clustered scRNA-seq count matrix (e.g., Seurat object such as demo_sc_data) lacks cell-type labels and cluster identities must be assigned via a pre-computed reference classifier before cell-cell communication analysis.

## When NOT to use

- The dataset already carries authoritative cell-type annotations (e.g., manually curated labels such as MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils) — re-classification is unnecessary.
- No suitable pre-computed reference training set exists for the tissue or species, since scDeepSort depends on pre-computed reference data.
- The goal is de novo cluster identification rather than labeling existing clusters — clustering, not reference-based classification, is the right step.

## Inputs

- Clustered single-cell count matrix with cluster assignments (e.g., demo_sc_data from scCrossTalk)
- Species designation (e.g., human)
- Pre-computed reference training set with reference-annotated cell types (scDeepSort)

## Outputs

- cell_annotations.csv: cell-to-celltype mapping whose cell-type column is directly usable as the sc_celltype argument of create_scCrossTalk
- Character vector of cell-type labels, one per cell, derived from cluster-level scDeepSort predictions

## How to apply

Load the clustered single-cell count matrix with cluster assignments (e.g., the demo_sc_data matrix shipped with scCrossTalk) together with the species designation (human). Run scDeepSort in pre-computed-reference mode on the clustered object so it predicts cell-type probabilities for each cell from the reference-annotated training set. Aggregate per-cell predictions to one cell type per cluster and propagate the cluster-level label back to all member cells as a character vector. Write the result as cell_annotations.csv with a cell-type column formatted so it can be passed directly as the sc_celltype argument of create_scCrossTalk, the entry point of the scCrossTalk cell-cell communication workflow (create_scCrossTalk -> find_lrpairs -> visualization).

## Related tools

- **scDeepSort** (Reference-based classifier run in pre-computed-reference mode to predict cell-type probabilities for each cell from reference-annotated training data)
- **Seurat** (Framework for the clustered single-cell count matrix / Seurat-compatible object that serves as classification input) — https://github.com/multitalk/scCrossTalk
- **scCrossTalk** (Downstream consumer of the cell_annotations.csv labels via create_scCrossTalk(sc_data, sc_celltype, species) for cell-cell communication inference) — https://github.com/multitalk/scCrossTalk
- **R** (Environment in which scDeepSort classification and export of cell_annotations.csv are executed) — https://github.com/multitalk/scCrossTalk

## Examples

```
annotations <- read.csv("cell_annotations.csv"); obj <- create_scCrossTalk(sc_data = demo_sc_data, sc_celltype = annotations$celltype, species = "human")
```

## Evaluation signals

- cell_annotations.csv exists after the run.
- Row count of cell_annotations.csv equals the number of cells in the input object (e.g., demo_sc_data).
- Every cell has a non-empty cell-type label in the output file.
- Each cluster receives exactly one aggregated cell type, and all member cells inherit that cluster-level label.
- The cell-type column loads without error as the sc_celltype argument of create_scCrossTalk, yielding an object of class scCrossTalk.

## Limitations

- Accuracy depends on the availability and quality of a pre-computed reference training set; novel or tissue-atypical states may be mislabeled.
- The classifier is applied per cluster via aggregation, so within-cluster heterogeneity (mixed or doublet populations) is collapsed to a single label.
- The species designation must match both the query data and the reference (e.g., human); mismatched species will invalidate predictions.
- No changelog was found for the scCrossTalk source repository, so version-specific behavior changes are not traceable.

## Evidence

- [methods] Cluster cell types were annotated by applying scDeepSort, a classifier that uses pre-computed reference data, to determine the cell type of each cluster in the single-cell analysis.: [evidence span withheld — anchor status: not_found]
