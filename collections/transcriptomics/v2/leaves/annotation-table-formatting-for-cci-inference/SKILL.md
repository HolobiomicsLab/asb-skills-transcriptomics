---
name: annotation-table-formatting-for-cci-inference
description: Use when a clustered single-cell count matrix needs cell-type labels
  before running scCrossTalk's find_lrpairs and downstream communication inference,
  and the labels must be formatted for create_scCrossTalk.
when_to_use_negative:
- Input is already an annotated single-cell dataset with an sc_celltype character
  vector — skip annotation and call create_scCrossTalk directly
- The analysis target is ligand-receptor pair enrichment or communication visualization
  (find_lrpairs, plot_cci_chord/heatmap) rather than producing the cell-type table
  itself
- Data are bulk RNA-seq or spatial transcriptomics without per-cell cluster assignments,
  since scDeepSort per-cell prediction and the sc_celltype character vector assume
  single-cell resolution
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDeepSort
  role: Classifies cell types of each cluster by predicting per-cell type probabilities
    against a pre-computed reference training set
- name: Seurat
  role: Holds the clustered scRNA-seq count matrix and cluster assignments that the
    cell-type labels are attached to
- name: scCrossTalk
  role: Consumes the formatted cell-type character vector as the sc_celltype argument
    of create_scCrossTalk to build the scCrossTalk object for LR-pair enrichment
  repo: https://github.com/multitalk/scCrossTalk
- name: R
  role: Execution environment in which scDeepSort annotation, CSV export, and create_scCrossTalk
    calls are run (installed via devtools::install_github)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.eng.2023.12.004
    title: A Single-Cell Landscape of Human Liver Transplantation Reveals a Pathogenic
      Immune Niche Associated with Early Allograft Dysfunction
  source_content_hash: sha256:cad2f5fe0a39dd1aaf5dd4859e7f1337c238f28b9b91c5e0d1b368c303a8bd68
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# annotation-table-formatting-for-cci-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Formats cluster-level cell-type annotations (produced via scDeepSort on a clustered scRNA-seq object) into a cell-to-celltype table (cell_annotations.csv) whose cell-type column plugs directly into scCrossTalk's create_scCrossTalk for cell-cell communication inference.

## When to use

Use when a clustered single-cell count matrix needs cell-type labels before running scCrossTalk's find_lrpairs and downstream communication inference, and the labels must be formatted for create_scCrossTalk.

## When NOT to use

- Input is already an annotated single-cell dataset with an sc_celltype character vector — skip annotation and call create_scCrossTalk directly
- The analysis target is ligand-receptor pair enrichment or communication visualization (find_lrpairs, plot_cci_chord/heatmap) rather than producing the cell-type table itself
- Data are bulk RNA-seq or spatial transcriptomics without per-cell cluster assignments, since scDeepSort per-cell prediction and the sc_celltype character vector assume single-cell resolution

## Inputs

- Clustered single-cell count matrix (e.g., demo_sc_data matrix from scCrossTalk) with cluster assignments
- Species designation (e.g., human)
- Pre-computed scDeepSort reference training set with reference-annotated cell types
- Seurat single-cell object (compatible with Seurat 5)

## Outputs

- cell_annotations.csv: cell-to-celltype mapping with one row per cell and a non-empty cell-type column
- Character vector of cell types (sc_celltype) positionally aligned with the scRNA-seq count matrix for create_scCrossTalk

## How to apply

Load the clustered single-cell count matrix with cluster assignments (e.g., the demo_sc_data matrix shipped with scCrossTalk) together with the species designation (human), then run scDeepSort in pre-computed-reference mode to predict cell-type probabilities for each cell from the reference-annotated training set. Aggregate the per-cell predictions to assign one cell type per cluster and propagate the cluster-level label back to all member cells as a character vector, since scCrossTalk's sc_celltype argument is 'a character containing the cell types for scRNA-seq data'. Write the cell-to-celltype mapping as cell_annotations.csv so the cell-type column can be supplied directly as the sc_celltype argument of create_scCrossTalk. Decision point: keep labels at cluster granularity (one label per cluster) rather than per-cell independent labels, because create_scCrossTalk pairs the character vector positionally with the scRNA-seq count matrix before find_lrpairs enriches highly expressed ligand-receptor pairs with the Z-test.

## Related tools

- **scDeepSort** (Classifies cell types of each cluster by predicting per-cell type probabilities against a pre-computed reference training set)
- **Seurat** (Holds the clustered scRNA-seq count matrix and cluster assignments that the cell-type labels are attached to)
- **scCrossTalk** (Consumes the formatted cell-type character vector as the sc_celltype argument of create_scCrossTalk to build the scCrossTalk object for LR-pair enrichment) — https://github.com/multitalk/scCrossTalk
- **R** (Execution environment in which scDeepSort annotation, CSV export, and create_scCrossTalk calls are run (installed via devtools::install_github))

## Examples

```
obj <- create_scCrossTalk(sc_data, sc_celltype, species)
```

## Evaluation signals

- cell_annotations.csv exists after the step completes
- Row count of cell_annotations.csv equals the number of cells in the input single-cell object
- Every cell in cell_annotations.csv has a non-empty cell-type label
- The cell-type column loads as a character vector whose length matches nrow(sc_data) and is accepted by create_scCrossTalk without positional mismatch errors
- The resulting scCrossTalk object reports a non-zero ligand-receptor interaction count after find_lrpairs (README demo shows 679 LR interactions found on demo data)

## Limitations

- Annotation quality depends on the pre-computed scDeepSort reference; reference coverage of tissue-specific states (e.g., the EAD-associated immune niche of MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils) determines label fidelity
- The sc_celltype argument must be 'a character containing the cell types for scRNA-seq data' — data frames or factor vectors are not directly accepted by create_scCrossTalk
- No changelog found for the source repository, so compatibility changes (beyond 'Compatible with Seurat 5') are not documented

## Evidence

- [readme] sc_celltype argument character format: "sc_celltype: A character containing the cell types for scRNA-seq data"
- [readme] create_scCrossTalk invocation: "> obj <- create_scCrossTalk(sc_data, sc_celltype, species)"
- [methods] cell_annotations.csv purpose: [evidence span withheld — anchor status: not_found]
- [methods] scDeepSort annotation of cluster cell types: [evidence span withheld — anchor status: not_found]
